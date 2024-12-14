# macOS GCD - Grand Central Dispatch

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

**Grand Central Dispatch (GCD),** επίσης γνωστό ως **libdispatch** (`libdispatch.dyld`), είναι διαθέσιμο τόσο σε macOS όσο και σε iOS. Είναι μια τεχνολογία που αναπτύχθηκε από την Apple για να βελτιστοποιήσει την υποστήριξη εφαρμογών για ταυτόχρονη (multithreaded) εκτέλεση σε υλικό πολλαπλών πυρήνων.

**GCD** παρέχει και διαχειρίζεται **FIFO queues** στις οποίες η εφαρμογή σας μπορεί να **υποβάλει εργασίες** με τη μορφή **block objects**. Τα blocks που υποβάλλονται σε dispatch queues εκτελούνται σε μια πισίνα νημάτων που διαχειρίζεται πλήρως το σύστημα. Το GCD δημιουργεί αυτόματα νήματα για την εκτέλεση των εργασιών στις dispatch queues και προγραμματίζει αυτές τις εργασίες να εκτελούνται στους διαθέσιμους πυρήνες.

{% hint style="success" %}
In summary, to execute code in **parallel**, processes can send **blocks of code to GCD**, which will take care of their execution. Therefore, processes don't create new threads; **GCD executes the given code with its own pool of threads** (which might increase or decrease as necessary).
{% endhint %}

Αυτό είναι πολύ χρήσιμο για τη διαχείριση της ταυτόχρονης εκτέλεσης με επιτυχία, μειώνοντας σημαντικά τον αριθμό των νημάτων που δημιουργούν οι διαδικασίες και βελτιστοποιώντας την ταυτόχρονη εκτέλεση. Αυτό είναι ιδανικό για εργασίες που απαιτούν **μεγάλο παραλληλισμό** (brute-forcing?) ή για εργασίες που δεν θα πρέπει να μπλοκάρουν το κύριο νήμα: Για παράδειγμα, το κύριο νήμα στο iOS χειρίζεται τις αλληλεπιδράσεις UI, οπότε οποιαδήποτε άλλη λειτουργικότητα που θα μπορούσε να κάνει την εφαρμογή να κολλήσει (αναζήτηση, πρόσβαση σε ιστό, ανάγνωση αρχείου...) διαχειρίζεται με αυτόν τον τρόπο.

### Blocks

Ένα block είναι μια **αυτοτελής ενότητα κώδικα** (όπως μια συνάρτηση με παραμέτρους που επιστρέφει μια τιμή) και μπορεί επίσης να καθορίσει δεσμευμένες μεταβλητές.\
Ωστόσο, σε επίπεδο μεταγλωττιστή, τα blocks δεν υπάρχουν, είναι `os_object`s. Κάθε ένα από αυτά τα αντικείμενα σχηματίζεται από δύο δομές:

* **block literal**:&#x20;
* Ξεκινά από το πεδίο **`isa`**, που δείχνει στην κλάση του block:
* `NSConcreteGlobalBlock` (blocks από `__DATA.__const`)
* `NSConcreteMallocBlock` (blocks στο heap)
* `NSConcreateStackBlock` (blocks στο stack)
* Έχει **`flags`** (που υποδεικνύουν τα πεδία που υπάρχουν στον περιγραφέα του block) και μερικά δεσμευμένα bytes
* Ο δείκτης συνάρτησης για κλήση
* Ένας δείκτης στον περιγραφέα του block
* Εισαγόμενες μεταβλητές block (αν υπάρχουν)
* **block descriptor**: Το μέγεθός του εξαρτάται από τα δεδομένα που είναι παρόντα (όπως υποδεικνύεται στα προηγούμενα flags)
* Έχει μερικά δεσμευμένα bytes
* Το μέγεθός του
* Συνήθως θα έχει έναν δείκτη σε μια υπογραφή στυλ Objective-C για να γνωρίζει πόσο χώρο χρειάζεται για τις παραμέτρους (flag `BLOCK_HAS_SIGNATURE`)
* Αν οι μεταβλητές αναφέρονται, αυτό το block θα έχει επίσης δείκτες σε έναν βοηθό αντιγραφής (αντιγράφοντας την τιμή στην αρχή) και σε έναν βοηθό απελευθέρωσης (απελευθερώνοντάς το).

### Queues

Μια dispatch queue είναι ένα ονομαστικό αντικείμενο που παρέχει FIFO σειρά blocks για εκτέλεση.

Τα blocks τοποθετούνται σε queues για εκτέλεση, και αυτές υποστηρίζουν 2 τρόπους: `DISPATCH_QUEUE_SERIAL` και `DISPATCH_QUEUE_CONCURRENT`. Φυσικά, η **σειριακή** δεν θα έχει προβλήματα **race condition** καθώς ένα block δεν θα εκτελείται μέχρι να έχει ολοκληρωθεί το προηγούμενο. Αλλά **ο άλλος τύπος queue μπορεί να έχει**.

Default queues:

* `.main-thread`: Από `dispatch_get_main_queue()`
* `.libdispatch-manager`: Διαχειριστής queue του GCD
* `.root.libdispatch-manager`: Διαχειριστής queue του GCD
* `.root.maintenance-qos`: Εργασίες χαμηλότερης προτεραιότητας
* `.root.maintenance-qos.overcommit`
* `.root.background-qos`: Διαθέσιμο ως `DISPATCH_QUEUE_PRIORITY_BACKGROUND`
* `.root.background-qos.overcommit`
* `.root.utility-qos`: Διαθέσιμο ως `DISPATCH_QUEUE_PRIORITY_NON_INTERACTIVE`
* `.root.utility-qos.overcommit`
* `.root.default-qos`: Διαθέσιμο ως `DISPATCH_QUEUE_PRIORITY_DEFAULT`
* `.root.background-qos.overcommit`
* `.root.user-initiated-qos`: Διαθέσιμο ως `DISPATCH_QUEUE_PRIORITY_HIGH`
* `.root.background-qos.overcommit`
* `.root.user-interactive-qos`: Υψηλότερη προτεραιότητα
* `.root.background-qos.overcommit`

Σημειώστε ότι θα είναι το σύστημα που θα αποφασίσει **ποια νήματα θα χειρίζονται ποιες queues κάθε στιγμή** (πολλαπλά νήματα μπορεί να εργάζονται στην ίδια queue ή το ίδιο νήμα μπορεί να εργάζεται σε διαφορετικές queues σε κάποια στιγμή)

#### Attributtes

Όταν δημιουργείτε μια queue με **`dispatch_queue_create`** το τρίτο επιχείρημα είναι ένα `dispatch_queue_attr_t`, το οποίο συνήθως είναι είτε `DISPATCH_QUEUE_SERIAL` (το οποίο είναι στην πραγματικότητα NULL) είτε `DISPATCH_QUEUE_CONCURRENT` που είναι ένας δείκτης σε μια δομή `dispatch_queue_attr_t` που επιτρέπει τον έλεγχο ορισμένων παραμέτρων της queue.

### Dispatch objects

Υπάρχουν διάφορα αντικείμενα που χρησιμοποιεί το libdispatch και οι queues και blocks είναι μόνο 2 από αυτά. Είναι δυνατή η δημιουργία αυτών των αντικειμένων με `dispatch_object_create`:

* `block`
* `data`: Δεδομένα blocks
* `group`: Ομάδα blocks
* `io`: Async I/O requests
* `mach`: Mach ports
* `mach_msg`: Mach messages
* `pthread_root_queue`: Μια queue με μια πισίνα νημάτων pthread και όχι workqueues
* `queue`
* `semaphore`
* `source`: Πηγή γεγονότων

## Objective-C

Στην Objective-C υπάρχουν διάφορες συναρτήσεις για την αποστολή ενός block για εκτέλεση παράλληλα:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Υποβάλλει ένα block για ασύγχρονη εκτέλεση σε μια dispatch queue και επιστρέφει αμέσως.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Υποβάλλει ένα block object για εκτέλεση και επιστρέφει αφού ολοκληρωθεί η εκτέλεση αυτού του block.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Εκτελεί ένα block object μόνο μία φορά για τη διάρκεια ζωής μιας εφαρμογής.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Υποβάλλει ένα στοιχείο εργασίας για εκτέλεση και επιστρέφει μόνο αφού ολοκληρωθεί η εκτέλεση. Σε αντίθεση με [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), αυτή η συνάρτηση σέβεται όλα τα χαρακτηριστικά της queue όταν εκτελεί το block.

Αυτές οι συναρτήσεις αναμένουν αυτές τις παραμέτρους: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Αυτή είναι η **δομή ενός Block**:
```c
struct Block {
void *isa; // NSConcreteStackBlock,...
int flags;
int reserved;
void *invoke;
struct BlockDescriptor *descriptor;
// captured variables go here
};
```
Και αυτό είναι ένα παράδειγμα χρήσης **παράλληλης εκτέλεσης** με **`dispatch_async`**:
```objectivec
#import <Foundation/Foundation.h>

// Define a block
void (^backgroundTask)(void) = ^{
// Code to be executed in the background
for (int i = 0; i < 10; i++) {
NSLog(@"Background task %d", i);
sleep(1);  // Simulate a long-running task
}
};

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Create a dispatch queue
dispatch_queue_t backgroundQueue = dispatch_queue_create("com.example.backgroundQueue", NULL);

// Submit the block to the queue for asynchronous execution
dispatch_async(backgroundQueue, backgroundTask);

// Continue with other work on the main queue or thread
for (int i = 0; i < 10; i++) {
NSLog(@"Main task %d", i);
sleep(1);  // Simulate a long-running task
}
}
return 0;
}
```
## Swift

**`libswiftDispatch`** είναι μια βιβλιοθήκη που παρέχει **Swift bindings** στο πλαίσιο Grand Central Dispatch (GCD) το οποίο είναι αρχικά γραμμένο σε C.\
Η βιβλιοθήκη **`libswiftDispatch`** περιτυλίγει τα C GCD APIs σε μια πιο φιλική προς το Swift διεπαφή, διευκολύνοντας και καθιστώντας πιο διαισθητική τη δουλειά των προγραμματιστών Swift με το GCD.

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**Code example**:
```swift
import Foundation

// Define a closure (the Swift equivalent of a block)
let backgroundTask: () -> Void = {
for i in 0..<10 {
print("Background task \(i)")
sleep(1)  // Simulate a long-running task
}
}

// Entry point
autoreleasepool {
// Create a dispatch queue
let backgroundQueue = DispatchQueue(label: "com.example.backgroundQueue")

// Submit the closure to the queue for asynchronous execution
backgroundQueue.async(execute: backgroundTask)

// Continue with other work on the main queue
for i in 0..<10 {
print("Main task \(i)")
sleep(1)  // Simulate a long-running task
}
}
```
## Frida

Το παρακάτω σενάριο Frida μπορεί να χρησιμοποιηθεί για να **hook into several `dispatch`** συναρτήσεις και να εξάγει το όνομα της ουράς, το backtrace και το block: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
```bash
frida -U <prog_name> -l libdispatch.js

dispatch_sync
Calling queue: com.apple.UIKit._UIReusePool.reuseSetAccess
Callback function: 0x19e3a6488 UIKitCore!__26-[_UIReusePool addObject:]_block_invoke
Backtrace:
0x19e3a6460 UIKitCore!-[_UIReusePool addObject:]
0x19e3a5db8 UIKitCore!-[UIGraphicsRenderer _enqueueContextForReuse:]
0x19e3a57fc UIKitCore!+[UIGraphicsRenderer _destroyCGContext:withRenderer:]
[...]
```
## Ghidra

Αυτή τη στιγμή, το Ghidra δεν κατανοεί ούτε τη δομή ObjectiveC **`dispatch_block_t`**, ούτε τη **`swift_dispatch_block`**.

Έτσι, αν θέλετε να την κατανοήσει, μπορείτε απλά να **τις δηλώσετε**:

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

Στη συνέχεια, βρείτε ένα μέρος στον κώδικα όπου χρησιμοποιούνται:

{% hint style="success" %}
Σημειώστε όλες τις αναφορές που γίνονται σε "block" για να κατανοήσετε πώς θα μπορούσατε να καταλάβετε ότι η δομή χρησιμοποιείται.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1164).png" alt="" width="563"><figcaption></figcaption></figure>

Κάντε δεξί κλικ στη μεταβλητή -> Επανατύπωση Μεταβλητής και επιλέξτε σε αυτή την περίπτωση **`swift_dispatch_block`**:

<figure><img src="../../.gitbook/assets/image (1165).png" alt="" width="563"><figcaption></figcaption></figure>

Το Ghidra θα ξαναγράψει αυτόματα τα πάντα:

<figure><img src="../../.gitbook/assets/image (1166).png" alt="" width="563"><figcaption></figcaption></figure>

## References

* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

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
