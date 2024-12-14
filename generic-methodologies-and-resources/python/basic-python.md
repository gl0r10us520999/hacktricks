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

list(xrange()) == range() --> Στην python3 το range είναι το xrange της python2 (δεν είναι λίστα αλλά γεννήτρια)\
Η διαφορά μεταξύ ενός Tuple και μιας List είναι ότι η θέση μιας τιμής σε ένα tuple της δίνει νόημα, ενώ οι λίστες είναι απλώς ταξινομημένες τιμές. Τα tuples έχουν δομές αλλά οι λίστες έχουν μια σειρά.

### Main operations

To raise a number you use: 3\*\*2 (not 3^2)\
If you do 2/3 it returns 1 because you are dividing two ints (integers). If you want decimals you should divide floats (2.0/3.0).\
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
dir(str) = List of all the available methods\
help(str) = Definition of the class str\
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
'abc’\[1:3] = ‘bc’ από \[1] έως \[2]\
"qwertyuiop"\[:-1] = 'qwertyuio'

**Comments**\
\# One line comment\
"""\
Several lines comment\
Another one\
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
d = () κενό tuple\
d += (4,) --> Προσθήκη σε ένα tuple\
CANT! --> t1\[1] == 'New value'\
list(t2) = \[5,6] --> Από tuple σε λίστα

### List (array)

d = \[] κενό\
a = \[1,2,3]\
b = \[4,5]\
a + b = \[1,2,3,4,5]\
b.append(6) = \[4,5,6]\
tuple(a) = (1,2,3) --> Από λίστα σε tuple

### Dictionary

d = {} κενό\
monthNumbers={1:’Jan’, 2: ‘feb’,’feb’:2}—> monthNumbers ->{1:’Jan’, 2: ‘feb’,’feb’:2}\
monthNumbers\[1] = ‘Jan’\
monthNumbers\[‘feb’] = 2\
list(monthNumbers) = \[1,2,’feb’]\
monthNumbers.values() = \[‘Jan’,’feb’,2]\
keys = \[k for k in monthNumbers]\
a={'9':9}\
monthNumbers.update(a) = {'9':9, 1:’Jan’, 2: ‘feb’,’feb’:2}\
mN = monthNumbers.copy() #Ανεξάρτητη αντιγραφή\
monthNumbers.get('key',0) #Έλεγχος αν υπάρχει το κλειδί, Επιστροφή της τιμής του monthNumbers\["key"] ή 0 αν δεν υπάρχει

### Set

Στα σύνολα δεν υπάρχουν επαναλήψεις\
myset = set(\['a', 'b']) = {'a', 'b'}\
myset.add('c') = {'a', 'b', 'c'}\
myset.add('a') = {'a', 'b', 'c'} #Χωρίς επαναλήψεις\
myset.update(\[1,2,3]) = set(\['a', 1, 2, 'b', 'c', 3])\
myset.discard(10) #Αν υπάρχει, αφαιρέστε το, αν όχι, τίποτα\
myset.remove(10) #Αν υπάρχει, αφαιρέστε το, αν όχι, ρίξτε εξαίρεση\
myset2 = set(\[1, 2, 3, 4])\
myset.union(myset2) #Τιμές που είναι myset Ή myset2\
myset.intersection(myset2) #Τιμές σε myset ΚΑΙ myset2\
myset.difference(myset2) #Τιμές σε myset αλλά όχι σε myset2\
myset.symmetric\_difference(myset2) #Τιμές που δεν είναι σε myset ΚΑΙ myset2 (όχι και στα δύο)\
myset.pop() #Πάρτε το πρώτο στοιχείο του συνόλου και αφαιρέστε το\
myset.intersection\_update(myset2) #myset = Στοιχεία και στα δύο myset και myset2\
myset.difference\_update(myset2) #myset = Στοιχεία σε myset αλλά όχι σε myset2\
myset.symmetric\_difference\_update(myset2) #myset = Στοιχεία που δεν είναι και στα δύο

### Classes

Η μέθοδος στο \_\_It\_\_ θα είναι αυτή που θα χρησιμοποιηθεί από το sort για να συγκρίνει αν ένα αντικείμενο αυτής της κλάσης είναι μεγαλύτερο από άλλο
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
### map, zip, filter, lambda, sorted and one-liners

**Map** είναι όπως: \[f(x) for x in iterable] --> map(tutple,\[a,b]) = \[(1,2,3),(4,5)]\
m = map(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) --> \[False, False, True, False, False, True, False, False, True]

**zip** σταματά όταν το πιο σύντομο από το foo ή το bar σταματά:
```
for f, b in zip(foo, bar):
print(f, b)
```
**Lambda** χρησιμοποιείται για να ορίσει μια συνάρτηση\
(lambda x,y: x+y)(5,3) = 8 --> Χρησιμοποιήστε το lambda ως απλή **συνάρτηση**\
**sorted**(range(-5,6), key=lambda x: x\*\* 2) = \[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5] --> Χρησιμοποιήστε το lambda για να ταξινομήσετε μια λίστα\
m = **filter**(lambda x: x % 3 == 0, \[1, 2, 3, 4, 5, 6, 7, 8, 9]) = \[3, 6, 9] --> Χρησιμοποιήστε το lambda για να φιλτράρετε\
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

### Εξαιρέσεις
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

Αν η συνθήκη είναι ψευδής, η συμβολοσειρά θα εκτυπωθεί στην οθόνη.
```
def avg(grades, weights):
assert not len(grades) == 0, 'no grades data'
assert len(grades) == 'wrong number grades'
```
### Γεννήτριες, yield

Μια γεννήτρια, αντί να επιστρέφει κάτι, "παραδίδει" κάτι. Όταν την προσπελάσετε, θα "επιστρέψει" την πρώτη τιμή που δημιουργήθηκε, στη συνέχεια, μπορείτε να την προσπελάσετε ξανά και θα επιστρέψει την επόμενη τιμή που δημιουργήθηκε. Έτσι, όλες οι τιμές δεν δημιουργούνται ταυτόχρονα και μπορεί να σωθεί πολύ μνήμη χρησιμοποιώντας αυτό αντί για μια λίστα με όλες τις τιμές.
```
def myGen(n):
yield n
yield n + 1
```
g = myGen(6) --> 6\
next(g) --> 7\
next(g) --> Σφάλμα

### Κανονικές Εκφράσεις

import re\
re.search("\w","hola").group() = "h"\
re.findall("\w","hola") = \['h', 'o', 'l', 'a']\
re.findall("\w+(la)","hola caracola") = \['la', 'la']

**Ειδικές σημασίες:**\
. --> Όλα\
\w --> \[a-zA-Z0-9\_]\
\d --> Αριθμός\
\s --> Χαρακτήρας κενής θέσης\[ \n\r\t\f]\
\S --> Μη κενός χαρακτήρας\
^ --> Ξεκινά με\
$ --> Τελειώνει με\
\+ --> Ένα ή περισσότερα\
\* --> 0 ή περισσότερα\
? --> 0 ή 1 εμφανίσεις

**Επιλογές:**\
re.search(pat,str,re.IGNORECASE)\
IGNORECASE\
DOTALL --> Επιτρέπει την τελεία να ταιριάζει με νέα γραμμή\
MULTILINE --> Επιτρέπει το ^ και $ να ταιριάζουν σε διαφορετικές γραμμές

re.findall("<.\*>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>foo\</b>and\<i>so on\</i>']\
re.findall("<.\*?>", "\<b>foo\</b>and\<i>so on\</i>") = \['\<b>', '\</b>', '\<i>', '\</i>']

IterTools\
**product**\
from **itertools** import product --> Δημιουργεί συνδυασμούς μεταξύ 1 ή περισσότερων λιστών, ίσως επαναλαμβάνοντας τιμές, καρτεσιανός προϊόν (διανεμητική ιδιότητα)\
print list(**product**(\[1,2,3],\[3,4])) = \[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]\
print list(**product**(\[1,2,3],repeat = 2)) = \[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]

**permutations**\
from **itertools** import **permutations** --> Δημιουργεί συνδυασμούς όλων των χαρακτήρων σε κάθε θέση\
print list(permutations(\['1','2','3'])) = \[('1', '2', '3'), ('1', '3', '2'), ('2', '1', '3'),... Κάθε πιθανός συνδυασμός\
print(list(permutations('123',2))) = \[('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')] Κάθε πιθανός συνδυασμός μήκους 2

**combinations**\
from itertools import **combinations** --> Δημιουργεί όλους τους πιθανούς συνδυασμούς χωρίς επαναλαμβανόμενους χαρακτήρες (αν υπάρχει "ab", δεν δημιουργεί "ba")\
print(list(**combinations**('123',2))) --> \[('1', '2'), ('1', '3'), ('2', '3')]

**combinations\_with\_replacement**\
from itertools import **combinations\_with\_replacement** --> Δημιουργεί όλους τους πιθανούς συνδυασμούς από τον χαρακτήρα και μετά (για παράδειγμα, ο 3ος είναι μίξη από τον 3ο και μετά αλλά όχι με τον 2ο ή τον πρώτο)\
print(list(**combinations\_with\_replacement**('1133',2))) = \[('1', '1'), ('1', '1'), ('1', '3'), ('1', '3'), ('1', '1'), ('1', '3'), ('1', '3'), ('3', '3'), ('3', '3'), ('3', '3')]

### Διακοσμητές

Διακοσμητής που μετράει τον χρόνο που χρειάζεται μια συνάρτηση για να εκτελεστεί (από [εδώ](https://towardsdatascience.com/decorating-functions-in-python-619cbbe82c74)):
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
Αν το εκτελέσετε, θα δείτε κάτι σαν το παρακάτω:
```
Let's call our decorated function
Decorated func!
Execution time: 4.792213439941406e-05 seconds
```
{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξη HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
