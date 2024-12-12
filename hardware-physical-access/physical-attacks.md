# Фізичні атаки

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
{% endhint %}

## Відновлення пароля BIOS та безпека системи

**Скидання BIOS** можна здійснити кількома способами. Більшість материнських плат містять **батарею**, яка, якщо її зняти на приблизно **30 хвилин**, скине налаштування BIOS, включаючи пароль. Альтернативно, **перемичка на материнській платі** може бути налаштована для скидання цих налаштувань шляхом з'єднання певних контактів.

У ситуаціях, коли апаратні зміни неможливі або не практичні, **програмні інструменти** пропонують рішення. Запуск системи з **Live CD/USB** з дистрибутивами, такими як **Kali Linux**, надає доступ до інструментів, таких як **_killCmos_** та **_CmosPWD_**, які можуть допомогти у відновленні пароля BIOS.

У випадках, коли пароль BIOS невідомий, неправильне введення його **тричі** зазвичай призводить до коду помилки. Цей код можна використовувати на веб-сайтах, таких як [https://bios-pw.org](https://bios-pw.org), щоб потенційно отримати придатний пароль.

### Безпека UEFI

Для сучасних систем, що використовують **UEFI** замість традиційного BIOS, інструмент **chipsec** може бути використаний для аналізу та модифікації налаштувань UEFI, включаючи вимкнення **Secure Boot**. Це можна зробити за допомогою наступної команди:

`python chipsec_main.py -module exploits.secure.boot.pk`

### Аналіз RAM та атаки холодного старту

RAM зберігає дані короткий час після відключення живлення, зазвичай протягом **1-2 хвилин**. Цю тривалість можна продовжити до **10 хвилин**, застосовуючи холодні речовини, такі як рідкий азот. Протягом цього розширеного періоду можна створити **дамп пам'яті** за допомогою інструментів, таких як **dd.exe** та **volatility** для аналізу.

### Атаки прямого доступу до пам'яті (DMA)

**INCEPTION** - це інструмент, призначений для **фізичної маніпуляції пам'яттю** через DMA, сумісний з інтерфейсами, такими як **FireWire** та **Thunderbolt**. Він дозволяє обійти процедури входу, модифікуючи пам'ять для прийняття будь-якого пароля. Однак він неефективний проти систем **Windows 10**.

### Live CD/USB для доступу до системи

Зміна системних бінарних файлів, таких як **_sethc.exe_** або **_Utilman.exe_**, на копію **_cmd.exe_** може надати командний рядок з системними привілеями. Інструменти, такі як **chntpw**, можуть бути використані для редагування файлу **SAM** Windows-інсталяції, що дозволяє змінювати паролі.

**Kon-Boot** - це інструмент, який полегшує вхід у системи Windows без знання пароля, тимчасово модифікуючи ядро Windows або UEFI. Більше інформації можна знайти на [https://www.raymond.cc](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/).

### Обробка функцій безпеки Windows

#### Скорочення для завантаження та відновлення

- **Supr**: Доступ до налаштувань BIOS.
- **F8**: Вхід у режим відновлення.
- Натискання **Shift** після банера Windows може обійти автоматичний вхід.

#### BAD USB пристрої

Пристрої, такі як **Rubber Ducky** та **Teensyduino**, слугують платформами для створення **поганих USB** пристроїв, здатних виконувати попередньо визначені корисні навантаження при підключенні до цільового комп'ютера.

#### Копія тіней томів

Привілеї адміністратора дозволяють створювати копії чутливих файлів, включаючи файл **SAM**, через PowerShell.

### Обхід шифрування BitLocker

Шифрування BitLocker може бути потенційно обійдено, якщо **відновлювальний пароль** знайдено в дамп-файлі пам'яті (**MEMORY.DMP**). Для цього можна використовувати інструменти, такі як **Elcomsoft Forensic Disk Decryptor** або **Passware Kit Forensic**.

### Соціальна інженерія для додавання ключа відновлення

Новий ключ відновлення BitLocker може бути доданий за допомогою тактик соціальної інженерії, переконуючи користувача виконати команду, яка додає новий ключ відновлення, що складається з нулів, спрощуючи процес розшифровки.
{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
</details>
{% endhint %}
