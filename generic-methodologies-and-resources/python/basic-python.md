# Basic Python

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

## Python Basics

### Useful information

list(xrange()) == range() --> Katika python3 range ni xrange ya python2 (siyo orodha bali ni jenereta)\
Tofauti kati ya Tuple na List ni kwamba nafasi ya thamani katika tuple inampa maana lakini orodha ni thamani zilizopangwa tu. Tuples zina muundo lakini orodha zina mpangilio.

### Main operations

Ili kuinua nambari unatumia: 3\*\*2 (siyo 3^2)\
Ikiwa unafanya 2/3 inarudisha 1 kwa sababu unagawa ints mbili (nambari nzima). Ikiwa unataka desimali unapaswa kugawa floats (2.0/3.0).\
i >= j\
i <= j\
i == j\
i != j\
a na b\
a au b\
siyo a\
float(a)\
int(a)\
str(d)\
ord("A") = 65\
chr(65) = 'A'\
hex(100) = '0x64'\
hex(100)\[2:] = '64'\
isinstance(1, int) = True\
"a b".split(" ") = \['a', 'b']\
" ".join(\['a', 'b']) = "a b"\
"abcdef".startswith("ab") = True\
"abcdef".contains("abc") = True\
"abc\n".strip() = "abc"\
"apbc".replace("p","") = "abc"\
dir(str) = Orodha ya mbinu zote zinazopatikana\
help(str) = Maelezo ya darasa str\
"a".upper() = "A"\
"A".lower() = "a"\
"abc".capitalize() = "Abc"\
sum(\[1,2,3]) = 6\
sorted(\[1,43,5,3,21,4])

**Join chars**\
3 \* ’a’ = ‘aaa’\
‘a’ + ‘b’ = ‘ab’\
‘a’ + str(3) = ‘a3’\
\[1,2,3]+\[4,5]=\[1,2,3,4,5]

**Parts of a list**\
‘abc’\[0] = ‘a’\
'abc’\[-1] = ‘c’\
'abc’\[1:3] = ‘bc’ kutoka \[1] hadi \[2]\
"qwertyuiop"\[:-1] = 'qwertyuio'

**Comments**\
\# Maoni ya mstari mmoja\
"""\
Maoni ya mistari kadhaa\
Nyingine\
"""

**Loops**
```
if a:
#somethig
elif b:
#something
else:
#something

while(a):
#comething

for i in range(0,100):
#something from 0 to 99

for letter in "hola":
#something with a letter in "hola"
```
### Tuples

t1 = (1,'2,'three')\
t2 = (5,6)\
t3 = t1 + t2 = (1, '2', 'three', 5, 6)\
(4,) = Singelton\
d = () tuple tupu\
d += (4,) --> Kuongeza kwenye tuple\
CANT! --> t1\[1] == 'New value'\
list(t2) = \[5,6] --> Kutoka tuple hadi orodha

### List (array)

d = \[] tupu\
a = \[1,2,3]\
b = \[4,5]\
a + b = \[1,2,3,4,5]\
b.append(6) = \[4,5,6]\
tuple(a) = (1,2,3) --> Kutoka orodha hadi tuple

### Dictionary

d = {} tupu\
monthNumbers={1:’Jan’, 2: ‘feb’,’feb’:2}—> monthNumbers ->{1:’Jan’, 2: ‘feb’,’feb’:2}\
monthNumbers\[1] = ‘Jan’\
monthNumbers\[‘feb’] = 2\
list(monthNumbers) = \[1,2,’feb’]\
monthNumbers.values() = \[‘Jan’,’feb’,2]\
keys = \[k for k in monthNumbers]\
a={'9':9}\
monthNumbers.update(a) = {'9':9, 1:’Jan’, 2: ‘feb’,’feb’:2}\
mN = monthNumbers.copy() #Nakala huru\
monthNumbers.get('key',0) #Angalia kama ufunguo upo, Rudisha thamani ya monthNumbers\["key"] au 0 kama haipo

### Set

Katika set hakuna kurudiwa\
myset = set(\['a', 'b']) = {'a', 'b'}\
myset.add('c') = {'a', 'b', 'c'}\
myset.add('a') = {'a', 'b', 'c'} #Hakuna kurudiwa\
myset.update(\[1,2,3]) = set(\['a', 1, 2, 'b', 'c', 3])\
myset.discard(10) #Kama ipo, iondoe, kama sio, hakuna\
myset.remove(10) #Kama ipo iondoe, kama sio, panda exception\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #Thamani ni myset AU myset2\
myset.intersection(myset2) #Thamani katika myset NA myset2\
myset.difference(myset2) #Thamani katika myset lakini sio katika myset2\
myset.symmetric\_difference(myset2) #Thamani ambazo hazipo katika myset NA myset2 (sio katika zote)\
myset.pop() #Pata kipengele cha kwanza cha set na uondoe\
myset.intersection\_update(myset2) #myset = Vipengele katika myset na myset2\
myset.difference\_update(myset2) #myset = Vipengele katika myset lakini sio katika myset2\
myset.symmetric\_difference\_update(myset2) #myset = Vipengele ambavyo havipo katika zote

### Classes

Njia katika \_\_It\_\_ itakuwa ile itakayotumika na sort kulinganisha kama kitu cha darasa hili ni kikubwa kuliko kingine
```python
class Person(name):
def __init__(self,name):
self.name= name
self.lastName = name.split(‘ ‘)[-1]
self.birthday = None
def __It__(self, other):
if self.lastName == other.lastName:
return self.name < other.name
return self.lastName < other.lastName #Return True if the lastname is smaller

def setBirthday(self, month, day. year):
self.birthday = date tame.date(year,month,day)
def getAge(self):
return (date time.date.today() - self.birthday).days


class MITPerson(Person):
nextIdNum = 0	# Attribute of the Class
def __init__(self, name):
Person.__init__(self,name)
self.idNum = MITPerson.nextIdNum  —> Accedemos al atributo de la clase
MITPerson.nextIdNum += 1 #Attribute of the class +1

def __it__(self, other):
return self.idNum < other.idNum
```
### ramani, zip, filter, lambda, sorted na mistari moja

**Map** ni kama: \[f(x) for x in iterable] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** inasimama wakati mfupi wa foo au bar inasimama:
```
for f, b in zip(foo, bar):
print(f, b)
```
**Lambda** inatumika kufafanua kazi\
(lambda x,y: x+y)(5,3) = 8 --> Tumia lambda kama **kazi**\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> Tumia lambda kupanga orodha\
m = **filter**(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) = \[3, 6, 9] --> Tumia lambda kuchuja\
**reduce** (lambda x,y: x\*y, \[1,2,3,4]) = 24
```
def make_adder(n):
return lambda x: x+n
plus3 = make_adder(3)
plus3(4) = 7 # 3 + 4 = 7

class Car:
crash = lambda self: print('Boom!')
my_car = Car(); my_car.crash() = 'Boom!'
```
mult1 = \[x for x in \[1, 2, 3, 4, 5, 6, 7, 8, 9] if x%3 == 0 ]

### Mifanozo
```
def divide(x,y):
try:
result = x/y
except ZeroDivisionError, e:
print “division by zero!” + str(e)
except TypeError:
divide(int(x),int(y))
else:
print “result i”, result
finally
print “executing finally clause in any case”
```
### Assert()

Ikiwa hali si ya kweli, maandiko yataonyeshwa kwenye skrini.
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### Generators, yield

Generator, badala ya kurudisha kitu, "hutoa" kitu. Unapokifikia, kitarejesha thamani ya kwanza iliyozalishwa, kisha, unaweza kukifikia tena na kitarejesha thamani inayofuata iliyozalishwa. Hivyo, thamani zote hazizalishwi kwa wakati mmoja na kumbukumbu nyingi zinaweza kuokolewa kwa kutumia hii badala ya orodha yenye thamani zote.
```
def myGen(n):
yield n
yield n + 1
```
g = myGen(6) --> 6\
next(g) --> 7\
next(g) --> Hitilafu

### Mifumo ya Kawaida

import re\
re.search("\w","hola").group() = "h"\
re.findall("\w","hola") = \['h', 'o', 'l', 'a']\
re.findall("\w+(la)","hola caracola") = \['la', 'la']

**Maana Maalum:**\
. --> Kila kitu\
\w --> \[a-zA-Z0-9\_]\
\d --> Nambari\
\s --> Karakteri ya Nafasi\[ \n\r\t\f]\
\S --> Karakteri isiyo na nafasi\
^ --> Anza na\
$ --> Maliza na\
\+ --> Moja au zaidi\
\* --> 0 au zaidi\
? --> Matukio 0 au 1

**Chaguo:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> Ruhusu nukta kuendana na newline\
MULTILINE --> Ruhusu ^ na $ kuendana katika mistari tofauti

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**product**\
from **itertools** import product --> Inazalisha mchanganyiko kati ya orodha 1 au zaidi, labda ikirudia thamani, bidhaa ya cartesian (sifa ya usambazaji)\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**permutations**\
from **itertools** import **permutations** --> Inazalisha mchanganyiko wa wahusika wote katika kila nafasi\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... Mchanganyiko wote unaowezekana\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] Mchanganyiko wote unaowezekana wa urefu 2

**combinations**\
from itertools import **combinations** --> Inazalisha mchanganyiko wote unaowezekana bila kurudia wahusika (ikiwa "ab" ipo, haiwezi kuunda "ba")\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> Inazalisha mchanganyiko wote unaowezekana kuanzia wahusika (kwa mfano, ya tatu inachanganywa kuanzia ya tatu lakini si na ya pili au ya kwanza)\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### Wapambo

Mwapambo unaopima muda ambao kazi inahitaji kutekelezwa (kutoka [hapa](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74)):
```python
from functools import wraps
import time
def timeme(func):
@wraps(func)
def wrapper(*args, **kwargs):
print("Let's call our decorated function")
start = time.time()
result = func(*args, **kwargs)
print('Execution time: {} seconds'.format(time.time() - start))
return result
return wrapper

@timeme
def decorated_func():
print("Decorated func!")
```
Ikiwa utaikimbia, utaona kitu kama ifuatavyo:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
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
