# Shadow Credentials

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

## Intro <a href="#3f17" id="3f17"></a>

**Check the original post for [όλες τις πληροφορίες σχετικά με αυτή την τεχνική](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab).**

As **summary**: αν μπορείτε να γράψετε στην **msDS-KeyCredentialLink** ιδιότητα ενός χρήστη/υπολογιστή, μπορείτε να ανακτήσετε το **NT hash αυτού του αντικειμένου**.

In the post, a method is outlined for setting up **public-private key authentication credentials** to acquire a unique **Service Ticket** that includes the target's NTLM hash. This process involves the encrypted NTLM_SUPPLEMENTAL_CREDENTIAL within the Privilege Attribute Certificate (PAC), which can be decrypted.

### Requirements

To apply this technique, certain conditions must be met:
- Απαιτείται τουλάχιστον ένας Windows Server 2016 Domain Controller.
- Ο Domain Controller πρέπει να έχει εγκατεστημένο ένα ψηφιακό πιστοποιητικό αυθεντικοποίησης διακομιστή.
- Το Active Directory πρέπει να είναι στο Windows Server 2016 Functional Level.
- Απαιτείται ένας λογαριασμός με εξουσιοδοτημένα δικαιώματα για να τροποποιήσει την ιδιότητα msDS-KeyCredentialLink του αντικειμένου στόχου.

## Abuse

The abuse of Key Trust for computer objects encompasses steps beyond obtaining a Ticket Granting Ticket (TGT) and the NTLM hash. The options include:
1. Δημιουργία ενός **RC4 silver ticket** για να ενεργεί ως προνομιούχοι χρήστες στον προοριζόμενο υπολογιστή.
2. Χρήση του TGT με **S4U2Self** για την προσποίηση **προνομιούχων χρηστών**, απαιτώντας τροποποιήσεις στο Service Ticket για να προστεθεί μια κατηγορία υπηρεσίας στο όνομα της υπηρεσίας.

A significant advantage of Key Trust abuse is its limitation to the attacker-generated private key, avoiding delegation to potentially vulnerable accounts and not requiring the creation of a computer account, which could be challenging to remove.

## Tools

### [**Whisker**](https://github.com/eladshamir/Whisker)

It's based on DSInternals providing a C# interface for this attack. Whisker and its Python counterpart, **pyWhisker**, enable manipulation of the `msDS-KeyCredentialLink` attribute to gain control over Active Directory accounts. These tools support various operations like adding, listing, removing, and clearing key credentials from the target object.

**Whisker** functions include:
- **Add**: Δημιουργεί ένα ζεύγος κλειδιών και προσθέτει μια πιστοποίηση κλειδιού.
- **List**: Εμφανίζει όλες τις καταχωρίσεις πιστοποίησης κλειδιού.
- **Remove**: Διαγράφει μια συγκεκριμένη πιστοποίηση κλειδιού.
- **Clear**: Διαγράφει όλες τις πιστοποιήσεις κλειδιού, ενδεχομένως διαταράσσοντας τη νόμιμη χρήση WHfB.
```shell
Whisker.exe add /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1
```
### [pyWhisker](https://github.com/ShutdownRepo/pywhisker)

Επεκτείνει τη λειτουργικότητα του Whisker σε **συστήματα βασισμένα σε UNIX**, αξιοποιώντας το Impacket και το PyDSInternals για ολοκληρωμένες δυνατότητες εκμετάλλευσης, συμπεριλαμβανομένης της καταγραφής, προσθήκης και αφαίρεσης KeyCredentials, καθώς και της εισαγωγής και εξαγωγής τους σε μορφή JSON.
```shell
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"
```
### [ShadowSpray](https://github.com/Dec0ne/ShadowSpray/)

Το ShadowSpray στοχεύει να **εκμεταλλευτεί τις άδειες GenericWrite/GenericAll που μπορεί να έχουν ευρείες ομάδες χρηστών σε αντικείμενα τομέα** για να εφαρμόσει ευρέως τα ShadowCredentials. Περιλαμβάνει την είσοδο στον τομέα, την επαλήθευση του λειτουργικού επιπέδου του τομέα, την καταμέτρηση αντικειμένων τομέα και την προσπάθεια προσθήκης KeyCredentials για την απόκτηση TGT και την αποκάλυψη NT hash. Οι επιλογές καθαρισμού και οι αναδρομικές τακτικές εκμετάλλευσης ενισχύουν τη χρησιμότητά του.

## References

* [https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
* [https://github.com/eladshamir/Whisker](https://github.com/eladshamir/Whisker)
* [https://github.com/Dec0ne/ShadowSpray/](https://github.com/Dec0ne/ShadowSpray/)
* [https://github.com/ShutdownRepo/pywhisker](https://github.com/ShutdownRepo/pywhisker)

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
