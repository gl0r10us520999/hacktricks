# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

На відміну від Kernel Extensions, **System Extensions працюють у просторі користувача** замість простору ядра, що зменшує ризик аварійної зупинки системи через несправність розширення.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Існує три типи системних розширень: **DriverKit** Extensions, **Network** Extensions та **Endpoint Security** Extensions.

### **DriverKit Extensions**

DriverKit є заміною для kernel extensions, які **надають апаратну підтримку**. Він дозволяє драйверам пристроїв (таким як USB, Serial, NIC та HID драйвери) працювати в просторі користувача, а не в просторі ядра. Фреймворк DriverKit включає **версії певних класів I/O Kit для простору користувача**, а ядро пересилає звичайні події I/O Kit у простір користувача, пропонуючи безпечніше середовище для роботи цих драйверів.

### **Network Extensions**

Network Extensions надають можливість налаштування мережевої поведінки. Існує кілька типів Network Extensions:

* **App Proxy**: Використовується для створення VPN-клієнта, який реалізує орієнтований на потоки, кастомний VPN-протокол. Це означає, що він обробляє мережевий трафік на основі з'єднань (або потоків), а не окремих пакетів.
* **Packet Tunnel**: Використовується для створення VPN-клієнта, який реалізує орієнтований на пакети, кастомний VPN-протокол. Це означає, що він обробляє мережевий трафік на основі окремих пакетів.
* **Filter Data**: Використовується для фільтрації мережевих "потоків". Він може моніторити або змінювати мережеві дані на рівні потоку.
* **Filter Packet**: Використовується для фільтрації окремих мережевих пакетів. Він може моніторити або змінювати мережеві дані на рівні пакета.
* **DNS Proxy**: Використовується для створення кастомного DNS-постачальника. Може використовуватися для моніторингу або зміни DNS-запитів і відповідей.

## Endpoint Security Framework

Endpoint Security — це фреймворк, наданий Apple в macOS, який забезпечує набір API для системної безпеки. Він призначений для використання **постачальниками безпеки та розробниками для створення продуктів, які можуть моніторити та контролювати системну активність** для виявлення та захисту від шкідливої активності.

Цей фреймворк надає **збірку API для моніторингу та контролю системної активності**, такої як виконання процесів, події файлової системи, мережеві та ядрові події.

Ядро цього фреймворку реалізовано в ядрі, як Kernel Extension (KEXT), розташоване за адресою **`/System/Library/Extensions/EndpointSecurity.kext`**. Цей KEXT складається з кількох ключових компонентів:

* **EndpointSecurityDriver**: Діє як "точка входу" для розширення ядра. Це основна точка взаємодії між ОС та фреймворком Endpoint Security.
* **EndpointSecurityEventManager**: Цей компонент відповідає за реалізацію хуків ядра. Хуки ядра дозволяють фреймворку моніторити системні події, перехоплюючи системні виклики.
* **EndpointSecurityClientManager**: Це управляє зв'язком з клієнтами простору користувача, відстежуючи, які клієнти підключені та потребують отримання сповіщень про події.
* **EndpointSecurityMessageManager**: Це надсилає повідомлення та сповіщення про події клієнтам простору користувача.

Події, які фреймворк Endpoint Security може моніторити, класифікуються на:

* Події файлів
* Події процесів
* Події сокетів
* Події ядра (такі як завантаження/вивантаження розширення ядра або відкриття пристрою I/O Kit)

### Архітектура фреймворку Endpoint Security

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Зв'язок у просторі користувача** з фреймворком Endpoint Security відбувається через клас IOUserClient. Використовуються два різні підкласи, залежно від типу виклику:

* **EndpointSecurityDriverClient**: Це вимагає привілею `com.apple.private.endpoint-security.manager`, який має лише системний процес `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Це вимагає привілею `com.apple.developer.endpoint-security.client`. Це зазвичай використовуватиметься стороннім програмним забезпеченням безпеки, яке потребує взаємодії з фреймворком Endpoint Security.

Розширення Endpoint Security:**`libEndpointSecurity.dylib`** — це C-бібліотека, яку системні розширення використовують для зв'язку з ядром. Ця бібліотека використовує I/O Kit (`IOKit`) для зв'язку з KEXT Endpoint Security.

**`endpointsecurityd`** — це ключовий системний демон, який бере участь в управлінні та запуску системних розширень безпеки кінцевих точок, особливо під час раннього процесу завантаження. **Тільки системні розширення**, позначені **`NSEndpointSecurityEarlyBoot`** у їхньому файлі `Info.plist`, отримують це раннє завантаження.

Ще один системний демон, **`sysextd`**, **перевіряє системні розширення** та переміщує їх у відповідні системні місця. Потім він запитує відповідний демон, щоб завантажити розширення. **`SystemExtensions.framework`** відповідає за активацію та деактивацію системних розширень.

## Обхід ESF

ESF використовується інструментами безпеки, які намагатимуться виявити червону команду, тому будь-яка інформація про те, як цього можна уникнути, звучить цікаво.

### CVE-2021-30965

Справа в тому, що безпекова програма повинна мати **дозволи на повний доступ до диска**. Тож, якщо зловмисник зможе це видалити, він зможе запобігти запуску програмного забезпечення:
```bash
tccutil reset All
```
Для **додаткової інформації** про цей обхід та пов'язані з ним, перегляньте доповідь [#OBTS v5.0: "Ахіллесова пета EndpointSecurity" - Фіцл Чаба](https://www.youtube.com/watch?v=lQO7tvNCoTI)

В кінці це було виправлено шляхом надання нового дозволу **`kTCCServiceEndpointSecurityClient`** безпековому додатку, яким керує **`tccd`**, щоб `tccutil` не очищав його дозволи, що заважає йому працювати.

## Посилання

* [**OBTS v3.0: "Безпека та небезпека Endpoint" - Скотт Найт**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
