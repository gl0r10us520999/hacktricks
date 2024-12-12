# Файли, теки, бінарні файли та пам'ять macOS

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Структура ієрархії файлів

* **/Applications**: Встановлені додатки повинні бути тут. Усі користувачі зможуть до них отримати доступ.
* **/bin**: Бінарні файли командного рядка
* **/cores**: Якщо існує, використовується для зберігання дампів ядра
* **/dev**: Все трактується як файл, тому можна побачити тут збережені пристрої апаратного забезпечення.
* **/etc**: Файли конфігурації
* **/Library**: Тут можна знайти багато підкаталогів та файлів, пов'язаних з налаштуваннями, кешами та журналами. Папка Library існує в корені та в кожному каталозі користувача.
* **/private**: Недокументовано, але багато згаданих тек є символічними посиланнями на приватну теку.
* **/sbin**: Основні системні бінарні файли (пов'язані з адмініструванням)
* **/System**: Файл для запуску OS X. Тут ви повинні знайти в основному лише файли, специфічні для Apple (не сторонні).
* **/tmp**: Файли видаляються через 3 дні (це символічне посилання на /private/tmp)
* **/Users**: Домашня тека для користувачів.
* **/usr**: Конфігураційні та системні бінарні файли
* **/var**: Файли журналів
* **/Volumes**: Підключені диски з'являться тут.
* **/.vol**: Запускаючи `stat a.txt`, ви отримаєте щось на зразок `16777223 7545753 -rw-r--r-- 1 username wheel ...`, де перше число - це ідентифікаційний номер тома, де знаходиться файл, а друге - номер іноди. Ви можете отримати доступ до вмісту цього файлу через /.vol/ з цією інформацією, запустивши `cat /.vol/16777223/7545753`

### Теки Додатків

* **Системні додатки** розташовані у `/System/Applications`
* **Встановлені** додатки зазвичай встановлюються у `/Applications` або у `~/Applications`
* **Дані додатків** можна знайти у `/Library/Application Support` для додатків, які працюють як root, та у `~/Library/Application Support` для додатків, які працюють як користувач.
* Демони сторонніх додатків, які **потребують запуску як root**, зазвичай розташовані у `/Library/PrivilegedHelperTools/`
* **Додатки в пісочниці** відображаються у теку `~/Library/Containers`. У кожного додатка є тека, названа відповідно до ідентифікатора пакета додатка (`com.apple.Safari`).
* **Ядро** розташоване у `/System/Library/Kernels/kernel`
* **Розширення ядра Apple** розташовані у `/System/Library/Extensions`
* **Розширення ядра сторонніх виробників** зберігаються у `/Library/Extensions`

### Файли з Чутливою Інформацією

macOS зберігає інформацію, таку як паролі, в декількох місцях:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Вразливі інсталятори pkg

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## Особливі Розширення OS X

* **`.dmg`**: Файли образів дисків Apple є дуже поширеними для програм-інсталяторів.
* **`.kext`**: Вони повинні мати певну структуру і є версією драйвера для OS X. (це пакет)
* **`.plist`**: Також відомий як property list, зберігає інформацію у форматі XML або бінарному форматі.
* Може бути XML або бінарний. Бінарні можна прочитати за допомогою:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Додатки Apple, які слідують структурі теки (це пакет).
* **`.dylib`**: Динамічні бібліотеки (схожі на файли DLL у Windows)
* **`.pkg`**: Це те саме, що й xar (eXtensible Archive format). Команда installer може використовуватися для встановлення вмісту цих файлів.
* **`.DS_Store`**: Цей файл є в кожній текі, він зберігає атрибути та налаштування теки.
* **`.Spotlight-V100`**: Ця тека з'являється в кореневій текі кожного тому в системі.
* **`.metadata_never_index`**: Якщо цей файл знаходиться в корені тому, Spotlight не індексуватиме цей том.
* **`.noindex`**: Файли та теки з цим розширенням не будуть індексуватися Spotlight.
* **`.sdef`**: Файли всередині пакетів, що вказують, як можна взаємодіяти з додатком з AppleScript.

### Пакети macOS

Пакет - це **тека**, яка **виглядає як об'єкт у Finder** (прикладом пакету є файли `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Кеш Спільних Бібліотек Dyld (SLC)

У macOS (і iOS) всі системні спільні бібліотеки, такі як фреймворки та dylib, **об'єднані в один файл**, який називається **кешем спільних бібліотек dyld**. Це покращує продуктивність, оскільки код може завантажуватися швидше.

Це розташовано в macOS у `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/`, а в старіших версіях ви можете знайти **спільний кеш** у **`/System/Library/dyld/`**.\
У iOS ви можете знайти їх у **`/System/Library/Caches/com.apple.dyld/`**.

Подібно до кешу спільних бібліотек dyld, ядро та розширення ядра також компілюються в кеш ядра, який завантажується під час завантаження системи.

Для вилучення бібліотек з одного файлу кешу спільних бібліотек dylib можна було використовувати бінарний файл [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip), який зараз може не працювати, але ви також можете використовувати [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
Зверніть увагу, що навіть якщо інструмент `dyld_shared_cache_util` не працює, ви можете передати **спільний dyld-бінарний файл Hopper** і Hopper зможе ідентифікувати всі бібліотеки та дозволить вам **вибрати, яку саме** ви хочете дослідити:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Деякі екстрактори можуть не працювати, оскільки dylibs попередньо зв'язані з жорстко закодованими адресами, тому вони можуть переходити на невідомі адреси.

{% hint style="success" %}
Також можна завантажити Спільний Кеш Бібліотек інших \*OS пристроїв у macOS, використовуючи емулятор у Xcode. Вони будуть завантажені за адресою: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, наприклад: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Картографування SLC

**`dyld`** використовує системний виклик **`shared_region_check_np`** для визначення того, чи було змаповано SLC (який повертає адресу), та **`shared_region_map_and_slide_np`** для мапування SLC.

Зверніть увагу, що навіть якщо SLC зміщено при першому використанні, всі **процеси** використовують **одну копію**, що **скасовує захист ASLR**, якщо зловмисник зміг запустити процеси в системі. Це фактично використовувалося у минулому і виправлено за допомогою спільного регіону пейджера.

Базові групи - це невеликі Mach-O dylibs, які створюють невеликі проміжки між відображеннями зображень, що робить неможливим перехоплення функцій.

### Перевизначення SLC

Використовуючи змінні середовища:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Це дозволить завантажити новий спільний кеш бібліотек
* **`DYLD_SHARED_CACHE_DIR=avoid`** та вручну замінюйте бібліотеки символічними посиланнями на спільний кеш з реальними (їх потрібно видобути)

## Спеціальні дозволи на файли

### Дозволи на теки

У **теці** **читання** дозволяє **переглядати її**, **запис** дозволяє **видаляти** та **записувати** файли в ній, а **виконання** дозволяє **переміщатися** по каталозі. Таким чином, наприклад, користувач з **дозволом на читання файлу** всередині каталогу, де він **не має дозволу на виконання**, **не зможе прочитати** файл.

### Модифікатори прапорців

Є деякі прапорці, які можна встановити в файлах, що змінять поведінку файлу. Ви можете **перевірити прапорці** файлів у каталозі за допомогою `ls -lO /path/directory`

* **`uchg`**: Відомий як прапорець **uchange**, запобігає будь-яким діям, що змінюють або видаляють **файл**. Щоб встановити його, виконайте: `chflags uchg file.txt`
* Користувач root може **видалити прапорець** та змінити файл
* **`restricted`**: Цей прапорець робить файл **захищеним SIP** (ви не можете додати цей прапорець до файлу).
* **`Sticky bit`**: Якщо в каталозі встановлено прапорець Sticky bit, **тільки** власник каталогу або root можуть **перейменовувати або видаляти** файли. Зазвичай це встановлено в каталозі /tmp, щоб запобігти звичайним користувачам видаленню або переміщенню файлів інших користувачів.

Усі прапорці можна знайти у файлі `sys/stat.h` (знайдіть його за допомогою `mdfind stat.h | grep stat.h`) і це:

* `UF_SETTABLE` 0x0000ffff: Маска флагів, які може змінювати власник.
* `UF_NODUMP` 0x00000001: Не викидати файл.
* `UF_IMMUTABLE` 0x00000002: Файл не може бути змінений.
* `UF_APPEND` 0x00000004: Записи в файл можуть бути лише додані.
* `UF_OPAQUE` 0x00000008: Каталог є непрозоримим щодо об'єднання.
* `UF_COMPRESSED` 0x00000020: Файл стиснутий (деякі файлові системи).
* `UF_TRACKED` 0x00000040: Немає сповіщень про видалення/перейменування для файлів з цим набором.
* `UF_DATAVAULT` 0x00000080: Потрібен дозвіл для читання та запису.
* `UF_HIDDEN` 0x00008000: Підказка, що цей елемент не повинен відображатися в графічному інтерфейсі.
* `SF_SUPPORTED` 0x009f0000: Маска флагів, які підтримує суперкористувач.
* `SF_SETTABLE` 0x3fff0000: Маска флагів, які може змінювати суперкористувач.
* `SF_SYNTHETIC` 0xc0000000: Маска системних флагів тільки для читання.
* `SF_ARCHIVED` 0x00010000: Файл заархівований.
* `SF_IMMUTABLE` 0x00020000: Файл не може бути змінений.
* `SF_APPEND` 0x00040000: Записи в файл можуть бути лише додані.
* `SF_RESTRICTED` 0x00080000: Потрібен дозвіл для запису.
* `SF_NOUNLINK` 0x00100000: Елемент не може бути видалений, перейменований або змонтований.
* `SF_FIRMLINK` 0x00800000: Файл є посиланням на програму.
* `SF_DATALESS` 0x40000000: Файл є об'єктом без даних.

### **ACL файлів**

ACL файлів містять **ACE** (Access Control Entries), де можна призначити більш **деталізовані дозволи** різним користувачам.

Можливо надати ці дозволи **каталогу**: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
А для **файлу**: `read`, `write`, `append`, `execute`.

Коли файл містить ACL, ви побачите **"+" при переліку дозволів, як у**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Ви можете **читати ACLs** файлу за допомогою:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Ви можете знайти **всі файли з ACL** за допомогою (це дуже повільно):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Розширені атрибути

Розширені атрибути мають назву та бажане значення, і можуть бути переглянуті за допомогою `ls -@` та змінені за допомогою команди `xattr`. Деякі поширені розширені атрибути:

* `com.apple.resourceFork`: Сумісність ресурсного виливу. Також видимий як `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: механізм карантину Gatekeeper (III/6)
* `metadata:*`: MacOS: різноманітна метадані, такі як `_backup_excludeItem`, або `kMD*`
* `com.apple.lastuseddate` (#PS): Дата останнього використання файлу
* `com.apple.FinderInfo`: MacOS: Інформація Finder (наприклад, кольорові мітки)
* `com.apple.TextEncoding`: Вказує кодування тексту для файлів ASCII
* `com.apple.logd.metadata`: Використовується logd на файлах у `/var/db/diagnostics`
* `com.apple.genstore.*`: Генераційне сховище (`/.DocumentRevisions-V100` в корені файлової системи)
* `com.apple.rootless`: MacOS: Використовується захистом цілісності системи для позначення файлу (III/10)
* `com.apple.uuidb.boot-uuid`: Позначки logd для епох завантаження з унікальним UUID
* `com.apple.decmpfs`: MacOS: Прозоре стиснення файлів (II/7)
* `com.apple.cprotect`: \*OS: Дані про шифрування для кожного файлу (III/11)
* `com.apple.installd.*`: \*OS: Метадані, використовувані installd, наприклад, `installType`, `uniqueInstallID`

### Ресурсні виливи | macOS ADS

Це спосіб отримання **Альтернативних потоків даних в MacOS**. Ви можете зберегти вміст всередині розширеного атрибуту під назвою **com.apple.ResourceFork** всередині файлу, зберігаючи його в **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Ви можете **знайти всі файли, що містять цей розширений атрибут**, за допомогою:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Розширений атрибут `com.apple.decmpfs` вказує на те, що файл зберігається зашифрованим, `ls -l` повідомить про **розмір 0**, а стислі дані знаходяться в цьому атрибуті. Кожного разу, коли файл доступний, він буде розшифрований в пам'яті.

Цей атрибут можна побачити за допомогою `ls -lO`, вказаний як стислий, оскільки стислі файли також позначаються прапорцем `UF_COMPRESSED`. Якщо стислий файл видалити цей прапорець за допомогою `chflags nocompressed </шлях/до/файлу>`, система не буде знати, що файл був стиснутий, і, отже, не зможе розпакувати та отримати доступ до даних (вона буде вважати, що файл насправді порожній).

Інструмент afscexpand можна використовувати для примусового розпакування файлу.

## **Універсальні бінарні файли та** Формат Mach-o

Бінарні файли Mac OS зазвичай компілюються як **універсальні бінарні файли**. **Універсальний бінарний файл** може **підтримувати кілька архітектур у одному файлі**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## Пам'ять процесів macOS

## Витяг пам'яті macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Файли категорії ризику Mac OS

Каталог `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` - це місце, де зберігається інформація про **ризики, пов'язані з різними розширеннями файлів**. Цей каталог категоризує файли на різні рівні ризику, впливаючи на те, як Safari обробляє ці файли під час завантаження. Категорії наступні:

* **LSRiskCategorySafe**: Файли в цій категорії вважаються **повністю безпечними**. Safari автоматично відкриє ці файли після їх завантаження.
* **LSRiskCategoryNeutral**: Ці файли не мають попереджень і **не відкриваються автоматично** Safari.
* **LSRiskCategoryUnsafeExecutable**: Файли цієї категорії **спричиняють попередження**, що файл є додатком. Це слугує як захисний захід для попередження користувача.
* **LSRiskCategoryMayContainUnsafeExecutable**: Ця категорія призначена для файлів, таких як архіви, які можуть містити виконуваний файл. Safari **спричинить попередження**, якщо вона не може перевірити, що всі вміст безпечні або нейтральні.

## Файли журналів

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Містить інформацію про завантажені файли, таку як URL, звідки вони були завантажені.
* **`/var/log/system.log`**: Основний журнал систем OSX. com.apple.syslogd.plist відповідає за виконання системного журналювання (можна перевірити, чи воно вимкнене, шукаючи "com.apple.syslogd" в `launchctl list`.
* **`/private/var/log/asl/*.asl`**: Це журнали системи Apple, які можуть містити цікаву інформацію.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Зберігає недавно відкриті файли та програми через "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Зберігає елементи для запуску при запуску системи
* **`$HOME/Library/Logs/DiskUtility.log`**: Файл журналу для програми DiskUtility (інформація про диски, включаючи USB-накопичувачі)
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Дані про бездротові точки доступу.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Список демонів, які вимкнені.

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
