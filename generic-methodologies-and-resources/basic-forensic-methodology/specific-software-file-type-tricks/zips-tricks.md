# ZIPs tricks

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

**Εργαλεία γραμμής εντολών** για τη διαχείριση **αρχείων zip** είναι απαραίτητα για τη διάγνωση, επισκευή και αποκρυπτογράφηση αρχείων zip. Ακολουθούν μερικά βασικά εργαλεία:

- **`unzip`**: Αποκαλύπτει γιατί ένα αρχείο zip μπορεί να μην αποσυμπιέζεται.
- **`zipdetails -v`**: Προσφέρει λεπτομερή ανάλυση των πεδίων μορφής αρχείου zip.
- **`zipinfo`**: Λίστα περιεχομένων ενός αρχείου zip χωρίς να τα εξάγει.
- **`zip -F input.zip --out output.zip`** και **`zip -FF input.zip --out output.zip`**: Προσπαθούν να επισκευάσουν κατεστραμμένα αρχεία zip.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Ένα εργαλείο για την αποκρυπτογράφηση κωδικών πρόσβασης zip με brute-force, αποτελεσματικό για κωδικούς πρόσβασης έως περίπου 7 χαρακτήρες.

Η [Προδιαγραφή μορφής αρχείου Zip](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) παρέχει λεπτομερείς πληροφορίες σχετικά με τη δομή και τα πρότυπα των αρχείων zip.

Είναι κρίσιμο να σημειωθεί ότι τα αρχεία zip που προστατεύονται με κωδικό πρόσβασης **δεν κρυπτογραφούν τα ονόματα αρχείων ή τα μεγέθη αρχείων** εντός τους, μια αδυναμία ασφαλείας που δεν μοιράζονται τα αρχεία RAR ή 7z που κρυπτογραφούν αυτές τις πληροφορίες. Επιπλέον, τα αρχεία zip που κρυπτογραφούνται με την παλαιότερη μέθοδο ZipCrypto είναι ευάλωτα σε μια **επίθεση σε απλή μορφή** εάν είναι διαθέσιμο ένα μη κρυπτογραφημένο αντίγραφο ενός συμπιεσμένου αρχείου. Αυτή η επίθεση εκμεταλλεύεται το γνωστό περιεχόμενο για να σπάσει τον κωδικό πρόσβασης του zip, μια ευπάθεια που αναλύεται στο [άρθρο του HackThis](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) και εξηγείται περαιτέρω σε [αυτή την ακαδημαϊκή εργασία](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). Ωστόσο, τα αρχεία zip που ασφαλίζονται με κρυπτογράφηση **AES-256** είναι ανθεκτικά σε αυτήν την επίθεση σε απλή μορφή, επιδεικνύοντας τη σημασία της επιλογής ασφαλών μεθόδων κρυπτογράφησης για ευαίσθητα δεδομένα.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

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
