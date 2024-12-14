# macOS IOKit

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

Το I/O Kit είναι ένα ανοιχτού κώδικα, αντικειμενοστραφές **framework οδηγών συσκευών** στον πυρήνα XNU, που διαχειρίζεται **δυναμικά φορτωμένους οδηγούς συσκευών**. Επιτρέπει την προσθήκη αρθρωτού κώδικα στον πυρήνα εν κινήσει, υποστηρίζοντας ποικιλία υλικού.

Οι οδηγοί IOKit θα **εξάγουν βασικά συναρτήσεις από τον πυρήνα**. Οι τύποι **παραμέτρων** αυτών των συναρτήσεων είναι **προκαθορισμένοι** και επαληθεύονται. Επιπλέον, παρόμοια με το XPC, το IOKit είναι απλώς ένα άλλο επίπεδο **πάνω από τα μηνύματα Mach**.

Ο **κώδικας IOKit XNU kernel** είναι ανοιχτού κώδικα από την Apple στο [https://github.com/apple-oss-distributions/xnu/tree/main/iokit](https://github.com/apple-oss-distributions/xnu/tree/main/iokit). Επιπλέον, τα στοιχεία IOKit του χώρου χρηστών είναι επίσης ανοιχτού κώδικα [https://github.com/opensource-apple/IOKitUser](https://github.com/opensource-apple/IOKitUser).

Ωστόσο, **κανένας οδηγός IOKit** δεν είναι ανοιχτού κώδικα. Ούτως ή άλλως, από καιρό σε καιρό μια έκδοση ενός οδηγού μπορεί να έρθει με σύμβολα που διευκολύνουν την αποσφαλμάτωσή του. Δείτε πώς να [**πάρετε τις επεκτάσεις οδηγών από το firmware εδώ**](./#ipsw)**.**

Είναι γραμμένο σε **C++**. Μπορείτε να αποκτήσετε αποσυμβολισμένα σύμβολα C++ με:
```bash
# Get demangled symbols
nm -C com.apple.driver.AppleJPEGDriver

# Demangled symbols from stdin
c++filt
__ZN16IOUserClient202222dispatchExternalMethodEjP31IOExternalMethodArgumentsOpaquePK28IOExternalMethodDispatch2022mP8OSObjectPv
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% hint style="danger" %}
Οι **εκτεθειμένες συναρτήσεις** του IOKit θα μπορούσαν να εκτελούν **επιπλέον ελέγχους ασφαλείας** όταν ένας πελάτης προσπαθεί να καλέσει μια συνάρτηση, αλλά σημειώστε ότι οι εφαρμογές είναι συνήθως **περιορισμένες** από το **sandbox** με το οποίο μπορούν να αλληλεπιδρούν οι συναρτήσεις του IOKit.
{% endhint %}

## Οδηγοί

Στο macOS βρίσκονται σε:

* **`/System/Library/Extensions`**
* Αρχεία KEXT ενσωματωμένα στο λειτουργικό σύστημα OS X.
* **`/Library/Extensions`**
* Αρχεία KEXT που εγκαθίστανται από λογισμικό τρίτων

Στο iOS βρίσκονται σε:

* **`/System/Library/Extensions`**
```bash
#Use kextstat to print the loaded drivers
kextstat
Executing: /usr/bin/kmutil showloaded
No variant specified, falling back to release
Index Refs Address            Size       Wired      Name (Version) UUID <Linked Against>
1  142 0                  0          0          com.apple.kpi.bsd (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
2   11 0                  0          0          com.apple.kpi.dsep (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
3  170 0                  0          0          com.apple.kpi.iokit (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
4    0 0                  0          0          com.apple.kpi.kasan (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
5  175 0                  0          0          com.apple.kpi.libkern (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
6  154 0                  0          0          com.apple.kpi.mach (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
7   88 0                  0          0          com.apple.kpi.private (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
8  106 0                  0          0          com.apple.kpi.unsupported (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
9    2 0xffffff8003317000 0xe000     0xe000     com.apple.kec.Libm (1) 6C1342CC-1D74-3D0F-BC43-97D5AD38200A <5>
10   12 0xffffff8003544000 0x92000    0x92000    com.apple.kec.corecrypto (11.1) F5F1255F-6552-3CF4-A9DB-D60EFDEB4A9A <8 7 6 5 3 1>
```
Μέχρι τον αριθμό 9, οι αναφερόμενοι οδηγοί είναι **φορτωμένοι στη διεύθυνση 0**. Αυτό σημαίνει ότι δεν είναι πραγματικοί οδηγοί αλλά **μέρος του πυρήνα και δεν μπορούν να αποφορτωθούν**.

Για να βρείτε συγκεκριμένες επεκτάσεις, μπορείτε να χρησιμοποιήσετε:
```bash
kextfind -bundle-id com.apple.iokit.IOReportFamily #Search by full bundle-id
kextfind -bundle-id -substring IOR #Search by substring in bundle-id
```
Για να φορτώσετε και να ξεφορτώσετε επεκτάσεις πυρήνα, κάντε:
```bash
kextload com.apple.iokit.IOReportFamily
kextunload com.apple.iokit.IOReportFamily
```
## IORegistry

Το **IORegistry** είναι ένα κρίσιμο μέρος του πλαισίου IOKit στο macOS και iOS που λειτουργεί ως βάση δεδομένων για την αναπαράσταση της υλικής διαμόρφωσης και κατάστασης του συστήματος. Είναι μια **ιεραρχική συλλογή αντικειμένων που αναπαριστούν όλο το υλικό και τους οδηγούς** που έχουν φορτωθεί στο σύστημα, και τις σχέσεις τους μεταξύ τους.

Μπορείτε να αποκτήσετε το IORegistry χρησιμοποιώντας το cli **`ioreg`** για να το επιθεωρήσετε από την κονσόλα (ιδιαίτερα χρήσιμο για το iOS).
```bash
ioreg -l #List all
ioreg -w 0 #Not cut lines
ioreg -p <plane> #Check other plane
```
Μπορείτε να κατεβάσετε **`IORegistryExplorer`** από τα **Xcode Additional Tools** από [**https://developer.apple.com/download/all/**](https://developer.apple.com/download/all/) και να επιθεωρήσετε το **macOS IORegistry** μέσω μιας **γραφικής** διεπαφής.

<figure><img src="../../../.gitbook/assets/image (1167).png" alt="" width="563"><figcaption></figcaption></figure>

Στο IORegistryExplorer, οι "επίπεδοι" χρησιμοποιούνται για να οργανώσουν και να εμφανίσουν τις σχέσεις μεταξύ διαφορετικών αντικειμένων στο IORegistry. Κάθε επίπεδο αντιπροσωπεύει έναν συγκεκριμένο τύπο σχέσης ή μια συγκεκριμένη άποψη της υλικού και της διαμόρφωσης οδηγών του συστήματος. Ακολουθούν μερικοί από τους κοινούς επίπεδους που μπορεί να συναντήσετε στο IORegistryExplorer:

1. **IOService Plane**: Αυτό είναι το πιο γενικό επίπεδο, που εμφανίζει τα αντικείμενα υπηρεσιών που αντιπροσωπεύουν οδηγούς και nubs (κανάλια επικοινωνίας μεταξύ οδηγών). Δείχνει τις σχέσεις προμηθευτή-πελάτη μεταξύ αυτών των αντικειμένων.
2. **IODeviceTree Plane**: Αυτό το επίπεδο αντιπροσωπεύει τις φυσικές συνδέσεις μεταξύ συσκευών καθώς συνδέονται στο σύστημα. Χρησιμοποιείται συχνά για να οπτικοποιήσει την ιεραρχία των συσκευών που συνδέονται μέσω λεωφόρων όπως USB ή PCI.
3. **IOPower Plane**: Εμφανίζει αντικείμενα και τις σχέσεις τους σε όρους διαχείρισης ενέργειας. Μπορεί να δείξει ποια αντικείμενα επηρεάζουν την κατάσταση ενέργειας άλλων, χρήσιμο για την αποσφαλμάτωση προβλημάτων που σχετίζονται με την ενέργεια.
4. **IOUSB Plane**: Ειδικά επικεντρωμένο σε συσκευές USB και τις σχέσεις τους, δείχνοντας την ιεραρχία των USB hubs και των συνδεδεμένων συσκευών.
5. **IOAudio Plane**: Αυτό το επίπεδο είναι για την αναπαράσταση συσκευών ήχου και των σχέσεών τους εντός του συστήματος.
6. ...

## Παράδειγμα Κώδικα Comm Driver

Ο παρακάτω κώδικας συνδέεται με την υπηρεσία IOKit `"YourServiceNameHere"` και καλεί τη συνάρτηση μέσα στον επιλεγέα 0. Για αυτό:

* πρώτα καλεί **`IOServiceMatching`** και **`IOServiceGetMatchingServices`** για να αποκτήσει την υπηρεσία.
* Στη συνέχεια, καθορίζει μια σύνδεση καλώντας **`IOServiceOpen`**.
* Και τελικά καλεί μια συνάρτηση με **`IOConnectCallScalarMethod`** υποδεικνύοντας τον επιλεγέα 0 (ο επιλεγέας είναι ο αριθμός που έχει ανατεθεί στη συνάρτηση που θέλετε να καλέσετε).
```objectivec
#import <Foundation/Foundation.h>
#import <IOKit/IOKitLib.h>

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Get a reference to the service using its name
CFMutableDictionaryRef matchingDict = IOServiceMatching("YourServiceNameHere");
if (matchingDict == NULL) {
NSLog(@"Failed to create matching dictionary");
return -1;
}

// Obtain an iterator over all matching services
io_iterator_t iter;
kern_return_t kr = IOServiceGetMatchingServices(kIOMasterPortDefault, matchingDict, &iter);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to get matching services");
return -1;
}

// Get a reference to the first service (assuming it exists)
io_service_t service = IOIteratorNext(iter);
if (!service) {
NSLog(@"No matching service found");
IOObjectRelease(iter);
return -1;
}

// Open a connection to the service
io_connect_t connect;
kr = IOServiceOpen(service, mach_task_self(), 0, &connect);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to open service");
IOObjectRelease(service);
IOObjectRelease(iter);
return -1;
}

// Call a method on the service
// Assume the method has a selector of 0, and takes no arguments
kr = IOConnectCallScalarMethod(connect, 0, NULL, 0, NULL, NULL);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to call method");
}

// Cleanup
IOServiceClose(connect);
IOObjectRelease(service);
IOObjectRelease(iter);
}
return 0;
}
```
There are **άλλες** functions that can be used to call IOKit functions apart of **`IOConnectCallScalarMethod`** like **`IOConnectCallMethod`**, **`IOConnectCallStructMethod`**...

## Reversing driver entrypoint

You could obtain these for example from a [**firmware image (ipsw)**](./#ipsw). Then, load it into your favourite decompiler.

You could start decompiling the **`externalMethod`** function as this is the driver function that will be receiving the call and calling the correct function:

<figure><img src="../../../.gitbook/assets/image (1168).png" alt="" width="315"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1169).png" alt=""><figcaption></figcaption></figure>

That awful call demagled means:

{% code overflow="wrap" %}
```cpp
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

Σημειώστε πώς στην προηγούμενη ορισμό λείπει η παράμετρος **`self`**, η καλή ορισμός θα ήταν:

{% code overflow="wrap" %}
```cpp
IOUserClient2022::dispatchExternalMethod(self, unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

Στην πραγματικότητα, μπορείτε να βρείτε τον πραγματικό ορισμό στο [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388):
```cpp
IOUserClient2022::dispatchExternalMethod(uint32_t selector, IOExternalMethodArgumentsOpaque *arguments,
const IOExternalMethodDispatch2022 dispatchArray[], size_t dispatchArrayCount,
OSObject * target, void * reference)
```
Με αυτές τις πληροφορίες μπορείτε να ξαναγράψετε Ctrl+Δεξί -> `Edit function signature` και να ορίσετε τους γνωστούς τύπους:

<figure><img src="../../../.gitbook/assets/image (1174).png" alt=""><figcaption></figcaption></figure>

Ο νέος αποσυμπιεσμένος κώδικας θα φαίνεται ως εξής:

<figure><img src="../../../.gitbook/assets/image (1175).png" alt=""><figcaption></figcaption></figure>

Για το επόμενο βήμα πρέπει να έχουμε ορίσει τη δομή **`IOExternalMethodDispatch2022`**. Είναι ανοιχτού κώδικα στο [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176), μπορείτε να το ορίσετε:

<figure><img src="../../../.gitbook/assets/image (1170).png" alt=""><figcaption></figcaption></figure>

Τώρα, ακολουθώντας το `(IOExternalMethodDispatch2022 *)&sIOExternalMethodArray` μπορείτε να δείτε πολλά δεδομένα:

<figure><img src="../../../.gitbook/assets/image (1176).png" alt="" width="563"><figcaption></figcaption></figure>

Αλλάξτε τον Τύπο Δεδομένων σε **`IOExternalMethodDispatch2022:`**

<figure><img src="../../../.gitbook/assets/image (1177).png" alt="" width="375"><figcaption></figcaption></figure>

μετά την αλλαγή:

<figure><img src="../../../.gitbook/assets/image (1179).png" alt="" width="563"><figcaption></figcaption></figure>

Και όπως γνωρίζουμε εκεί έχουμε ένα **πίνακα 7 στοιχείων** (ελέγξτε τον τελικό αποσυμπιεσμένο κώδικα), κάντε κλικ για να δημιουργήσετε έναν πίνακα 7 στοιχείων:

<figure><img src="../../../.gitbook/assets/image (1180).png" alt="" width="563"><figcaption></figcaption></figure>

Αφού δημιουργηθεί ο πίνακας, μπορείτε να δείτε όλες τις εξαγόμενες συναρτήσεις:

<figure><img src="../../../.gitbook/assets/image (1181).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Αν θυμάστε, για να **καλέσουμε** μια **εξαγόμενη** συνάρτηση από τον χώρο χρήστη δεν χρειάζεται να καλέσουμε το όνομα της συνάρτησης, αλλά τον **αριθμό επιλεγέα**. Εδώ μπορείτε να δείτε ότι ο επιλεγέας **0** είναι η συνάρτηση **`initializeDecoder`**, ο επιλεγέας **1** είναι **`startDecoder`**, ο επιλεγέας **2** **`initializeEncoder`**...
{% endhint %}

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξη HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
