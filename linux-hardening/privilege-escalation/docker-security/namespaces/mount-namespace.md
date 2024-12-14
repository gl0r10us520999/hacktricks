# Mount Namespace

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

## Basic Information

Точка монтування простору імен є функцією ядра Linux, яка забезпечує ізоляцію точок монтування файлової системи, які бачить група процесів. Кожен простір імен монтування має свій власний набір точок монтування файлової системи, і **зміни в точках монтування в одному просторі імен не впливають на інші простори імен**. Це означає, що процеси, що виконуються в різних просторах імен монтування, можуть мати різні уявлення про ієрархію файлової системи.

Простори імен монтування особливо корисні в контейнеризації, де кожен контейнер повинен мати свою власну файлову систему та конфігурацію, ізольовану від інших контейнерів і хост-системи.

### How it works:

1. Коли створюється новий простір імен монтування, він ініціалізується з **копією точок монтування з його батьківського простору імен**. Це означає, що при створенні новий простір імен ділить те ж саме уявлення про файлову систему, що й його батько. Однак будь-які подальші зміни в точках монтування в межах простору імен не вплинуть на батьківський або інші простори імен.
2. Коли процес змінює точку монтування в межах свого простору імен, наприклад, монтує або демонтує файлову систему, **зміна є локальною для цього простору імен** і не впливає на інші простори імен. Це дозволяє кожному простору імен мати свою власну незалежну ієрархію файлової системи.
3. Процеси можуть переміщатися між просторами імен, використовуючи системний виклик `setns()`, або створювати нові простори імен, використовуючи системні виклики `unshare()` або `clone()` з прапором `CLONE_NEWNS`. Коли процес переходить до нового простору імен або створює його, він почне використовувати точки монтування, пов'язані з цим простором імен.
4. **Файлові дескриптори та іноди діляться між просторами імен**, що означає, що якщо процес в одному просторі імен має відкритий файловий дескриптор, що вказує на файл, він може **передати цей файловий дескриптор** процесу в іншому просторі імен, і **обидва процеси отримають доступ до одного й того ж файлу**. Однак шлях до файлу може не бути однаковим в обох просторах імен через різницю в точках монтування.

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -m [--mount-proc] /bin/bash
```
При монтуванні нового екземпляра файлової системи `/proc`, якщо ви використовуєте параметр `--mount-proc`, ви забезпечуєте, що новий простір монтування має **точний та ізольований вигляд інформації про процеси, специфічної для цього простору**.

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

Забезпечивши, що `unshare` виконується з прапором `-f`, новий PID простір правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без виникнення помилки виділення пам'яті.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;Перевірте, в якому просторі імен знаходиться ваш процес
```bash
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```
### Знайти всі простори монтування

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

{% code overflow="wrap" %}
```bash
findmnt
```
{% endcode %}

### Увійти всередину простору імен монтування
```bash
nsenter -m TARGET_PID --pid /bin/bash
```
Також ви можете **входити в інший простір процесів лише якщо ви root**. І ви **не можете** **входити** в інший простір **без дескриптора**, що вказує на нього (наприклад, `/proc/self/ns/mnt`).

Оскільки нові монтування доступні лише в межах простору, можливо, що простір містить чутливу інформацію, яка може бути доступна лише з нього.

### Монтувати щось
```bash
# Generate new mount ns
unshare -m /bin/bash
mkdir /tmp/mount_ns_example
mount -t tmpfs tmpfs /tmp/mount_ns_example
mount | grep tmpfs # "tmpfs on /tmp/mount_ns_example"
echo test > /tmp/mount_ns_example/test
ls /tmp/mount_ns_example/test # Exists

# From the host
mount | grep tmpfs # Cannot see "tmpfs on /tmp/mount_ns_example"
ls /tmp/mount_ns_example/test # Doesn't exist
```

```
# findmnt # List existing mounts
TARGET                                SOURCE                                                                                                           FSTYPE     OPTIONS
/                                     /dev/mapper/web05--vg-root

# unshare --mount  # run a shell in a new mount namespace
# mount --bind /usr/bin/ /mnt/
# ls /mnt/cp
/mnt/cp
# exit  # exit the shell, and hence the mount namespace
# ls /mnt/cp
ls: cannot access '/mnt/cp': No such file or directory

## Notice there's different files in /tmp
# ls /tmp
revshell.elf

# ls /mnt/tmp
krb5cc_75401103_X5yEyy
systemd-private-3d87c249e8a84451994ad692609cd4b6-apache2.service-77w9dT
systemd-private-3d87c249e8a84451994ad692609cd4b6-systemd-resolved.service-RnMUhT
systemd-private-3d87c249e8a84451994ad692609cd4b6-systemd-timesyncd.service-FAnDql
vmware-root_662-2689143848

```
## Посилання
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)
* [https://unix.stackexchange.com/questions/464033/understanding-how-mount-namespaces-work-in-linux](https://unix.stackexchange.com/questions/464033/understanding-how-mount-namespaces-work-in-linux)


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
