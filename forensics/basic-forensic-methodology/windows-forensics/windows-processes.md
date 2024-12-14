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


## smss.exe

**Διαχειριστής Συνεδρίας**.\
Η Συνεδρία 0 ξεκινά **csrss.exe** και **wininit.exe** (**Υπηρεσίες** **OS**) ενώ η Συνεδρία 1 ξεκινά **csrss.exe** και **winlogon.exe** (**Συνεδρία** **Χρήστη**). Ωστόσο, θα πρέπει να δείτε **μόνο μία διαδικασία** αυτού του **δυαδικού** χωρίς παιδιά στο δέντρο διαδικασιών.

Επίσης, συνεδρίες εκτός από τις 0 και 1 μπορεί να σημαίνουν ότι συμβαίνουν συνεδρίες RDP.


## csrss.exe

**Διαδικασία Υποσυστήματος Πελάτη/Διακομιστή**.\
Διαχειρίζεται **διαδικασίες** και **νήματα**, καθιστά διαθέσιμη την **API** **Windows** για άλλες διαδικασίες και επίσης **χαρτογραφεί γράμματα δίσκων**, δημιουργεί **temp αρχεία** και χειρίζεται τη διαδικασία **τερματισμού**.

Υπάρχει μία **που εκτελείται στη Συνεδρία 0 και άλλη μία στη Συνεδρία 1** (έτσι **2 διαδικασίες** στο δέντρο διαδικασιών). Μία άλλη δημιουργείται **ανά νέα Συνεδρία**.


## winlogon.exe

**Διαδικασία Σύνδεσης Windows**.\
Είναι υπεύθυνη για τη **σύνδεση**/**αποσύνδεση** του χρήστη. Ξεκινά το **logonui.exe** για να ζητήσει όνομα χρήστη και κωδικό πρόσβασης και στη συνέχεια καλεί το **lsass.exe** για να τα επαληθεύσει.

Στη συνέχεια, ξεκινά το **userinit.exe** το οποίο καθορίζεται στο **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** με το κλειδί **Userinit**.

Επιπλέον, το προηγούμενο μητρώο θα πρέπει να έχει **explorer.exe** στο **κλειδί Shell** ή μπορεί να καταχραστεί ως μέθοδος **επιμονής κακόβουλου λογισμικού**.


## wininit.exe

**Διαδικασία Αρχικοποίησης Windows**. \
Ξεκινά **services.exe**, **lsass.exe** και **lsm.exe** στη Συνεδρία 0. Θα πρέπει να υπάρχει μόνο 1 διαδικασία.


## userinit.exe

**Εφαρμογή Σύνδεσης Userinit**.\
Φορτώνει το **ntduser.dat στο HKCU** και αρχικοποιεί το **περιβάλλον** **χρήστη** και εκτελεί **σενάρια** **σύνδεσης** και **GPO**.

Ξεκινά το **explorer.exe**.


## lsm.exe

**Διαχειριστής Τοπικής Συνεδρίας**.\
Δουλεύει με το smss.exe για να χειριστεί τις συνεδρίες χρηστών: Σύνδεση/αποσύνδεση, εκκίνηση κελύφους, κλείδωμα/ξεκλείδωμα επιφάνειας εργασίας, κ.λπ.

Μετά το W7, το lsm.exe μετατράπηκε σε υπηρεσία (lsm.dll).

Θα πρέπει να υπάρχει μόνο 1 διαδικασία στο W7 και από αυτές μια υπηρεσία που εκτελεί τη DLL.


## services.exe

**Διαχειριστής Ελέγχου Υπηρεσιών**.\
Φορτώνει **υπηρεσίες** που έχουν ρυθμιστεί ως **αυτόματη εκκίνηση** και **οδηγούς**.

Είναι η γονική διαδικασία του **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** και πολλών άλλων.

Οι υπηρεσίες ορίζονται στο `HKLM\SYSTEM\CurrentControlSet\Services` και αυτή η διαδικασία διατηρεί μια βάση δεδομένων στη μνήμη με πληροφορίες υπηρεσιών που μπορούν να ερωτηθούν από το sc.exe.

Σημειώστε πώς **ορισμένες** **υπηρεσίες** θα εκτελούνται σε **δική τους διαδικασία** και άλλες θα **μοιράζονται μια διαδικασία svchost.exe**.

Θα πρέπει να υπάρχει μόνο 1 διαδικασία.


## lsass.exe

**Υποσύστημα Τοπικής Ασφάλειας**.\
Είναι υπεύθυνη για την **αυθεντικοποίηση** του χρήστη και τη δημιουργία των **ασφαλιστικών** **token**. Χρησιμοποιεί πακέτα αυθεντικοποίησης που βρίσκονται στο `HKLM\System\CurrentControlSet\Control\Lsa`.

Γράφει στο **μητρώο** **σφαλμάτων** **ασφαλείας** και θα πρέπει να υπάρχει μόνο 1 διαδικασία.

Λάβετε υπόψη ότι αυτή η διαδικασία είναι πολύ επιρρεπής σε επιθέσεις για την εξαγωγή κωδικών πρόσβασης.


## svchost.exe

**Γενική Διαδικασία Φιλοξενίας Υπηρεσιών**.\
Φιλοξενεί πολλές υπηρεσίες DLL σε μία κοινή διαδικασία.

Συνήθως, θα βρείτε ότι το **svchost.exe** εκκινείται με την παράμετρο `-k`. Αυτό θα εκκινήσει ένα ερώτημα στο μητρώο **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** όπου θα υπάρχει ένα κλειδί με το επιχείρημα που αναφέρεται στο -k που θα περιέχει τις υπηρεσίες που θα εκκινηθούν στην ίδια διαδικασία.

Για παράδειγμα: `-k UnistackSvcGroup` θα εκκινήσει: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Εάν η **παράμετρος `-s`** χρησιμοποιείται επίσης με ένα επιχείρημα, τότε ζητείται από το svchost να **εκκινήσει μόνο την καθορισμένη υπηρεσία** σε αυτό το επιχείρημα.

Θα υπάρχουν πολλές διαδικασίες του `svchost.exe`. Εάν καμία από αυτές **δεν χρησιμοποιεί την παράμετρο `-k`**, τότε αυτό είναι πολύ ύποπτο. Εάν διαπιστώσετε ότι **το services.exe δεν είναι η γονική διαδικασία**, αυτό είναι επίσης πολύ ύποπτο.


## taskhost.exe

Αυτή η διαδικασία λειτουργεί ως φιλοξενητής για διαδικασίες που εκτελούνται από DLLs. Φορτώνει επίσης τις υπηρεσίες που εκτελούνται από DLLs.

Στο W8 αυτό ονομάζεται taskhostex.exe και στο W10 taskhostw.exe.


## explorer.exe

Αυτή είναι η διαδικασία που είναι υπεύθυνη για την **επιφάνεια εργασίας του χρήστη** και την εκκίνηση αρχείων μέσω επεκτάσεων αρχείων.

**Μόνο 1** διαδικασία θα πρέπει να δημιουργείται **ανά χρήστη που έχει συνδεθεί**.

Αυτό εκτελείται από το **userinit.exe** το οποίο θα πρέπει να τερματιστεί, έτσι **κανένας γονέας** δεν θα πρέπει να εμφανίζεται για αυτή τη διαδικασία.


# Ανίχνευση Κακόβουλων Διαδικασιών

* Εκτελείται από την αναμενόμενη διαδρομή; (Καμία δυαδική Windows δεν εκτελείται από προσωρινή τοποθεσία)
* Επικοινωνεί με περίεργες IP;
* Ελέγξτε τις ψηφιακές υπογραφές (τα αρχεία της Microsoft θα πρέπει να είναι υπογεγραμμένα)
* Είναι σωστά γραμμένο;
* Εκτελείται υπό την αναμενόμενη SID;
* Είναι η γονική διαδικασία η αναμενόμενη (αν υπάρχει);
* Είναι οι διαδικασίες παιδιών οι αναμενόμενες; (κανένα cmd.exe, wscript.exe, powershell.exe..?)

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
