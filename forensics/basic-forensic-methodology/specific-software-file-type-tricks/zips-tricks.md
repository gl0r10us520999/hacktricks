# ZIPs tricks

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

**Εργαλεία γραμμής εντολών** για τη διαχείριση **zip αρχείων** είναι απαραίτητα για τη διάγνωση, επισκευή και σπάσιμο zip αρχείων. Ακολουθούν μερικά βασικά εργαλεία:

- **`unzip`**: Αποκαλύπτει γιατί ένα zip αρχείο μπορεί να μην αποσυμπιέζεται.
- **`zipdetails -v`**: Προσφέρει λεπτομερή ανάλυση των πεδίων μορφής zip αρχείου.
- **`zipinfo`**: Λίστα περιεχομένων ενός zip αρχείου χωρίς να τα εξάγει.
- **`zip -F input.zip --out output.zip`** και **`zip -FF input.zip --out output.zip`**: Προσπαθούν να επισκευάσουν κατεστραμμένα zip αρχεία.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Ένα εργαλείο για brute-force σπάσιμο κωδικών πρόσβασης zip, αποτελεσματικό για κωδικούς πρόσβασης έως περίπου 7 χαρακτήρες.

Η [προδιαγραφή μορφής αρχείου Zip](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) παρέχει λεπτομερείς πληροφορίες σχετικά με τη δομή και τα πρότυπα των zip αρχείων.

Είναι κρίσιμο να σημειωθεί ότι τα zip αρχεία που προστατεύονται με κωδικό πρόσβασης **δεν κρυπτογραφούν τα ονόματα αρχείων ή τα μεγέθη αρχείων** εντός τους, μια αδυναμία ασφαλείας που δεν μοιράζονται τα RAR ή 7z αρχεία που κρυπτογραφούν αυτές τις πληροφορίες. Επιπλέον, τα zip αρχεία που κρυπτογραφούνται με τη παλαιότερη μέθοδο ZipCrypto είναι ευάλωτα σε μια **επίθεση σε απλή μορφή** αν είναι διαθέσιμη μια μη κρυπτογραφημένη αντίγραφο ενός συμπιεσμένου αρχείου. Αυτή η επίθεση εκμεταλλεύεται το γνωστό περιεχόμενο για να σπάσει τον κωδικό πρόσβασης του zip, μια ευπάθεια που αναλύεται στο [άρθρο του HackThis](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) και εξηγείται περαιτέρω σε [αυτή την ακαδημαϊκή εργασία](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). Ωστόσο, τα zip αρχεία που ασφαλίζονται με κρυπτογράφηση **AES-256** είναι ανθεκτικά σε αυτή την επίθεση σε απλή μορφή, επιδεικνύοντας τη σημασία της επιλογής ασφαλών μεθόδων κρυπτογράφησης για ευαίσθητα δεδομένα.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
