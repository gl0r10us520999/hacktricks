{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}


## smss.exe

**Менеджер сеансів**.\
Сеанс 0 запускає **csrss.exe** та **wininit.exe** (**служби ОС**), тоді як сеанс 1 запускає **csrss.exe** та **winlogon.exe** (**сеанс користувача**). Однак ви повинні бачити **лише один процес** цього **виконуваного файлу без дочірніх процесів у дереві процесів**.

Також, сеанси, відмінні від 0 та 1, можуть означати, що відбуваються сеанси RDP.


## csrss.exe

**Процес підсистеми клієнт-сервер**.\
Він керує **процесами** та **потоками**, робить **Windows API** доступним для інших процесів, а також **відображає літери диска**, створює **тимчасові файли** та обробляє **процес завершення роботи**.

Є один **запущений у Сеансі 0 та ще один у Сеансі 1** (тобто **2 процеси** у дереві процесів). Ще один створюється **для кожного нового сеансу**.


## winlogon.exe

**Процес входу в Windows**.\
Він відповідає за вхід/вихід користувача. Запускає **logonui.exe**, щоб запросити ім'я користувача та пароль, а потім викликає **lsass.exe**, щоб їх перевірити.

Потім він запускає **userinit.exe**, який вказаний у **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** з ключем **Userinit**.

Крім того, попередній реєстр повинен містити **explorer.exe** у ключі **Shell**, інакше його можна використовувати як **метод постійства шкідливого ПЗ**.


## wininit.exe

**Процес ініціалізації Windows**. \
Запускає **services.exe**, **lsass.exe** та **lsm.exe** у Сеансі 0. Повинен бути лише 1 процес.


## userinit.exe

**Додаток входу користувача**.\
Завантажує **ntduser.dat у HKCU** та ініціалізує **середовище користувача** та запускає **скрипти входу** та **GPO**.

Запускає **explorer.exe**.


## lsm.exe

**Локальний менеджер сеансів**.\
Працює з smss.exe для маніпулювання користувацькими сеансами: вхід/вихід, запуск оболонки, блокування/розблокування робочого столу тощо.

Після W7 lsm.exe був перетворений на службу (lsm.dll).

Повинен бути лише 1 процес у W7, а з них служба, що запускає DLL.


## services.exe

**Менеджер керування службами**.\
Він **завантажує** **служби**, налаштовані як **автозапуск**, та **драйвери**.

Це батьківський процес для **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** та багатьох інших.

Служби визначаються в `HKLM\SYSTEM\CurrentControlSet\Services`, і цей процес підтримує базу даних у пам'яті з інформацією про служби, яку можна опитати за допомогою sc.exe.

Зверніть увагу, що **деякі** **служби будуть працювати у власному процесі**, а інші **будуть використовувати процес svchost.exe**.

Повинен бути лише 1 процес.


## lsass.exe

**Підсистема локальної безпеки авторитету**.\
Він відповідає за **аутентифікацію користувача** та створення **токенів безпеки**. Він використовує пакети аутентифікації, розташовані в `HKLM\System\CurrentControlSet\Control\Lsa`.

Він записує в **журнал подій безпеки**, і повинен бути лише 1 процес.

Пам'ятайте, що цей процес часто атакується для вилучення паролів.


## svchost.exe

**Загальний процес хостингу служб**.\
Він розміщує кілька служб DLL у одному спільному процесі.

Зазвичай ви побачите, що **svchost.exe** запускається з прапорцем `-k`. Це запустить запит до реєстру **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost**, де буде ключ з вказаним аргументом у `-k`, який міститиме служби для запуску в одному процесі.

Наприклад: `-k UnistackSvcGroup` запустить: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Якщо також використовується **прапорець `-s`** з аргументом, то svchost запитується запустити **лише вказану службу** у цьому аргументі.

Буде кілька процесів `svchost.exe`. Якщо який-небудь з них **не використовує прапорець `-k`**, це дуже підозріло. Якщо ви виявите, що **services.exe не є батьківським процесом**, це також дуже підозрливо.


## taskhost.exe

Цей процес діє як хост для процесів, що працюють з DLL. Він також завантажує служби, які працюють з DLL.

У W8 це називається taskhostex.exe, а у W10 - taskhostw.exe.


## explorer.exe

Цей процес відповідає за **робочий стіл користувача** та запуск файлів через розширення файлів.

**Лише 1** процес повинен бути створений **на кожного ввійшовшого користувача.**

Цей процес запускається з **userinit.exe**, який повинен бути завершений, тому **батьківський процес** для цього процесу не повинен з'являтися.


# Виявлення шкідливих процесів

* Чи він запущений з очікуваного шляху? (Жоден виконуваний файл Windows не запускається з тимчасового місця розташування)
* Чи він спілкується з дивними IP-адресами?
* Перевірте цифрові підписи (артефакти Microsoft повинні бути підписані)
* Чи правильно написано?
* Чи виконується під очікуваним SID?
* Чи батьківський процес є очікуваним (якщо є)?
* Чи дочірні процеси є очікуваними? (немає cmd.exe, wscript.exe, powershell.exe..?)


{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
