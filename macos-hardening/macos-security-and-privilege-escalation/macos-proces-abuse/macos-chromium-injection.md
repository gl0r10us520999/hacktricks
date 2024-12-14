# macOS Chromium Injection

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

## Basic Information

Οι περιηγητές που βασίζονται στο Chromium, όπως το Google Chrome, το Microsoft Edge, το Brave και άλλοι. Αυτοί οι περιηγητές είναι κατασκευασμένοι πάνω στο έργο ανοιχτού κώδικα Chromium, που σημαίνει ότι μοιράζονται μια κοινή βάση και, επομένως, έχουν παρόμοιες λειτουργίες και επιλογές προγραμματιστή.

#### `--load-extension` Flag

Η σημαία `--load-extension` χρησιμοποιείται κατά την εκκίνηση ενός περιηγητή που βασίζεται στο Chromium από τη γραμμή εντολών ή ένα σενάριο. Αυτή η σημαία επιτρέπει να **φορτωθεί αυτόματα μία ή περισσότερες επεκτάσεις** στον περιηγητή κατά την εκκίνηση.

#### `--use-fake-ui-for-media-stream` Flag

Η σημαία `--use-fake-ui-for-media-stream` είναι μια άλλη επιλογή γραμμής εντολών που μπορεί να χρησιμοποιηθεί για να ξεκινήσει περιηγητές που βασίζονται στο Chromium. Αυτή η σημαία έχει σχεδιαστεί για να **παρακάμπτει τις κανονικές προτροπές χρήστη που ζητούν άδεια για πρόσβαση σε ροές πολυμέσων από την κάμερα και το μικρόφωνο**. Όταν χρησιμοποιείται αυτή η σημαία, ο περιηγητής χορηγεί αυτόματα άδεια σε οποιαδήποτε ιστοσελίδα ή εφαρμογή ζητά πρόσβαση στην κάμερα ή το μικρόφωνο.

### Tools

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### Example
```bash
# Intercept traffic
voodoo intercept -b chrome
```
Βρείτε περισσότερα παραδείγματα στους συνδέσμους εργαλείων

## Αναφορές

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
