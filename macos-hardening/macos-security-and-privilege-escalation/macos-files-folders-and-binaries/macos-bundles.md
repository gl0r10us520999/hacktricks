# macOS Bundles

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

Τα bundles στο macOS λειτουργούν ως δοχεία για μια ποικιλία πόρων, συμπεριλαμβανομένων εφαρμογών, βιβλιοθηκών και άλλων απαραίτητων αρχείων, κάνοντάς τα να εμφανίζονται ως ενιαία αντικείμενα στο Finder, όπως τα γνωστά αρχεία `*.app`. Το πιο συχνά συναντώμενο bundle είναι το `.app` bundle, αν και άλλοι τύποι όπως `.framework`, `.systemextension` και `.kext` είναι επίσης διαδεδομένοι.

### Essential Components of a Bundle

Μέσα σε ένα bundle, ιδιαίτερα μέσα στον φάκελο `<application>.app/Contents/`, φιλοξενούνται διάφοροι σημαντικοί πόροι:

* **\_CodeSignature**: Αυτός ο φάκελος αποθηκεύει λεπτομέρειες υπογραφής κώδικα που είναι ζωτικής σημασίας για την επαλήθευση της ακεραιότητας της εφαρμογής. Μπορείτε να ελέγξετε τις πληροφορίες υπογραφής κώδικα χρησιμοποιώντας εντολές όπως: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Περιέχει το εκτελέσιμο δυαδικό αρχείο της εφαρμογής που εκτελείται κατά την αλληλεπίδραση του χρήστη.
* **Resources**: Ένας αποθηκευτικός χώρος για τα στοιχεία διεπαφής χρήστη της εφαρμογής, συμπεριλαμβανομένων εικόνων, εγγράφων και περιγραφών διεπαφής (nib/xib αρχεία).
* **Info.plist**: Λειτουργεί ως το κύριο αρχείο ρύθμισης της εφαρμογής, κρίσιμο για το σύστημα ώστε να αναγνωρίζει και να αλληλεπιδρά με την εφαρμογή κατάλληλα.

#### Important Keys in Info.plist

Το αρχείο `Info.plist` είναι θεμελιώδες για τη ρύθμιση της εφαρμογής, περιέχοντας κλειδιά όπως:

* **CFBundleExecutable**: Προσδιορίζει το όνομα του κύριου εκτελέσιμου αρχείου που βρίσκεται στον φάκελο `Contents/MacOS`.
* **CFBundleIdentifier**: Παρέχει έναν παγκόσμιο αναγνωριστικό για την εφαρμογή, που χρησιμοποιείται εκτενώς από το macOS για τη διαχείριση εφαρμογών.
* **LSMinimumSystemVersion**: Υποδεικνύει την ελάχιστη έκδοση του macOS που απαιτείται για να εκτελείται η εφαρμογή.

### Exploring Bundles

Για να εξερευνήσετε το περιεχόμενο ενός bundle, όπως το `Safari.app`, μπορεί να χρησιμοποιηθεί η εξής εντολή: `bash ls -lR /Applications/Safari.app/Contents`

Αυτή η εξερεύνηση αποκαλύπτει φακέλους όπως `_CodeSignature`, `MacOS`, `Resources`, και αρχεία όπως `Info.plist`, καθένα από τα οποία εξυπηρετεί μια μοναδική σκοπιμότητα από την ασφάλιση της εφαρμογής μέχρι τον καθορισμό της διεπαφής χρήστη και των λειτουργικών παραμέτρων.

#### Additional Bundle Directories

Πέρα από τους κοινούς φακέλους, τα bundles μπορεί επίσης να περιλαμβάνουν:

* **Frameworks**: Περιέχει bundled frameworks που χρησιμοποιούνται από την εφαρμογή. Τα frameworks είναι σαν dylibs με επιπλέον πόρους.
* **PlugIns**: Ένας φάκελος για plug-ins και επεκτάσεις που ενισχύουν τις δυνατότητες της εφαρμογής.
* **XPCServices**: Περιέχει XPC υπηρεσίες που χρησιμοποιούνται από την εφαρμογή για επικοινωνία εκτός διαδικασίας.

Αυτή η δομή διασφαλίζει ότι όλα τα απαραίτητα στοιχεία είναι ενσωματωμένα μέσα στο bundle, διευκολύνοντας ένα αρθρωτό και ασφαλές περιβάλλον εφαρμογής.

Για περισσότερες λεπτομέρειες σχετικά με τα κλειδιά του `Info.plist` και τις σημασίες τους, η τεκμηρίωση προγραμματιστών της Apple παρέχει εκτενείς πόρους: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
