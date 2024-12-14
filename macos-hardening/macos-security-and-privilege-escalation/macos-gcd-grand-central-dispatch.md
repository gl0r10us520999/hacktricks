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

**Grand Central Dispatch (GCD),** також відомий як **libdispatch** (`libdispatch.dyld`), доступний як в macOS, так і в iOS. Це технологія, розроблена Apple для оптимізації підтримки додатків для паралельного (мультитредового) виконання на багатоядерному апаратному забезпеченні.

**GCD** надає та керує **FIFO чергами**, до яких ваш додаток може **подавати завдання** у формі **блок-об'єктів**. Блоки, подані до черг, **виконуються на пулі потоків**, повністю керованих системою. GCD автоматично створює потоки для виконання завдань у чергах і планує ці завдання для виконання на доступних ядрах.

{% hint style="success" %}
У підсумку, для виконання коду в **паралельному** режимі процеси можуть надсилати **блоки коду до GCD**, який подбає про їх виконання. Тому процеси не створюють нові потоки; **GCD виконує даний код зі своїм власним пулом потоків** (який може збільшуватися або зменшуватися за необхідності).
{% endhint %}

Це дуже корисно для успішного управління паралельним виконанням, значно зменшуючи кількість потоків, які створюють процеси, і оптимізуючи паралельне виконання. Це ідеально підходить для завдань, які вимагають **великого паралелізму** (брутфорс?) або для завдань, які не повинні блокувати основний потік: наприклад, основний потік на iOS обробляє взаємодії з UI, тому будь-яка інша функціональність, яка може призвести до зависання програми (пошук, доступ до вебу, читання файлу...) управляється таким чином.

### Blocks

Блок — це **самостійна секція коду** (як функція з аргументами, що повертає значення) і може також вказувати зв'язані змінні.\
Однак на рівні компілятора блоки не існують, вони є `os_object`s. Кожен з цих об'єктів складається з двох структур:

* **блок-літерал**:&#x20;
* Він починається з поля **`isa`**, що вказує на клас блоку:
* `NSConcreteGlobalBlock` (блоки з `__DATA.__const`)
* `NSConcreteMallocBlock` (блоки в купі)
* `NSConcreateStackBlock` (блоки в стеку)
* Має **`flags`** (які вказують на поля, присутні в дескрипторі блоку) і кілька зарезервованих байтів
* Вказівник на функцію для виклику
* Вказівник на дескриптор блоку
* Імпортовані змінні блоку (якщо є)
* **дескриптор блоку**: Його розмір залежить від даних, які присутні (як вказано в попередніх флагах)
* Має кілька зарезервованих байтів
* Розмір його
* Зазвичай матиме вказівник на підпис у стилі Objective-C, щоб знати, скільки місця потрібно для параметрів (флаг `BLOCK_HAS_SIGNATURE`)
* Якщо змінні посилаються, цей блок також матиме вказівники на допоміжний засіб копіювання (копіюючи значення на початку) і допоміжний засіб звільнення (вивільняючи його).

### Queues

Черга розподілу — це іменований об'єкт, що забезпечує FIFO порядок блоків для виконання.

Блоки встановлюються в черги для виконання, і ці черги підтримують 2 режими: `DISPATCH_QUEUE_SERIAL` і `DISPATCH_QUEUE_CONCURRENT`. Звичайно, **послідовна** черга **не матиме проблем з гонками**, оскільки блок не буде виконуватись, поки попередній не закінчиться. Але **інший тип черги може мати їх**.

Черги за замовчуванням:

* `.main-thread`: З `dispatch_get_main_queue()`
* `.libdispatch-manager`: Менеджер черг GCD
* `.root.libdispatch-manager`: Менеджер черг GCD
* `.root.maintenance-qos`: Завдання з найнижчим пріоритетом
* `.root.maintenance-qos.overcommit`
* `.root.background-qos`: Доступно як `DISPATCH_QUEUE_PRIORITY_BACKGROUND`
* `.root.background-qos.overcommit`
* `.root.utility-qos`: Доступно як `DISPATCH_QUEUE_PRIORITY_NON_INTERACTIVE`
* `.root.utility-qos.overcommit`
* `.root.default-qos`: Доступно як `DISPATCH_QUEUE_PRIORITY_DEFAULT`
* `.root.background-qos.overcommit`
* `.root.user-initiated-qos`: Доступно як `DISPATCH_QUEUE_PRIORITY_HIGH`
* `.root.background-qos.overcommit`
* `.root.user-interactive-qos`: Найвищий пріоритет
* `.root.background-qos.overcommit`

Зверніть увагу, що саме система вирішує, **які потоки обробляють які черги в кожен момент часу** (кілька потоків можуть працювати в одній черзі або один і той же потік може працювати в різних чергах в певний момент)

#### Attributtes

При створенні черги з **`dispatch_queue_create`** третій аргумент є `dispatch_queue_attr_t`, який зазвичай є або `DISPATCH_QUEUE_SERIAL` (що насправді є NULL), або `DISPATCH_QUEUE_CONCURRENT`, що є вказівником на структуру `dispatch_queue_attr_t`, яка дозволяє контролювати деякі параметри черги.

### Dispatch objects

Існує кілька об'єктів, які використовує libdispatch, і черги та блоки — це лише 2 з них. Можна створити ці об'єкти за допомогою `dispatch_object_create`:

* `block`
* `data`: Блоки даних
* `group`: Група блоків
* `io`: Асинхронні запити I/O
* `mach`: Порти Mach
* `mach_msg`: Повідомлення Mach
* `pthread_root_queue`: Черга з пулом потоків pthread і не робочими чергами
* `queue`
* `semaphore`
* `source`: Джерело подій

## Objective-C

В Objective-C є різні функції для надсилання блоку для виконання в паралельному режимі:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Подає блок для асинхронного виконання в черзі розподілу та повертає негайно.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Подає об'єкт блоку для виконання та повертає після завершення виконання цього блоку.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Виконує об'єкт блоку лише один раз протягом життєвого циклу програми.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Подає робочий елемент для виконання та повертає лише після його завершення. На відміну від [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), ця функція поважає всі атрибути черги під час виконання блоку.

Ці функції очікують такі параметри: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Це **структура блоку**:
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
І це приклад використання **паралелізму** з **`dispatch_async`**:
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

**`libswiftDispatch`** - це бібліотека, яка надає **Swift прив'язки** до фреймворку Grand Central Dispatch (GCD), який спочатку написаний на C.\
Бібліотека **`libswiftDispatch`** обгортає C GCD API в більш дружній до Swift інтерфейс, що робить роботу з GCD легшою та інтуїтивно зрозумілішою для розробників Swift.

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

Наступний скрипт Frida можна використовувати для **перехоплення кількох `dispatch`** функцій та витягнення назви черги, зворотного сліду та блоку: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
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

Наразі Ghidra не розуміє ні структуру ObjectiveC **`dispatch_block_t`**, ні **`swift_dispatch_block`**.

Отже, якщо ви хочете, щоб вона їх розуміла, ви можете просто **оголосити їх**:

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

Потім знайдіть місце в коді, де вони **використовуються**:

{% hint style="success" %}
Зверніть увагу на всі посилання на "block", щоб зрозуміти, як ви могли б зрозуміти, що структура використовується.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1164).png" alt="" width="563"><figcaption></figcaption></figure>

Клацніть правою кнопкою миші на змінній -> Змінити тип змінної і виберіть у цьому випадку **`swift_dispatch_block`**:

<figure><img src="../../.gitbook/assets/image (1165).png" alt="" width="563"><figcaption></figcaption></figure>

Ghidra автоматично перепише все:

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
