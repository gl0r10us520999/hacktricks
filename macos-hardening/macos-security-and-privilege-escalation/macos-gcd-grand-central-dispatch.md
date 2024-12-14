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

**Grand Central Dispatch (GCD),** pia inajulikana kama **libdispatch** (`libdispatch.dyld`), inapatikana katika macOS na iOS. Ni teknolojia iliyotengenezwa na Apple kuboresha msaada wa programu kwa ajili ya utekelezaji wa sambamba (multithreaded) kwenye vifaa vya multicore.

**GCD** inatoa na kusimamia **FIFO queues** ambazo programu yako inaweza **kuwasilisha kazi** katika mfumo wa **block objects**. Blocks zilizowasilishwa kwa dispatch queues zina **tekelezwa kwenye mchanganyiko wa nyuzi** zinazodhibitiwa kikamilifu na mfumo. GCD kiotomatiki huunda nyuzi za kutekeleza kazi katika dispatch queues na kupanga kazi hizo kufanyika kwenye cores zinazopatikana.

{% hint style="success" %}
Kwa muhtasari, ili kutekeleza msimbo kwa **pamoja**, michakato inaweza kutuma **blocks za msimbo kwa GCD**, ambayo itashughulikia utekelezaji wao. Hivyo, michakato haisababisha nyuzi mpya; **GCD inatekeleza msimbo uliotolewa kwa mchanganyiko wake wa nyuzi** (ambayo inaweza kuongezeka au kupungua kadri inavyohitajika).
{% endhint %}

Hii ni muhimu sana katika kusimamia utekelezaji wa sambamba kwa mafanikio, ikipunguza kwa kiasi kikubwa idadi ya nyuzi ambazo michakato inaunda na kuboresha utekelezaji wa sambamba. Hii ni bora kwa kazi zinazohitaji **paralelism mkubwa** (brute-forcing?) au kwa kazi ambazo hazipaswi kuzuia nyuzi kuu: Kwa mfano, nyuzi kuu kwenye iOS inashughulikia mwingiliano wa UI, hivyo kazi nyingine yoyote ambayo inaweza kufanya programu ikasimama (kutafuta, kufikia wavuti, kusoma faili...) inasimamiwa kwa njia hii.

### Blocks

Block ni **sehemu ya msimbo iliyo na uhuru** (kama kazi yenye hoja inayorejesha thamani) na inaweza pia kubainisha mabadiliko yaliyofungwa.\
Hata hivyo, katika ngazi ya mkusanyiko blocks hazipo, ni `os_object`s. Kila moja ya vitu hivi inaundwa na muundo miwili:

* **block literal**:&#x20;
* Inaanza na **`isa`** uwanja, ikielekeza kwenye darasa la block:
* `NSConcreteGlobalBlock` (blocks kutoka `__DATA.__const`)
* `NSConcreteMallocBlock` (blocks kwenye heap)
* `NSConcreateStackBlock` (blocks kwenye stack)
* Ina **`flags`** (zinazoashiria maeneo yaliyopo katika block descriptor) na baadhi ya bytes zilizohifadhiwa
* Pointer ya kazi ya kuita
* Pointer kwa block descriptor
* Mabadiliko yaliyopatikana ya block (ikiwa yapo)
* **block descriptor**: Ukubwa wake unategemea data iliyopo (kama ilivyoashiriwa katika flags zilizopita)
* Ina baadhi ya bytes zilizohifadhiwa
* Ukubwa wake
* Kwa kawaida itakuwa na pointer kwa saini ya mtindo wa Objective-C ili kujua ni nafasi ngapi inahitajika kwa params (bendera `BLOCK_HAS_SIGNATURE`)
* Ikiwa mabadiliko yanarejelewa, block hii pia itakuwa na pointers kwa msaada wa nakala (kuiga thamani mwanzoni) na msaada wa kutupa (kuachilia).

### Queues

Dispatch queue ni kitu chenye jina kinachotoa mpangilio wa FIFO wa blocks kwa utekelezaji.

Blocks huwekwa katika queues ili kutekelezwa, na hizi zinasaidia njia 2: `DISPATCH_QUEUE_SERIAL` na `DISPATCH_QUEUE_CONCURRENT`. Bila shaka **serial** moja **haitakuwa na matatizo ya hali ya mbio** kwani block haitatekelezwa mpaka ya awali iwe imekamilika. Lakini **aina nyingine ya queue inaweza kuwa nayo**.

Queues za kawaida:

* `.main-thread`: Kutoka `dispatch_get_main_queue()`
* `.libdispatch-manager`: Meneja wa queue wa GCD
* `.root.libdispatch-manager`: Meneja wa queue wa GCD
* `.root.maintenance-qos`: Kazi za kipaumbele cha chini
* `.root.maintenance-qos.overcommit`
* `.root.background-qos`: Inapatikana kama `DISPATCH_QUEUE_PRIORITY_BACKGROUND`
* `.root.background-qos.overcommit`
* `.root.utility-qos`: Inapatikana kama `DISPATCH_QUEUE_PRIORITY_NON_INTERACTIVE`
* `.root.utility-qos.overcommit`
* `.root.default-qos`: Inapatikana kama `DISPATCH_QUEUE_PRIORITY_DEFAULT`
* `.root.background-qos.overcommit`
* `.root.user-initiated-qos`: Inapatikana kama `DISPATCH_QUEUE_PRIORITY_HIGH`
* `.root.background-qos.overcommit`
* `.root.user-interactive-qos`: Kipaumbele cha juu zaidi
* `.root.background-qos.overcommit`

Kumbuka kwamba itakuwa mfumo ambao utaamua **ni nyuzi zipi zinashughulikia queues zipi kila wakati** (nyuzi nyingi zinaweza kufanya kazi katika queue moja au nyuzi moja inaweza kufanya kazi katika queues tofauti kwa wakati fulani)

#### Attributtes

Wakati wa kuunda queue na **`dispatch_queue_create`** hoja ya tatu ni `dispatch_queue_attr_t`, ambayo kwa kawaida ni `DISPATCH_QUEUE_SERIAL` (ambayo kwa kweli ni NULL) au `DISPATCH_QUEUE_CONCURRENT` ambayo ni pointer kwa muundo wa `dispatch_queue_attr_t` ambao unaruhusu kudhibiti baadhi ya vigezo vya queue.

### Dispatch objects

Kuna vitu kadhaa ambavyo libdispatch inatumia na queues na blocks ni 2 tu kati yao. Inawezekana kuunda vitu hivi kwa `dispatch_object_create`:

* `block`
* `data`: Data blocks
* `group`: Kundi la blocks
* `io`: Maombi ya Async I/O
* `mach`: Mach ports
* `mach_msg`: Mach messages
* `pthread_root_queue`: Queue yenye mchanganyiko wa nyuzi za pthread na sio workqueues
* `queue`
* `semaphore`
* `source`: Chanzo cha tukio

## Objective-C

Katika Objetive-C kuna kazi tofauti za kutuma block kutekelezwa kwa sambamba:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Inawasilisha block kwa utekelezaji wa asynchronous kwenye dispatch queue na inarudi mara moja.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Inawasilisha block object kwa utekelezaji na inarudi baada ya block hiyo kumaliza kutekeleza.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Inatekeleza block object mara moja tu kwa muda wa programu.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Inawasilisha kipengele cha kazi kwa utekelezaji na inarudi tu baada ya kumaliza kutekeleza. Tofauti na [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), kazi hii inaheshimu vigezo vyote vya queue wakati inatekeleza block.

Kazi hizi zinatarajia vigezo hivi: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Hii ni **struct ya Block**:
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
Na hii ni mfano wa kutumia **parallelism** na **`dispatch_async`**:
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

**`libswiftDispatch`** ni maktaba inayotoa **Swift bindings** kwa mfumo wa Grand Central Dispatch (GCD) ambao awali umeandikwa kwa C.\
Maktaba ya **`libswiftDispatch`** inafunika API za C GCD katika kiolesura kinachofaa zaidi kwa Swift, na kufanya iwe rahisi na ya kueleweka zaidi kwa waendelezaji wa Swift kufanya kazi na GCD.

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**Mfano wa msimbo**:
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

Script ifuatayo ya Frida inaweza kutumika **kuunganisha kwenye kazi kadhaa za `dispatch`** na kutoa jina la foleni, backtrace na block: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
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

Kwa sasa Ghidra haielewi ama muundo wa ObjectiveC **`dispatch_block_t`**, wala muundo wa **`swift_dispatch_block`**.

Hivyo kama unataka iweze kuelewa, unaweza tu **kuutangaza**:

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

Kisha, pata mahali katika msimbo ambapo zinatumika **kuitwa**:

{% hint style="success" %}
Kumbuka rejea zote zilizofanywa kwa "block" ili kuelewa jinsi unavyoweza kugundua kuwa muundo unatumika.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1164).png" alt="" width="563"><figcaption></figcaption></figure>

Bonyeza kulia kwenye variable -> Re-type Variable na uchague katika kesi hii **`swift_dispatch_block`**:

<figure><img src="../../.gitbook/assets/image (1165).png" alt="" width="563"><figcaption></figcaption></figure>

Ghidra itaandika upya kila kitu kiotomatiki:

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
