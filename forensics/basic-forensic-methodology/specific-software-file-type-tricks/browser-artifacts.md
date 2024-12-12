# Browser Artifacts

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

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Browsers Artifacts <a href="#id-3def" id="id-3def"></a>

Артефакти браузера включають різні типи даних, збережених веб-браузерами, такі як історія навігації, закладки та кешовані дані. Ці артефакти зберігаються в специфічних папках в операційній системі, які відрізняються за місцем розташування та назвою в різних браузерах, але зазвичай зберігають подібні типи даних.

Ось короткий огляд найпоширеніших артефактів браузера:

* **Історія навігації**: Відстежує відвідування користувачем веб-сайтів, корисно для ідентифікації відвідувань шкідливих сайтів.
* **Дані автозаповнення**: Пропозиції на основі частих пошуків, що надають інформацію в поєднанні з історією навігації.
* **Закладки**: Сайти, збережені користувачем для швидкого доступу.
* **Розширення та додатки**: Розширення браузера або додатки, встановлені користувачем.
* **Кеш**: Зберігає веб-контент (наприклад, зображення, файли JavaScript), щоб покращити час завантаження веб-сайтів, цінно для судової експертизи.
* **Логіни**: Збережені облікові дані для входу.
* **Фавіконки**: Іконки, пов'язані з веб-сайтами, що з'являються на вкладках і в закладках, корисні для додаткової інформації про відвідування користувача.
* **Сесії браузера**: Дані, пов'язані з відкритими сесіями браузера.
* **Завантаження**: Записи файлів, завантажених через браузер.
* **Дані форм**: Інформація, введена у веб-форму, збережена для майбутніх пропозицій автозаповнення.
* **Ескізи**: Попередні зображення веб-сайтів.
* **Custom Dictionary.txt**: Слова, додані користувачем до словника браузера.

## Firefox

Firefox організовує дані користувача в профілях, які зберігаються в специфічних місцях залежно від операційної системи:

* **Linux**: `~/.mozilla/firefox/`
* **MacOS**: `/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**: `%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

Файл `profiles.ini` в цих каталогах містить список профілів користувача. Дані кожного профілю зберігаються в папці, назва якої вказана в змінній `Path` у `profiles.ini`, розташованій в тому ж каталозі, що й `profiles.ini`. Якщо папка профілю відсутня, вона могла бути видалена.

У кожній папці профілю ви можете знайти кілька важливих файлів:

* **places.sqlite**: Зберігає історію, закладки та завантаження. Інструменти, такі як [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) на Windows, можуть отримати доступ до даних історії.
* Використовуйте специфічні SQL-запити для витягнення інформації про історію та завантаження.
* **bookmarkbackups**: Містить резервні копії закладок.
* **formhistory.sqlite**: Зберігає дані веб-форм.
* **handlers.json**: Керує обробниками протоколів.
* **persdict.dat**: Слова з користувацького словника.
* **addons.json** та **extensions.sqlite**: Інформація про встановлені додатки та розширення.
* **cookies.sqlite**: Зберігання куків, з [MZCookiesView](https://www.nirsoft.net/utils/mzcv.html) доступним для перевірки на Windows.
* **cache2/entries** або **startupCache**: Дані кешу, доступні через інструменти, такі як [MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html).
* **favicons.sqlite**: Зберігає фавіконки.
* **prefs.js**: Налаштування та переваги користувача.
* **downloads.sqlite**: Стара база даних завантажень, тепер інтегрована в places.sqlite.
* **thumbnails**: Ескізи веб-сайтів.
* **logins.json**: Зашифрована інформація для входу.
* **key4.db** або **key3.db**: Зберігає ключі шифрування для захисту чутливої інформації.

Крім того, перевірити налаштування антифішингу браузера можна, шукаючи записи `browser.safebrowsing` у `prefs.js`, що вказує, чи увімкнені або вимкнені функції безпечного перегляду.

Щоб спробувати розшифрувати майстер-пароль, ви можете використовувати [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt)\
З наступним скриптом і викликом ви можете вказати файл паролів для брутфорсу:

{% code title="brute.sh" %}
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
{% endcode %}

![](<../../../.gitbook/assets/image (417).png>)

## Google Chrome

Google Chrome зберігає профілі користувачів у специфічних місцях залежно від операційної системи:

* **Linux**: `~/.config/google-chrome/`
* **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
* **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

У цих каталогах більшість даних користувача можна знайти у папках **Default/** або **ChromeDefaultData/**. Наступні файли містять значні дані:

* **History**: Містить URL-адреси, завантаження та ключові слова пошуку. На Windows можна використовувати [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) для читання історії. Стовпець "Transition Type" має різні значення, включаючи кліки користувача на посилання, введені URL-адреси, відправлення форм та перезавантаження сторінок.
* **Cookies**: Зберігає куки. Для перевірки доступний [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html).
* **Cache**: Містить кешовані дані. Для перевірки користувачі Windows можуть використовувати [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html).
* **Bookmarks**: Закладки користувача.
* **Web Data**: Містить історію форм.
* **Favicons**: Зберігає фавіконки веб-сайтів.
* **Login Data**: Включає облікові дані для входу, такі як імена користувачів та паролі.
* **Current Session**/**Current Tabs**: Дані про поточну сесію перегляду та відкриті вкладки.
* **Last Session**/**Last Tabs**: Інформація про сайти, активні під час останньої сесії перед закриттям Chrome.
* **Extensions**: Каталоги для розширень браузера та аддонів.
* **Thumbnails**: Зберігає ескізи веб-сайтів.
* **Preferences**: Файл, багатий на інформацію, включаючи налаштування для плагінів, розширень, спливаючих вікон, сповіщень та інше.
* **Browser’s built-in anti-phishing**: Щоб перевірити, чи увімкнено захист від фішингу та шкідливого ПЗ, виконайте `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`. Шукайте `{"enabled: true,"}` у виході.

## **Відновлення даних SQLite DB**

Як видно з попередніх розділів, як Chrome, так і Firefox використовують **SQLite** бази даних для зберігання даних. Можливо **відновити видалені записи за допомогою інструменту** [**sqlparse**](https://github.com/padfoot999/sqlparse) **або** [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases).

## **Internet Explorer 11**

Internet Explorer 11 управляє своїми даними та метаданими в різних місцях, що допомагає розділити збережену інформацію та відповідні деталі для легкого доступу та управління.

### Зберігання метаданих

Метадані для Internet Explorer зберігаються в `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` (з VX, що може бути V01, V16 або V24). У супроводі цього файл `V01.log` може показувати розбіжності в часі модифікації з `WebcacheVX.data`, що вказує на необхідність ремонту за допомогою `esentutl /r V01 /d`. Ці метадані, що зберігаються в базі даних ESE, можна відновити та перевірити за допомогою таких інструментів, як photorec та [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html). У таблиці **Containers** можна розрізнити конкретні таблиці або контейнери, де зберігається кожен сегмент даних, включаючи деталі кешу для інших інструментів Microsoft, таких як Skype.

### Перевірка кешу

Інструмент [IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) дозволяє перевіряти кеш, вимагаючи місцезнаходження папки для витягування даних кешу. Метадані для кешу включають ім'я файлу, каталог, кількість доступів, URL-адресу походження та часові мітки, що вказують на час створення, доступу, модифікації та закінчення терміну дії кешу.

### Управління куками

Куки можна досліджувати за допомогою [IECookiesView](https://www.nirsoft.net/utils/iecookies.html), з метаданими, що охоплюють імена, URL-адреси, кількість доступів та різні часові деталі. Постійні куки зберігаються в `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies`, а сесійні куки знаходяться в пам'яті.

### Деталі завантажень

Метадані завантажень доступні через [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html), з конкретними контейнерами, що містять дані, такі як URL, тип файлу та місце завантаження. Фізичні файли можна знайти в `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory`.

### Історія переглядів

Щоб переглянути історію переглядів, можна використовувати [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html), вимагаючи місцезнаходження витягнутих файлів історії та конфігурації для Internet Explorer. Метадані тут включають час модифікації та доступу, а також кількість доступів. Файли історії розташовані в `%userprofile%\Appdata\Local\Microsoft\Windows\History`.

### Введені URL-адреси

Введені URL-адреси та час їх використання зберігаються в реєстрі під `NTUSER.DAT` за адресами `Software\Microsoft\InternetExplorer\TypedURLs` та `Software\Microsoft\InternetExplorer\TypedURLsTime`, відстежуючи останні 50 URL-адрес, введених користувачем, та час їх останнього введення.

## Microsoft Edge

Microsoft Edge зберігає дані користувачів у `%userprofile%\Appdata\Local\Packages`. Шляхи для різних типів даних:

* **Profile Path**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
* **History, Cookies, and Downloads**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
* **Settings, Bookmarks, and Reading List**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
* **Cache**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
* **Last Active Sessions**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Дані Safari зберігаються за адресою `/Users/$User/Library/Safari`. Ключові файли включають:

* **History.db**: Містить таблиці `history_visits` та `history_items` з URL-адресами та часовими мітками відвідувань. Використовуйте `sqlite3` для запитів.
* **Downloads.plist**: Інформація про завантажені файли.
* **Bookmarks.plist**: Зберігає закладені URL-адреси.
* **TopSites.plist**: Найчастіше відвідувані сайти.
* **Extensions.plist**: Список розширень браузера Safari. Використовуйте `plutil` або `pluginkit` для отримання.
* **UserNotificationPermissions.plist**: Доменні імена, яким дозволено надсилати сповіщення. Використовуйте `plutil` для парсингу.
* **LastSession.plist**: Вкладки з останньої сесії. Використовуйте `plutil` для парсингу.
* **Browser’s built-in anti-phishing**: Перевірте за допомогою `defaults read com.apple.Safari WarnAboutFraudulentWebsites`. Відповідь 1 вказує на те, що функція активна.

## Opera

Дані Opera знаходяться в `/Users/$USER/Library/Application Support/com.operasoftware.Opera` і використовують формат Chrome для історії та завантажень.

* **Browser’s built-in anti-phishing**: Перевірте, чи `fraud_protection_enabled` у файлі Preferences встановлено на `true`, використовуючи `grep`.

Ці шляхи та команди є важливими для доступу та розуміння даних перегляду, збережених різними веб-браузерами.

## References

* [https://nasbench.medium.com/web-browsers-forensics-7e99940c579a](https://nasbench.medium.com/web-browsers-forensics-7e99940c579a)
* [https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/](https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/)
* [https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file](https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file)
* **Книга: OS X Incident Response: Scripting and Analysis By Jaron Bradley сторінка 123**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкого створення та **автоматизації робочих процесів**, підтримуваних найсучаснішими інструментами спільноти.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% hint style="success" %}
Вчіться та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вчіться та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
