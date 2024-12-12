# macOS Launch/Environment Constraints & Trust Cache

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

Обмеження запуску в macOS були введені для підвищення безпеки шляхом **регулювання того, як, ким і звідки може бути ініційований процес**. Введені в macOS Ventura, вони надають структуру, яка категоризує **кожен системний бінарний файл у різні категорії обмежень**, які визначені в **кеші довіри**, списку, що містить системні бінарні файли та їх відповідні хеші. Ці обмеження поширюються на кожен виконуваний бінар у системі, що передбачає набір **правил**, які визначають вимоги для **запуску конкретного бінарного файлу**. Правила охоплюють самостійні обмеження, які бінарний файл повинен задовольнити, обмеження батьківського процесу, які повинні бути виконані його батьківським процесом, та відповідальні обмеження, які повинні дотримуватися інші відповідні суб'єкти.

Механізм поширюється на сторонні програми через **обмеження середовища**, починаючи з macOS Sonoma, що дозволяє розробникам захищати свої програми, вказуючи **набір ключів і значень для обмежень середовища.**

Ви визначаєте **обмеження середовища запуску та бібліотеки** в словниках обмежень, які ви або зберігаєте в **файлах списку властивостей `launchd`**, або в **окремих файлах списку властивостей**, які ви використовуєте для підписування коду.

Існує 4 типи обмежень:

* **Самостійні обмеження**: Обмеження, що застосовуються до **запущеного** бінарного файлу.
* **Батьківський процес**: Обмеження, що застосовуються до **батька процесу** (наприклад, **`launchd`**, що запускає службу XP)
* **Відповідальні обмеження**: Обмеження, що застосовуються до **процесу, що викликає службу** в комунікації XPC
* **Обмеження завантаження бібліотеки**: Використовуйте обмеження завантаження бібліотеки, щоб вибірково описати код, який може бути завантажений

Отже, коли процес намагається запустити інший процес — викликом `execve(_:_:_:)` або `posix_spawn(_:_:_:_:_:_:)` — операційна система перевіряє, що **виконуваний** файл **задовольняє** своє **власне самостійне обмеження**. Вона також перевіряє, що **виконуваний файл батьківського** **процесу** **задовольняє** **обмеження батька** виконуваного файлу, і що **виконуваний файл відповідального** **процесу** **задовольняє обмеження відповідального процесу** виконуваного файлу. Якщо будь-яке з цих обмежень запуску не задовольняється, операційна система не запускає програму.

Якщо під час завантаження бібліотеки будь-яка частина **обмеження бібліотеки не є істинною**, ваш процес **не завантажує** бібліотеку.

## LC Categories

LC складається з **фактів** та **логічних операцій** (і, або..), які поєднують факти.

[**Факти, які може використовувати LC, задокументовані**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints). Наприклад:

* is-init-proc: Логічне значення, яке вказує, чи повинен виконуваний файл бути процесом ініціалізації операційної системи (`launchd`).
* is-sip-protected: Логічне значення, яке вказує, чи повинен виконуваний файл бути файлом, захищеним захистом цілісності системи (SIP).
* `on-authorized-authapfs-volume:` Логічне значення, яке вказує, чи завантажила операційна система виконуваний файл з авторизованого, аутентифікованого обсягу APFS.
* `on-authorized-authapfs-volume`: Логічне значення, яке вказує, чи завантажила операційна система виконуваний файл з авторизованого, аутентифікованого обсягу APFS.
* Обсяг Cryptexes
* `on-system-volume:` Логічне значення, яке вказує, чи завантажила операційна система виконуваний файл з поточного завантаженого системного обсягу.
* Всередині /System...
* ...

Коли бінарний файл Apple підписується, він **призначає його до категорії LC** всередині **кешу довіри**.

* **Категорії LC iOS 16** були [**перевернуті та задокументовані тут**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056).
* Поточні **категорії LC (macOS 14** - Somona) були перевернуті, і їх [**описи можна знайти тут**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53).

Наприклад, категорія 1 є:
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: Повинен бути в системному або Cryptexes обсязі.
* `launch-type == 1`: Повинен бути системною службою (plist в LaunchDaemons).
* `validation-category == 1`: Виконуваний файл операційної системи.
* `is-init-proc`: Launchd

### Реверсування LC категорій

У вас є більше інформації [**про це тут**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints), але в основному, вони визначені в **AMFI (AppleMobileFileIntegrity)**, тому вам потрібно завантажити набір для розробки ядра, щоб отримати **KEXT**. Символи, що починаються з **`kConstraintCategory`**, є **цікавими**. Витягуючи їх, ви отримаєте DER (ASN.1) закодований потік, який вам потрібно буде декодувати за допомогою [ASN.1 Decoder](https://holtstrom.com/michael/tools/asn1decoder.php) або бібліотеки python-asn1 та її скрипта `dump.py`, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master), що надасть вам більш зрозумілий рядок.

## Обмеження середовища

Це обмеження запуску, налаштовані в **додатках третіх сторін**. Розробник може вибрати **факти** та **логічні операнди для використання** у своєму додатку, щоб обмежити доступ до нього.

Можливо перерахувати обмеження середовища додатка за допомогою:
```bash
codesign -d -vvvv app.app
```
## Кеші Довіри

В **macOS** є кілька кешів довіри:

* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
* **`/System/Library/Security/OSLaunchPolicyData`**

А в iOS, здається, це знаходиться в **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`**.

{% hint style="warning" %}
На macOS, що працює на пристроях Apple Silicon, якщо бінарний файл, підписаний Apple, не знаходиться в кеші довіри, AMFI відмовиться його завантажити.
{% endhint %}

### Перерахування Кешів Довіри

Попередні файли кешу довіри мають формат **IMG4** та **IM4P**, причому IM4P є секцією корисного навантаження формату IMG4.

Ви можете використовувати [**pyimg4**](https://github.com/m1stadev/PyIMG4) для витягнення корисного навантаження баз даних:

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
{% endcode %}

(Інший варіант - використати інструмент [**img4tool**](https://github.com/tihmstar/img4tool), який працюватиме навіть на M1, навіть якщо випуск старий, і для x86\_64, якщо ви встановите його в правильні місця).

Тепер ви можете використовувати інструмент [**trustcache**](https://github.com/CRKatri/trustcache), щоб отримати інформацію в читабельному форматі:
```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```
Кеш довіри має таку структуру, тому **категорія LC є 4-м стовпцем**
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
Тоді ви можете використовувати скрипт, такий як [**цей**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30), щоб витягти дані.

З цих даних ви можете перевірити програми з **значенням обмежень запуску `0`**, які не мають обмежень ([**перевірте тут**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056), що означає кожне значення).

## Заходи щодо пом'якшення атак

Обмеження запуску могли б пом'якшити кілька старих атак, **переконавшись, що процес не буде виконуватись в несподіваних умовах:** Наприклад, з несподіваних місць або за запитом несподіваного батьківського процесу (якщо тільки launchd не повинен його запускати).

Більше того, обмеження запуску також **пом'якшують атаки з пониженням рівня.**

Однак вони **не пом'якшують загальні зловживання XPC**, **впровадження коду Electron** або **впровадження dylib** без валідації бібліотек (якщо тільки ідентифікатори команди, які можуть завантажувати бібліотеки, не відомі).

### Захист від демонів XPC

У випуску Sonoma важливим моментом є **конфігурація відповідальності** служби демонів XPC. Служба XPC відповідає за себе, на відміну від підключеного клієнта, який несе відповідальність. Це задокументовано у звіті зворотного зв'язку FB13206884. Ця конфігурація може здаватися недосконалою, оскільки дозволяє певні взаємодії зі службою XPC:

- **Запуск служби XPC**: Якщо вважати це помилкою, ця конфігурація не дозволяє ініціювати службу XPC через код зловмисника.
- **Підключення до активної служби**: Якщо служба XPC вже працює (можливо, активована її оригінальним додатком), немає перешкод для підключення до неї.

Хоча впровадження обмежень на службу XPC може бути корисним, **звужуючи вікно для потенційних атак**, це не вирішує основну проблему. Забезпечення безпеки служби XPC вимагає **ефективної валідації підключеного клієнта**. Це залишається єдиним способом зміцнити безпеку служби. Також варто зазначити, що згадана конфігурація відповідальності наразі діє, що може не відповідати запланованому дизайну.

### Захист Electron

Навіть якщо вимагається, щоб додаток був **відкритий через LaunchService** (в обмеженнях батьків). Це можна досягти, використовуючи **`open`** (який може встановлювати змінні середовища) або використовуючи **API Launch Services** (де можуть бути вказані змінні середовища).

## Посилання

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

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
