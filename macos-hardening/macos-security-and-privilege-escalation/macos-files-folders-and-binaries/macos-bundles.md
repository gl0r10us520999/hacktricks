# macOS Bundles

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

Бандли в macOS слугують контейнерами для різноманітних ресурсів, включаючи програми, бібліотеки та інші необхідні файли, що дозволяє їм з'являтися як єдині об'єкти в Finder, такі як знайомі файли `*.app`. Найбільш поширеним бандлом є бандл `.app`, хоча також поширені інші типи, такі як `.framework`, `.systemextension` та `.kext`.

### Essential Components of a Bundle

Усередині бандла, зокрема в каталозі `<application>.app/Contents/`, зберігається різноманіття важливих ресурсів:

* **\_CodeSignature**: Цей каталог зберігає деталі підпису коду, які є важливими для перевірки цілісності програми. Ви можете перевірити інформацію про підпис коду, використовуючи команди, такі як: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Містить виконуваний бінарний файл програми, який запускається при взаємодії з користувачем.
* **Resources**: Сховище для компонентів інтерфейсу користувача програми, включаючи зображення, документи та описи інтерфейсу (файли nib/xib).
* **Info.plist**: Виконує роль основного конфігураційного файлу програми, що є важливим для системи, щоб правильно розпізнавати та взаємодіяти з програмою.

#### Important Keys in Info.plist

Файл `Info.plist` є основою для конфігурації програми, містячи такі ключі:

* **CFBundleExecutable**: Вказує на ім'я основного виконуваного файлу, розташованого в каталозі `Contents/MacOS`.
* **CFBundleIdentifier**: Надає глобальний ідентифікатор для програми, який широко використовується macOS для управління програмами.
* **LSMinimumSystemVersion**: Вказує на мінімальну версію macOS, необхідну для запуску програми.

### Exploring Bundles

Щоб дослідити вміст бандла, такого як `Safari.app`, можна використовувати наступну команду: `bash ls -lR /Applications/Safari.app/Contents`

Це дослідження виявляє каталоги, такі як `_CodeSignature`, `MacOS`, `Resources`, та файли, такі як `Info.plist`, кожен з яких виконує унікальну роль, від забезпечення безпеки програми до визначення її інтерфейсу користувача та операційних параметрів.

#### Additional Bundle Directories

Окрім загальних каталогів, бандли можуть також включати:

* **Frameworks**: Містить упаковані фреймворки, які використовуються програмою. Фреймворки подібні до dylibs з додатковими ресурсами.
* **PlugIns**: Каталог для плагінів та розширень, які покращують можливості програми.
* **XPCServices**: Містить XPC-сервіси, які використовуються програмою для міжпроцесної комунікації.

Ця структура забезпечує, що всі необхідні компоненти інкапсульовані в бандлі, що сприяє модульному та безпечному середовищу програми.

Для отримання більш детальної інформації про ключі `Info.plist` та їх значення, документація розробника Apple надає обширні ресурси: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
