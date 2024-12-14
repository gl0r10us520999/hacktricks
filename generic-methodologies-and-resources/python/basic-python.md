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

list(xrange()) == range() --> In python3 range è l'xrange di python2 (non è una lista ma un generatore)\
La differenza tra una Tupla e una Lista è che la posizione di un valore in una tupla le dà significato, mentre le liste sono solo valori ordinati. Le tuple hanno strutture ma le liste hanno un ordine.

### Main operations

Per elevare un numero si usa: 3\*\*2 (non 3^2)\
Se fai 2/3 restituisce 1 perché stai dividendo due int (interi). Se vuoi decimali dovresti dividere float (2.0/3.0).\
i >= j\
i <= j\
i == j\
i != j\
a e b\
a o b\
non a\
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
dir(str) = Elenco di tutti i metodi disponibili\
help(str) = Definizione della classe str\
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
'abc’\[1:3] = ‘bc’ da \[1] a \[2]\
"qwertyuiop"\[:-1] = 'qwertyuio'

**Comments**\
\# Commento su una riga\
"""\
Commento su più righe\
Un altro\
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
### Tuple

t1 = (1,'2,'three')\
t2 = (5,6)\
t3 = t1 + t2 = (1, '2', 'three', 5, 6)\
(4,) = Singelton\
d = () tupla vuota\
d += (4,) --> Aggiunta in una tupla\
CANT! --> t1\[1] == 'Nuovo valore'\
list(t2) = \[5,6] --> Da tupla a lista

### Lista (array)

d = \[] vuota\
a = \[1,2,3]\
b = \[4,5]\
a + b = \[1,2,3,4,5]\
b.append(6) = \[4,5,6]\
tuple(a) = (1,2,3) --> Da lista a tupla

### Dizionario

d = {} vuoto\
monthNumbers={1:’Gen’, 2: ‘feb’,’feb’:2}—> monthNumbers ->{1:’Gen’, 2: ‘feb’,’feb’:2}\
monthNumbers\[1] = ‘Gen’\
monthNumbers\[‘feb’] = 2\
list(monthNumbers) = \[1,2,’feb’]\
monthNumbers.values() = \[‘Gen’,’feb’,2]\
keys = \[k for k in monthNumbers]\
a={'9':9}\
monthNumbers.update(a) = {'9':9, 1:’Gen’, 2: ‘feb’,’feb’:2}\
mN = monthNumbers.copy() #Copia indipendente\
monthNumbers.get('key',0) #Controlla se la chiave esiste, Restituisce il valore di monthNumbers\["key"] o 0 se non esiste

### Insieme

Negli insiemi non ci sono ripetizioni\
myset = set(\['a', 'b']) = {'a', 'b'}\
myset.add('c') = {'a', 'b', 'c'}\
myset.add('a') = {'a', 'b', 'c'} #Nessuna ripetizione\
myset.update(\[1,2,3]) = set(\['a', 1, 2, 'b', 'c', 3])\
myset.discard(10) #Se presente, rimuovilo, altrimenti, niente\
myset.remove(10) #Se presente rimuovilo, altrimenti, solleva un'eccezione\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #Valori in myset O myset2\
myset.intersection(myset2) #Valori in myset E myset2\
myset.difference(myset2) #Valori in myset ma non in myset2\
myset.symmetric\_difference(myset2) #Valori che non sono in myset E myset2 (non in entrambi)\
myset.pop() #Ottieni il primo elemento dell'insieme e rimuovilo\
myset.intersection\_update(myset2) #myset = Elementi in entrambi myset e myset2\
myset.difference\_update(myset2) #myset = Elementi in myset ma non in myset2\
myset.symmetric\_difference\_update(myset2) #myset = Elementi che non sono in entrambi

### Classi

Il metodo in \_\_It\_\_ sarà quello utilizzato da sort per confrontare se un oggetto di questa classe è più grande di un altro
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
### map, zip, filter, lambda, sorted e one-liners

**Map** è come: \[f(x) for x in iterable] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** si ferma quando il più corto tra foo o bar si ferma:
```
for f, b in zip(foo, bar):
print(f, b)
```
**Lambda** viene utilizzato per definire una funzione\
(lambda x,y: x+y)(5,3) = 8 --> Usa lambda come semplice **funzione**\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> Usa lambda per ordinare una lista\
m = **filter**(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) = \[3, 6, 9] --> Usa lambda per filtrare\
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

### Eccezioni
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

Se la condizione è falsa, la stringa verrà stampata sullo schermo.
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### Generatori, yield

Un generatore, invece di restituire qualcosa, "produce" qualcosa. Quando lo accedi, "restituirà" il primo valore generato, poi, puoi accedervi di nuovo e restituirà il prossimo valore generato. Quindi, tutti i valori non vengono generati contemporaneamente e si potrebbe risparmiare molta memoria utilizzando questo invece di una lista con tutti i valori.
```
def myGen(n):
yield n
yield n + 1
```
g = myGen(6) --> 6\
next(g) --> 7\
next(g) --> Errore

### Espressioni Regolari

import re\
re.search("\w","hola").group() = "h"\
re.findall("\w","hola") = \['h', 'o', 'l', 'a']\
re.findall("\w+(la)","hola caracola") = \['la', 'la']

**Significati speciali:**\
. --> Tutto\
\w --> \[a-zA-Z0-9\_]\
\d --> Numero\
\s --> Carattere di spazi bianchi\[ \n\r\t\f]\
\S --> Carattere non bianco\
^ --> Inizia con\
$ --> Finisce con\
\+ --> Uno o più\
\* --> 0 o più\
? --> 0 o 1 occorrenze

**Opzioni:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> Consente al punto di corrispondere a una nuova riga\
MULTILINE --> Consente a ^ e $ di corrispondere in righe diverse

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**product**\
from **itertools** import product --> Genera combinazioni tra 1 o più liste, forse ripetendo valori, prodotto cartesiano (proprietà distributiva)\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**permutations**\
from **itertools** import **permutations** --> Genera combinazioni di tutti i caratteri in ogni posizione\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... Ogni possibile combinazione\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] Ogni possibile combinazione di lunghezza 2

**combinations**\
from itertools import **combinations** --> Genera tutte le possibili combinazioni senza ripetere caratteri (se "ab" esiste, non genera "ba")\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> Genera tutte le possibili combinazioni a partire dal carattere (ad esempio, il 3° è mescolato a partire dal 3° ma non con il 2° o il 1°)\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### Decoratori

Decoratore che misura il tempo necessario per eseguire una funzione (da [qui](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74)):
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
Se lo esegui, vedrai qualcosa di simile al seguente:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
```
{% hint style="success" %}
Impara e pratica AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos su github.

</details>
{% endhint %}
