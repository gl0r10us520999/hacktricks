# macOS Файли, Папки, Бінарні Файли та Пам'ять

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

## Ієрархія файлів

* **/Applications**: Встановлені програми повинні бути тут. Усі користувачі зможуть отримати до них доступ.
* **/bin**: Бінарні файли командного рядка
* **/cores**: Якщо існує, використовується для зберігання дампів пам'яті
* **/dev**: Все розглядається як файл, тому ви можете побачити апаратні пристрої, збережені тут.
* **/etc**: Конфігураційні файли
* **/Library**: Тут можна знайти багато підкаталогів і файлів, пов'язаних з налаштуваннями, кешами та журналами. Папка Library існує в кореневому каталозі та в каталозі кожного користувача.
* **/private**: Не задокументовано, але багато з вказаних папок є символічними посиланнями на приватний каталог.
* **/sbin**: Основні системні бінарні файли (пов'язані з адмініструванням)
* **/System**: Файли для запуску OS X. Тут ви повинні знайти в основному лише специфічні для Apple файли (не сторонні).
* **/tmp**: Файли видаляються через 3 дні (це м'яке посилання на /private/tmp)
* **/Users**: Домашній каталог для користувачів.
* **/usr**: Конфігураційні та системні бінарні файли
* **/var**: Журнальні файли
* **/Volumes**: Змонтовані диски з'являться тут.
* **/.vol**: Запустивши `stat a.txt`, ви отримаєте щось на зразок `16777223 7545753 -rw-r--r-- 1 username wheel ...`, де перше число - це ідентифікаційний номер тому, де існує файл, а друге - номер inode. Ви можете отримати доступ до вмісту цього файлу через /.vol/ з цією інформацією, запустивши `cat /.vol/16777223/7545753`

### Папки програм

* **Системні програми** розташовані в `/System/Applications`
* **Встановлені** програми зазвичай встановлюються в `/Applications` або в `~/Applications`
* **Дані програм** можна знайти в `/Library/Application Support` для програм, що працюють від імені root, і `~/Library/Application Support` для програм, що працюють від імені користувача.
* Сторонні програми **демони**, які **потрібно запускати від імені root**, зазвичай розташовані в `/Library/PrivilegedHelperTools/`
* **Пісочниці** програми відображаються в папці `~/Library/Containers`. Кожна програма має папку, названу відповідно до ідентифікатора пакета програми (`com.apple.Safari`).
* **Ядро** розташоване в `/System/Library/Kernels/kernel`
* **Розширення ядра Apple** розташовані в `/System/Library/Extensions`
* **Сторонні розширення ядра** зберігаються в `/Library/Extensions`

### Файли з чутливою інформацією

MacOS зберігає інформацію, таку як паролі, у кількох місцях:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Вразливі установники pkg

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## Специфічні розширення OS X

* **`.dmg`**: Файли образу диска Apple дуже поширені для установників.
* **`.kext`**: Повинен дотримуватися специфічної структури і є версією драйвера для OS X. (це пакет)
* **`.plist`**: Відомий також як список властивостей, зберігає інформацію у форматі XML або бінарному.
* Може бути XML або бінарним. Бінарні можна прочитати за допомогою:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Програми Apple, які дотримуються структури каталогу (це пакет).
* **`.dylib`**: Динамічні бібліотеки (як файли DLL Windows)
* **`.pkg`**: Це те ж саме, що і xar (формат розширеного архіву). Команда установника може бути використана для встановлення вмісту цих файлів.
* **`.DS_Store`**: Цей файл є в кожному каталозі, він зберігає атрибути та налаштування каталогу.
* **`.Spotlight-V100`**: Ця папка з'являється в кореневому каталозі кожного тому в системі.
* **`.metadata_never_index`**: Якщо цей файл знаходиться в корені тому, Spotlight не буде індексувати цей том.
* **`.noindex`**: Файли та папки з цим розширенням не будуть індексуватися Spotlight.
* **`.sdef`**: Файли всередині пакетів, що вказують, як можна взаємодіяти з програмою з AppleScript.

### Пакети macOS

Пакет - це **каталог**, який **виглядає як об'єкт у Finder** (приклад пакета - це файли `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Кеш спільних бібліотек Dyld (SLC)

У macOS (і iOS) всі системні спільні бібліотеки, такі як фреймворки та dylibs, **об'єднуються в один файл**, який називається **кешем спільного використання dyld**. Це покращує продуктивність, оскільки код може завантажуватися швидше.

Це розташовано в macOS в `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/`, а в старіших версіях ви можете знайти **спільний кеш** в **`/System/Library/dyld/`**.\
В iOS ви можете знайти їх у **`/System/Library/Caches/com.apple.dyld/`**.

Подібно до кешу спільного використання dyld, ядро та розширення ядра також компілюються в кеш ядра, який завантажується під час завантаження.

Щоб витягти бібліотеки з єдиного файлу кешу спільного використання dylib, можна було використовувати бінарний [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip), який, можливо, зараз не працює, але ви також можете використовувати [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

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
Зверніть увагу, що навіть якщо інструмент `dyld_shared_cache_util` не працює, ви можете передати **спільний двійковий файл dyld в Hopper**, і Hopper зможе ідентифікувати всі бібліотеки та дозволить вам **вибрати, яку з них** ви хочете дослідити:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Деякі екстрактори не працюватимуть, оскільки dylibs попередньо зв'язані з жорстко закодованими адресами, тому вони можуть переходити на невідомі адреси.

{% hint style="success" %}
Також можливо завантажити кеш спільних бібліотек інших \*OS пристроїв в macos, використовуючи емулятор в Xcode. Вони будуть завантажені в: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, наприклад: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Mapping SLC

**`dyld`** використовує системний виклик **`shared_region_check_np`**, щоб дізнатися, чи був SLC відображений (що повертає адресу) і **`shared_region_map_and_slide_np`**, щоб відобразити SLC.

Зверніть увагу, що навіть якщо SLC зсувається при першому використанні, всі **процеси** використовують **одну й ту ж копію**, що **усуває захист ASLR**, якщо зловмисник зміг запустити процеси в системі. Це насправді було використано в минулому і виправлено за допомогою спільного регіонального пейджера.

Пул гілок - це маленькі Mach-O dylibs, які створюють невеликі простори між відображеннями зображень, що ускладнює перехоплення функцій.

### Override SLCs

Використовуючи змінні середовища:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Це дозволить завантажити новий кеш спільних бібліотек.
* **`DYLD_SHARED_CACHE_DIR=avoid`** і вручну замінити бібліотеки на символічні посилання на спільний кеш з реальними (вам потрібно буде їх витягти).

## Special File Permissions

### Folder permissions

У **папці**, **читання** дозволяє **переглядати її**, **запис** дозволяє **видаляти** та **записувати** файли в ній, а **виконання** дозволяє **переміщатися** по директорії. Отже, наприклад, користувач з **дозволом на читання файлу** всередині директорії, де він **не має дозволу на виконання**, **не зможе прочитати** файл.

### Flag modifiers

Є деякі прапори, які можуть бути встановлені у файлах, що змусить файл поводитися інакше. Ви можете **перевірити прапори** файлів всередині директорії за допомогою `ls -lO /path/directory`

* **`uchg`**: Відомий як **uchange** прапор, який **запобігає будь-якій дії** зміни або видалення **файлу**. Щоб встановити його, виконайте: `chflags uchg file.txt`
* Користувач root може **видалити прапор** і змінити файл.
* **`restricted`**: Цей прапор робить файл **захищеним SIP** (ви не можете додати цей прапор до файлу).
* **`Sticky bit`**: Якщо директорія має sticky bit, **тільки** власник **директорії або root можуть перейменовувати або видаляти** файли. Зазвичай це встановлюється на директорії /tmp, щоб запобігти звичайним користувачам видаляти або переміщати файли інших користувачів.

Усі прапори можна знайти у файлі `sys/stat.h` (знайдіть його за допомогою `mdfind stat.h | grep stat.h`) і вони є:

* `UF_SETTABLE` 0x0000ffff: Маска змінних прапорів власника.
* `UF_NODUMP` 0x00000001: Не вивантажувати файл.
* `UF_IMMUTABLE` 0x00000002: Файл не може бути змінений.
* `UF_APPEND` 0x00000004: Записи у файл можуть лише додаватися.
* `UF_OPAQUE` 0x00000008: Директорія є непрозорою щодо об'єднання.
* `UF_COMPRESSED` 0x00000020: Файл стиснутий (деякі файлові системи).
* `UF_TRACKED` 0x00000040: Немає сповіщень про видалення/перейменування для файлів з цим прапором.
* `UF_DATAVAULT` 0x00000080: Потрібно право для читання та запису.
* `UF_HIDDEN` 0x00008000: Підказка, що цей елемент не повинен відображатися в GUI.
* `SF_SUPPORTED` 0x009f0000: Маска підтримуваних прапорів суперкористувача.
* `SF_SETTABLE` 0x3fff0000: Маска змінних прапорів суперкористувача.
* `SF_SYNTHETIC` 0xc0000000: Маска системних синтетичних прапорів тільки для читання.
* `SF_ARCHIVED` 0x00010000: Файл архівований.
* `SF_IMMUTABLE` 0x00020000: Файл не може бути змінений.
* `SF_APPEND` 0x00040000: Записи у файл можуть лише додаватися.
* `SF_RESTRICTED` 0x00080000: Потрібно право для запису.
* `SF_NOUNLINK` 0x00100000: Елемент не може бути видалений, перейменований або змонтований.
* `SF_FIRMLINK` 0x00800000: Файл є firmlink.
* `SF_DATALESS` 0x40000000: Файл є об'єктом без даних.

### **File ACLs**

Файлові **ACLs** містять **ACE** (Записи контролю доступу), де можуть бути призначені більш **деталізовані дозволи** для різних користувачів.

Можна надати **директорії** ці дозволи: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
А для **файлу**: `read`, `write`, `append`, `execute`.

Коли файл містить ACLs, ви **знайдете "+" при переліку дозволів, як у**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Ви можете **прочитати ACL** файлу за допомогою:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Ви можете знайти **всі файли з ACL** за допомогою (це дуже повільно):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Розширені Атрибути

Розширені атрибути мають ім'я та будь-яке бажане значення, і їх можна переглядати за допомогою `ls -@` та маніпулювати за допомогою команди `xattr`. Деякі поширені розширені атрибути:

* `com.apple.resourceFork`: Сумісність з ресурсними fork. Також видно як `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: механізм карантину Gatekeeper (III/6)
* `metadata:*`: MacOS: різні метадані, такі як `_backup_excludeItem`, або `kMD*`
* `com.apple.lastuseddate` (#PS): Дата останнього використання файлу
* `com.apple.FinderInfo`: MacOS: інформація Finder (наприклад, кольорові мітки)
* `com.apple.TextEncoding`: Визначає кодування тексту файлів ASCII
* `com.apple.logd.metadata`: Використовується logd для файлів у `/var/db/diagnostics`
* `com.apple.genstore.*`: Генераційне зберігання (`/.DocumentRevisions-V100` в корені файлової системи)
* `com.apple.rootless`: MacOS: Використовується System Integrity Protection для маркування файлу (III/10)
* `com.apple.uuidb.boot-uuid`: маркування logd епох завантаження з унікальним UUID
* `com.apple.decmpfs`: MacOS: Прозора компресія файлів (II/7)
* `com.apple.cprotect`: \*OS: Дані шифрування для кожного файлу (III/11)
* `com.apple.installd.*`: \*OS: Метадані, що використовуються installd, наприклад, `installType`, `uniqueInstallID`

### Ресурсні Forks | macOS ADS

Це спосіб отримати **Альтернативні Потоки Даних у MacOS**. Ви можете зберігати вміст всередині розширеного атрибута під назвою **com.apple.ResourceFork** всередині файлу, зберігаючи його в **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Ви можете **знайти всі файли, що містять цей розширений атрибут** за допомогою: 

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Розширена атрибутика `com.apple.decmpfs` вказує на те, що файл зберігається в зашифрованому вигляді, `ls -l` повідомить про **розмір 0** і стиснуті дані знаходяться всередині цього атрибута. Кожного разу, коли файл відкривається, він буде розшифрований в пам'яті.

Цей атрибут можна побачити за допомогою `ls -lO`, вказаним як стиснутий, оскільки стиснуті файли також позначені прапором `UF_COMPRESSED`. Якщо стиснутий файл видалити з цього прапора за допомогою `chflags nocompressed </path/to/file>`, система не знатиме, що файл був стиснутий, і тому не зможе розпакувати та отримати доступ до даних (вона подумає, що він насправді порожній).

Інструмент afscexpand можна використовувати для примусового розпакування файлу.

## **Універсальні бінарні файли &** Формат Mach-o

Бінарні файли Mac OS зазвичай компілюються як **універсальні бінарні файли**. **Універсальний бінарний файл** може **підтримувати кілька архітектур в одному файлі**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS Процес Пам'яті

## Вивантаження пам'яті macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Категорії ризику файлів Mac OS

Каталог `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` є місцем, де зберігається інформація про **ризик, пов'язаний з різними розширеннями файлів**. Цей каталог класифікує файли за різними рівнями ризику, впливаючи на те, як Safari обробляє ці файли під час завантаження. Категорії такі:

* **LSRiskCategorySafe**: Файли в цій категорії вважаються **абсолютно безпечними**. Safari автоматично відкриє ці файли після їх завантаження.
* **LSRiskCategoryNeutral**: Ці файли не супроводжуються попередженнями і **не відкриваються автоматично** Safari.
* **LSRiskCategoryUnsafeExecutable**: Файли в цій категорії **викликають попередження**, що вказує на те, що файл є додатком. Це служить заходом безпеки для попередження користувача.
* **LSRiskCategoryMayContainUnsafeExecutable**: Ця категорія призначена для файлів, таких як архіви, які можуть містити виконуваний файл. Safari **викличе попередження**, якщо не зможе підтвердити, що всі вмісти безпечні або нейтральні.

## Лог файли

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Містить інформацію про завантажені файли, такі як URL, з якого вони були завантажені.
* **`/var/log/system.log`**: Основний журнал систем OSX. com.apple.syslogd.plist відповідає за виконання syslogging (ви можете перевірити, чи він вимкнений, шукаючи "com.apple.syslogd" у `launchctl list`).
* **`/private/var/log/asl/*.asl`**: Це журнали системи Apple, які можуть містити цікаву інформацію.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Зберігає нещодавно відкриті файли та програми через "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Зберігає елементи для запуску під час старту системи.
* **`$HOME/Library/Logs/DiskUtility.log`**: Журнал для програми DiskUtility (інформація про диски, включаючи USB).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Дані про бездротові точки доступу.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Список деактивованих демонів.

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
