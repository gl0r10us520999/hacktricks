# macOS Defensive Apps

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Firewalls

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): Θα παρακολουθεί κάθε σύνδεση που γίνεται από κάθε διαδικασία. Ανάλογα με τη λειτουργία (σιωπηλή επιτρεπόμενη σύνδεση, σιωπηλή άρνηση σύνδεσης και ειδοποίηση) θα **σας δείξει μια ειδοποίηση** κάθε φορά που καθορίζεται μια νέα σύνδεση. Έχει επίσης μια πολύ ωραία διεπαφή χρήστη για να δείτε όλες αυτές τις πληροφορίες.
* [**LuLu**](https://objective-see.org/products/lulu.html): Τείχος προστασίας Objective-See. Αυτό είναι ένα βασικό τείχος προστασίας που θα σας ειδοποιεί για ύποπτες συνδέσεις (έχει διεπαφή χρήστη αλλά δεν είναι τόσο εντυπωσιακή όσο αυτή του Little Snitch).

## Persistence detection

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Εφαρμογή Objective-See που θα αναζητήσει σε διάφορες τοποθεσίες όπου **το κακόβουλο λογισμικό θα μπορούσε να επιμένει** (είναι ένα εργαλείο μιας χρήσης, όχι υπηρεσία παρακολούθησης).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): Όπως το KnockKnock παρακολουθώντας διαδικασίες που δημιουργούν επιμονή.

## Keyloggers detection

* [**ReiKey**](https://objective-see.org/products/reikey.html): Εφαρμογή Objective-See για να βρείτε **keyloggers** που εγκαθιστούν "event taps" πληκτρολογίου.
