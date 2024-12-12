# Системні розширення macOS

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа взлому AWS для Червоної Команди (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа взлому GCP для Червоної Команди (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Системні розширення / Фреймворк безпеки кінцевої точки

На відміну від Ядерних розширень, **Системні розширення працюють в просторі користувача** замість простору ядра, що зменшує ризик аварії системи через несправність розширення.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Існує три типи системних розширень: Розширення **DriverKit**, Розширення **Network** та Розширення **Безпеки кінцевої точки**.

### **Розширення DriverKit**

DriverKit є заміною для ядерних розширень, які **забезпечують підтримку обладнання**. Воно дозволяє драйверам пристроїв (таким як USB, Serial, NIC та HID драйвери) працювати в просторі користувача, а не в просторі ядра. Фреймворк DriverKit включає **версії простору користувача певних класів I/O Kit**, і ядро пересилає звичайні події I/O Kit в простір користувача, пропонуючи безпечне середовище для роботи цих драйверів.

### **Розширення Network**

Розширення Network надають можливість налаштування мережевих поведінок. Існує кілька типів Розширень Network:

* **App Proxy**: Використовується для створення клієнта VPN, який реалізує протокол VPN з орієнтацією на потік. Це означає, що він обробляє мережевий трафік на основі з'єднань (або потоків), а не окремих пакетів.
* **Packet Tunnel**: Використовується для створення клієнта VPN, який реалізує протокол VPN з орієнтацією на пакет. Це означає, що він обробляє мережевий трафік на основі окремих пакетів.
* **Filter Data**: Використовується для фільтрації мережевих "потоків". Він може відстежувати або змінювати мережеві дані на рівні потоку.
* **Filter Packet**: Використовується для фільтрації окремих мережевих пакетів. Він може відстежувати або змінювати мережеві дані на рівні пакету.
* **DNS Proxy**: Використовується для створення власного постачальника DNS. Він може використовуватися для відстеження або зміни запитів та відповідей DNS.

## Фреймворк безпеки кінцевої точки

Фреймворк безпеки кінцевої точки - це фреймворк, наданий Apple в macOS, який надає набір API для системної безпеки. Він призначений для використання **вендорами безпеки та розробниками для створення продуктів, які можуть відстежувати та контролювати діяльність системи** для виявлення та захисту від зловмисної діяльності.

Цей фреймворк надає **колекцію API для відстеження та контролю діяльності системи**, таких як виконання процесів, події файлової системи, мережеві та ядерні події.

Основа цього фреймворку реалізована в ядрі, як Ядерне розширення (KEXT), розташоване за адресою **`/System/Library/Extensions/EndpointSecurity.kext`**. Це KEXT складається з кількох ключових компонентів:

* **EndpointSecurityDriver**: Діє як "точка входу" для ядерного розширення. Це основна точка взаємодії між ОС та фреймворком безпеки кінцевої точки.
* **EndpointSecurityEventManager**: Цей компонент відповідає за реалізацію ядерних гуків. Ядерні гуки дозволяють фреймворку відстежувати системні події, перехоплюючи системні виклики.
* **EndpointSecurityClientManager**: Цей компонент відповідає за взаємодію з клієнтами простору користувача, відстежує, які клієнти підключені та потребують отримання сповіщень про події.
* **EndpointSecurityMessageManager**: Цей компонент надсилає повідомлення та сповіщення про події клієнтам простору користувача.

Події, які може відстежувати фреймворк безпеки кінцевої точки, поділяються на:

* Події файлів
* Події процесів
* Події сокетів
* Ядерні події (такі як завантаження/вивантаження ядерного розширення або відкриття пристрою I/O Kit)

### Архітектура фреймворку безпеки кінцевої точки

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Взаємодія з простором користувача** фреймворку безпеки кінцевої точки відбувається через клас IOUserClient. Використовуються два різні підкласи, залежно від типу викликача:

* **EndpointSecurityDriverClient**: Це вимагає дозволу `com.apple.private.endpoint-security.manager`, який має лише процес системи `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Це вимагає дозволу `com.apple.developer.endpoint-security.client`. Це зазвичай використовується стороннім програмним забезпеченням безпеки, яке потребує взаємодії з фреймворком безпеки кінцевої точки.

Розширення безпеки кінцевої точки: **`libEndpointSecurity.dylib`** - це бібліотека С, яку використовують системні розширення для взаємодії з ядром. Ця бібліотека використовує I/O Kit (`IOKit`) для взаємодії з Ядерним розширенням безпеки кінцевої точки.

**`endpointsecurityd`** - це ключовий системний демон, який бере участь у керуванні та запуску системних розширень безпеки кінцевої точки, зокрема під час раннього процесу завантаження. **Тільки системні розширення**, позначені як **`NSEndpointSecurityEarlyBoot`** у їх файлі `Info.plist`, отримують цю обробку під час раннього завантаження.

Інший системний демон, **`sysextd`**, **перевіряє системні розширення** та переміщає їх у відповідні системні розташування. Потім він просить відповідний демон завантажити розширення. **`SystemExtensions.framework`** відповідає за активацію та деактивацію системних розширень.

## Ухилення від ESF

ESF використовується безпековими інструментами, які намагатимуться виявити червоного командира, тому будь-яка інформація про те, як цього уникнути, звучить цікаво.

### CVE-2021-30965

Справа в тому, що додатку безпеки потрібно мати **повний доступ до диска**. Таким чином, якщо зловмисник може це видалити, він може запобігти запуску програмного забезпечення:
```bash
tccutil reset All
```
Для **додаткової інформації** про цей обхід та пов'язані з ним перевірте виступ [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

В кінці це було виправлено, надавши новий дозвіл **`kTCCServiceEndpointSecurityClient`** додатку безпеки, керованому **`tccd`**, щоб `tccutil` не очищав його дозволи, запобігаючи його запуску.

## Посилання

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
