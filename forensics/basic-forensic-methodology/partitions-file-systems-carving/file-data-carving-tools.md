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


# Εργαλεία Carving

## Autopsy

Το πιο κοινό εργαλείο που χρησιμοποιείται στην ψηφιακή εγκληματολογία για την εξαγωγή αρχείων από εικόνες είναι το [**Autopsy**](https://www.autopsy.com/download/). Κατεβάστε το, εγκαταστήστε το και κάντε το να επεξεργαστεί το αρχείο για να βρείτε "κρυφά" αρχεία. Σημειώστε ότι το Autopsy έχει σχεδιαστεί για να υποστηρίζει δισκοειδείς εικόνες και άλλου είδους εικόνες, αλλά όχι απλά αρχεία.

## Binwalk <a id="binwalk"></a>

**Binwalk** είναι ένα εργαλείο για την αναζήτηση δυαδικών αρχείων όπως εικόνες και αρχεία ήχου για ενσωματωμένα αρχεία και δεδομένα. Μπορεί να εγκατασταθεί με `apt`, ωστόσο η [πηγή](https://github.com/ReFirmLabs/binwalk) μπορεί να βρεθεί στο github.  
**Χρήσιμες εντολές**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

Ένα άλλο κοινό εργαλείο για να βρείτε κρυφά αρχεία είναι το **foremost**. Μπορείτε να βρείτε το αρχείο ρύθμισης του foremost στο `/etc/foremost.conf`. Αν θέλετε απλώς να αναζητήσετε ορισμένα συγκεκριμένα αρχεία, αποσχολιάστε τα. Αν δεν αποσχολιάσετε τίποτα, το foremost θα αναζητήσει τους προεπιλεγμένους τύπους αρχείων που είναι ρυθμισμένοι.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** είναι ένα άλλο εργαλείο που μπορεί να χρησιμοποιηθεί για να βρει και να εξάγει **αρχεία ενσωματωμένα σε ένα αρχείο**. Σε αυτή την περίπτωση, θα χρειαστεί να αφαιρέσετε το σχόλιο από το αρχείο ρυθμίσεων \(_/etc/scalpel/scalpel.conf_\) τους τύπους αρχείων που θέλετε να εξάγει.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

Αυτό το εργαλείο έρχεται μέσα στο kali αλλά μπορείτε να το βρείτε εδώ: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

Αυτό το εργαλείο μπορεί να σαρώσει μια εικόνα και θα **εξάγει pcaps** μέσα σε αυτή, **πληροφορίες δικτύου (URLs, τομείς, IPs, MACs, emails)** και περισσότερα **αρχεία**. Πρέπει απλώς να κάνετε:
```text
bulk_extractor memory.img -o out_folder
```
Πλοηγηθείτε μέσα από **όλες τις πληροφορίες** που έχει συγκεντρώσει το εργαλείο \(κωδικοί πρόσβασης;\), **αναλύστε** τα **πακέτα** \(διαβάστε[ **Ανάλυση Pcaps**](../pcap-inspection/)\), αναζητήστε **παράξενους τομείς** \(τομείς που σχετίζονται με **malware** ή **μη υπάρχοντες**\).

## PhotoRec

Μπορείτε να το βρείτε στο [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)

Έρχεται με GUI και CLI έκδοση. Μπορείτε να επιλέξετε τους **τύπους αρχείων** που θέλετε να αναζητήσει το PhotoRec.

![](../../../.gitbook/assets/image%20%28524%29.png)

# Ειδικά Εργαλεία Κατασκευής Δεδομένων

## FindAES

Αναζητά κλειδιά AES ψάχνοντας για τα χρονοδιαγράμματα κλειδιών τους. Ικανό να βρει κλειδιά 128, 192 και 256 bit, όπως αυτά που χρησιμοποιούνται από το TrueCrypt και το BitLocker.

Κατεβάστε [εδώ](https://sourceforge.net/projects/findaes/).

# Συμπληρωματικά εργαλεία

Μπορείτε να χρησιμοποιήσετε το [**viu** ](https://github.com/atanunq/viu) για να δείτε εικόνες από το τερματικό.  
Μπορείτε να χρησιμοποιήσετε το εργαλείο γραμμής εντολών linux **pdftotext** για να μετατρέψετε ένα pdf σε κείμενο και να το διαβάσετε.

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
