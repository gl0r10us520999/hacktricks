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


# Έλεγχος BSSIDs

Όταν λάβετε μια καταγραφή της οποίας η κύρια κίνηση είναι Wifi χρησιμοποιώντας το WireShark, μπορείτε να αρχίσετε να ερευνάτε όλα τα SSIDs της καταγραφής με _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## Brute Force

Μία από τις στήλες της οθόνης αυτής υποδεικνύει αν **βρέθηκε οποιαδήποτε αυθεντικοποίηση μέσα στο pcap**. Αν αυτό ισχύει, μπορείτε να προσπαθήσετε να το σπάσετε χρησιμοποιώντας το `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
Για παράδειγμα, θα ανακτήσει το WPA passphrase που προστατεύει ένα PSK (προκαθορισμένο κλειδί), το οποίο θα απαιτηθεί για να αποκρυπτογραφήσετε την κίνηση αργότερα.

# Δεδομένα σε Beacons / Παράπλευρη Διάσταση

Εάν υποψιάζεστε ότι **δεδομένα διαρρέουν μέσα σε beacons ενός δικτύου Wifi**, μπορείτε να ελέγξετε τα beacons του δικτύου χρησιμοποιώντας ένα φίλτρο όπως το εξής: `wlan contains <NAMEofNETWORK>`, ή `wlan.ssid == "NAMEofNETWORK"` αναζητώντας μέσα στα φιλτραρισμένα πακέτα για ύποπτες αλυσίδες.

# Βρείτε Άγνωστες Διευθύνσεις MAC σε Ένα Δίκτυο Wifi

Ο παρακάτω σύνδεσμος θα είναι χρήσιμος για να βρείτε τις **μηχανές που στέλνουν δεδομένα μέσα σε ένα Δίκτυο Wifi**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Εάν ήδη γνωρίζετε **διευθύνσεις MAC μπορείτε να τις αφαιρέσετε από την έξοδο** προσθέτοντας ελέγχους όπως αυτόν: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Μόλις εντοπίσετε **άγνωστες διευθύνσεις MAC** που επικοινωνούν μέσα στο δίκτυο, μπορείτε να χρησιμοποιήσετε **φίλτρα** όπως το εξής: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)` για να φιλτράρετε την κίνησή τους. Σημειώστε ότι τα φίλτρα ftp/http/ssh/telnet είναι χρήσιμα εάν έχετε αποκρυπτογραφήσει την κίνηση.

# Αποκρυπτογράφηση Κίνησης

Edit --> Preferences --> Protocols --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (426).png>)





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
