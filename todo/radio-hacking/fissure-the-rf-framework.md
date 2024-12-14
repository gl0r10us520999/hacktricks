# FISSURE - The RF Framework

**Ανεξάρτητη Συχνότητα SDR-βασισμένη Κατανόηση και Αντίστροφη Μηχανική Σημάτων**

Το FISSURE είναι ένα ανοιχτού κώδικα πλαίσιο RF και αντίστροφης μηχανικής σχεδιασμένο για όλα τα επίπεδα δεξιοτήτων με hooks για ανίχνευση και ταξινόμηση σημάτων, ανακάλυψη πρωτοκόλλων, εκτέλεση επιθέσεων, χειρισμό IQ, ανάλυση ευπαθειών, αυτοματοποίηση και AI/ML. Το πλαίσιο έχει κατασκευαστεί για να προάγει την ταχεία ενσωμάτωση λογισμικών μονάδων, ραδιοφώνων, πρωτοκόλλων, δεδομένων σημάτων, scripts, ροών γραφημάτων, υλικού αναφοράς και εργαλείων τρίτων. Το FISSURE είναι ένας επιταχυντής ροής εργασίας που διατηρεί το λογισμικό σε μία τοποθεσία και επιτρέπει στις ομάδες να προσαρμόζονται εύκολα ενώ μοιράζονται την ίδια αποδεδειγμένη βασική διαμόρφωση για συγκεκριμένες διανομές Linux.

Το πλαίσιο και τα εργαλεία που περιλαμβάνονται στο FISSURE έχουν σχεδιαστεί για να ανιχνεύουν την παρουσία RF ενέργειας, να κατανοούν τα χαρακτηριστικά ενός σήματος, να συλλέγουν και να αναλύουν δείγματα, να αναπτύσσουν τεχνικές μετάδοσης και/ή ένεσης, και να δημιουργούν προσαρμοσμένα payloads ή μηνύματα. Το FISSURE περιέχει μια αυξανόμενη βιβλιοθήκη πληροφοριών πρωτοκόλλων και σημάτων για να βοηθήσει στην αναγνώριση, τη δημιουργία πακέτων και το fuzzing. Υπάρχουν δυνατότητες online αρχείου για τη λήψη αρχείων σημάτων και τη δημιουργία playlists για την προσομοίωση κυκλοφορίας και τη δοκιμή συστημάτων.

Η φιλική βάση κώδικα Python και η διεπαφή χρήστη επιτρέπουν στους αρχάριους να μάθουν γρήγορα για δημοφιλή εργαλεία και τεχνικές που σχετίζονται με RF και αντίστροφη μηχανική. Οι εκπαιδευτές στον τομέα της κυβερνοασφάλειας και της μηχανικής μπορούν να εκμεταλλευτούν το ενσωματωμένο υλικό ή να χρησιμοποιήσουν το πλαίσιο για να επιδείξουν τις δικές τους εφαρμογές στον πραγματικό κόσμο. Οι προγραμματιστές και οι ερευνητές μπορούν να χρησιμοποιήσουν το FISSURE για τις καθημερινές τους εργασίες ή για να εκθέσουν τις πρωτοποριακές λύσεις τους σε ένα ευρύτερο κοινό. Καθώς η ευαισθητοποίηση και η χρήση του FISSURE αυξάνονται στην κοινότητα, θα αυξάνεται και η έκταση των δυνατοτήτων του και η ποικιλία της τεχνολογίας που περιλαμβάνει.

**Επιπλέον Πληροφορίες**

* [AIS Page](https://www.ainfosec.com/technologies/fissure/)
* [GRCon22 Slides](https://events.gnuradio.org/event/18/contributions/246/attachments/84/164/FISSURE\_Poore\_GRCon22.pdf)
* [GRCon22 Paper](https://events.gnuradio.org/event/18/contributions/246/attachments/84/167/FISSURE\_Paper\_Poore\_GRCon22.pdf)
* [GRCon22 Video](https://www.youtube.com/watch?v=1f2umEKhJvE)
* [Hack Chat Transcript](https://hackaday.io/event/187076-rf-hacking-hack-chat/log/212136-hack-chat-transcript-part-1)

## Getting Started

**Supported**

Υπάρχουν τρεις κλάδοι μέσα στο FISSURE για να διευκολύνουν την πλοήγηση στα αρχεία και να μειώσουν την επαναληψιμότητα του κώδικα. Ο κλάδος Python2\_maint-3.7 περιέχει μια βάση κώδικα που έχει κατασκευαστεί γύρω από Python2, PyQt4 και GNU Radio 3.7; ο κλάδος Python3\_maint-3.8 έχει κατασκευαστεί γύρω από Python3, PyQt5 και GNU Radio 3.8; και ο κλάδος Python3\_maint-3.10 έχει κατασκευαστεί γύρω από Python3, PyQt5 και GNU Radio 3.10.

|   Operating System   |   FISSURE Branch   |
| :------------------: | :----------------: |
|  Ubuntu 18.04 (x64)  | Python2\_maint-3.7 |
| Ubuntu 18.04.5 (x64) | Python2\_maint-3.7 |
| Ubuntu 18.04.6 (x64) | Python2\_maint-3.7 |
| Ubuntu 20.04.1 (x64) | Python3\_maint-3.8 |
| Ubuntu 20.04.4 (x64) | Python3\_maint-3.8 |
|  KDE neon 5.25 (x64) | Python3\_maint-3.8 |

**In-Progress (beta)**

Αυτά τα λειτουργικά συστήματα είναι ακόμα σε κατάσταση beta. Είναι υπό ανάπτυξη και αρκετές δυνατότητες είναι γνωστό ότι λείπουν. Αντικείμενα στον εγκαταστάτη μπορεί να συγκρούονται με υπάρχοντα προγράμματα ή να αποτύχουν να εγκατασταθούν μέχρι να αφαιρεθεί η κατάσταση.

|     Operating System     |    FISSURE Branch   |
| :----------------------: | :-----------------: |
| DragonOS Focal (x86\_64) |  Python3\_maint-3.8 |
|    Ubuntu 22.04 (x64)    | Python3\_maint-3.10 |

Σημείωση: Ορισμένα εργαλεία λογισμικού δεν λειτουργούν για κάθε OS. Ανατρέξτε στο [Software And Conflicts](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Help/Markdown/SoftwareAndConflicts.md)

**Installation**
```
git clone https://github.com/ainfosec/FISSURE.git
cd FISSURE
git checkout <Python2_maint-3.7> or <Python3_maint-3.8> or <Python3_maint-3.10>
git submodule update --init
./install
```
Αυτό θα εγκαταστήσει τις εξαρτήσεις λογισμικού PyQt που απαιτούνται για να ξεκινήσει η εγκατάσταση GUIs αν δεν βρεθούν.

Στη συνέχεια, επιλέξτε την επιλογή που ταιριάζει καλύτερα στο λειτουργικό σας σύστημα (θα ανιχνευθεί αυτόματα αν το λειτουργικό σας σύστημα ταιριάζει με μια επιλογή).

|                                          Python2\_maint-3.7                                          |                                          Python3\_maint-3.8                                          |                                          Python3\_maint-3.10                                         |
| :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| ![install1b](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1b.png) | ![install1a](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1a.png) | ![install1c](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1c.png) |

Συνιστάται να εγκαταστήσετε το FISSURE σε ένα καθαρό λειτουργικό σύστημα για να αποφύγετε υπάρχουσες συγκρούσεις. Επιλέξτε όλα τα προτεινόμενα πλαίσια ελέγχου (κουμπί προεπιλογής) για να αποφύγετε σφάλματα κατά τη λειτουργία των διαφόρων εργαλείων εντός του FISSURE. Θα υπάρχουν πολλαπλές προτροπές κατά τη διάρκεια της εγκατάστασης, κυρίως ζητώντας ανυψωμένα δικαιώματα και ονόματα χρηστών. Αν ένα στοιχείο περιέχει μια ενότητα "Επαλήθευση" στο τέλος, ο εγκαταστάτης θα εκτελέσει την εντολή που ακολουθεί και θα επισημάνει το στοιχείο του πλαισίου ελέγχου πράσινο ή κόκκινο ανάλογα με το αν παραχθούν σφάλματα από την εντολή. Τα ελεγμένα στοιχεία χωρίς ενότητα "Επαλήθευση" θα παραμείνουν μαύρα μετά την εγκατάσταση.

![install2](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install2.png)

**Χρήση**

Ανοίξτε ένα τερματικό και εισάγετε:
```
fissure
```
Refer to the FISSURE Help menu for more details on usage.

## Details

**Components**

* Dashboard
* Central Hub (HIPRFISR)
* Target Signal Identification (TSI)
* Protocol Discovery (PD)
* Flow Graph & Script Executor (FGE)

![components](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/components.png)

**Capabilities**

| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/detector.png)_**Ανιχνευτής Σημάτων**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/iq.png)_**Manipulation IQ**_      | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/library.png)_**Αναζήτηση Σημάτων**_          | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/pd.png)_**Αναγνώριση Προτύπων**_ |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/attack.png)_**Επιθέσεις**_           | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/fuzzing.png)_**Fuzzing**_         | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/archive.png)_**Λίστες Σημάτων**_       | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/gallery.png)_**Γκαλερί Εικόνας**_  |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/packet.png)_**Δημιουργία Πακέτων**_   | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/scapy.png)_**Ενσωμάτωση Scapy**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/crc\_calculator.png)_**Υπολογιστής CRC**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/log.png)_**Καταγραφή**_            |

**Hardware**

The following is a list of "supported" hardware with varying levels of integration:

* USRP: X3xx, B2xx, B20xmini, USRP2, N2xx
* HackRF
* RTL2832U
* 802.11 Adapters
* LimeSDR
* bladeRF, bladeRF 2.0 micro
* Open Sniffer
* PlutoSDR

## Lessons

FISSURE comes with several helpful guides to become familiar with different technologies and techniques. Many include steps for using various tools that are integrated into FISSURE.

* [Lesson1: OpenBTS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson1\_OpenBTS.md)
* [Lesson2: Lua Dissectors](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson2\_LuaDissectors.md)
* [Lesson3: Sound eXchange](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson3\_Sound\_eXchange.md)
* [Lesson4: ESP Boards](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson4\_ESP\_Boards.md)
* [Lesson5: Radiosonde Tracking](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson5\_Radiosonde\_Tracking.md)
* [Lesson6: RFID](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson6\_RFID.md)
* [Lesson7: Data Types](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson7\_Data\_Types.md)
* [Lesson8: Custom GNU Radio Blocks](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson8\_Custom\_GNU\_Radio\_Blocks.md)
* [Lesson9: TPMS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson9\_TPMS.md)
* [Lesson10: Ham Radio Exams](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson10\_Ham\_Radio\_Exams.md)
* [Lesson11: Wi-Fi Tools](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson11\_WiFi\_Tools.md)

## Roadmap

* [ ] Add more hardware types, RF protocols, signal parameters, analysis tools
* [ ] Support more operating systems
* [ ] Develop class material around FISSURE (RF Attacks, Wi-Fi, GNU Radio, PyQt, etc.)
* [ ] Create a signal conditioner, feature extractor, and signal classifier with selectable AI/ML techniques
* [ ] Implement recursive demodulation mechanisms for producing a bitstream from unknown signals
* [ ] Transition the main FISSURE components to a generic sensor node deployment scheme

## Contributing

Suggestions for improving FISSURE are strongly encouraged. Leave a comment in the [Discussions](https://github.com/ainfosec/FISSURE/discussions) page or in the Discord Server if you have any thoughts regarding the following:

* New feature suggestions and design changes
* Software tools with installation steps
* New lessons or additional material for existing lessons
* RF protocols of interest
* More hardware and SDR types for integration
* IQ analysis scripts in Python
* Installation corrections and improvements

Contributions to improve FISSURE are crucial to expediting its development. Any contributions you make are greatly appreciated. If you wish to contribute through code development, please fork the repo and create a pull request:

1. Fork the project
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a pull request

Creating [Issues](https://github.com/ainfosec/FISSURE/issues) to bring attention to bugs is also welcomed.

## Collaborating

Contact Assured Information Security, Inc. (AIS) Business Development to propose and formalize any FISSURE collaboration opportunities–whether that is through dedicating time towards integrating your software, having the talented people at AIS develop solutions for your technical challenges, or integrating FISSURE into other platforms/applications.

## License

GPL-3.0

For license details, see LICENSE file.

## Contact

Join the Discord Server: [https://discord.gg/JZDs5sgxcG](https://discord.gg/JZDs5sgxcG)

Follow on Twitter: [@FissureRF](https://twitter.com/fissurerf), [@AinfoSec](https://twitter.com/ainfosec)

Chris Poore - Assured Information Security, Inc. - poorec@ainfosec.com

Business Development - Assured Information Security, Inc. - bd@ainfosec.com

## Credits

We acknowledge and are grateful to these developers:

[Credits](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/CREDITS.md)

## Acknowledgments

Special thanks to Dr. Samuel Mantravadi and Joseph Reith for their contributions to this project.
