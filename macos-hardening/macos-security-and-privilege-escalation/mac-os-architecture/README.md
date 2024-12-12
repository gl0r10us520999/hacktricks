# macOS Kernel & System Extensions

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## XNU Kernel

**Ядро macOS - це XNU**, що означає "X is Not Unix". Це ядро в основному складається з **Mach мікроядра** (про яке буде сказано пізніше) та елементів з Berkeley Software Distribution (**BSD**). XNU також забезпечує платформу для **драйверів ядра через систему, відому як I/O Kit**. Ядро XNU є частиною проекту з відкритим вихідним кодом Darwin, що означає, що **його вихідний код є вільно доступним**.

З точки зору дослідника безпеки або розробника Unix, **macOS** може здаватися досить **схожим** на систему **FreeBSD** з елегантним графічним інтерфейсом та безліччю спеціальних додатків. Більшість додатків, розроблених для BSD, будуть компілюватися та працювати на macOS без необхідності модифікацій, оскільки командні інструменти, знайомі користувачам Unix, присутні в macOS. Однак, оскільки ядро XNU включає Mach, існують деякі суттєві відмінності між традиційною системою, схожою на Unix, та macOS, і ці відмінності можуть викликати потенційні проблеми або надавати унікальні переваги.

Відкрита версія XNU: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach - це **мікроядро**, розроблене для того, щоб бути **сумісним з UNIX**. Одним з його ключових принципів дизайну було **мінімізувати** кількість **коду**, що виконується в **ядровому** просторі, і натомість дозволити багатьом типовим функціям ядра, таким як файлові системи, мережі та I/O, **виконуватися як завдання на рівні користувача**.

У XNU Mach **відповідає за багато критично важливих низькорівневих операцій**, які зазвичай обробляє ядро, таких як планування процесора, багатозадачність та управління віртуальною пам'яттю.

### BSD

Ядро XNU також **включає** значну кількість коду, отриманого з проекту **FreeBSD**. Цей код **виконується як частина ядра разом з Mach**, в одному адресному просторі. Однак код FreeBSD в XNU може суттєво відрізнятися від оригінального коду FreeBSD, оскільки були потрібні модифікації для забезпечення його сумісності з Mach. FreeBSD сприяє багатьом операціям ядра, включаючи:

* Управління процесами
* Обробка сигналів
* Основні механізми безпеки, включаючи управління користувачами та групами
* Інфраструктура системних викликів
* Стек TCP/IP та сокети
* Брандмауер та фільтрація пакетів

Розуміння взаємодії між BSD та Mach може бути складним через їх різні концептуальні рамки. Наприклад, BSD використовує процеси як свою основну одиницю виконання, тоді як Mach працює на основі потоків. Ця розбіжність узгоджується в XNU шляхом **асоціювання кожного процесу BSD з завданням Mach**, яке містить точно один потік Mach. Коли використовується системний виклик fork() BSD, код BSD в ядрі використовує функції Mach для створення структури завдання та потоку.

Більше того, **Mach та BSD кожен підтримує різні моделі безпеки**: **модель безпеки Mach** базується на **правах портів**, тоді як модель безпеки BSD працює на основі **власності процесу**. Відмінності між цими двома моделями іноді призводили до вразливостей підвищення локальних привілеїв. Окрім типових системних викликів, також існують **Mach traps, які дозволяють програмам користувацького простору взаємодіяти з ядром**. Ці різні елементи разом формують багатогранну, гібридну архітектуру ядра macOS.

### I/O Kit - Драйвери

I/O Kit - це відкритий, об'єктно-орієнтований **фреймворк драйверів пристроїв** в ядрі XNU, який обробляє **динамічно завантажувані драйвери пристроїв**. Він дозволяє модульному коду додаватися до ядра на льоту, підтримуючи різноманітне апаратне забезпечення.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - Міжпроцесна комунікація

{% content-ref url="../macos-proces-abuse/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../macos-proces-abuse/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

## macOS Kernel Extensions

macOS є **надзвичайно обмеженим для завантаження розширень ядра** (.kext) через високі привілеї, з якими буде виконуватися код. Насправді, за замовчуванням це практично неможливо (якщо не знайдено обхід).

На наступній сторінці ви також можете побачити, як відновити `.kext`, які macOS завантажує всередині свого **kernelcache**:

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOS System Extensions

Замість використання розширень ядра macOS створив системні розширення, які пропонують API на рівні користувача для взаємодії з ядром. Таким чином, розробники можуть уникнути використання розширень ядра.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## References

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
