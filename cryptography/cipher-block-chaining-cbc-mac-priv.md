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


# CBC

Αν το **cookie** είναι **μόνο** το **όνομα χρήστη** (ή το πρώτο μέρος του cookie είναι το όνομα χρήστη) και θέλεις να προσποιηθείς το όνομα χρήστη "**admin**". Τότε, μπορείς να δημιουργήσεις το όνομα χρήστη **"bdmin"** και να **bruteforce** το **πρώτο byte** του cookie.

# CBC-MAC

**Κωδικός αυθεντικοποίησης μηνύματος με αλυσίδα μπλοκ κρυπτογράφησης** (**CBC-MAC**) είναι μια μέθοδος που χρησιμοποιείται στην κρυπτογραφία. Λειτουργεί παίρνοντας ένα μήνυμα και κρυπτογραφώντας το μπλοκ προς μπλοκ, όπου η κρυπτογράφηση κάθε μπλοκ συνδέεται με την προηγούμενη. Αυτή η διαδικασία δημιουργεί μια **αλυσίδα μπλοκ**, διασφαλίζοντας ότι η αλλαγή έστω και ενός bit του αρχικού μηνύματος θα οδηγήσει σε μια απρόβλεπτη αλλαγή στο τελευταίο μπλοκ των κρυπτογραφημένων δεδομένων. Για να γίνει ή να αντιστραφεί μια τέτοια αλλαγή, απαιτείται το κλειδί κρυπτογράφησης, διασφαλίζοντας την ασφάλεια.

Για να υπολογίσεις το CBC-MAC του μηνύματος m, κρυπτογραφείς το m σε λειτουργία CBC με μηδενικό αρχικοποιημένο διανύσμα και κρατάς το τελευταίο μπλοκ. Η παρακάτω εικόνα σκιαγραφεί τον υπολογισμό του CBC-MAC ενός μηνύματος που αποτελείται από μπλοκ![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) χρησιμοποιώντας ένα μυστικό κλειδί k και έναν κωδικό μπλοκ E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

Με το CBC-MAC συνήθως το **IV που χρησιμοποιείται είναι 0**.\
Αυτό είναι ένα πρόβλημα γιατί 2 γνωστά μηνύματα (`m1` και `m2`) ανεξάρτητα θα δημιουργήσουν 2 υπογραφές (`s1` και `s2`). Έτσι:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Τότε ένα μήνυμα που αποτελείται από τα m1 και m2 που συνδυάζονται (m3) θα δημιουργήσει 2 υπογραφές (s31 και s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Το οποίο είναι δυνατό να υπολογιστεί χωρίς να γνωρίζεις το κλειδί της κρυπτογράφησης.**

Φαντάσου ότι κρυπτογραφείς το όνομα **Administrator** σε **8bytes** μπλοκ:

* `Administ`
* `rator\00\00\00`

Μπορείς να δημιουργήσεις ένα όνομα χρήστη που ονομάζεται **Administ** (m1) και να ανακτήσεις την υπογραφή (s1).\
Τότε, μπορείς να δημιουργήσεις ένα όνομα χρήστη που ονομάζεται το αποτέλεσμα του `rator\00\00\00 XOR s1`. Αυτό θα δημιουργήσει `E(m2 XOR s1 XOR 0)` που είναι s32.\
Τώρα, μπορείς να χρησιμοποιήσεις το s32 ως την υπογραφή του πλήρους ονόματος **Administrator**.

### Summary

1. Πάρε την υπογραφή του ονόματος χρήστη **Administ** (m1) που είναι s1
2. Πάρε την υπογραφή του ονόματος χρήστη **rator\x00\x00\x00 XOR s1 XOR 0** είναι s32**.**
3. Ρύθμισε το cookie σε s32 και θα είναι ένα έγκυρο cookie για τον χρήστη **Administrator**.

# Attack Controlling IV

Αν μπορείς να ελέγξεις το χρησιμοποιούμενο IV η επίθεση θα μπορούσε να είναι πολύ εύκολη.\
Αν το cookie είναι απλώς το όνομα χρήστη που έχει κρυπτογραφηθεί, για να προσποιηθείς τον χρήστη "**administrator**" μπορείς να δημιουργήσεις τον χρήστη "**Administrator**" και θα πάρεις το cookie του.\
Τώρα, αν μπορείς να ελέγξεις το IV, μπορείς να αλλάξεις το πρώτο Byte του IV έτσι ώστε **IV\[0] XOR "A" == IV'\[0] XOR "a"** και να αναγεννήσεις το cookie για τον χρήστη **Administrator.** Αυτό το cookie θα είναι έγκυρο για **να προσποιηθείς** τον χρήστη **administrator** με το αρχικό **IV**.

## References

Περισσότερες πληροφορίες στο [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
