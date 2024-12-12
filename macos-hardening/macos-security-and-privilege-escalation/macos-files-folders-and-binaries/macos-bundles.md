# Бандли macOS

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Основна інформація

Бандли в macOS служать контейнерами для різноманітних ресурсів, включаючи програми, бібліотеки та інші необхідні файли, що робить їх вигляд схожими на один об'єкт у Finder, наприклад, відомі файли `*.app`. Найпоширенішим бандлом є бандл `.app`, хоча інші типи, такі як `.framework`, `.systemextension` та `.kext`, також поширені.

### Основні компоненти бандлу

У бандлі, зокрема у каталозі `<application>.app/Contents/`, розміщено різноманітні важливі ресурси:

* **\_CodeSignature**: Цей каталог зберігає важливі дані підписування коду, необхідні для перевірки цілісності програми. Ви можете переглянути інформацію про підпис коду за допомогою команд, наприклад: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Містить виконуваний бінарний файл програми, який запускається при взаємодії з користувачем.
* **Resources**: Сховище компонентів інтерфейсу користувача програми, включаючи зображення, документи та описи інтерфейсу (файли nib/xib).
* **Info.plist**: Діє як основний файл конфігурації програми, важливий для системи для визнання та взаємодії з програмою належним чином.

#### Важливі ключі в Info.plist

Файл `Info.plist` є важливим для конфігурації програми, містить ключі, такі як:

* **CFBundleExecutable**: Вказує назву основного виконуваного файлу, розташованого в каталозі `Contents/MacOS`.
* **CFBundleIdentifier**: Надає глобальний ідентифікатор програми, який широко використовується macOS для управління програмами.
* **LSMinimumSystemVersion**: Вказує мінімальну версію macOS, необхідну для запуску програми.

### Дослідження бандлів

Для дослідження вмісту бандлу, такого як `Safari.app`, можна використати наступну команду: `bash ls -lR /Applications/Safari.app/Contents`

Це дослідження розкриває каталоги, такі як `_CodeSignature`, `MacOS`, `Resources`, та файли, такі як `Info.plist`, кожен з яких виконує унікальну функцію від забезпечення програми до визначення її інтерфейсу користувача та операційних параметрів.

#### Додаткові каталоги бандлу

Поза загальними каталогами, бандли також можуть містити:

* **Frameworks**: Містить упаковані фреймворки, використовувані програмою. Фреймворки схожі на dylibs з додатковими ресурсами.
* **PlugIns**: Каталог для плагінів та розширень, які покращують можливості програми.
* **XPCServices**: Містить XPC-сервіси, використовувані програмою для міжпроцесної комунікації.

Ця структура забезпечує усі необхідні компоненти, укладені в бандл, що сприяє модульному та безпечному середовищу програми.

Для отримання більш детальної інформації про ключі `Info.plist` та їх значення, документація розробника Apple надає обширні ресурси: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
