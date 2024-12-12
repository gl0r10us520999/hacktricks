{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа взлому AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа взлому GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# Відмітки часу

Атакуючому може бути цікаво **змінити відмітки часу файлів**, щоб уникнути виявлення.\
Можна знайти відмітки часу всередині MFT в атрибутах `$STANDARD_INFORMATION` __ та __ `$FILE_NAME`.

Обидва атрибути мають 4 відмітки часу: **модифікація**, **доступ**, **створення** та **модифікація реєстру MFT** (MACE або MACB).

**Провідник Windows** та інші інструменти показують інформацію з **`$STANDARD_INFORMATION`**.

## TimeStomp - Анти-форензичний інструмент

Цей інструмент **змінює** інформацію про відмітки часу всередині **`$STANDARD_INFORMATION`** **але** **не** інформацію всередині **`$FILE_NAME`**. Тому можна **виявити** **підозрілу** **активність**.

## Usnjrnl

**Журнал USN** (Update Sequence Number Journal) - це функція файлової системи NTFS (Windows NT), яка відстежує зміни обсягу. Інструмент [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) дозволяє переглядати ці зміни.

![](<../../.gitbook/assets/image (449).png>)

Попереднє зображення - це **вихід**, показаний **інструментом**, де можна побачити, що до файлу було внесено **зміни**.

## $LogFile

**Усі зміни метаданих файлової системи реєструються** в процесі, відомому як [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead_logging). Зареєстровані метадані зберігаються в файлі під назвою `**$LogFile**`, розташованому в кореневому каталозі файлової системи NTFS. Інструменти, такі як [LogFileParser](https://github.com/jschicht/LogFileParser), можуть використовуватися для аналізу цього файлу та ідентифікації змін.

![](<../../.gitbook/assets/image (450).png>)

Знову ж таки, у виході інструменту можна побачити, що **було виконано деякі зміни**.

За допомогою цього ж інструменту можна визначити, **коли були змінені відмітки часу**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Час створення файлу
* ATIME: Час модифікації файлу
* MTIME: Модифікація реєстру MFT файлу
* RTIME: Час доступу до файлу

## Порівняння `$STANDARD_INFORMATION` та `$FILE_NAME`

Ще один спосіб виявлення підозрілих змінених файлів - порівняти час в обох атрибутах, шукаючи **неспівпадіння**.

## Наносекунди

Відмітки часу **NTFS** мають **точність** **100 наносекунд**. Тому знаходження файлів з відмітками часу, наприклад, 2010-10-10 10:10:**00.000:0000, є дуже підозрілим**.

## SetMace - Анти-форензичний інструмент

Цей інструмент може змінювати обидва атрибути `$STARNDAR_INFORMATION` та `$FILE_NAME`. Однак починаючи з Windows Vista, для зміни цієї інформації потрібна робоча ОС.

# Приховування даних

NFTS використовує кластер та мінімальний розмір інформації. Це означає, що якщо файл займає кластер і половину, **решта половина ніколи не буде використана**, поки файл не буде видалено. Тому можна **приховати дані в цьому вільному просторі**.

Існують інструменти, такі як slacker, які дозволяють приховувати дані в цьому "прихованому" просторі. Однак аналіз `$logfile` та `$usnjrnl` може показати, що деякі дані було додано:

![](<../../.gitbook/assets/image (452).png>)

Тому можна відновити вільний простір за допомогою інструментів, таких як FTK Imager. Зверніть увагу, що цей тип інструменту може зберігати вміст затемненим або навіть зашифрованим.

# UsbKill

Це інструмент, який **вимкне комп'ютер**, якщо виявлено будь-які зміни в портах **USB**.\
Щоб виявити це, слід перевірити запущені процеси та **переглянути кожен запущений сценарій Python**.

# Живі дистрибутиви Linux

Ці дистрибутиви **виконуються в оперативній пам'яті**. Єдиний спосіб виявити їх - **якщо файлова система NTFS монтується з правами на запис**. Якщо вона монтується лише з правами на читання, виявити вторгнення буде неможливо.

# Безпечне видалення

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Налаштування Windows

Можливо вимкнути кілька методів журналювання Windows, щоб ускладнити розслідування форензіки.

## Вимкнення відміток часу - UserAssist

Це реєстровий ключ, який зберігає дати та години, коли кожен виконуваний виконуваний файл користувачем.

Вимкнення UserAssist потребує двох кроків:

1. Встановіть два реєстрових ключа, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` та `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, обидва на нуль, щоб позначити, що ми хочемо вимкнути UserAssist.
2. Очистіть піддерева реєстру, які виглядають як `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<хеш>`.

## Вимкнення відміток часу - Prefetch

Це збереже інформацію про виконані програми з метою покращення продуктивності системи Windows. Однак це також може бути корисним для практик форензіки.

* Виконайте `regedit`
* Виберіть шлях файлу `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Клацніть правою кнопкою миші на `EnablePrefetcher` та `EnableSuperfetch`
* Виберіть Змінити для кожного з них, щоб змінити значення з 1 (або 3) на 0
* Перезавантажте

## Вимкнення відміток часу - Останній час доступу

Кожного разу, коли папка відкривається з тома NTFS на сервері Windows NT, система витрачає час на **оновлення полі відмітки часу на кожній перерахованій папці**, яке називається останнім часом доступу. На сильно використовуваному томі NTFS це може вплинути на продуктивність.

1. Відкрийте Редактор реєстру (Regedit.exe).
2. Перейдіть до `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Знайдіть `NtfsDisableLastAccessUpdate`. Якщо його немає, додайте цей DWORD та встановіть його значення на 1, що вимкне процес.
4. Закрийте Редактор реєстру та перезавантажте сервер.
## Видалення історії USB

Усі **записи пристроїв USB** зберігаються в реєстрі Windows під ключем реєстру **USBSTOR**, який містить підключені ключі, які створюються кожного разу, коли ви підключаєте пристрій USB до свого ПК або ноутбука. Ви можете знайти цей ключ тут `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Видалення цього** ключа призведе до видалення історії USB.\
Ви також можете скористатися інструментом [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html), щоб переконатися, що ви їх видалили (і видалити їх).

Ще один файл, який зберігає інформацію про USB, це файл `setupapi.dev.log` всередині `C:\Windows\INF`. Його також слід видалити.

## Вимкнення тіньових копій

**Вивести список** тіньових копій за допомогою `vssadmin list shadowstorage`\
**Видалити** їх, запустивши `vssadmin delete shadow`

Ви також можете видалити їх через GUI, виконавши кроки, запропоновані за посиланням [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Щоб вимкнути тіньові копії [кроки звідси](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows):

1. Відкрийте програму Служби, набравши "services" у текстовому полі пошуку після натискання кнопки запуску Windows.
2. Зі списку знайдіть "Тіньову копію тому", виберіть її, а потім отримайте доступ до властивостей, клацнувши правою кнопкою миші.
3. Виберіть "Вимкнено" зі списку випадаючих меню "Тип запуску", а потім підтвердіть зміну, клацнувши "Застосувати" і "OK".

Також можливо змінити конфігурацію файлів, які будуть скопійовані в тіньову копію, в реєстрі `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

## Перезапис видалених файлів

* Ви можете скористатися **інструментом Windows**: `cipher /w:C`. Це вказує cipher видалити будь-які дані з вільного місця на диску в диску C.
* Ви також можете скористатися інструментами, такими як [**Eraser**](https://eraser.heidi.ie)

## Видалення журналів подій Windows

* Windows + R --> eventvwr.msc --> Розгорнути "Журнали Windows" --> Клацніть правою кнопкою миші на кожній категорії та виберіть "Очистити журнал"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Вимкнення журналів подій Windows

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* У розділі служб вимкніть службу "Журнал подій Windows"
* `WEvtUtil.exec clear-log` або `WEvtUtil.exe cl`

## Вимкнення $UsnJrnl

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}
