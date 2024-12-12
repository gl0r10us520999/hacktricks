# Active Directory Methodology

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

## Basic overview

**Active Directory** служить основною технологією, що дозволяє **мережевим адміністраторам** ефективно створювати та керувати **доменами**, **користувачами** та **об'єктами** в межах мережі. Вона спроектована для масштабування, полегшуючи організацію великої кількості користувачів у керовані **групи** та **підгрупи**, контролюючи **права доступу** на різних рівнях.

Структура **Active Directory** складається з трьох основних рівнів: **домени**, **дерева** та **ліси**. **Домен** охоплює колекцію об'єктів, таких як **користувачі** або **пристрої**, які ділять спільну базу даних. **Дерева** — це групи цих доменів, пов'язані спільною структурою, а **ліс** представляє колекцію кількох дерев, з'єднаних через **довірчі відносини**, формуючи найвищий рівень організаційної структури. Специфічні **права доступу** та **права на спілкування** можуть бути призначені на кожному з цих рівнів.

Ключові концепції в **Active Directory** включають:

1. **Каталог** – Містить всю інформацію, що стосується об'єктів Active Directory.
2. **Об'єкт** – Позначає сутності в каталозі, включаючи **користувачів**, **групи** або **спільні папки**.
3. **Домен** – Служить контейнером для об'єктів каталогу, з можливістю для кількох доменів співіснувати в межах **лісу**, кожен з яких підтримує свою власну колекцію об'єктів.
4. **Дерево** – Групування доменів, які ділять спільний кореневий домен.
5. **Ліс** – Вершина організаційної структури в Active Directory, що складається з кількох дерев з **довірчими відносинами** між ними.

**Active Directory Domain Services (AD DS)** охоплює ряд послуг, критично важливих для централізованого управління та спілкування в межах мережі. Ці послуги включають:

1. **Послуги домену** – Централізує зберігання даних і керує взаємодією між **користувачами** та **доменами**, включаючи функції **автентифікації** та **пошуку**.
2. **Послуги сертифікатів** – Контролює створення, розподіл та управління безпечними **цифровими сертифікатами**.
3. **Легкі служби каталогу** – Підтримує програми, що використовують каталог, через **протокол LDAP**.
4. **Служби федерації каталогу** – Надає можливості **єдиного входу** для автентифікації користувачів через кілька веб-додатків в одній сесії.
5. **Управління правами** – Допомагає захистити авторські матеріали, регулюючи їх несанкціоноване розповсюдження та використання.
6. **Служба DNS** – Критично важлива для розв'язання **доменних імен**.

Для більш детального пояснення перегляньте: [**TechTerms - Визначення Active Directory**](https://techterms.com/definition/active\_directory)

### **Kerberos Authentication**

Щоб дізнатися, як **атакувати AD**, вам потрібно **дуже добре розуміти** **процес автентифікації Kerberos**.\
[**Прочитайте цю сторінку, якщо ви ще не знаєте, як це працює.**](kerberos-authentication.md)

## Cheat Sheet

Ви можете відвідати [https://wadcoms.github.io/](https://wadcoms.github.io), щоб швидко ознайомитися з командами, які ви можете виконати для перерахунку/експлуатації AD.

## Recon Active Directory (No creds/sessions)

Якщо у вас є доступ до середовища AD, але немає жодних облікових даних/сесій, ви можете:

* **Pentest the network:**
* Сканувати мережу, знаходити машини та відкриті порти та намагатися **експлуатувати вразливості** або **витягувати облікові дані** з них (наприклад, [принтери можуть бути дуже цікавими цілями](ad-information-in-printers.md)).
* Перерахунок DNS може надати інформацію про ключові сервери в домені, такі як веб, принтери, спільні ресурси, vpn, медіа тощо.
* `gobuster dns -d domain.local -t 25 -w /opt/Seclist/Discovery/DNS/subdomain-top2000.txt`
* Ознайомтеся з Загальною [**Методологією Pentesting**](../../generic-methodologies-and-resources/pentesting-methodology.md), щоб знайти більше інформації про те, як це зробити.
* **Перевірте наявність доступу null та Guest на smb-сервісах** (це не спрацює на сучасних версіях Windows):
* `enum4linux -a -u "" -p "" <DC IP> && enum4linux -a -u "guest" -p "" <DC IP>`
* `smbmap -u "" -p "" -P 445 -H <DC IP> && smbmap -u "guest" -p "" -P 445 -H <DC IP>`
* `smbclient -U '%' -L //<DC IP> && smbclient -U 'guest%' -L //`
* Більш детальну інструкцію про те, як перерахувати SMB-сервер, можна знайти тут:

{% content-ref url="../../network-services-pentesting/pentesting-smb/" %}
[pentesting-smb](../../network-services-pentesting/pentesting-smb/)
{% endcontent-ref %}

* **Перерахувати Ldap**
* `nmap -n -sV --script "ldap* and not brute" -p 389 <DC IP>`
* Більш детальну інструкцію про те, як перерахувати LDAP, можна знайти тут (зверніть **особливу увагу на анонімний доступ**):

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

* **Отруїти мережу**
* Збирати облікові дані [**імітуючи сервіси з Responder**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
* Доступ до хоста [**зловживаючи атакою реле**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack)
* Збирати облікові дані **викриваючи** [**фальшиві UPnP сервіси з evil-S**](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md)[**SDP**](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)
* [**OSINT**](https://book.hacktricks.xyz/external-recon-methodology):
* Витягувати імена користувачів/імена з внутрішніх документів, соціальних мереж, сервісів (в основному веб) в середовищах домену, а також з публічно доступних джерел.
* Якщо ви знайдете повні імена працівників компанії, ви можете спробувати різні конвенції **імен користувачів AD** (**[читайте це](https://activedirectorypro.com/active-directory-user-naming-convention/)**). Найбільш поширені конвенції: _NameSurname_, _Name.Surname_, _NamSur_ (3 літери з кожного), _Nam.Sur_, _NSurname_, _N.Surname_, _SurnameName_, _Surname.Name_, _SurnameN_, _Surname.N_, 3 _випадкові літери та 3 випадкові цифри_ (abc123).
* Інструменти:
* [w0Tx/generate-ad-username](https://github.com/w0Tx/generate-ad-username)
* [urbanadventurer/username-anarchy](https://github.com/urbanadventurer/username-anarchy)

### User enumeration

* **Анонімний SMB/LDAP enum:** Перевірте сторінки [**pentesting SMB**](../../network-services-pentesting/pentesting-smb/) та [**pentesting LDAP**](../../network-services-pentesting/pentesting-ldap.md).
* **Kerbrute enum**: Коли запитується **недійсне ім'я користувача**, сервер відповість, використовуючи **код помилки Kerberos** _KRB5KDC\_ERR\_C\_PRINCIPAL\_UNKNOWN_, що дозволяє нам визначити, що ім'я користувача було недійсним. **Дійсні імена користувачів** викличуть або **TGT в AS-REP** відповіді, або помилку _KRB5KDC\_ERR\_PREAUTH\_REQUIRED_, що вказує на те, що користувачеві потрібно виконати попередню автентифікацію.
```bash
./kerbrute_linux_amd64 userenum -d lab.ropnop.com --dc 10.10.10.10 usernames.txt #From https://github.com/ropnop/kerbrute/releases

nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='DOMAIN'" <IP>
Nmap -p 88 --script=krb5-enum-users --script-args krb5-enum-users.realm='<domain>',userdb=/root/Desktop/usernames.txt <IP>

msf> use auxiliary/gather/kerberos_enumusers

crackmapexec smb dominio.es  -u '' -p '' --users | awk '{print $4}' | uniq
```
* **OWA (Outlook Web Access) Сервер**

Якщо ви знайшли один з цих серверів у мережі, ви також можете виконати **перерахунок користувачів проти нього**. Наприклад, ви можете використовувати інструмент [**MailSniper**](https://github.com/dafthack/MailSniper):
```bash
ipmo C:\Tools\MailSniper\MailSniper.ps1
# Get info about the domain
Invoke-DomainHarvestOWA -ExchHostname [ip]
# Enumerate valid users from a list of potential usernames
Invoke-UsernameHarvestOWA -ExchHostname [ip] -Domain [domain] -UserList .\possible-usernames.txt -OutFile valid.txt
# Password spraying
Invoke-PasswordSprayOWA -ExchHostname [ip] -UserList .\valid.txt -Password Summer2021
# Get addresses list from the compromised mail
Get-GlobalAddressList -ExchHostname [ip] -UserName [domain]\[username] -Password Summer2021 -OutFile gal.txt
```
{% hint style="warning" %}
Ви можете знайти списки імен користувачів у [**цьому репозиторії github**](https://github.com/danielmiessler/SecLists/tree/master/Usernames/Names) \*\*\*\* та у цьому ([**статистично-імовірні імена користувачів**](https://github.com/insidetrust/statistically-likely-usernames)).

Однак, ви повинні мати **імена людей, які працюють у компанії** з етапу розвідки, який ви повинні були виконати раніше. З іменем та прізвищем ви можете використовувати скрипт [**namemash.py**](https://gist.github.com/superkojiman/11076951) для генерації потенційних дійсних імен користувачів.
{% endhint %}

### Знання одного або кількох імен користувачів

Отже, ви знаєте, що у вас вже є дійсне ім'я користувача, але немає паролів... Тоді спробуйте:

* [**ASREPRoast**](asreproast.md): Якщо у користувача **немає** атрибута _DONT\_REQ\_PREAUTH_, ви можете **запросити AS\_REP повідомлення** для цього користувача, яке міститиме деякі дані, зашифровані похідною пароля користувача.
* [**Password Spraying**](password-spraying.md): Спробуємо найпоширеніші **паролі** з кожним з виявлених користувачів, можливо, деякий користувач використовує поганий пароль (пам'ятайте про політику паролів!).
* Зверніть увагу, що ви також можете **спрейти OWA сервери**, щоб спробувати отримати доступ до поштових серверів користувачів.

{% content-ref url="password-spraying.md" %}
[password-spraying.md](password-spraying.md)
{% endcontent-ref %}

### LLMNR/NBT-NS отруєння

Ви можете бути в змозі **отримати** деякі челендж **хеші** для злому **отруєння** деяких протоколів **мережі**:

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### NTML Relay

Якщо вам вдалося перерахувати активний каталог, ви отримаєте **більше електронних адрес і краще розуміння мережі**. Ви можете спробувати примусити NTML [**relay attacks**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack) \*\*\*\* для отримання доступу до середовища AD.

### Вкрасти NTLM креденціали

Якщо ви можете **отримати доступ до інших ПК або загальних ресурсів** з **нульовим або гостьовим користувачем**, ви можете **розмістити файли** (наприклад, SCF файл), які, якщо їх якось отримають доступ, **запустять NTML аутентифікацію проти вас**, щоб ви могли **вкрасти** **NTLM челендж** для злому:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

## Перерахунок Active Directory ЗА допомогою облікових даних/сесії

Для цього етапу вам потрібно мати **компрометовані облікові дані або сесію дійсного облікового запису домену.** Якщо у вас є дійсні облікові дані або оболонка як доменний користувач, **ви повинні пам'ятати, що варіанти, наведені раніше, все ще є варіантами для компрометації інших користувачів**.

Перед початком аутентифікованого перерахунку ви повинні знати, що таке **проблема подвійного стрибка Kerberos.**

{% content-ref url="kerberos-double-hop-problem.md" %}
[kerberos-double-hop-problem.md](kerberos-double-hop-problem.md)
{% endcontent-ref %}

### Перерахунок

Маючи компрометований обліковий запис, це **великий крок до початку компрометації всього домену**, оскільки ви зможете почати **перерахунок Active Directory:**

Щодо [**ASREPRoast**](asreproast.md), ви тепер можете знайти кожного можливого вразливого користувача, а щодо [**Password Spraying**](password-spraying.md) ви можете отримати **список усіх імен користувачів** і спробувати пароль компрометованого облікового запису, порожні паролі та нові перспективні паролі.

* Ви можете використовувати [**CMD для виконання базової розвідки**](../basic-cmd-for-pentesters.md#domain-info)
* Ви також можете використовувати [**powershell для розвідки**](../basic-powershell-for-pentesters/), що буде менш помітно
* Ви також можете [**використовувати powerview**](../basic-powershell-for-pentesters/powerview.md) для отримання більш детальної інформації
* Інший чудовий інструмент для розвідки в Active Directory - це [**BloodHound**](bloodhound.md). Це **не дуже непомітно** (залежно від методів збору, які ви використовуєте), але **якщо вам це не важливо**, ви повинні обов'язково спробувати. Знайдіть, де користувачі можуть RDP, знайдіть шлях до інших груп тощо.
* **Інші автоматизовані інструменти для перерахунку AD:** [**AD Explorer**](bloodhound.md#ad-explorer)**,** [**ADRecon**](bloodhound.md#adrecon)**,** [**Group3r**](bloodhound.md#group3r)**,** [**PingCastle**](bloodhound.md#pingcastle)**.**
* [**DNS записи AD**](ad-dns-records.md), оскільки вони можуть містити цікаву інформацію.
* **Інструмент з GUI**, який ви можете використовувати для перерахунку каталогу, - це **AdExplorer.exe** з **SysInternal** Suite.
* Ви також можете шукати в LDAP базі даних за допомогою **ldapsearch**, щоб шукати облікові дані в полях _userPassword_ & _unixUserPassword_, або навіть для _Description_. cf. [Пароль в коментарі користувача AD на PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#password-in-ad-user-comment) для інших методів.
* Якщо ви використовуєте **Linux**, ви також можете перерахувати домен, використовуючи [**pywerview**](https://github.com/the-useless-one/pywerview).
* Ви також можете спробувати автоматизовані інструменти, такі як:
* [**tomcarver16/ADSearch**](https://github.com/tomcarver16/ADSearch)
* [**61106960/adPEAS**](https://github.com/61106960/adPEAS)
*   **Витягування всіх користувачів домену**

Дуже легко отримати всі імена користувачів домену з Windows (`net user /domain`, `Get-DomainUser` або `wmic useraccount get name,sid`). У Linux ви можете використовувати: `GetADUsers.py -all -dc-ip 10.10.10.110 domain.com/username` або `enum4linux -a -u "user" -p "password" <DC IP>`

> Навіть якщо цей розділ перерахунку виглядає маленьким, це найважливіша частина всього. Перейдіть за посиланнями (в основному на cmd, powershell, powerview і BloodHound), навчіться перераховувати домен і практикуйтеся, поки не відчуєте себе комфортно. Під час оцінки це буде ключовий момент, щоб знайти свій шлях до DA або вирішити, що нічого не можна зробити.

### Kerberoast

Kerberoasting передбачає отримання **TGS квитків**, які використовуються службами, пов'язаними з обліковими записами користувачів, і злому їх шифрування, яке базується на паролях користувачів, **офлайн**.

Більше про це в:

{% content-ref url="kerberoast.md" %}
[kerberoast.md](kerberoast.md)
{% endcontent-ref %}

### Віддалене з'єднання (RDP, SSH, FTP, Win-RM тощо)

Якщо ви отримали деякі облікові дані, ви можете перевірити, чи маєте доступ до будь-якої **машини**. Для цього ви можете використовувати **CrackMapExec**, щоб спробувати підключитися до кількох серверів з різними протоколами, відповідно до ваших сканувань портів.

### Локальне підвищення привілеїв

Якщо ви компрометували облікові дані або сесію як звичайний доменний користувач і маєте **доступ** з цим користувачем до **будь-якої машини в домені**, ви повинні спробувати знайти спосіб **підвищити привілеї локально та шукати облікові дані**. Це тому, що тільки з правами локального адміністратора ви зможете **вивантажити хеші інших користувачів** в пам'яті (LSASS) та локально (SAM).

У цій книзі є повна сторінка про [**локальне підвищення привілеїв у Windows**](../windows-local-privilege-escalation/) та [**контрольний список**](../checklist-windows-privilege-escalation.md). Також не забудьте використовувати [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite).

### Поточні сесійні квитки

Дуже **малоймовірно**, що ви знайдете **квитки** у поточного користувача, **які надають вам дозвіл на доступ** до несподіваних ресурсів, але ви можете перевірити:
```bash
## List all tickets (if not admin, only current user tickets)
.\Rubeus.exe triage
## Dump the interesting one by luid
.\Rubeus.exe dump /service:krbtgt /luid:<luid> /nowrap
[IO.File]::WriteAllBytes("ticket.kirbi", [Convert]::FromBase64String("<BASE64_TICKET>"))
```
### NTML Relay

Якщо вам вдалося перерахувати активний каталог, ви отримаєте **більше електронних листів і краще розуміння мережі**. Ви можете змусити NTML [**атаки реле**](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#relay-attack)**.**

### **Шукайте креденціали в комп'ютерних спільних ресурсах**

Тепер, коли у вас є деякі базові креденціали, ви повинні перевірити, чи можете ви **знайти** будь-які **цікаві файли, які діляться всередині AD**. Ви можете зробити це вручну, але це дуже нудне повторюване завдання (і ще більше, якщо ви знайдете сотні документів, які потрібно перевірити).

[**Слідуйте за цим посиланням, щоб дізнатися про інструменти, які ви можете використовувати.**](../../network-services-pentesting/pentesting-smb/#domain-shared-folders-search)

### Вкрасти NTLM креденціали

Якщо ви можете **доступитися до інших ПК або спільних ресурсів**, ви можете **розмістити файли** (наприклад, файл SCF), які, якщо їх якось відкриють, **викличуть NTML аутентифікацію проти вас**, щоб ви могли **вкрасти** **NTLM виклик** для його зламу:

{% content-ref url="../ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### CVE-2021-1675/CVE-2021-34527 PrintNightmare

Ця вразливість дозволила будь-якому аутентифікованому користувачу **компрометувати контролер домену**.

{% content-ref url="printnightmare.md" %}
[printnightmare.md](printnightmare.md)
{% endcontent-ref %}

## Підвищення привілеїв в Active Directory З привілейованими креденціалами/сесією

**Для наступних технік звичайного доменного користувача недостатньо, вам потрібні деякі спеціальні привілеї/креденціали для виконання цих атак.**

### Витягування хешів

Сподіваюся, вам вдалося **компрометувати деякий локальний адміністратор** обліковий запис, використовуючи [AsRepRoast](asreproast.md), [Password Spraying](password-spraying.md), [Kerberoast](kerberoast.md), [Responder](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md) включаючи реле, [EvilSSDP](../../generic-methodologies-and-resources/pentesting-network/spoofing-ssdp-and-upnp-devices.md), [підвищення привілеїв локально](../windows-local-privilege-escalation/).\
Тепер час вивантажити всі хеші в пам'яті та локально.\
[**Прочитайте цю сторінку про різні способи отримання хешів.**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/active-directory-methodology/broken-reference/README.md)

### Pass the Hash

**Якщо у вас є хеш користувача**, ви можете використовувати його для **імітуювання** його.\
Вам потрібно використовувати якийсь **інструмент**, який **виконає** **NTLM аутентифікацію, використовуючи** цей **хеш**, **або** ви можете створити новий **sessionlogon** і **впровадити** цей **хеш** всередину **LSASS**, так що коли будь-яка **NTLM аутентифікація виконується**, цей **хеш буде використаний.** Останній варіант - це те, що робить mimikatz.\
[**Прочитайте цю сторінку для отримання додаткової інформації.**](../ntlm/#pass-the-hash)

### Over Pass the Hash/Pass the Key

Ця атака має на меті **використати NTLM хеш користувача для запиту квитків Kerberos**, як альтернатива звичайному Pass The Hash через NTLM протокол. Тому це може бути особливо **корисно в мережах, де NTLM протокол вимкнено** і лише **Kerberos дозволено** як протокол аутентифікації.

{% content-ref url="over-pass-the-hash-pass-the-key.md" %}
[over-pass-the-hash-pass-the-key.md](over-pass-the-hash-pass-the-key.md)
{% endcontent-ref %}

### Pass the Ticket

У методі атаки **Pass The Ticket (PTT)** зловмисники **вкрадають квиток аутентифікації користувача** замість їх пароля або значень хешу. Цей вкрадений квиток потім використовується для **імітуювання користувача**, отримуючи несанкціонований доступ до ресурсів і послуг у мережі.

{% content-ref url="pass-the-ticket.md" %}
[pass-the-ticket.md](pass-the-ticket.md)
{% endcontent-ref %}

### Повторне використання креденціалів

Якщо у вас є **хеш** або **пароль** **локального адміністратора**, ви повинні спробувати **увійти локально** до інших **ПК** з ним.
```bash
# Local Auth Spray (once you found some local admin pass or hash)
## --local-auth flag indicate to only try 1 time per machine
crackmapexec smb --local-auth 10.10.10.10/23 -u administrator -H 10298e182387f9cab376ecd08491764a0 | grep +
```
{% hint style="warning" %}
Зверніть увагу, що це досить **гучно** і **LAPS** це **зменшить**.
{% endhint %}

### Зловживання MSSQL та Довірені Посилання

Якщо користувач має привілеї для **доступу до екземплярів MSSQL**, він може використовувати це для **виконання команд** на хості MSSQL (якщо працює як SA), **викрадення** NetNTLM **хешу** або навіть виконання **атаки** **реле**.\
Також, якщо екземпляр MSSQL є довіреним (посилання на базу даних) іншим екземпляром MSSQL. Якщо користувач має привілеї над довіреною базою даних, він зможе **використовувати довірчі відносини для виконання запитів також в іншому екземплярі**. Ці довіри можуть бути з'єднані, і в якийсь момент користувач може знайти неправильно налаштовану базу даних, де він може виконувати команди.\
**Посилання між базами даних працюють навіть через довіри лісу.**

{% content-ref url="abusing-ad-mssql.md" %}
[abusing-ad-mssql.md](abusing-ad-mssql.md)
{% endcontent-ref %}

### Неконтрольована Делегація

Якщо ви знайдете будь-який об'єкт комп'ютера з атрибутом [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) і у вас є доменні привілеї на комп'ютері, ви зможете вивантажити TGT з пам'яті кожного користувача, який входить на комп'ютер.\
Отже, якщо **Domain Admin входить на комп'ютер**, ви зможете вивантажити його TGT і видавати себе за нього, використовуючи [Pass the Ticket](pass-the-ticket.md).\
Завдяки контрольованій делегації ви навіть могли б **автоматично скомпрометувати сервер друку** (надіємось, це буде DC).

{% content-ref url="unconstrained-delegation.md" %}
[unconstrained-delegation.md](unconstrained-delegation.md)
{% endcontent-ref %}

### Контрольована Делегація

Якщо користувач або комп'ютер дозволено для "Контрольованої Делегації", він зможе **видавати себе за будь-якого користувача для доступу до деяких сервісів на комп'ютері**.\
Тоді, якщо ви **скомпрометуєте хеш** цього користувача/комп'ютера, ви зможете **видавати себе за будь-якого користувача** (навіть доменних адміністраторів) для доступу до деяких сервісів.

{% content-ref url="constrained-delegation.md" %}
[constrained-delegation.md](constrained-delegation.md)
{% endcontent-ref %}

### Делегація на основі ресурсів

Маючи привілей **WRITE** на об'єкт Active Directory віддаленого комп'ютера, можна отримати виконання коду з **підвищеними привілеями**:

{% content-ref url="resource-based-constrained-delegation.md" %}
[resource-based-constrained-delegation.md](resource-based-constrained-delegation.md)
{% endcontent-ref %}

### Зловживання ACL

Скомпрометований користувач може мати деякі **цікаві привілеї над деякими об'єктами домену**, які можуть дозволити вам **переміщатися** латерально/**підвищувати** привілеї.

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### Зловживання службою Spooler принтера

Виявлення **служби Spool** в домені може бути **зловжито** для **отримання нових облікових даних** та **підвищення привілеїв**.

{% content-ref url="printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](printers-spooler-service-abuse.md)
{% endcontent-ref %}

### Зловживання сесіями третіх сторін

Якщо **інші користувачі** **доступають** до **скомпрометованої** машини, можливо **збирати облікові дані з пам'яті** і навіть **впроваджувати маяки в їхні процеси** для видачі себе за них.\
Зазвичай користувачі отримують доступ до системи через RDP, тому ось як виконати кілька атак на сесії RDP третіх сторін:

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### LAPS

**LAPS** забезпечує систему для управління **паролем локального адміністратора** на комп'ютерах, що приєднані до домену, забезпечуючи його **випадковість**, унікальність та часту **зміну**. Ці паролі зберігаються в Active Directory, а доступ контролюється через ACL лише для авторизованих користувачів. З достатніми правами для доступу до цих паролів стає можливим переміщення до інших комп'ютерів.

{% content-ref url="laps.md" %}
[laps.md](laps.md)
{% endcontent-ref %}

### Крадіжка Сертифікатів

**Збір сертифікатів** з скомпрометованої машини може бути способом підвищення привілеїв у середовищі:

{% content-ref url="ad-certificates/certificate-theft.md" %}
[certificate-theft.md](ad-certificates/certificate-theft.md)
{% endcontent-ref %}

### Зловживання Шаблонами Сертифікатів

Якщо **вразливі шаблони** налаштовані, їх можна зловживати для підвищення привілеїв:

{% content-ref url="ad-certificates/domain-escalation.md" %}
[domain-escalation.md](ad-certificates/domain-escalation.md)
{% endcontent-ref %}

## Пост-експлуатація з обліковим записом з високими привілеями

### Вивантаження Доменних Облікових Даних

Якщо ви отримали привілеї **Domain Admin** або навіть краще **Enterprise Admin**, ви можете **вивантажити** **доменну базу даних**: _ntds.dit_.

[**Більше інформації про атаку DCSync можна знайти тут**](dcsync.md).

[**Більше інформації про те, як вкрасти NTDS.dit можна знайти тут**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/active-directory-methodology/broken-reference/README.md)

### Підвищення привілеїв як Постійність

Деякі з технік, обговорених раніше, можуть бути використані для постійності.\
Наприклад, ви могли б:

*   Зробити користувачів вразливими до [**Kerberoast**](kerberoast.md)

```powershell
Set-DomainObject -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}r
```
*   Зробити користувачів вразливими до [**ASREPRoast**](asreproast.md)

```powershell
Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
```
*   Надати [**DCSync**](./#dcsync) привілеї користувачу

```powershell
Add-DomainObjectAcl -TargetIdentity "DC=SUB,DC=DOMAIN,DC=LOCAL" -PrincipalIdentity bfarmer -Rights DCSync
```

### Срібний Квиток

**Атака Срібного Квитка** створює **легітимний квиток на надання послуг (TGS)** для конкретної служби, використовуючи **NTLM хеш** (наприклад, **хеш облікового запису ПК**). Цей метод використовується для **доступу до привілеїв служби**.

{% content-ref url="silver-ticket.md" %}
[silver-ticket.md](silver-ticket.md)
{% endcontent-ref %}

### Золотий Квиток

**Атака Золотого Квитка** передбачає, що зловмисник отримує доступ до **NTLM хешу облікового запису krbtgt** в середовищі Active Directory (AD). Цей обліковий запис є особливим, оскільки використовується для підписання всіх **квитків на надання послуг (TGT)**, які є необхідними для аутентифікації в мережі AD.

Якщо зловмисник отримує цей хеш, він може створити **TGT** для будь-якого облікового запису, який вибере (атака Срібного Квитка).

{% content-ref url="golden-ticket.md" %}
[golden-ticket.md](golden-ticket.md)
{% endcontent-ref %}

### Діамантовий Квиток

Ці квитки схожі на золоті, але підроблені так, щоб **обійти звичайні механізми виявлення золотих квитків.**

{% content-ref url="diamond-ticket.md" %}
[diamond-ticket.md](diamond-ticket.md)
{% endcontent-ref %}

### **Постійність Облікових Записів Сертифікатів**

**Маючи сертифікати облікового запису або можливість їх запитувати** є дуже хорошим способом зберегти постійність в обліковому записі користувача (навіть якщо він змінює пароль):

{% content-ref url="ad-certificates/account-persistence.md" %}
[account-persistence.md](ad-certificates/account-persistence.md)
{% endcontent-ref %}

### **Постійність Сертифікатів в Домені**

**Використання сертифікатів також можливе для постійності з високими привілеями в домені:**

{% content-ref url="ad-certificates/domain-persistence.md" %}
[domain-persistence.md](ad-certificates/domain-persistence.md)
{% endcontent-ref %}

### Група AdminSDHolder

Об'єкт **AdminSDHolder** в Active Directory забезпечує безпеку **привілейованих груп** (таких як Domain Admins та Enterprise Admins), застосовуючи стандартний **Список Контролю Доступу (ACL)** до цих груп, щоб запобігти несанкціонованим змінам. Однак цю функцію можна експлуатувати; якщо зловмисник змінює ACL AdminSDHolder, щоб надати повний доступ звичайному користувачу, цей користувач отримує значний контроль над усіма привілейованими групами. Ця міра безпеки, призначена для захисту, може таким чином обернутися проти, дозволяючи неналежний доступ, якщо не контролювати її.

[**Більше інформації про групу AdminDSHolder тут.**](privileged-groups-and-token-privileges.md#adminsdholder-group)

### Облікові Дані DSRM

У кожному **Контролері Домену (DC)** існує обліковий запис **локального адміністратора**. Отримавши права адміністратора на такій машині, хеш локального адміністратора можна витягти за допомогою **mimikatz**. Після цього необхідно внести зміни до реєстру, щоб **дозволити використання цього пароля**, що дозволяє віддалений доступ до облікового запису локального адміністратора.

{% content-ref url="dsrm-credentials.md" %}
[dsrm-credentials.md](dsrm-credentials.md)
{% endcontent-ref %}

### Постійність ACL

Ви могли б **надати** деякі **спеціальні привілеї** **користувачу** над деякими конкретними об'єктами домену, які дозволять користувачу **підвищити привілеї в майбутньому**.

{% content-ref url="acl-persistence-abuse/" %}
[acl-persistence-abuse](acl-persistence-abuse/)
{% endcontent-ref %}

### Описники Безпеки

**Описники безпеки** використовуються для **зберігання** **привілеїв**, які має **об'єкт** **над** **об'єктом**. Якщо ви зможете просто **зробити** **невелику зміну** в **описнику безпеки** об'єкта, ви зможете отримати дуже цікаві привілеї над цим об'єктом без необхідності бути членом привілейованої групи.

{% content-ref url="security-descriptors.md" %}
[security-descriptors.md](security-descriptors.md)
{% endcontent-ref %}

### Скелетний Ключ

Змініть **LSASS** в пам'яті, щоб встановити **універсальний пароль**, що надає доступ до всіх доменних облікових записів.

{% content-ref url="skeleton-key.md" %}
[skeleton-key.md](skeleton-key.md)
{% endcontent-ref %}

### Користувацький SSP

[Дізнайтеся, що таке SSP (Постачальник Підтримки Безпеки) тут.](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
Ви можете створити свій **власний SSP**, щоб **захоплювати** в **чистому тексті** **облікові дані**, які використовуються для доступу до машини.\\

{% content-ref url="custom-ssp.md" %}
[custom-ssp.md](custom-ssp.md)
{% endcontent-ref %}

### DCShadow

Це реєструє **новий Контролер Домену** в AD і використовує його для **поштовхування атрибутів** (SIDHistory, SPNs...) на вказані об'єкти **без** залишення будь-яких **журналів** щодо **змін**. Вам **потрібні DA** привілеї і бути всередині **кореневого домену**.\
Зверніть увагу, що якщо ви використовуєте неправильні дані, з'являться досить неприємні журнали.

{% content-ref url="dcshadow.md" %}
[dcshadow.md](dcshadow.md)
{% endcontent-ref %}

### Постійність LAPS

Раніше ми обговорювали, як підвищити привілеї, якщо у вас є **достатні права для читання паролів LAPS**. Однак ці паролі також можуть бути використані для **підтримки постійності**.\
Перевірте:

{% content-ref url="laps.md" %}
[laps.md](laps.md)
{% endcontent-ref %}

## Підвищення Привілеїв Лісу - Доменні Довіри

Microsoft розглядає **Ліс** як межу безпеки. Це означає, що **скомпрометування одного домену може потенційно призвести до компрометації всього Лісу**.

### Основна Інформація

[**Доменна довіра**](http://technet.microsoft.com/en-us/library/cc759554\(v=ws.10\).aspx) є механізмом безпеки, який дозволяє користувачу з одного **домену** отримувати доступ до ресурсів в іншому **домені**. Це фактично створює зв'язок між системами аутентифікації двох доменів, дозволяючи безперешкодно проходити перевірки аутентифікації. Коли домени встановлюють довіру, вони обмінюються та зберігають специфічні **ключі** в своїх **Контролерах Домену (DC)**, які є критично важливими для цілісності довіри.

У типовій ситуації, якщо користувач має намір отримати доступ до служби в **довіреному домені**, спочатку він повинен запитати спеціальний квиток, відомий як **міждоменний TGT**, у DC свого власного домену. Цей TGT зашифрований спільним **ключем**, на якому обидва домени погодилися. Користувач потім представляє цей TGT **DC довіреного домену**, щоб отримати квиток на службу (**TGS**). Після успішної перевірки міждоменного TGT DC довіреного домену видає TGS, надаючи користувачу доступ до служби.

**Кроки**:

1. **Клієнтський комп'ютер** в **Домені 1** починає процес, використовуючи свій **NTLM хеш** для запиту **Квитка на Надання Квитків (TGT)** у свого **Контролера Домену (DC1)**.
2. DC1 видає новий TGT, якщо клієнт успішно аутентифікований.
3. Клієнт потім запитує **міждоменний TGT** у DC1, який потрібен для доступу до ресурсів у **Домені 2**.
4. Міждоменний TGT зашифрований спільним **ключем довіри**, що ділиться між DC1 та DC2 в рамках двосторонньої довіри домену.
5. Клієнт приносить міждоменний TGT до **Контролера Домену 2 (DC2)**.
6. DC2 перевіряє міждоменний TGT, використовуючи свій спільний ключ довіри, і, якщо він дійсний, видає **Квиток на Надання Послуг (TGS)** для сервера в Домені 2, до якого клієнт хоче отримати доступ.
7. Нарешті, клієнт представляє цей TGS серверу, який зашифрований хешем облікового запису сервера, щоб отримати доступ до служби в Домені 2.

### Різні довіри

Важливо помітити, що **довіра може бути односторонньою або двосторонньою**. У двосторонньому варіанті обидва домени довіряють один одному, але в **односторонній** довірі один з доменів буде **довіреним**, а інший - **доверяючим**. У останньому випадку **ви зможете отримати доступ до ресурсів лише всередині довірчого домену з довіреного**.

Якщо Домен A довіряє Домену B, A є довірчим доменом, а B - довіреним. Більше того, в **Доміні A** це буде **вихідна довіра**; а в **Доміні B** це буде **вхідна довіра**.

**Різні довірчі відносини**

* **Довіри Батьків-Дітей**: Це звичайна налаштування в межах одного лісу, де дитячий домен автоматично має двосторонню транзитивну довіру з батьківським доменом. Це означає, що запити на аутентифікацію можуть безперешкодно проходити між батьком і дитиною.
* **Перехресні Довіри**: Відомі як "скорочені довіри", вони встановлюються між дитячими доменами для прискорення процесів посилання. У складних лісах запити на аутентифікацію зазвичай повинні проходити до кореня лісу, а потім вниз до цільового домену. Створюючи перехресні зв'язки, подорож скорочується, що особливо корисно в географічно розподілених середовищах.
* **Зовнішні Довіри**: Вони встановлюються між різними, не пов'язаними доменами і за своєю природою є нетранзитивними. Згідно з [документацією Microsoft](https://technet.microsoft.com/en-us/library/cc773178\(v=ws.10\).aspx), зовнішні довіри корисні для доступу до ресурсів у домені поза поточним лісом, який не підключений через довіру лісу. Безпека посилюється через фільтрацію SID з зовнішніми довірами.
* **Довіри Кореня Дерева**: Ці довіри автоматично встановлюються між кореневим доменом лісу та новододаним коренем дерева. Хоча їх не часто зустрічають, довіри кореня дерева важливі для додавання нових доменних дерев до лісу, дозволяючи їм зберігати унікальну назву домену та забезпечуючи двосторонню транзитивність. Більше інформації можна знайти в [посібнику Microsoft](https://technet.microsoft.com/en-us/library/cc773178\(v=ws.10\).aspx).
* **Довіри Лісу**: Цей тип довіри є двосторонньою транзитивною довірою між двома кореневими доменами лісу, також забезпечуючи фільтрацію SID для підвищення заходів безпеки.
* **Довіри MIT**: Ці довіри встановлюються з не-Windows, [RFC4120-сумісними](https://tools.ietf.org/html/rfc4120) доменами Kerberos. Довіри MIT є дещо більш спеціалізованими і призначені для середовищ, які потребують інтеграції з системами на основі Kerberos поза екосистемою Windows.

#### Інші відмінності в **довірчих відносинах**

* Довірчі відносини можуть бути також **транзитивними** (A довіряє B, B довіряє C, тоді A довіряє C) або **нетранзитивними**.
* Довірчі відносини можуть бути налаштовані як **двостороння довіра** (обидва довіряють один одному) або як **одностороння довіра** (лише один з них довіряє іншому).

### Шлях Атаки

1. **Перелічити** довірчі відносини
2. Перевірте, чи має будь-який **суб'єкт безпеки** (користувач/група/комп'ютер) **доступ** до ресурсів **іншого домену**, можливо, через записи ACE або через членство в групах іншого домену. Шукайте **відносини між доменами** (довіра була створена для цього, напевно).
1. У цьому випадку kerberoast може бути ще одним варіантом.
3. **Скомпрометувати** **облікові записи**, які можуть **переміщатися** між доменами.

Зловмисники можуть отримати доступ до ресурсів в іншому домені через три основні механізми:

* **Членство в Локальних Групах**: Суб'єкти можуть бути додані до локальних груп на машинах, таких як група "Адміністратори" на сервері, що надає їм значний контроль над цією машиною.
* **Членство в Групах Зовнішнього Домену**: Суб'єкти також можуть бути членами груп у зовнішньому домені. Однак ефективність цього методу залежить від природи довіри та обсягу групи.
* **Списки Контролю Доступу (ACL)**: Суб'єкти можуть бути вказані в **ACL**, особливо як сутності в **ACE** в рамках **DACL**, надаючи їм доступ до специфічних ресурсів. Для тих, хто хоче глибше зануритися в механіку ACL, DACL та ACE, документ під назвою “[An ACE Up The Sleeve](https://specterops.io/assets/resources/an\_ace\_up\_the\_sleeve.pdf)” є безцінним ресурсом.

### Підвищення Привілеїв з Дитини до Батька в Лісі
```
Get-DomainTrust

SourceName      : sub.domain.local    --> current domain
TargetName      : domain.local        --> foreign domain
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST       --> WITHIN_FOREST: Both in the same forest
TrustDirection  : Bidirectional       --> Trust direction (2ways in this case)
WhenCreated     : 2/19/2021 1:28:00 PM
WhenChanged     : 2/19/2021 1:28:00 PM
```
{% hint style="warning" %}
Є **2 довірених ключі**, один для _Дитина --> Батько_ і ще один для _Батько_ --> _Дитина_.\
Ви можете використовувати той, що використовується поточним доменом, з:
```bash
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\mcorp$"'
```
{% endhint %}

#### SID-History Injection

Ескалація як Enterprise admin до дочірнього/батьківського домену, зловживаючи довірою з ін'єкцією SID-History:

{% content-ref url="sid-history-injection.md" %}
[sid-history-injection.md](sid-history-injection.md)
{% endcontent-ref %}

#### Exploit writeable Configuration NC

Розуміння того, як можна експлуатувати Configuration Naming Context (NC), є критично важливим. Configuration NC слугує центральним репозиторієм для конфігураційних даних у лісі в середовищах Active Directory (AD). Ці дані реплікуються на кожен Domain Controller (DC) у лісі, при цьому записувані DC підтримують записувану копію Configuration NC. Щоб це експлуатувати, потрібно мати **SYSTEM привілеї на DC**, бажано на дочірньому DC.

**Link GPO to root DC site**

Контейнер Sites Configuration NC містить інформацію про всі комп'ютери, приєднані до домену, у лісі AD. Працюючи з привілеями SYSTEM на будь-якому DC, зловмисники можуть прив'язувати GPO до кореневих сайтів DC. Ця дія потенційно компрометує кореневий домен, маніпулюючи політиками, які застосовуються до цих сайтів.

Для детальної інформації можна дослідити матеріали про [Bypassing SID Filtering](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research).

**Compromise any gMSA in the forest**

Вектор атаки передбачає націлювання на привілейовані gMSA в домені. Ключ KDS Root, необхідний для обчислення паролів gMSA, зберігається в Configuration NC. Маючи привілеї SYSTEM на будь-якому DC, можна отримати доступ до ключа KDS Root і обчислити паролі для будь-якого gMSA у лісі.

Детальний аналіз можна знайти в обговоренні [Golden gMSA Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent).

**Schema change attack**

Цей метод вимагає терпіння, очікуючи створення нових привілейованих об'єктів AD. Маючи привілеї SYSTEM, зловмисник може змінити схему AD, щоб надати будь-якому користувачу повний контроль над усіма класами. Це може призвести до несанкціонованого доступу та контролю над новоствореними об'єктами AD.

Додаткова інформація доступна на [Schema Change Trust Attacks](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent).

**From DA to EA with ADCS ESC5**

Уразливість ADCS ESC5 націлюється на контроль над об'єктами інфраструктури відкритих ключів (PKI), щоб створити шаблон сертифіката, який дозволяє автентифікацію як будь-який користувач у лісі. Оскільки об'єкти PKI знаходяться в Configuration NC, компрометація записуваного дочірнього DC дозволяє виконувати атаки ESC5.

Більше деталей про це можна прочитати в [From DA to EA with ESC5](https://posts.specterops.io/from-da-to-ea-with-esc5-f9f045aa105c). У сценаріях, де відсутній ADCS, зловмисник має можливість налаштувати необхідні компоненти, як обговорюється в [Escalating from Child Domain Admins to Enterprise Admins](https://www.pkisolutions.com/escalating-from-child-domains-admins-to-enterprise-admins-in-5-minutes-by-abusing-ad-cs-a-follow-up/).

### External Forest Domain - One-Way (Inbound) or bidirectional
```powershell
Get-DomainTrust
SourceName      : a.domain.local   --> Current domain
TargetName      : domain.external  --> Destination domain
TrustType       : WINDOWS-ACTIVE_DIRECTORY
TrustAttributes :
TrustDirection  : Inbound          --> Inboud trust
WhenCreated     : 2/19/2021 10:50:56 PM
WhenChanged     : 2/19/2021 10:50:56 PM
```
У цьому сценарії **ваш домен довірений** зовнішньому, що надає вам **невизначені дозволи** над ним. Вам потрібно буде з'ясувати, **які принципи вашого домену мають який доступ до зовнішнього домену** і потім спробувати це експлуатувати:

{% content-ref url="external-forest-domain-oneway-inbound.md" %}
[external-forest-domain-oneway-inbound.md](external-forest-domain-oneway-inbound.md)
{% endcontent-ref %}

### Зовнішній лісовий домен - односпрямований (вихідний)
```powershell
Get-DomainTrust -Domain current.local

SourceName      : current.local   --> Current domain
TargetName      : external.local  --> Destination domain
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound        --> Outbound trust
WhenCreated     : 2/19/2021 10:15:24 PM
WhenChanged     : 2/19/2021 10:15:24 PM
```
У цьому сценарії **ваш домен** **довіряє** деяким **привілеям** принципу з **інших доменів**.

Однак, коли **домен довіряється** довіреним доменом, довірений домен **створює користувача** з **передбачуваним ім'ям**, який використовує **довірений пароль**. Це означає, що можливо **отримати доступ до користувача з довіреного домену, щоб потрапити всередину довіреного** для його перерахунку та спроби ескалації більше привілеїв:

{% content-ref url="external-forest-domain-one-way-outbound.md" %}
[external-forest-domain-one-way-outbound.md](external-forest-domain-one-way-outbound.md)
{% endcontent-ref %}

Інший спосіб скомпрометувати довірений домен - це знайти [**SQL довірене з'єднання**](abusing-ad-mssql.md#mssql-trusted-links), створене в **протилежному напрямку** довіреності домену (що не є дуже поширеним).

Ще один спосіб скомпрометувати довірений домен - це чекати на машині, до якої **користувач з довіреного домену може отримати доступ** для входу через **RDP**. Тоді зловмисник може впровадити код у процес сесії RDP і **отримати доступ до початкового домену жертви** звідти.\
Більше того, якщо **жертва змонтувала свій жорсткий диск**, з процесу **сесії RDP** зловмисник може зберігати **бекдори** в **папці автозавантаження жорсткого диска**. Цю техніку називають **RDPInception.**

{% content-ref url="rdp-sessions-abuse.md" %}
[rdp-sessions-abuse.md](rdp-sessions-abuse.md)
{% endcontent-ref %}

### Зменшення зловживань довірою домену

### **Фільтрація SID:**

* Ризик атак, що використовують атрибут історії SID через довіри лісу, зменшується за допомогою фільтрації SID, яка активується за замовчуванням на всіх міжлісових довірах. Це підкріплюється припущенням, що внутрішні довіри лісу є безпечними, вважаючи ліс, а не домен, як межу безпеки відповідно до позиції Microsoft.
* Однак є підводний камінь: фільтрація SID може порушити роботу додатків і доступ користувачів, що призводить до її періодичного деактивування.

### **Вибіркова аутентифікація:**

* Для міжлісових довірів використання вибіркової аутентифікації забезпечує, що користувачі з двох лісів не аутентифікуються автоматично. Натомість для доступу до доменів і серверів у довіреному домені або лісі потрібні явні дозволи.
* Важливо зазначити, що ці заходи не захищають від експлуатації запису конфігурації, що підлягає запису (NC), або атак на обліковий запис довіри.

[**Більше інформації про довіри доменів на ired.team.**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

## AD -> Azure & Azure -> AD

{% embed url="https://cloud.hacktricks.xyz/pentesting-cloud/azure-security/az-lateral-movements/azure-ad-connect-hybrid-identity" %}

## Деякі загальні заходи захисту

[**Дізнайтеся більше про те, як захистити облікові дані тут.**](../stealing-credentials/credentials-protections.md)\\

### **Заходи захисту для захисту облікових даних**

* **Обмеження для адміністраторів домену**: Рекомендується, щоб адміністраторам домену дозволялося входити лише на контролери домену, уникаючи їх використання на інших хостах.
* **Привілеї облікових записів служб**: Служби не повинні працювати з привілеями адміністратора домену (DA) для підтримки безпеки.
* **Обмеження тривалості привілеїв**: Для завдань, що вимагають привілеїв DA, їх тривалість повинна бути обмежена. Це можна досягти за допомогою: `Add-ADGroupMember -Identity ‘Domain Admins’ -Members newDA -MemberTimeToLive (New-TimeSpan -Minutes 20)`

### **Впровадження технік обману**

* Впровадження обману передбачає встановлення пасток, таких як приманкові користувачі або комп'ютери, з такими функціями, як паролі, які не закінчуються або позначені як довірені для делегування. Детальний підхід включає створення користувачів з певними правами або додавання їх до груп з високими привілеями.
* Практичний приклад включає використання інструментів, таких як: `Create-DecoyUser -UserFirstName user -UserLastName manager-uncommon -Password Pass@123 | DeployUserDeception -UserFlag PasswordNeverExpires -GUID d07da11f-8a3d-42b6-b0aa-76c962be719a -Verbose`
* Більше про впровадження технік обману можна знайти на [Deploy-Deception на GitHub](https://github.com/samratashok/Deploy-Deception).

### **Виявлення обману**

* **Для об'єктів користувачів**: Підозрілі ознаки включають нетиповий ObjectSID, рідкісні входи, дати створення та низькі кількості неправильних паролів.
* **Загальні ознаки**: Порівняння атрибутів потенційних приманкових об'єктів з атрибутами справжніх може виявити невідповідності. Інструменти, такі як [HoneypotBuster](https://github.com/JavelinNetworks/HoneypotBuster), можуть допомогти в ідентифікації таких обманів.

### **Обхід систем виявлення**

* **Обхід виявлення Microsoft ATA**:
* **Перерахунок користувачів**: Уникнення перерахунку сесій на контролерах домену, щоб запобігти виявленню ATA.
* **Імітація квитків**: Використання **aes** ключів для створення квитків допомагає уникнути виявлення, не знижуючи до NTLM.
* **Атаки DCSync**: Рекомендується виконувати з не-контролера домену, щоб уникнути виявлення ATA, оскільки безпосереднє виконання з контролера домену викличе сповіщення.

## Посилання

* [http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/](http://www.harmj0y.net/blog/redteaming/a-guide-to-attacking-domain-trusts/)
* [https://www.labofapenetrationtester.com/2018/10/deploy-deception.html](https://www.labofapenetrationtester.com/2018/10/deploy-deception.html)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/child-domain-da-to-ea-in-parent-domain)

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
