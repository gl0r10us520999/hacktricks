# User Namespace

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Basic Information

User namespace — це функція ядра Linux, яка **забезпечує ізоляцію відображень ідентифікаторів користувачів і груп**, дозволяючи кожному простору користувача мати **власний набір ідентифікаторів користувачів і груп**. Ця ізоляція дозволяє процесам, що працюють у різних просторах користувачів, **мати різні привілеї та власність**, навіть якщо вони мають однакові числові ідентифікатори користувачів і груп.

Простори користувачів особливо корисні в контейнеризації, де кожен контейнер повинен мати свій незалежний набір ідентифікаторів користувачів і груп, що дозволяє забезпечити кращу безпеку та ізоляцію між контейнерами та хост-системою.

### How it works:

1. Коли створюється новий простір користувача, він **починається з порожнього набору відображень ідентифікаторів користувачів і груп**. Це означає, що будь-який процес, що працює в новому просторі користувача, **спочатку не матиме привілеїв поза межами простору**.
2. Відображення ідентифікаторів можуть бути встановлені між ідентифікаторами користувачів і груп у новому просторі та тими, що в батьківському (або хост) просторі. Це **дозволяє процесам у новому просторі мати привілеї та власність, що відповідають ідентифікаторам користувачів і груп у батьківському просторі**. Однак відображення ідентифікаторів можуть бути обмежені до конкретних діапазонів і підмножин ідентифікаторів, що дозволяє точно контролювати привілеї, надані процесам у новому просторі.
3. У межах простору користувача **процеси можуть мати повні привілеї root (UID 0) для операцій всередині простору**, при цьому все ще маючи обмежені привілеї поза межами простору. Це дозволяє **контейнерам працювати з можливостями, подібними до root, у межах свого власного простору без повних привілеїв root на хост-системі**.
4. Процеси можуть переміщатися між просторами, використовуючи системний виклик `setns()` або створювати нові простори, використовуючи системні виклики `unshare()` або `clone()` з прапором `CLONE_NEWUSER`. Коли процес переходить до нового простору або створює його, він почне використовувати відображення ідентифікаторів користувачів і груп, пов'язані з цим простором.

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -U [--mount-proc] /bin/bash
```
Монтування нової інстанції файлової системи `/proc`, якщо ви використовуєте параметр `--mount-proc`, забезпечує, що новий простір монтування має **точний та ізольований вигляд інформації про процеси, специфічної для цього простору**.

<details>

<summary>Помилка: bash: fork: Не вдається виділити пам'ять</summary>

Коли `unshare` виконується без параметра `-f`, виникає помилка через те, як Linux обробляє нові PID (ідентифікатори процесів) простори. Основні деталі та рішення наведені нижче:

1. **Пояснення проблеми**:
- Ядро Linux дозволяє процесу створювати нові простори за допомогою системного виклику `unshare`. Однак процес, який ініціює створення нового PID простору (який називається "процесом unshare"), не входить до нового простору; лише його дочірні процеси входять.
- Виконання `%unshare -p /bin/bash%` запускає `/bin/bash` в тому ж процесі, що й `unshare`. Відповідно, `/bin/bash` та його дочірні процеси знаходяться в оригінальному PID просторі.
- Перший дочірній процес `/bin/bash` у новому просторі стає PID 1. Коли цей процес завершується, це викликає очищення простору, якщо немає інших процесів, оскільки PID 1 має особливу роль усиновлення сирітських процесів. Ядро Linux тоді вимкне виділення PID у цьому просторі.

2. **Наслідок**:
- Завершення PID 1 у новому просторі призводить до очищення прапора `PIDNS_HASH_ADDING`. Це призводить до того, що функція `alloc_pid` не може виділити новий PID при створенні нового процесу, що викликає помилку "Не вдається виділити пам'ять".

3. **Рішення**:
- Проблему можна вирішити, використовуючи параметр `-f` з `unshare`. Цей параметр змушує `unshare` створити новий процес після створення нового PID простору.
- Виконання `%unshare -fp /bin/bash%` забезпечує, що команда `unshare` сама стає PID 1 у новому просторі. `/bin/bash` та його дочірні процеси тоді безпечно містяться в цьому новому просторі, запобігаючи передчасному завершенню PID 1 та дозволяючи нормальне виділення PID.

Забезпечуючи, що `unshare` виконується з прапором `-f`, новий PID простір правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без виникнення помилки виділення пам'яті.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
Щоб використовувати простір імен користувача, демон Docker потрібно запустити з **`--userns-remap=default`** (в Ubuntu 14.04 це можна зробити, змінивши `/etc/default/docker`, а потім виконавши `sudo service docker restart`)

### &#x20;Перевірте, в якому просторі імен знаходиться ваш процес
```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```
Можна перевірити мапу користувачів з контейнера docker за допомогою:
```bash
cat /proc/self/uid_map
0          0 4294967295  --> Root is root in host
0     231072      65536  --> Root is 231072 userid in host
```
Або з хоста за допомогою:
```bash
cat /proc/<pid>/uid_map
```
### Знайти всі простори імен користувачів

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Увійти в простір користувача
```bash
nsenter -U TARGET_PID --pid /bin/bash
```
Також ви можете **входити в інший простір процесів лише якщо ви root**. І ви **не можете** **входити** в інший простір **без дескриптора**, що вказує на нього (наприклад, `/proc/self/ns/user`).

### Створити новий простір користувачів (з відображеннями)

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
{% endcode %}
```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```
### Відновлення можливостей

У випадку з просторами користувачів, **коли створюється новий простір користувачів, процес, який входить до простору, отримує повний набір можливостей у цьому просторі**. Ці можливості дозволяють процесу виконувати привілейовані операції, такі як **монтування** **файлових систем**, створення пристроїв або зміна власності файлів, але **тільки в контексті його простору користувачів**.

Наприклад, коли у вас є можливість `CAP_SYS_ADMIN` у просторі користувачів, ви можете виконувати операції, які зазвичай вимагають цієї можливості, такі як монтування файлових систем, але тільки в контексті вашого простору користувачів. Будь-які операції, які ви виконуєте з цією можливістю, не вплинуть на хост-систему або інші простори.

{% hint style="warning" %}
Отже, навіть якщо отримання нового процесу всередині нового простору користувачів **дасть вам всі можливості назад** (CapEff: 000001ffffffffff), ви насправді можете **використовувати лише ті, що пов'язані з простором** (монтування, наприклад), але не всі. Тож цього самого по собі недостатньо, щоб втекти з контейнера Docker.
{% endhint %}
```bash
# There are the syscalls that are filtered after changing User namespace with:
unshare -UmCpf  bash

Probando: 0x067 . . . Error
Probando: 0x070 . . . Error
Probando: 0x074 . . . Error
Probando: 0x09b . . . Error
Probando: 0x0a3 . . . Error
Probando: 0x0a4 . . . Error
Probando: 0x0a7 . . . Error
Probando: 0x0a8 . . . Error
Probando: 0x0aa . . . Error
Probando: 0x0ab . . . Error
Probando: 0x0af . . . Error
Probando: 0x0b0 . . . Error
Probando: 0x0f6 . . . Error
Probando: 0x12c . . . Error
Probando: 0x130 . . . Error
Probando: 0x139 . . . Error
Probando: 0x140 . . . Error
Probando: 0x141 . . . Error
```
{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
