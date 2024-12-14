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

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# Timestamps

Зловмисник може бути зацікавлений у **зміні часових міток файлів**, щоб уникнути виявлення.\
Можна знайти часові мітки всередині MFT в атрибутах `$STANDARD_INFORMATION` __ та __ `$FILE_NAME`.

Обидва атрибути мають 4 часові мітки: **Зміна**, **доступ**, **створення** та **зміна реєстрації MFT** (MACE або MACB).

**Windows explorer** та інші інструменти показують інформацію з **`$STANDARD_INFORMATION`**.

## TimeStomp - Anti-forensic Tool

Цей інструмент **модифікує** інформацію про часові мітки всередині **`$STANDARD_INFORMATION`**, **але** **не** інформацію всередині **`$FILE_NAME`**. Тому можливо **виявити** **підозрілу** **активність**.

## Usnjrnl

**USN Journal** (Журнал номерів послідовності оновлень) є функцією NTFS (файлова система Windows NT), яка відстежує зміни обсягу. Інструмент [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) дозволяє досліджувати ці зміни.

![](<../../.gitbook/assets/image (449).png>)

Попереднє зображення є **виходом**, показаним **інструментом**, де можна спостерігати, що деякі **зміни були виконані** до файлу.

## $LogFile

**Всі зміни метаданих файлової системи реєструються** в процесі, відомому як [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead_logging). Зареєстровані метадані зберігаються у файлі з назвою `**$LogFile**`, розташованому в кореневому каталозі файлової системи NTFS. Інструменти, такі як [LogFileParser](https://github.com/jschicht/LogFileParser), можуть бути використані для парсингу цього файлу та виявлення змін.

![](<../../.gitbook/assets/image (450).png>)

Знову ж таки, у виході інструмента можна побачити, що **деякі зміни були виконані**.

Використовуючи той же інструмент, можна визначити, **коли були змінені часові мітки**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Час створення файлу
* ATIME: Час модифікації файлу
* MTIME: Зміна реєстрації MFT файлу
* RTIME: Час доступу до файлу

## Порівняння `$STANDARD_INFORMATION` та `$FILE_NAME`

Ще один спосіб виявити підозрілі модифіковані файли - це порівняти час на обох атрибутах, шукаючи **невідповідності**.

## Наносекунди

**Часові мітки NTFS** мають **точність** **100 наносекунд**. Тому знаходження файлів з часовими мітками, такими як 2010-10-10 10:10:**00.000:0000, є дуже підозрілим**.

## SetMace - Anti-forensic Tool

Цей інструмент може модифікувати обидва атрибути `$STARNDAR_INFORMATION` та `$FILE_NAME`. Однак, починаючи з Windows Vista, необхідно, щоб ОС була активною для зміни цієї інформації.

# Data Hiding

NFTS використовує кластер і мінімальний розмір інформації. Це означає, що якщо файл займає кластер і півтора, то **залишкова половина ніколи не буде використана** до тих пір, поки файл не буде видалено. Тоді можливо **сховати дані в цьому слек-просторі**.

Існують інструменти, такі як slacker, які дозволяють ховати дані в цьому "прихованому" просторі. Однак аналіз `$logfile` та `$usnjrnl` може показати, що деякі дані були додані:

![](<../../.gitbook/assets/image (452).png>)

Тоді можливо відновити слек-простір, використовуючи інструменти, такі як FTK Imager. Зверніть увагу, що цей тип інструменту може зберігати вміст в обфусцированому або навіть зашифрованому вигляді.

# UsbKill

Це інструмент, який **вимкне комп'ютер, якщо буде виявлено будь-які зміни в USB** портах.\
Спосіб виявлення цього - перевірити запущені процеси та **переглянути кожен запущений python-скрипт**.

# Live Linux Distributions

Ці дистрибутиви **виконуються в пам'яті RAM**. Єдиний спосіб виявити їх - **якщо файлову систему NTFS змонтовано з правами на запис**. Якщо вона змонтована лише з правами на читання, виявити вторгнення не вдасться.

# Secure Deletion

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Windows Configuration

Можливо відключити кілька методів ведення журналів Windows, щоб ускладнити розслідування.

## Disable Timestamps - UserAssist

Це ключ реєстру, який зберігає дати та години, коли кожен виконуваний файл був запущений користувачем.

Вимкнення UserAssist вимагає двох кроків:

1. Встановіть два ключі реєстру, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` та `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, обидва на нуль, щоб сигналізувати, що ми хочемо вимкнути UserAssist.
2. Очистіть свої піддерева реєстру, які виглядають як `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

## Disable Timestamps - Prefetch

Це зберігатиме інформацію про програми, які виконувалися з метою покращення продуктивності системи Windows. Однак це також може бути корисним для судово-медичних практик.

* Виконайте `regedit`
* Виберіть шлях до файлу `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Клацніть правою кнопкою миші на `EnablePrefetcher` та `EnableSuperfetch`
* Виберіть Змінити для кожного з них, щоб змінити значення з 1 (або 3) на 0
* Перезавантажте

## Disable Timestamps - Last Access Time

Кожного разу, коли папка відкривається з обсягу NTFS на сервері Windows NT, система витрачає час на **оновлення поля часової мітки для кожної вказаної папки**, яке називається часом останнього доступу. На сильно використовуваному обсязі NTFS це може вплинути на продуктивність.

1. Відкрийте Редактор реєстру (Regedit.exe).
2. Перейдіть до `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Знайдіть `NtfsDisableLastAccessUpdate`. Якщо його немає, додайте цей DWORD і встановіть його значення на 1, що вимкне процес.
4. Закрийте Редактор реєстру та перезавантажте сервер.

## Delete USB History

Всі **USB Device Entries** зберігаються в реєстрі Windows під ключем **USBSTOR**, який містить підключі, які створюються щоразу, коли ви підключаєте USB-пристрій до свого ПК або ноутбука. Ви можете знайти цей ключ тут H`KEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Видаливши це**, ви видалите історію USB.\
Ви також можете використовувати інструмент [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html), щоб переконатися, що ви їх видалили (і щоб видалити їх).

Ще один файл, який зберігає інформацію про USB, - це файл `setupapi.dev.log` всередині `C:\Windows\INF`. Цей файл також слід видалити.

## Disable Shadow Copies

**Список** тіньових копій за допомогою `vssadmin list shadowstorage`\
**Видалити** їх, запустивши `vssadmin delete shadow`

Ви також можете видалити їх через GUI, дотримуючись кроків, запропонованих у [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Щоб вимкнути тіньові копії, [кроки звідси](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows):

1. Відкрийте програму Служби, ввівши "services" у текстовому полі пошуку після натискання кнопки "Пуск" Windows.
2. У списку знайдіть "Volume Shadow Copy", виберіть його, а потім отримайте доступ до Властивостей, клацнувши правою кнопкою миші.
3. Виберіть Вимкнено з випадаючого меню "Тип запуску", а потім підтвердіть зміну, натиснувши Застосувати та ОК.

Також можливо змінити конфігурацію, які файли будуть копіюватися в тіньовій копії в реєстрі `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

## Overwrite deleted files

* Ви можете використовувати **інструмент Windows**: `cipher /w:C` Це вказує шифрувати, щоб видалити будь-які дані з доступного невикористаного дискового простору всередині диска C.
* Ви також можете використовувати інструменти, такі як [**Eraser**](https://eraser.heidi.ie)

## Delete Windows event logs

* Windows + R --> eventvwr.msc --> Розгорніть "Windows Logs" --> Клацніть правою кнопкою миші на кожній категорії та виберіть "Очистити журнал"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Disable Windows event logs

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* У розділі служб вимкніть службу "Windows Event Log"
* `WEvtUtil.exec clear-log` або `WEvtUtil.exe cl`

## Disable $UsnJrnl

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


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
