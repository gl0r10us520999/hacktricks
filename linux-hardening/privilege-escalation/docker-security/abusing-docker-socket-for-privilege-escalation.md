# Κατάχρηση του Docker Socket για Ανύψωση Δικαιωμάτων

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

Υπάρχουν περιπτώσεις όπου έχετε **πρόσβαση στο docker socket** και θέλετε να το χρησιμοποιήσετε για να **ανυψώσετε δικαιώματα**. Ορισμένες ενέργειες μπορεί να είναι πολύ ύποπτες και μπορεί να θέλετε να τις αποφύγετε, οπότε εδώ μπορείτε να βρείτε διάφορες σημαίες που μπορεί να είναι χρήσιμες για την ανύψωση δικαιωμάτων:

### Μέσω mount

Μπορείτε να **mount** διάφορα μέρη του **filesystem** σε ένα κοντέινερ που τρέχει ως root και να τα **πρόσβαση**.\
Μπορείτε επίσης να **καταχρήσετε ένα mount για να ανυψώσετε δικαιώματα** μέσα στο κοντέινερ.

* **`-v /:/host`** -> Mount το filesystem του host στο κοντέινερ ώστε να μπορείτε να **διαβάσετε το filesystem του host.**
* Αν θέλετε να **νιώσετε ότι είστε στον host** αλλά να είστε στο κοντέινερ μπορείτε να απενεργοποιήσετε άλλους μηχανισμούς άμυνας χρησιμοποιώντας σημαίες όπως:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> Αυτό είναι παρόμοιο με την προηγούμενη μέθοδο, αλλά εδώ **mountάρουμε τη συσκευή δίσκου**. Στη συνέχεια, μέσα στο κοντέινερ εκτελέστε `mount /dev/sda1 /mnt` και μπορείτε να **πρόσβαση** στο **filesystem του host** στο `/mnt`
* Εκτελέστε `fdisk -l` στον host για να βρείτε τη συσκευή `</dev/sda1>` για να mount
* **`-v /tmp:/host`** -> Αν για κάποιο λόγο μπορείτε **μόνο να mountάρετε κάποιον κατάλογο** από τον host και έχετε πρόσβαση μέσα στον host. Mount το και δημιουργήστε ένα **`/bin/bash`** με **suid** στον mounted κατάλογο ώστε να μπορείτε να **το εκτελέσετε από τον host και να ανυψώσετε σε root**.

{% hint style="info" %}
Σημειώστε ότι ίσως δεν μπορείτε να mountάρετε τον φάκελο `/tmp` αλλά μπορείτε να mountάρετε έναν **διαφορετικό εγγράψιμο φάκελο**. Μπορείτε να βρείτε εγγράψιμους καταλόγους χρησιμοποιώντας: `find / -writable -type d 2>/dev/null`

**Σημειώστε ότι δεν υποστηρίζουν όλοι οι κατάλογοι σε μια μηχανή linux το suid bit!** Για να ελέγξετε ποιες καταλόγοι υποστηρίζουν το suid bit εκτελέστε `mount | grep -v "nosuid"` Για παράδειγμα, συνήθως οι `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` και `/var/lib/lxcfs` δεν υποστηρίζουν το suid bit.

Σημειώστε επίσης ότι αν μπορείτε να **mountάρετε το `/etc`** ή οποιονδήποτε άλλο φάκελο **που περιέχει αρχεία ρυθμίσεων**, μπορείτε να τα αλλάξετε από το docker κοντέινερ ως root προκειμένου να **τα καταχραστείτε στον host** και να ανυψώσετε δικαιώματα (ίσως τροποποιώντας το `/etc/shadow`)
{% endhint %}

### Διαφυγή από το κοντέινερ

* **`--privileged`** -> Με αυτή τη σημαία [αφαιρείτε όλη την απομόνωση από το κοντέινερ](docker-privileged.md#what-affects). Ελέγξτε τεχνικές για [να διαφύγετε από κοντέινερ με προνόμια ως root](docker-breakout-privilege-escalation/#automatic-enumeration-and-escape).
* **`--cap-add=<CAPABILITY/ALL> [--security-opt apparmor=unconfined] [--security-opt seccomp=unconfined] [-security-opt label:disable]`** -> Για [να ανυψώσετε καταχρώντας ικανότητες](../linux-capabilities.md), **δώστε αυτή την ικανότητα στο κοντέινερ** και απενεργοποιήστε άλλες μεθόδους προστασίας που μπορεί να εμποδίσουν την εκμετάλλευση να λειτουργήσει.

### Curl

Σε αυτή τη σελίδα έχουμε συζητήσει τρόπους για να ανυψώσετε δικαιώματα χρησιμοποιώντας σημαίες docker, μπορείτε να βρείτε **τρόπους να καταχραστείτε αυτές τις μεθόδους χρησιμοποιώντας την εντολή curl** στη σελίδα:

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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
