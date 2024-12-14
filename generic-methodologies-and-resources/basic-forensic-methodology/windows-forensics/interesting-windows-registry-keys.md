# Interesting Windows Registry Keys

### Interesting Windows Registry Keys

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


### **Windows Version and Owner Info**
- Βρίσκεται στο **`Software\Microsoft\Windows NT\CurrentVersion`**, θα βρείτε την έκδοση των Windows, το Service Pack, τον χρόνο εγκατάστασης και το όνομα του καταχωρημένου ιδιοκτήτη με σαφή τρόπο.

### **Computer Name**
- Το όνομα υπολογιστή βρίσκεται κάτω από **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Time Zone Setting**
- Η ζώνη ώρας του συστήματος αποθηκεύεται στο **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Access Time Tracking**
- Από προεπιλογή, η παρακολούθηση του τελευταίου χρόνου πρόσβασης είναι απενεργοποιημένη (**`NtfsDisableLastAccessUpdate=1`**). Για να την ενεργοποιήσετε, χρησιμοποιήστε:
`fsutil behavior set disablelastaccess 0`

### Windows Versions and Service Packs
- Η **έκδοση των Windows** υποδεικνύει την έκδοση (π.χ. Home, Pro) και την κυκλοφορία της (π.χ. Windows 10, Windows 11), ενώ τα **Service Packs** είναι ενημερώσεις που περιλαμβάνουν διορθώσεις και, μερικές φορές, νέες δυνατότητες.

### Enabling Last Access Time
- Η ενεργοποίηση της παρακολούθησης του τελευταίου χρόνου πρόσβασης σας επιτρέπει να δείτε πότε άνοιξαν τελευταία τα αρχεία, κάτι που μπορεί να είναι κρίσιμο για την εγκληματολογική ανάλυση ή την παρακολούθηση του συστήματος.

### Network Information Details
- Η μητρώο περιέχει εκτενή δεδομένα σχετικά με τις ρυθμίσεις δικτύου, συμπεριλαμβανομένων **τύπων δικτύων (ασύρματα, καλωδιακά, 3G)** και **κατηγοριών δικτύου (Δημόσιο, Ιδιωτικό/Σπίτι, Τομέας/Εργασία)**, που είναι ζωτικής σημασίας για την κατανόηση των ρυθμίσεων ασφαλείας δικτύου και των δικαιωμάτων.

### Client Side Caching (CSC)
- **CSC** βελτιώνει την πρόσβαση σε αρχεία εκτός σύνδεσης αποθηκεύοντας αντίγραφα κοινών αρχείων. Διαφορετικές ρυθμίσεις **CSCFlags** ελέγχουν πώς και ποια αρχεία αποθηκεύονται στην κρυφή μνήμη, επηρεάζοντας την απόδοση και την εμπειρία του χρήστη, ειδικά σε περιβάλλοντα με διαλείπουσα συνδεσιμότητα.

### AutoStart Programs
- Τα προγράμματα που αναφέρονται σε διάφορα κλειδιά μητρώου `Run` και `RunOnce` εκκινούν αυτόματα κατά την εκκίνηση, επηρεάζοντας τον χρόνο εκκίνησης του συστήματος και ενδεχομένως αποτελώντας σημεία ενδιαφέροντος για την αναγνώριση κακόβουλου λογισμικού ή ανεπιθύμητου λογισμικού.

### Shellbags
- **Shellbags** όχι μόνο αποθηκεύουν προτιμήσεις για τις προβολές φακέλων αλλά παρέχουν επίσης εγκληματολογικά στοιχεία πρόσβασης φακέλων ακόμη και αν ο φάκελος δεν υπάρχει πια. Είναι ανεκτίμητα για τις έρευνες, αποκαλύπτοντας δραστηριότητα χρηστών που δεν είναι προφανής μέσω άλλων μέσων.

### USB Information and Forensics
- Οι λεπτομέρειες που αποθηκεύονται στο μητρώο σχετικά με τις συσκευές USB μπορούν να βοηθήσουν στην ανίχνευση ποια συσκευές συνδέθηκαν σε έναν υπολογιστή, ενδεχομένως συνδέοντας μια συσκευή με ευαίσθητες μεταφορές αρχείων ή περιστατικά μη εξουσιοδοτημένης πρόσβασης.

### Volume Serial Number
- Ο **Αριθμός Σειράς Όγκου** μπορεί να είναι κρίσιμος για την παρακολούθηση της συγκεκριμένης περίπτωσης ενός συστήματος αρχείων, χρήσιμος σε εγκληματολογικά σενάρια όπου πρέπει να καθοριστεί η προέλευση ενός αρχείου σε διάφορες συσκευές.

### **Shutdown Details**
- Ο χρόνος και ο αριθμός τερματισμού (ο τελευταίος μόνο για XP) διατηρούνται στο **`System\ControlSet001\Control\Windows`** και **`System\ControlSet001\Control\Watchdog\Display`**.

### **Network Configuration**
- Για λεπτομερείς πληροφορίες σχετικά με τη διεπαφή δικτύου, ανατρέξτε στο **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Οι πρώτοι και τελευταίοι χρόνοι σύνδεσης δικτύου, συμπεριλαμβανομένων των συνδέσεων VPN, καταγράφονται κάτω από διάφορες διαδρομές στο **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Shared Folders**
- Οι κοινόχρηστοι φάκελοι και οι ρυθμίσεις βρίσκονται κάτω από **`System\ControlSet001\Services\lanmanserver\Shares`**. Οι ρυθμίσεις Client Side Caching (CSC) καθορίζουν τη διαθεσιμότητα αρχείων εκτός σύνδεσης.

### **Programs that Start Automatically**
- Διαδρομές όπως **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** και παρόμοιες καταχωρήσεις κάτω από `Software\Microsoft\Windows\CurrentVersion` περιγράφουν προγράμματα που έχουν ρυθμιστεί να εκκινούν κατά την εκκίνηση.

### **Searches and Typed Paths**
- Οι αναζητήσεις του Explorer και οι πληκτρολογημένες διαδρομές παρακολουθούνται στο μητρώο κάτω από **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** για WordwheelQuery και TypedPaths, αντίστοιχα.

### **Recent Documents and Office Files**
- Τα πρόσφατα έγγραφα και τα αρχεία Office που έχουν προσπελαστεί σημειώνονται στο `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` και σε συγκεκριμένες διαδρομές έκδοσης Office.

### **Most Recently Used (MRU) Items**
- Οι λίστες MRU, που υποδεικνύουν πρόσφατες διαδρομές αρχείων και εντολές, αποθηκεύονται σε διάφορους υποκλειδικούς `ComDlg32` και `Explorer` κάτω από `NTUSER.DAT`.

### **User Activity Tracking**
- Η δυνατότητα User Assist καταγράφει λεπτομερείς στατιστικές χρήσης εφαρμογών, συμπεριλαμβανομένου του αριθμού εκτελέσεων και του τελευταίου χρόνου εκτέλεσης, στο **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Shellbags Analysis**
- Τα Shellbags, που αποκαλύπτουν λεπτομέρειες πρόσβασης φακέλων, αποθηκεύονται στο `USRCLASS.DAT` και `NTUSER.DAT` κάτω από `Software\Microsoft\Windows\Shell`. Χρησιμοποιήστε **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** για ανάλυση.

### **USB Device History**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** και **`HKLM\SYSTEM\ControlSet001\Enum\USB`** περιέχουν πλούσιες λεπτομέρειες σχετικά με τις συνδεδεμένες συσκευές USB, συμπεριλαμβανομένου του κατασκευαστή, του ονόματος προϊόντος και των χρονικών σημείων σύνδεσης.
- Ο χρήστης που σχετίζεται με μια συγκεκριμένη συσκευή USB μπορεί να προσδιοριστεί αναζητώντας τις βάσεις `NTUSER.DAT` για το **{GUID}** της συσκευής.
- Η τελευταία τοποθετημένη συσκευή και ο αριθμός σειράς όγκου της μπορούν να ανιχνευθούν μέσω των `System\MountedDevices` και `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, αντίστοιχα.

This guide condenses the crucial paths and methods for accessing detailed system, network, and user activity information on Windows systems, aiming for clarity and usability.



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
