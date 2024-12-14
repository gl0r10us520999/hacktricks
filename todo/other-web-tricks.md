# Άλλες Τεχνικές Ιστού

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξη HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε τεχνικές hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

### Header Host

Πολλές φορές το back-end εμπιστεύεται το **Host header** για να εκτελέσει κάποιες ενέργειες. Για παράδειγμα, μπορεί να χρησιμοποιήσει την τιμή του ως το **domain για να στείλει μια επαναφορά κωδικού πρόσβασης**. Έτσι, όταν λάβετε ένα email με έναν σύνδεσμο για να επαναφέρετε τον κωδικό σας, το domain που χρησιμοποιείται είναι αυτό που βάλατε στο Host header. Στη συνέχεια, μπορείτε να ζητήσετε την επαναφορά κωδικού πρόσβασης άλλων χρηστών και να αλλάξετε το domain σε ένα που ελέγχετε εσείς για να κλέψετε τους κωδικούς επαναφοράς τους. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
Σημειώστε ότι είναι πιθανό να μην χρειαστεί καν να περιμένετε τον χρήστη να κάνει κλικ στον σύνδεσμο επαναφοράς κωδικού για να αποκτήσετε το token, καθώς ίσως ακόμη και **φίλτρα spam ή άλλες ενδιάμεσες συσκευές/bots να κάνουν κλικ σε αυτό για να το αναλύσουν**.
{% endhint %}

### Boolean συνεδρίας

Ορισμένες φορές όταν ολοκληρώνετε σωστά κάποια επαλήθευση, το back-end θα **προσθέσει απλώς ένα boolean με την τιμή "True" σε ένα χαρακτηριστικό ασφαλείας της συνεδρίας σας**. Στη συνέχεια, ένα διαφορετικό endpoint θα γνωρίζει αν περάσατε επιτυχώς αυτή την επαλήθευση.\
Ωστόσο, αν **περάσετε την επαλήθευση** και η συνεδρία σας αποκτήσει αυτή την τιμή "True" στο χαρακτηριστικό ασφαλείας, μπορείτε να προσπαθήσετε να **πρόσβαση σε άλλους πόρους** που **εξαρτώνται από το ίδιο χαρακτηριστικό** αλλά που **δεν θα έπρεπε να έχετε άδειες** για πρόσβαση. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Λειτουργία εγγραφής

Δοκιμάστε να εγγραφείτε ως ήδη υπάρχων χρήστης. Δοκιμάστε επίσης να χρησιμοποιήσετε ισοδύναμους χαρακτήρες (τελείες, πολλές κενές θέσεις και Unicode).

### Κατάληψη emails

Εγγραφείτε σε ένα email, πριν το επιβεβαιώσετε αλλάξτε το email, στη συνέχεια, αν το νέο email επιβεβαίωσης σταλεί στο πρώτο εγγεγραμμένο email, μπορείτε να καταλάβετε οποιοδήποτε email. Ή αν μπορείτε να ενεργοποιήσετε το δεύτερο email επιβεβαιώνοντας το πρώτο, μπορείτε επίσης να καταλάβετε οποιονδήποτε λογαριασμό.

### Πρόσβαση στο Εσωτερικό servicedesk εταιρειών που χρησιμοποιούν atlassian

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### Μέθοδος TRACE

Οι προγραμματιστές μπορεί να ξεχάσουν να απενεργοποιήσουν διάφορες επιλογές αποσφαλμάτωσης στο περιβάλλον παραγωγής. Για παράδειγμα, η μέθοδος HTTP `TRACE` έχει σχεδιαστεί για διαγνωστικούς σκοπούς. Αν είναι ενεργοποιημένη, ο web server θα απαντήσει σε αιτήματα που χρησιμοποιούν τη μέθοδο `TRACE` επαναλαμβάνοντας στην απάντηση το ακριβές αίτημα που ελήφθη. Αυτή η συμπεριφορά είναι συχνά αβλαβής, αλλά περιστασιακά οδηγεί σε αποκάλυψη πληροφοριών, όπως το όνομα εσωτερικών επικεφαλίδων αυθεντικοποίησης που μπορεί να προστεθούν σε αιτήματα από αντίστροφους μεσολαβητές.![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξη HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε τεχνικές hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
