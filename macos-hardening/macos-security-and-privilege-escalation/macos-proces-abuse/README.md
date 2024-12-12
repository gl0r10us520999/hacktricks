# macOS İşlem Kötüye Kullanımı

{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitimi AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitimi GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) katılın veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarını paylaşarak PR göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>
{% endhint %}

## İşlemler Temel Bilgileri

Bir işlem, çalışan bir yürütülebilir dosyanın bir örneğidir, ancak işlemler kod çalıştırmaz, bunlar thread'lerdir. Bu nedenle **işlemler yalnızca çalışan thread'ler için konteynerlerdir**, belleği, tanımlayıcıları, bağlantı noktalarını, izinleri sağlarlar...

Geleneksel olarak, işlemler diğer işlemler içinde (PID 1 hariç) **`fork`** çağrısı yaparak başlatılırdı, bu işlem mevcut işlemi tam bir kopya oluşturur ve ardından **çocuk işlem** genellikle yeni yürütülebilir dosyayı yüklemek ve çalıştırmak için **`execve`** çağrısını yapardı. Daha sonra, bu işlemi hafızaya kopyalamadan daha hızlı hale getirmek için **`vfork`** tanıtıldı.\
Daha sonra **`posix_spawn`** tanıtıldı, **`vfork`** ve **`execve`**'yi bir araya getirerek tek bir çağrıda birleştirdi ve bayrakları kabul etti:

* `POSIX_SPAWN_RESETIDS`: Etkili kimlikleri gerçek kimliklere sıfırlar
* `POSIX_SPAWN_SETPGROUP`: İşlem grubu üyeliğini ayarlar
* `POSUX_SPAWN_SETSIGDEF`: Sinyal varsayılan davranışını ayarlar
* `POSIX_SPAWN_SETSIGMASK`: Sinyal maskesini ayarlar
* `POSIX_SPAWN_SETEXEC`: Aynı işlemde yürütme (`execve` ile daha fazla seçenekli)
* `POSIX_SPAWN_START_SUSPENDED`: Askıya alınmış olarak başlat
* `_POSIX_SPAWN_DISABLE_ASLR`: ASLR olmadan başlat
* `_POSIX_SPAWN_NANO_ALLOCATOR:` libmalloc'ın Nano tahsisatçısını kullan
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Veri segmentlerinde `rwx`'e izin ver
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: Varsayılan olarak exec(2) ile tüm dosya tanımlamalarını kapat
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` ASLR kaydırmasının yüksek bitlerini rastgeleleştir

Ayrıca, `posix_spawn`, başlatılan işlemin bazı yönlerini kontrol eden bir dizi **`posix_spawnattr`** belirtmeye ve tanımlayıcıların durumunu değiştirmek için **`posix_spawn_file_actions`**'ı değiştirmeye olanak tanır.

Bir işlem öldüğünde, **dönüş kodunu ebeveyn işleme** (ebeveyn öldüyse, yeni ebeveyn PID 1'dir) `SIGCHLD` sinyali ile gönderir. Ebeveyn bu değeri `wait4()` veya `waitid()` çağrısını yaparak almalı ve bu gerçekleşene kadar çocuk, hala listelenmiş ancak kaynak tüketmeyen bir zombi durumunda kalır.

### PID'ler

PID'ler, işlem tanımlayıcıları, benzersiz bir işlemi tanımlar. XNU'da **PID'ler** **64 bit**'dir, monotonik olarak artar ve **asla sarılmaz** (istismarları önlemek için).

### İşlem Grupları, Oturumlar ve Koalisyonlar

**İşlemler**, onları daha kolay işlemek için **gruplara** yerleştirilebilir. Örneğin, bir kabuk betiğindeki komutlar aynı işlem grubunda olacak, bu nedenle örneğin kill kullanarak **onlara birlikte sinyal göndermek mümkün olacaktır**.\
Ayrıca, işlemleri **oturumlarda gruplandırmak mümkündür**. Bir işlem bir oturum başlattığında (`setsid(2)`), çocuk işlemler oturumun içine yerleştirilir, onlar kendi oturumlarını başlatmadıkça.

Koalisyon, Darwin'de işlemleri gruplamanın başka bir yoludur. Bir koalisyona katılan bir işlem, havuz kaynaklarına erişim sağlar, bir defteri paylaşır veya Jetsam ile karşılaşır. Koalisyonların farklı rolleri vardır: Lider, XPC hizmeti, Uzantı.

### Kimlik Bilgileri ve Personalar

Her işlem, sistemdeki ayrıcalıklarını tanımlayan **kimlik bilgilerini** tutar. Her işlemin birincil `uid` ve birincil `gid`'si olacaktır (ancak birkaç gruba ait olabilir).\
Binary'nin `setuid/setgid` bitine sahip olması durumunda kullanıcı ve grup kimliğini değiştirmek mümkündür.\
Yeni uid/gid'ler belirlemek için birkaç işlev vardır.

Syscall **`persona`**, bir **alternatif** kimlik bilgisi kümesi sağlar. Bir persona'yı benimsemek, uid'sini, gid'sini ve grup üyeliklerini **tek seferde** alır. [**Kaynak kodunda**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h) yapıyı bulmak mümkündür.
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## İplikler Temel Bilgileri

1. **POSIX İplikler (pthreads):** macOS, C/C++ için standart bir iplik API'si olan POSIX iplikleri (`pthreads`) destekler. macOS'taki pthreads uygulaması, `/usr/lib/system/libsystem_pthread.dylib` içinde bulunur ve genelde bulunan `libpthread` projesinden gelir. Bu kütüphane iplik oluşturmak ve yönetmek için gerekli işlevleri sağlar.
2. **İplik Oluşturma:** Yeni iplikler oluşturmak için `pthread_create()` işlevi kullanılır. Bu işlev, içsel olarak `bsdthread_create()` işlevini çağırır, bu işlev XNU çekirdeğine (macOS'un temel aldığı çekirdek) özgü düşük seviyeli bir sistem çağrısıdır. Bu sistem çağrısı, iplik davranışını belirten `pthread_attr` (nitelikler) türetilmiş çeşitli bayrakları alır, bu davranışlar arasında zamanlama politikaları ve yığın boyutu bulunur.
* **Varsayılan Yığın Boyutu:** Yeni iplikler için varsayılan yığın boyutu 512 KB'dir, tipik işlemler için yeterli olsa da daha fazla veya daha az alan gerekiyorsa iplik nitelikleri aracılığıyla ayarlanabilir.
3. **İplik Başlatma:** `__pthread_init()` işlevi, iplik kurulumu sırasında önemlidir ve yığının konumu ve boyutu gibi ayrıntıları içerebilen çevre değişkenlerini ayrıştırmak için `env[]` argümanını kullanır.

#### macOS'ta İplik Sonlandırma

1. **İpliklerin Sonlandırılması:** İplikler genellikle `pthread_exit()` çağrılarak sonlandırılır. Bu işlev, bir ipliğin temiz bir şekilde çıkmasına izin verir, gerekli temizliği yapar ve ipliği bekleyenlere bir dönüş değeri göndermesine olanak tanır.
2. **İplik Temizliği:** `pthread_exit()` çağrıldığında, `pthread_terminate()` işlevi çağrılır ve tüm ilişkili iplik yapılarının kaldırılmasını ele alır. Bu işlev, Mach iplik bağlantı noktalarını (Mach, XNU çekirdeğindeki iletişim alt sistemi) serbest bırakır ve iplikle ilişkili çekirdek düzeyindeki yapıları kaldıran bir sistem çağrısı olan `bsdthread_terminate` işlevini çağırır.

#### Senkronizasyon Mekanizmaları

Paylaşılan kaynaklara erişimi yönetmek ve yarış koşullarını önlemek için macOS, birkaç senkronizasyon ilkesi sağlar. Bu, veri bütünlüğünü ve sistem kararlılığını sağlamak için çoklu iplik ortamlarında kritiktir:

1. **Müteksinler:**
* **Normal Müteksin (İmza: 0x4D555458):** 60 bayt (müteksin için 56 bayt ve imza için 4 bayt) bellek izine sahip standart müteksin.
* **Hızlı Müteksin (İmza: 0x4d55545A):** Normal müteksine benzer ancak daha hızlı işlemler için optimize edilmiş, aynı zamanda 60 bayttır.
2. **Durum Değişkenleri:**
* Belirli koşulların gerçekleşmesini beklemek için kullanılır, 44 bayt büyüklüğündedir (40 bayt artı 4 bayt imza).
* **Durum Değişkeni Nitelikleri (İmza: 0x434e4441):** Durum değişkenleri için yapılandırma nitelikleri, 12 bayt büyüklüğündedir.
3. **Bir Kez Değişkeni (İmza: 0x4f4e4345):**
* Başlatma kodunun yalnızca bir kez yürütülmesini sağlar. Boyutu 12 bayttır.
4. **Okuma-Yazma Kilidi:**
* Aynı anda birden fazla okuyucuya veya bir yazıcıya izin verir, paylaşılan verilere verimli erişimi kolaylaştırır.
* **Okuma Yazma Kilidi (İmza: 0x52574c4b):** 196 bayt büyüklüğündedir.
* **Okuma Yazma Kilidi Nitelikleri (İmza: 0x52574c41):** Okuma-yazma kilitleri için nitelikler, 20 bayt büyüklüğündedir.

{% hint style="success" %}
Bu nesnelerin son 4 baytı taşmaları algılamak için kullanılır.
{% endhint %}

### İplik Yerel Değişkenler (TLV)

**İplik Yerel Değişkenler (TLV)**, macOS'ta yürütülebilir dosyalar için biçim olan Mach-O dosyaları bağlamında, çoklu iplikli bir uygulamada **her iplik** için özgü değişkenlerin bildirilmesinde kullanılır. Bu, her ipliğin değişkenin kendi ayrı örneğine sahip olduğundan emin olur, çakışmaları önlemek ve müteksinler gibi açık senkronizasyon mekanizmalarına gerek duymadan veri bütünlüğünü korumak için bir yol sağlar.

C ve ilgili dillerde, bir iplik yerel değişkeni **`__thread`** anahtar kelimesini kullanarak bildirebilirsiniz. İşte örneğinizde nasıl çalıştığı:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
Bu parça, `tlv_var`'ı bir iş parçacığı yerel değişkeni olarak tanımlar. Bu kodu çalıştıran her iş parçacığı, kendi `tlv_var`'ına sahip olacak ve bir iş parçacığının `tlv_var`'ına yaptığı değişiklikler diğer iş parçacığındaki `tlv_var`'ı etkilemeyecektir.

Mach-O ikili dosyasında, iş parçacığı yerel değişkenlerle ilgili veriler belirli bölümlere düzenlenmiştir:

* **`__DATA.__thread_vars`**: Bu bölüm, iş parçacığı yerel değişkenleri hakkında metadata içerir, türleri ve başlatma durumları gibi.
* **`__DATA.__thread_bss`**: Bu bölüm, açıkça başlatılmayan iş parçacığı yerel değişkenleri için kullanılır. Sıfırlanmış veriler için ayrılan belleğin bir parçasıdır.

Mach-O ayrıca bir iş parçacığının sonlandığında iş parçacığı yerel değişkenlerini yönetmek için **`tlv_atexit`** adında özel bir API sağlar. Bu API, bir iş parçacığı sonlandığında iş parçacığı yerel verileri temizlemek için özel fonksiyonlar olan **yıkıcıları kaydetmenizi** sağlar.

### İş Parçacığı Öncelikleri

İş parçacığı önceliklerini anlamak, işletim sisteminin hangi iş parçacıklarını çalıştıracağına ve ne zaman çalıştıracağına karar verirken hangi önceliğin her iş parçacığına atandığına bakmayı içerir. macOS ve Unix benzeri sistemlerde, bu kavramlar kullanılarak işlenir: `nice`, `renice` ve Kalite Hizmeti (QoS) sınıfları.

#### Nice ve Renice

1. **Nice:**
* Bir işlemin `nice` değeri, önceliğini etkileyen bir numaradır. Her işlemin genellikle 0 olan -20 (en yüksek öncelik) ile 19 (en düşük öncelik) arasında bir güzel değeri vardır. Bir işlem oluşturulduğunda varsayılan güzel değer genellikle 0'dır.
* Daha düşük bir güzel değer (-20'ye daha yakın) bir işlemi daha "bencil" yapar, ona diğer işlemlere göre daha fazla CPU zamanı verir.
2. **Renice:**
* `renice`, zaten çalışan bir işlemin güzel değerini değiştirmek için kullanılan bir komuttur. Bu, işlemlerin önceliğini dinamik olarak ayarlamak için kullanılabilir, yeni güzel değerlere dayanarak CPU zamanı tahsisini artırarak veya azaltarak işlemlerin önceliğini ayarlamak için kullanılabilir.
* Örneğin, bir işlemin geçici olarak daha fazla CPU kaynağına ihtiyacı varsa, `renice` kullanarak güzel değerini düşürebilirsiniz.

#### Kalite Hizmeti (QoS) Sınıfları

QoS sınıfları, özellikle **Grand Central Dispatch (GCD)**'yi destekleyen macOS gibi sistemlerde iş parçacığı önceliklerini ele almanın daha modern bir yaklaşımıdır. QoS sınıfları, işleri önem veya aciliyetlerine göre farklı seviyelere kategorize etmeye olanak tanır. macOS, bu QoS sınıflarına dayanarak iş parçacığı önceliğini otomatik olarak yönetir:

1. **Kullanıcı Etkileşimli:**
* Bu sınıf, şu anda kullanıcıyla etkileşimde olan veya iyi bir kullanıcı deneyimi sağlamak için hemen sonuçlar gerektiren görevler içindir. Bu görevler, arayüzün yanıt vermesini sağlamak için en yüksek önceliği alır (örneğin, animasyonlar veya etkinlik işleme).
2. **Kullanıcı Başlatılan:**
* Kullanıcının başlattığı ve hemen sonuçlar beklediği görevler, belge açma veya hesaplama gerektiren bir düğmeye tıklama gibi. Bunlar yüksek öncelikli ancak kullanıcı etkileşimli görevlerin altındadır.
3. **Yardımcı:**
* Bu görevler uzun süre çalışır ve genellikle bir ilerleme göstergesi gösterir (örneğin, dosyaları indirme, veri içe aktarma). Kullanıcı başlatılan görevlerden daha düşük önceliğe sahiptir ve hemen bitmesi gerekmez.
4. **Arka Plan:**
* Bu sınıf, arka planda çalışan ve kullanıcı tarafından görülmeyen görevler içindir. Bunlar dizinleme, senkronizasyon veya yedekleme gibi görevler olabilir. En düşük önceliğe sahiptirler ve sistem performansı üzerinde minimal etkiye sahiptirler.

QoS sınıflarını kullanarak, geliştiricilerin kesin öncelik numaralarını yönetmeleri gerekmez, ancak görevin doğasına odaklanabilirler ve sistem CPU kaynaklarını buna göre optimize eder.

Ayrıca, **iş parçacığı zamanlama politikaları** adlı farklı politikalar vardır ve bu politikaların dikkate alınacağı bir dizi zamanlama parametresini belirtmek için kullanılır. Bu, `thread_policy_[set/get]` kullanılarak yapılabilir. Bu, yarış koşulu saldırılarında faydalı olabilir.
### Python Enjeksiyonu

Eğer **`PYTHONINSPECT`** ortam değişkeni ayarlanmışsa, python işlemi işini bitirdikten sonra bir python komut satırına düşer. Ayrıca, etkileşimli bir oturumun başında yürütülecek bir python betiğini belirtmek için **`PYTHONSTARTUP`** kullanmak da mümkündür.\
Ancak, **`PYTHONINSPECT`** etkileşimli oturum oluşturduğunda **`PYTHONSTARTUP`** betiği yürütülmeyecektir.

**`PYTHONPATH`** ve **`PYTHONHOME`** gibi diğer ortam değişkenleri de bir python komutunun keyfi kodu yürütmesini sağlamak için kullanışlı olabilir.

**`pyinstaller`** ile derlenen yürütülebilir dosyalar, gömülü bir python kullanıyor olsalar bile bu ortam değişkenlerini kullanmayacaktır.

{% hint style="danger" %}
Genel olarak, ortam değişkenlerini kötüye kullanarak python'un keyfi kod yürütmesini sağlayacak bir yol bulamadım.\
Ancak, insanların çoğu **Hombrew** kullanarak python'u yükler, bu da python'u varsayılan yönetici kullanıcısı için bir **yazılabilir konuma** yükler. Bunu şu şekilde ele geçirebilirsiniz:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
## Tespit

### Kalkan

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)), **environmen değişkenlerini izleyerek** şu işlemleri **tespit edebilen ve engelleyebilen** açık kaynaklı bir uygulamadır:

- **Çevresel Değişkenler Kullanarak**: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** ve **`ELECTRON_RUN_AS_NODE`** değişkenlerinin varlığını izleyecektir.
- **`task_for_pid`** çağrıları Kullanarak: Bir işlemin başka bir işlemin **görev bağlantı noktasını almak istediğinde** (bu, işleme kod enjekte etmeyi sağlar) tespit eder.
- **Electron uygulama parametreleri**: Birisi bir Electron uygulamasını hata ayıklama modunda başlatmak için **`--inspect`**, **`--inspect-brk`** ve **`--remote-debugging-port`** komut satırı argümanlarını kullanabilir ve böylece kod enjekte edebilir.
- **Sembolik bağlantılar** veya **sabit bağlantılar** Kullanarak: Genellikle en yaygın kötüye kullanım, **kullanıcı ayrıcalıklarımızla bir bağlantı oluşturmak** ve **daha yüksek ayrıcalıklı bir konuma işaret etmek**tir. Hem sabit bağlantılar hem de sembolik bağlantılar için tespit çok basittir. Bağlantıyı oluşturan işlem hedef dosyadan **farklı bir ayrıcalık seviyesine** sahipse bir **uyarı** oluşturur. Ne yazık ki sembolik bağlantılar durumunda engelleme mümkün değildir, çünkü bağlantının oluşturulmasından önce bağlantının hedefi hakkında bilgiye sahip değiliz. Bu, Apple'ın EndpointSecuriy çerçevesinin bir kısıtlamasıdır.

### Diğer işlemler tarafından yapılan çağrılar

Bu [**blog yazısında**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) işlemlerin bir işleme kod enjekte ettiği bilgisi hakkında bilgi almak için **`task_name_for_pid`** işlevini kullanmanın mümkün olduğu açıklanmaktadır ve ardından o diğer işlem hakkında bilgi alınmaktadır.

Bu işlevi çağırmak için işlemi çalıştıran kullanıcı kimliğinin **aynı olması** veya **root** olmanız gerektiğini unutmayın (ve bu işlem, kod enjekte etme yöntemi değil, işlem hakkında bilgi döndürür).

## Referanslar

- [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
- [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)
