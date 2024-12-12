# IPC Namespace

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

## Основна інформація

IPC (Міжпроцесорна комунікація) namespace - це функція ядра Linux, яка забезпечує **ізоляцію** об'єктів IPC System V, таких як черги повідомлень, сегменти спільної пам'яті та семафори. Ця ізоляція гарантує, що процеси в **різних IPC namespaces не можуть безпосередньо отримувати доступ або змінювати об'єкти IPC один одного**, забезпечуючи додатковий рівень безпеки та конфіденційності між групами процесів.

### Як це працює:

1. Коли створюється новий IPC namespace, він починається з **повністю ізольованого набору об'єктів IPC System V**. Це означає, що процеси, що працюють у новому IPC namespace, не можуть отримати доступ або втручатися в об'єкти IPC в інших namespaces або в хост-системі за замовчуванням.
2. Об'єкти IPC, створені в межах namespace, видимі та **доступні лише для процесів у цьому namespace**. Кожен об'єкт IPC ідентифікується унікальним ключем у своєму namespace. Хоча ключ може бути ідентичним у різних namespaces, самі об'єкти ізольовані і не можуть бути доступні між namespaces.
3. Процеси можуть переміщатися між namespaces, використовуючи системний виклик `setns()`, або створювати нові namespaces, використовуючи системні виклики `unshare()` або `clone()` з прапором `CLONE_NEWIPC`. Коли процес переходить до нового namespace або створює його, він почне використовувати об'єкти IPC, пов'язані з цим namespace.

## Лабораторія:

### Створити різні Namespaces

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
Монтування нової інстанції файлової системи `/proc`, якщо ви використовуєте параметр `--mount-proc`, забезпечує, що новий простір монтування має **точний та ізольований вигляд інформації про процеси, специфічної для цього простору**.

<details>

<summary>Помилка: bash: fork: Не вдалося виділити пам'ять</summary>

Коли `unshare` виконується без параметра `-f`, виникає помилка через те, як Linux обробляє нові PID (ідентифікатори процесів) простори. Основні деталі та рішення наведені нижче:

1. **Пояснення проблеми**:
- Ядро Linux дозволяє процесу створювати нові простори за допомогою системного виклику `unshare`. Однак процес, який ініціює створення нового PID простору (який називається "процесом unshare"), не входить до нового простору; лише його дочірні процеси входять.
- Виконання `%unshare -p /bin/bash%` запускає `/bin/bash` в тому ж процесі, що й `unshare`. Відповідно, `/bin/bash` та його дочірні процеси знаходяться в оригінальному PID просторі.
- Перший дочірній процес `/bin/bash` у новому просторі стає PID 1. Коли цей процес завершується, це викликає очищення простору, якщо немає інших процесів, оскільки PID 1 має особливу роль усиновлення сирітських процесів. Ядро Linux тоді вимкне виділення PID у цьому просторі.

2. **Наслідок**:
- Завершення PID 1 у новому просторі призводить до очищення прапора `PIDNS_HASH_ADDING`. Це призводить до того, що функція `alloc_pid` не може виділити новий PID при створенні нового процесу, що викликає помилку "Не вдалося виділити пам'ять".

3. **Рішення**:
- Проблему можна вирішити, використовуючи параметр `-f` з `unshare`. Цей параметр змушує `unshare` створити новий процес після створення нового PID простору.
- Виконання `%unshare -fp /bin/bash%` забезпечує, що команда `unshare` сама стає PID 1 у новому просторі. `/bin/bash` та його дочірні процеси тоді безпечно містяться в цьому новому просторі, запобігаючи передчасному завершенню PID 1 та дозволяючи нормальне виділення PID.

Забезпечивши, що `unshare` виконується з прапором `-f`, новий PID простір правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без виникнення помилки виділення пам'яті.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;Перевірте, в якому просторі імен знаходиться ваш процес
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### Знайти всі IPC простори імен

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Увійти в IPC простір імен
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
Також ви можете **входити в інший простір процесів лише якщо ви root**. І ви **не можете** **входити** в інший простір **без дескриптора**, що вказує на нього (наприклад, `/proc/self/ns/net`).

### Створити об'єкт IPC
```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
```
## Посилання
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
</details>
{% endhint %}
