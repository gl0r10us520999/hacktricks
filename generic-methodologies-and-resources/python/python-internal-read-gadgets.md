# Python Internal Read Gadgets

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

Διαφορετικές ευπάθειες όπως [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) ή [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) μπορεί να σας επιτρέψουν να **διαβάσετε τα εσωτερικά δεδομένα του python αλλά δεν θα σας επιτρέψουν να εκτελέσετε κώδικα**. Επομένως, ένας pentester θα χρειαστεί να εκμεταλλευτεί αυτές τις άδειες ανάγνωσης για να **αποκτήσει ευαίσθητα προνόμια και να κλιμακώσει την ευπάθεια**.

### Flask - Read secret key

Η κύρια σελίδα μιας εφαρμογής Flask θα έχει πιθανώς το **`app`** παγκόσμιο αντικείμενο όπου αυτή η **μυστική ρύθμιση είναι διαμορφωμένη**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
Σε αυτή την περίπτωση είναι δυνατό να αποκτήσετε πρόσβαση σε αυτό το αντικείμενο χρησιμοποιώντας οποιοδήποτε gadget για **πρόσβαση σε παγκόσμια αντικείμενα** από τη σελίδα [**Bypass Python sandboxes**](bypass-python-sandboxes/).

Στην περίπτωση όπου **η ευπάθεια είναι σε διαφορετικό αρχείο python**, χρειάζεστε ένα gadget για να διασχίσετε τα αρχεία ώστε να φτάσετε στο κύριο για να **πρόσβαση στο παγκόσμιο αντικείμενο `app.secret_key`** για να αλλάξετε το μυστικό κλειδί του Flask και να μπορείτε να [**κλιμακώσετε δικαιώματα** γνωρίζοντας αυτό το κλειδί](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Ένα payload όπως αυτό [από αυτή τη γραφή](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Χρησιμοποιήστε αυτό το payload για να **αλλάξετε το `app.secret_key`** (το όνομα στην εφαρμογή σας μπορεί να είναι διαφορετικό) ώστε να μπορείτε να υπογράφετε νέα και πιο προνομιακά cookies flask.

### Werkzeug - machine\_id και node uuid

[**Χρησιμοποιώντας αυτά τα payload από αυτήν την αναφορά**](https://vozec.fr/writeups/tweedle-dum-dee/) θα μπορείτε να αποκτήσετε πρόσβαση στο **machine\_id** και το **uuid** node, τα οποία είναι τα **κύρια μυστικά** που χρειάζεστε για να [**δημιουργήσετε το Werkzeug pin**](../../network-services-pentesting/pentesting-web/werkzeug.md) που μπορείτε να χρησιμοποιήσετε για να αποκτήσετε πρόσβαση στην κονσόλα python στο `/console` αν είναι **ενεργοποιημένη η λειτουργία αποσφαλμάτωσης:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Σημειώστε ότι μπορείτε να αποκτήσετε την **τοπική διαδρομή του διακομιστή για το `app.py`** δημιουργώντας κάποιο **σφάλμα** στη σελίδα web που θα **δώσει τη διαδρομή**.
{% endhint %}

Αν η ευπάθεια είναι σε διαφορετικό αρχείο python, ελέγξτε το προηγούμενο κόλπο Flask για να αποκτήσετε πρόσβαση στα αντικείμενα από το κύριο αρχείο python.

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
