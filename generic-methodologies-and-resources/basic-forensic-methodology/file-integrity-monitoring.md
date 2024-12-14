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


# Βασική Γραμμή

Μια βασική γραμμή συνίσταται στη λήψη μιας στιγμιότυπης εικόνας ορισμένων τμημάτων ενός συστήματος για **να τη συγκρίνουμε με μια μελλοντική κατάσταση ώστε να επισημάνουμε τις αλλαγές**.

Για παράδειγμα, μπορείτε να υπολογίσετε και να αποθηκεύσετε το hash κάθε αρχείου του συστήματος αρχείων για να μπορέσετε να διαπιστώσετε ποια αρχεία έχουν τροποποιηθεί.\
Αυτό μπορεί επίσης να γίνει με τους λογαριασμούς χρηστών που έχουν δημιουργηθεί, τις διαδικασίες που εκτελούνται, τις υπηρεσίες που εκτελούνται και οτιδήποτε άλλο δεν θα έπρεπε να αλλάξει πολύ ή καθόλου.

## Παρακολούθηση Ακεραιότητας Αρχείων

Η Παρακολούθηση Ακεραιότητας Αρχείων (FIM) είναι μια κρίσιμη τεχνική ασφάλειας που προστατεύει τα IT περιβάλλοντα και τα δεδομένα παρακολουθώντας τις αλλαγές στα αρχεία. Περιλαμβάνει δύο βασικά βήματα:

1. **Σύγκριση Βασικής Γραμμής:** Καθιερώστε μια βασική γραμμή χρησιμοποιώντας χαρακτηριστικά αρχείων ή κρυπτογραφικούς ελέγχους (όπως MD5 ή SHA-2) για μελλοντικές συγκρίσεις ώστε να ανιχνεύσετε τροποποιήσεις.
2. **Ειδοποίηση Αλλαγών σε Πραγματικό Χρόνο:** Λάβετε άμεσες ειδοποιήσεις όταν τα αρχεία προσπελάζονται ή τροποποιούνται, συνήθως μέσω επεκτάσεων πυρήνα OS.

## Εργαλεία

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

## Αναφορές

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


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
