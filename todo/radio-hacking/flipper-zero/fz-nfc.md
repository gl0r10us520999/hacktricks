# FZ - NFC

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Вступ <a href="#id-9wrzi" id="id-9wrzi"></a>

Для отримання інформації про RFID та NFC перегляньте наступну сторінку:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## Підтримувані NFC картки <a href="#id-9wrzi" id="id-9wrzi"></a>

{% hint style="danger" %}
Окрім NFC карток, Flipper Zero підтримує **інші типи карток високої частоти**, такі як кілька **Mifare** Classic та Ultralight і **NTAG**.
{% endhint %}

Нові типи NFC карток будуть додані до списку підтримуваних карток. Flipper Zero підтримує наступні **NFC картки типу A** (ISO 14443A):

* ﻿**Банківські картки (EMV)** — лише читання UID, SAK та ATQA без збереження.
* ﻿**Невідомі картки** — читання (UID, SAK, ATQA) та емуляція UID.

Для **NFC карток типу B, типу F та типу V**, Flipper Zero може читати UID без його збереження.

### NFC картки типу A <a href="#uvusf" id="uvusf"></a>

#### Банківська картка (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero може лише читати UID, SAK, ATQA та збережені дані на банківських картках **без збереження**.

Екран читання банківської картки. Для банківських карток Flipper Zero може лише читати дані **без збереження та емуляції**.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### Невідомі картки <a href="#id-37eo8" id="id-37eo8"></a>

Коли Flipper Zero **не може визначити тип NFC картки**, тоді можна **читати та зберігати лише UID, SAK та ATQA**.

Екран читання невідомої картки. Для невідомих NFC карток Flipper Zero може емуляціювати лише UID.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC картки типів B, F та V <a href="#wyg51" id="wyg51"></a>

Для **NFC карток типів B, F та V**, Flipper Zero може лише **читати та відображати UID** без його збереження.

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## Дії

Для вступу про NFC [**прочитайте цю сторінку**](../pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz).

### Читання

Flipper Zero може **читати NFC картки**, однак він **не розуміє всі протоколи**, що базуються на ISO 14443. Однак, оскільки **UID є атрибутом низького рівня**, ви можете опинитися в ситуації, коли **UID вже прочитано, але протокол передачі даних високого рівня все ще невідомий**. Ви можете читати, емуляціювати та вручну вводити UID, використовуючи Flipper для примітивних зчитувачів, які використовують UID для авторизації.

#### Читання UID VS Читання даних всередині <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

У Flipper читання міток 13.56 МГц можна поділити на дві частини:

* **Читання низького рівня** — читає лише UID, SAK та ATQA. Flipper намагається вгадати протокол високого рівня на основі цих даних, прочитаних з картки. Ви не можете бути на 100% впевненими в цьому, оскільки це лише припущення на основі певних факторів.
* **Читання високого рівня** — читає дані з пам'яті картки, використовуючи специфічний протокол високого рівня. Це буде читання даних з Mifare Ultralight, читання секторів з Mifare Classic або читання атрибутів картки з PayPass/Apple Pay.

### Читання специфічного

У разі, якщо Flipper Zero не може визначити тип картки з даних низького рівня, у `Додаткових діях` ви можете вибрати `Читати специфічний тип картки` та **вручну** **вказати тип картки, яку ви хочете прочитати**.

#### EMV банківські картки (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

Окрім простого читання UID, ви можете витягти набагато більше даних з банківської картки. Можливо **отримати повний номер картки** (16 цифр на лицьовій стороні картки), **дату дії**, а в деяких випадках навіть **ім'я власника** разом зі списком **найновіших транзакцій**.\
Однак, ви **не можете прочитати CVV таким чином** (3 цифри на зворотному боці картки). Також **банківські картки захищені від атак повторного відтворення**, тому копіювання їх за допомогою Flipper і спроба емуляції для оплати чогось не спрацює.

## Посилання

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
