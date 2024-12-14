# macOS Keychain

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Основні ключові ланцюги

* **Ключовий ланцюг користувача** (`~/Library/Keychains/login.keychain-db`), який використовується для зберігання **облікових даних, специфічних для користувача**, таких як паролі додатків, паролі в Інтернеті, сертифікати, створені користувачем, паролі мережі та публічні/приватні ключі, створені користувачем.
* **Системний ключовий ланцюг** (`/Library/Keychains/System.keychain`), який зберігає **системні облікові дані**, такі як паролі WiFi, кореневі сертифікати системи, приватні ключі системи та паролі додатків системи.
* Можна знайти інші компоненти, такі як сертифікати в `/System/Library/Keychains/*`
* У **iOS** є лише один **ключовий ланцюг**, розташований у `/private/var/Keychains/`. Ця папка також містить бази даних для `TrustStore`, органів сертифікації (`caissuercache`) та записів OSCP (`ocspache`).
* Додатки будуть обмежені в ключовому ланцюзі лише до їх приватної області на основі їх ідентифікатора додатка.

### Доступ до ключового ланцюга паролів

Ці файли, хоча й не мають вбудованого захисту та можуть бути **завантажені**, зашифровані та вимагають **плоского пароля користувача для розшифровки**. Інструмент, такий як [**Chainbreaker**](https://github.com/n0fate/chainbreaker), може бути використаний для розшифровки.

## Захист записів ключового ланцюга

### ACLs

Кожен запис у ключовому ланцюзі регулюється **Списками контролю доступу (ACLs)**, які визначають, хто може виконувати різні дії над записом ключового ланцюга, включаючи:

* **ACLAuhtorizationExportClear**: Дозволяє власнику отримати відкритий текст секрету.
* **ACLAuhtorizationExportWrapped**: Дозволяє власнику отримати відкритий текст, зашифрований іншим наданим паролем.
* **ACLAuhtorizationAny**: Дозволяє власнику виконувати будь-яку дію.

ACL супроводжуються **переліком довірених додатків**, які можуть виконувати ці дії без запиту. Це може бути:

* **N`il`** (не потрібна авторизація, **всі довірені**)
* **порожній** список (**ніхто** не довірений)
* **Список** конкретних **додатків**.

Також запис може містити ключ **`ACLAuthorizationPartitionID`,** який використовується для ідентифікації **teamid, apple,** та **cdhash.**

* Якщо **teamid** вказано, тоді для **доступу до значення запису** **без** **запиту** використовуваний додаток повинен мати **той же teamid**.
* Якщо **apple** вказано, тоді додаток повинен бути **підписаний** **Apple**.
* Якщо **cdhash** вказано, тоді **додаток** повинен мати конкретний **cdhash**.

### Створення запису в ключовому ланцюзі

Коли **новий** **запис** створюється за допомогою **`Keychain Access.app`**, застосовуються такі правила:

* Усі додатки можуть шифрувати.
* **Жоден додаток** не може експортувати/розшифровувати (без запиту користувача).
* Усі додатки можуть бачити перевірку цілісності.
* Жоден додаток не може змінювати ACLs.
* **partitionID** встановлюється на **`apple`**.

Коли **додаток створює запис у ключовому ланцюзі**, правила трохи інші:

* Усі додатки можуть шифрувати.
* Тільки **додаток, що створює** (або будь-які інші додатки, які явно додані) можуть експортувати/розшифровувати (без запиту користувача).
* Усі додатки можуть бачити перевірку цілісності.
* Жоден додаток не може змінювати ACLs.
* **partitionID** встановлюється на **`teamid:[teamID here]`**.

## Доступ до ключового ланцюга

### `security`
```bash
# List keychains
security list-keychains

# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S

# Dump specifically the user keychain
security dump-keychain ~/Library/Keychains/login.keychain-db
```
### APIs

{% hint style="success" %}
Перерахунок та вивантаження секретів з **keychain**, які **не викликають запит**, можна виконати за допомогою інструмента [**LockSmith**](https://github.com/its-a-feature/LockSmith)

Інші кінцеві точки API можна знайти в [**SecKeyChain.h**](https://opensource.apple.com/source/libsecurity\_keychain/libsecurity\_keychain-55017/lib/SecKeychain.h.auto.html) вихідному коді.
{% endhint %}

Список та отримання **інформації** про кожен запис у keychain за допомогою **Security Framework** або ви також можете перевірити відкритий інструмент командного рядка Apple [**security**](https://opensource.apple.com/source/Security/Security-59306.61.1/SecurityTool/macOS/security.c.auto.html)**.** Деякі приклади API:

* API **`SecItemCopyMatching`** надає інформацію про кожен запис, і є кілька атрибутів, які ви можете встановити при його використанні:
* **`kSecReturnData`**: Якщо true, він спробує розшифрувати дані (встановіть на false, щоб уникнути потенційних спливаючих вікон)
* **`kSecReturnRef`**: Отримати також посилання на елемент keychain (встановіть на true, якщо пізніше ви побачите, що можете розшифрувати без спливаючого вікна)
* **`kSecReturnAttributes`**: Отримати метадані про записи
* **`kSecMatchLimit`**: Скільки результатів повернути
* **`kSecClass`**: Який тип запису в keychain

Отримати **ACL** кожного запису:

* За допомогою API **`SecAccessCopyACLList`** ви можете отримати **ACL для елемента keychain**, і він поверне список ACL (таких як `ACLAuhtorizationExportClear` та інші, згадані раніше), де кожен список має:
* Опис
* **Список надійних додатків**. Це може бути:
* Додаток: /Applications/Slack.app
* Бінарний файл: /usr/libexec/airportd
* Група: group://AirPort

Експортувати дані:

* API **`SecKeychainItemCopyContent`** отримує відкритий текст
* API **`SecItemExport`** експортує ключі та сертифікати, але може знадобитися встановити паролі для експорту вмісту в зашифрованому вигляді

І це **вимоги** для того, щоб **експортувати секрет без запиту**:

* Якщо **1+ надійних** додатків у списку:
* Потрібні відповідні **авторизації** (**`Nil`**, або бути **частиною** дозволеного списку додатків у авторизації для доступу до секретної інформації)
* Потрібен код підпису, щоб відповідати **PartitionID**
* Потрібен код підпису, щоб відповідати одному **надійному додатку** (або бути членом правильного KeychainAccessGroup)
* Якщо **всі додатки надійні**:
* Потрібні відповідні **авторизації**
* Потрібен код підпису, щоб відповідати **PartitionID**
* Якщо **немає PartitionID**, тоді це не потрібно

{% hint style="danger" %}
Отже, якщо є **1 додаток у списку**, вам потрібно **впровадити код у цей додаток**.

Якщо **apple** вказано в **partitionID**, ви можете отримати до нього доступ за допомогою **`osascript`**, тому все, що довіряє всім додаткам з apple в partitionID. **`Python`** також можна використовувати для цього.
{% endhint %}

### Два додаткові атрибути

* **Невидимий**: Це булевий прапорець для **приховування** запису з **UI** додатку Keychain
* **Загальний**: Це для зберігання **метаданих** (тому це НЕ ЗАШИФРОВАНО)
* Microsoft зберігав у відкритому тексті всі токени оновлення для доступу до чутливих кінцевих точок.

## References

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

{% hint style="success" %}
Вчіться та практикуйте AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Вчіться та практикуйте GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
