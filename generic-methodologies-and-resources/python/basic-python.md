# Основи Python

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи [**telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Основи Python

### Корисна інформація

list(xrange()) == range() --> У Python3 range є xrange у Python2 (це не список, а генератор)\
Відмінність між Tuple та List полягає в тому, що позиція значення в кортежі має значення, а списки - це просто упорядковані значення. У кортежах є структури, а у списків - порядок.

### Основні операції

Для піднесення числа використовується: 3\*\*2 (не 3^2)\
Якщо ви виконуєте 2/3, воно повертає 1, оскільки ви ділите два цілих числа (integers). Якщо ви хочете десяткові числа, вам слід ділити числа з плаваючою комою (2.0/3.0).\
i >= j\
i <= j\
i == j\
i != j\
a and b\
a or b\
not a\
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
dir(str) = Список усіх доступних методів\
help(str) = Визначення класу str\
"a".upper() = "A"\
"A".lower() = "a"\
"abc".capitalize() = "Abc"\
sum(\[1,2,3]) = 6\
sorted(\[1,43,5,3,21,4])

**Об'єднання символів**\
3 \* ’a’ = ‘aaa’\
‘a’ + ‘b’ = ‘ab’\
‘a’ + str(3) = ‘a3’\
\[1,2,3]+\[4,5]=\[1,2,3,4,5]

**Частини списку**\
‘abc’\[0] = ‘a’\
'abc’\[-1] = ‘c’\
'abc’\[1:3] = ‘bc’ від \[1] до \[2]\
"qwertyuiop"\[:-1] = 'qwertyuio'

**Коментарі**\
\# Коментар у один рядок\
"""\
Кілька рядків коментаря\
Ще один\
"""

**Цикли**
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
### Кортежі

t1 = (1, '2', 'three')\
t2 = (5, 6)\
t3 = t1 + t2 = (1, '2', 'three', 5, 6)\
(4,) = Одиночний\
d = () порожній кортеж\
d += (4,) --> Додавання до кортежу\
НЕ МОЖЛИВО! --> t1\[1] == 'Нове значення'\
list(t2) = \[5, 6] --> З кортежу у список

### Список (масив)

d = \[] порожній\
a = \[1, 2, 3]\
b = \[4, 5]\
a + b = \[1, 2, 3, 4, 5]\
b.append(6) = \[4, 5, 6]\
tuple(a) = (1, 2, 3) --> Зі списку у кортеж

### Словник

d = {} порожній\
monthNumbers={1:'Jan', 2: 'feb','feb':2}—> monthNumbers ->{1:'Jan', 2: 'feb','feb':2}\
monthNumbers\[1] = 'Jan'\
monthNumbers\['feb'] = 2\
list(monthNumbers) = \[1, 2, 'feb']\
monthNumbers.values() = \['Jan', 'feb', 2]\
keys = \[k for k in monthNumbers]\
a={'9':9}\
monthNumbers.update(a) = {'9':9, 1:'Jan', 2: 'feb','feb':2}\
mN = monthNumbers.copy() #Незалежна копія\
monthNumbers.get('key',0) #Перевірка наявності ключа, Повертає значення monthNumbers\["key"] або 0, якщо його немає

### Множини

У множинах немає повторень\
myset = set(\['a', 'b']) = {'a', 'b'}\
myset.add('c') = {'a', 'b', 'c'}\
myset.add('a') = {'a', 'b', 'c'} #Без повторень\
myset.update(\[1, 2, 3]) = set(\['a', 1, 2, 'b', 'c', 3])\
myset.discard(10) #Якщо присутній, видалити, якщо нічого\
myset.remove(10) #Якщо присутній, видалити, якщо нічого, викликати виняток\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #Значення myset АБО myset2\
myset.intersection(myset2) #Значення в myset ТА myset2\
myset.difference(myset2) #Значення в myset, але не в myset2\
myset.symmetric\_difference(myset2) #Значення, які не в myset ТА myset2 (не в обох)\
myset.pop() #Отримати перший елемент множини та видалити його\
myset.intersection\_update(myset2) #myset = Елементи як в myset, так і в myset2\
myset.difference\_update(myset2) #myset = Елементи в myset, але не в myset2\
myset.symmetric\_difference\_update(myset2) #myset = Елементи, які не в обох

### Класи

Метод в \_\_It\_\_ буде використовуватися для порівняння об'єкта цього класу з іншим, щоб визначити, чи є він більшим.
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
### map, zip, filter, lambda, sorted та однорядкові вирази

**Map** це як: \[f(x) для x в ітерабельному] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** зупиняється, коли коротший з foo або bar зупиняється:
```
for f, b in zip(foo, bar):
print(f, b)
```
**Лямбда** використовується для визначення функції\
(lambda x,y: x+y)(5,3) = 8 --> Використання лямбди як простої **функції**\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> Використання лямбди для сортування списку\
m = **filter**(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) = \[3, 6, 9] --> Використання лямбди для фільтрації\
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
mult1 = \[x для x в \[1, 2, 3, 4, 5, 6, 7, 8, 9\] якщо x%3 == 0 ]

### Винятки
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

Якщо умова є хибною, рядок буде надрукований на екрані
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### Генератори, yield

Генератор, замість повернення чогось, "подає" щось. Коли ви звертаєтеся до нього, він "повертає" перше згенероване значення, після цього ви можете звертатися до нього знову, і він поверне наступне згенероване значення. Таким чином, всі значення не генеруються одночасно, і використання цього може заощадити багато пам'яті порівняно зі списком усіх значень.
```
def myGen(n):
yield n
yield n + 1
```
```markdown
g = myGen(6) --> 6\
next(g) --> 7\
next(g) --> Помилка

### Регулярні вирази

import re\
re.search("\w","hola").group() = "h"\
re.findall("\w","hola") = \['h', 'o', 'l', 'a']\
re.findall("\w+(la)","hola caracola") = \['la', 'la']

**Спеціальні значення:**\
. --> Все\
\w --> \[a-zA-Z0-9\_]\
\d --> Число\
\s --> Пробільний символ\[ \n\r\t\f]\
\S --> Непробільний символ\
^ --> Починається з\
$ --> Закінчується на\
\+ --> Один або більше\
\* --> 0 або більше\
? --> 0 або 1 входження

**Опції:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> Дозволяє крапці відповідати символу нового рядка\
MULTILINE --> Дозволяє ^ та $ відповідати на різних рядках

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**product**\
from **itertools** import product --> Генерує комбінації між 1 або більше списками, можливо повторюючи значення, декартовий добуток (розподільний закон)\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**permutations**\
from **itertools** import **permutations** --> Генерує комбінації всіх символів на кожній позиції\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... Кожна можлива комбінація\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] Кожна можлива комбінація довжини 2

**combinations**\
from itertools import **combinations** --> Генерує всі можливі комбінації без повторення символів (якщо "ab" існує, не генерує "ba")\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> Генерує всі можливі комбінації від символу далі (наприклад, 3-й змішаний з 3-го далі, але не з 2-го або першого)\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### Декоратори

Декоратор, який визначає час, необхідний для виконання функції (з [тут](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74)):
```
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
Якщо ви запустите його, ви побачите щось схоже на наступне:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
```
{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
