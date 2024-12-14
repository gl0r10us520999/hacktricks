# Checklist - Локальне підвищення привілеїв Windows

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

### **Найкращий інструмент для пошуку векторів підвищення привілеїв Windows:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [Інформація про систему](windows-local-privilege-escalation/#system-info)

* [ ] Отримати [**інформацію про систему**](windows-local-privilege-escalation/#system-info)
* [ ] Шукати **експлойти ядра** [**за допомогою скриптів**](windows-local-privilege-escalation/#version-exploits)
* [ ] Використовувати **Google для пошуку** експлойтів ядра
* [ ] Використовувати **searchsploit для пошуку** експлойтів ядра
* [ ] Цікава інформація в [**змінних середовища**](windows-local-privilege-escalation/#environment)?
* [ ] Паролі в [**історії PowerShell**](windows-local-privilege-escalation/#powershell-history)?
* [ ] Цікава інформація в [**налаштуваннях Інтернету**](windows-local-privilege-escalation/#internet-settings)?
* [ ] [**Диски**](windows-local-privilege-escalation/#drives)?
* [ ] [**Експлойт WSUS**](windows-local-privilege-escalation/#wsus)?
* [ ] [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)?

### [Перерахування журналів/AV](windows-local-privilege-escalation/#enumeration)

* [ ] Перевірте [**налаштування аудиту**](windows-local-privilege-escalation/#audit-settings) та [**WEF**](windows-local-privilege-escalation/#wef)
* [ ] Перевірте [**LAPS**](windows-local-privilege-escalation/#laps)
* [ ] Перевірте, чи активний [**WDigest**](windows-local-privilege-escalation/#wdigest)
* [ ] [**Захист LSA**](windows-local-privilege-escalation/#lsa-protection)?
* [ ] [**Credentials Guard**](windows-local-privilege-escalation/#credentials-guard)[?](windows-local-privilege-escalation/#cached-credentials)
* [ ] [**Кешовані облікові дані**](windows-local-privilege-escalation/#cached-credentials)?
* [ ] Перевірте, чи є [**AV**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/windows-av-bypass/README.md)
* [ ] [**Політика AppLocker**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/README.md#applocker-policy)?
* [ ] [**UAC**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control/README.md)
* [ ] [**Привілеї користувачів**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Перевірте [**поточні** привілеї **користувача**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Чи є ви [**членом будь-якої привілейованої групи**](windows-local-privilege-escalation/#privileged-groups)?
* [ ] Перевірте, чи є у вас [будь-які з цих токенів, активованих](windows-local-privilege-escalation/#token-manipulation): **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege** ?
* [ ] [**Сесії користувачів**](windows-local-privilege-escalation/#logged-users-sessions)?
* [ ] Перевірте [**домашні папки користувачів**](windows-local-privilege-escalation/#home-folders) (доступ?)
* [ ] Перевірте [**Політику паролів**](windows-local-privilege-escalation/#password-policy)
* [ ] Що [**всередині буфера обміну**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)?

### [Мережа](windows-local-privilege-escalation/#network)

* [ ] Перевірте **поточну** [**мережеву** **інформацію**](windows-local-privilege-escalation/#network)
* [ ] Перевірте **приховані локальні служби**, обмежені для зовнішнього доступу

### [Запущені процеси](windows-local-privilege-escalation/#running-processes)

* [ ] Бінарні файли процесів [**дозволи на файли та папки**](windows-local-privilege-escalation/#file-and-folder-permissions)
* [ ] [**Видобуток паролів з пам'яті**](windows-local-privilege-escalation/#memory-password-mining)
* [ ] [**Небезпечні GUI додатки**](windows-local-privilege-escalation/#insecure-gui-apps)
* [ ] Вкрасти облікові дані з **цікавих процесів** за допомогою `ProcDump.exe` ? (firefox, chrome тощо ...)

### [Служби](windows-local-privilege-escalation/#services)

* [ ] [Чи можете ви **модифікувати будь-яку службу**?](windows-local-privilege-escalation/#permissions)
* [ ] [Чи можете ви **модифікувати** **бінарний файл**, який **виконується** будь-якою **службою**?](windows-local-privilege-escalation/#modify-service-binary-path)
* [ ] [Чи можете ви **модифікувати** **реєстр** будь-якої **служби**?](windows-local-privilege-escalation/#services-registry-modify-permissions)
* [ ] [Чи можете ви скористатися будь-яким **нецитованим шляхом** бінарного файлу **служби**?](windows-local-privilege-escalation/#unquoted-service-paths)

### [**Додатки**](windows-local-privilege-escalation/#applications)

* [ ] **Дозволи на запис** [**встановлені додатки**](windows-local-privilege-escalation/#write-permissions)
* [ ] [**Додатки автозавантаження**](windows-local-privilege-escalation/#run-at-startup)
* [ ] **Вразливі** [**драйвери**](windows-local-privilege-escalation/#drivers)

### [DLL Hijacking](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] Чи можете ви **записувати в будь-яку папку всередині PATH**?
* [ ] Чи є відомий бінарний файл служби, який **намагається завантажити будь-який неіснуючий DLL**?
* [ ] Чи можете ви **записувати** в будь-яку **папку з бінарними файлами**?

### [Мережа](windows-local-privilege-escalation/#network)

* [ ] Перерахувати мережу (спільні ресурси, інтерфейси, маршрути, сусіди тощо ...)
* [ ] Уважно перевірте мережеві служби, які слухають на localhost (127.0.0.1)

### [Облікові дані Windows](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Облікові дані Winlogon**](windows-local-privilege-escalation/#winlogon-credentials)
* [ ] [**Облікові дані Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault), які ви могли б використовувати?
* [ ] Цікаві [**облікові дані DPAPI**](windows-local-privilege-escalation/#dpapi)?
* [ ] Паролі збережених [**Wifi мереж**](windows-local-privilege-escalation/#wifi)?
* [ ] Цікава інформація в [**збережених RDP з'єднаннях**](windows-local-privilege-escalation/#saved-rdp-connections)?
* [ ] Паролі в [**недавніх командах**](windows-local-privilege-escalation/#recently-run-commands)?
* [ ] [**Менеджер облікових даних віддаленого робочого столу**](windows-local-privilege-escalation/#remote-desktop-credential-manager) паролі?
* [ ] [**AppCmd.exe** існує](windows-local-privilege-escalation/#appcmd-exe)? Облікові дані?
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)? Завантаження DLL з боку?

### [Файли та реєстр (Облікові дані)](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**Облікові дані**](windows-local-privilege-escalation/#putty-creds) **та** [**SSH ключі хоста**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] [**SSH ключі в реєстрі**](windows-local-privilege-escalation/#ssh-keys-in-registry)?
* [ ] Паролі в [**непідконтрольних файлах**](windows-local-privilege-escalation/#unattended-files)?
* [ ] Будь-яка [**резервна копія SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups)?
* [ ] [**Облікові дані хмари**](windows-local-privilege-escalation/#cloud-credentials)?
* [ ] [**Файл McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml)?
* [ ] [**Кешований GPP пароль**](windows-local-privilege-escalation/#cached-gpp-pasword)?
* [ ] Пароль у [**файлі конфігурації IIS Web**](windows-local-privilege-escalation/#iis-web-config)?
* [ ] Цікава інформація в [**веб** **журналах**](windows-local-privilege-escalation/#logs)?
* [ ] Чи хочете ви [**попросити облікові дані**](windows-local-privilege-escalation/#ask-for-credentials) у користувача?
* [ ] Цікаві [**файли всередині Кошика**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)?
* [ ] Інші [**реєстри, що містять облікові дані**](windows-local-privilege-escalation/#inside-the-registry)?
* [ ] Всередині [**даних браузера**](windows-local-privilege-escalation/#browsers-history) (бази даних, історія, закладки тощо)?
* [ ] [**Загальний пошук паролів**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry) у файлах та реєстрі
* [ ] [**Інструменти**](windows-local-privilege-escalation/#tools-that-search-for-passwords) для автоматичного пошуку паролів

### [Витік обробників](windows-local-privilege-escalation/#leaked-handlers)

* [ ] Чи маєте ви доступ до будь-якого обробника процесу, запущеного адміністратором?

### [Імітація клієнта Pipe](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] Перевірте, чи можете ви це зловживати

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
