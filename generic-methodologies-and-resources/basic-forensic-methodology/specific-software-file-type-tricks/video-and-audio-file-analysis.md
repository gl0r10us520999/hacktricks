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

**Η χειραγώγηση αρχείων ήχου και βίντεο** είναι βασικό στοιχείο στις **προκλήσεις ψηφιακής εγκληματολογίας CTF**, εκμεταλλευόμενη **στεγανографία** και ανάλυση μεταδεδομένων για να κρύψει ή να αποκαλύψει μυστικά μηνύματα. Εργαλεία όπως το **[mediainfo](https://mediaarea.net/en/MediaInfo)** και το **`exiftool`** είναι απαραίτητα για την επιθεώρηση μεταδεδομένων αρχείων και την αναγνώριση τύπων περιεχομένου.

Για προκλήσεις ήχου, το **[Audacity](http://www.audacityteam.org/)** ξεχωρίζει ως ένα κορυφαίο εργαλείο για την προβολή κυματομορφών και την ανάλυση φασματογραμμάτων, απαραίτητο για την αποκάλυψη κειμένου που έχει κωδικοποιηθεί σε ήχο. Το **[Sonic Visualiser](http://www.sonicvisualiser.org/)** συνιστάται ιδιαίτερα για λεπτομερή ανάλυση φασματογραμμάτων. Το **Audacity** επιτρέπει τη χειραγώγηση ήχου όπως η επιβράδυνση ή η αναστροφή κομματιών για την ανίχνευση κρυμμένων μηνυμάτων. Το **[Sox](http://sox.sourceforge.net/)**, ένα εργαλείο γραμμής εντολών, διαπρέπει στη μετατροπή και επεξεργασία αρχείων ήχου.

**Η χειραγώγηση των λιγότερο σημαντικών bit (LSB)** είναι μια κοινή τεχνική στη στεγανографία ήχου και βίντεο, εκμεταλλευόμενη τα σταθερού μεγέθους κομμάτια αρχείων πολυμέσων για να ενσωματώσει δεδομένα διακριτικά. Το **[Multimon-ng](http://tools.kali.org/wireless-attacks/multimon-ng)** είναι χρήσιμο για την αποκωδικοποίηση μηνυμάτων που είναι κρυμμένα ως **DTMF τόνοι** ή **κωδικός Μορς**.

Οι προκλήσεις βίντεο συχνά περιλαμβάνουν μορφές κοντέινερ που συνδυάζουν ροές ήχου και βίντεο. Το **[FFmpeg](http://ffmpeg.org/)** είναι το εργαλείο αναφοράς για την ανάλυση και χειραγώγηση αυτών των μορφών, ικανό να απο-πολυπλέκει και να αναπαράγει περιεχόμενο. Για προγραμματιστές, το **[ffmpy](http://ffmpy.readthedocs.io/en/latest/examples.html)** ενσωματώνει τις δυνατότητες του FFmpeg σε Python για προηγμένες αλληλεπιδράσεις μέσω σεναρίων.

Αυτή η σειρά εργαλείων υπογραμμίζει την ευελιξία που απαιτείται στις προκλήσεις CTF, όπου οι συμμετέχοντες πρέπει να χρησιμοποιήσουν ένα ευρύ φάσμα τεχνικών ανάλυσης και χειραγώγησης για να αποκαλύψουν κρυμμένα δεδομένα μέσα σε αρχεία ήχου και βίντεο.

## References
* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)


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
