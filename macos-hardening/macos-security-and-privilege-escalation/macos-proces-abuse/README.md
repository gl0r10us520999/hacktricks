# macOS Process Abuse

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **Bize katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya **bizi** **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## İşlemler Temel Bilgiler

Bir işlem, çalışan bir yürütülebilir dosyanın bir örneğidir, ancak işlemler kod çalıştırmaz, bunlar ipliklerdir. Bu nedenle **işlemler sadece çalışan iplikler için konteynerdir** ve bellek, tanımlayıcılar, portlar, izinler sağlar...

Geleneksel olarak, işlemler diğer işlemler içinde (PID 1 hariç) **`fork`** çağrılarak başlatılır, bu da mevcut işlemin tam bir kopyasını oluşturur ve ardından **çocuk işlem** genellikle yeni yürütülebilir dosyayı yüklemek ve çalıştırmak için **`execve`** çağrısını yapar. Daha sonra, bu işlemi bellek kopyalamadan daha hızlı hale getirmek için **`vfork`** tanıtıldı.\
Ardından **`posix_spawn`** tanıtıldı ve **`vfork`** ile **`execve`**'yi tek bir çağrıda birleştirerek bayraklar kabul etti:

* `POSIX_SPAWN_RESETIDS`: Etkili kimlikleri gerçek kimliklere sıfırla
* `POSIX_SPAWN_SETPGROUP`: İşlem grup üyeliğini ayarla
* `POSUX_SPAWN_SETSIGDEF`: Sinyal varsayılan davranışını ayarla
* `POSIX_SPAWN_SETSIGMASK`: Sinyal maskesini ayarla
* `POSIX_SPAWN_SETEXEC`: Aynı işlemde exec (daha fazla seçenekle `execve` gibi)
* `POSIX_SPAWN_START_SUSPENDED`: Askıya alınmış olarak başlat
* `_POSIX_SPAWN_DISABLE_ASLR`: ASLR olmadan başlat
* `_POSIX_SPAWN_NANO_ALLOCATOR:` libmalloc'un Nano allocator'ını kullan
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Veri segmentlerinde `rwx`'e izin ver
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: exec(2) ile varsayılan olarak tüm dosya tanımlayıcılarını kapat
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` ASLR kaydırmasının yüksek bitlerini rastgeleleştir

Ayrıca, `posix_spawn`, oluşturulan işlemin bazı yönlerini kontrol eden bir **`posix_spawnattr`** dizisi belirtmeye ve tanımlayıcıların durumunu değiştirmek için **`posix_spawn_file_actions`** kullanmaya olanak tanır.

Bir işlem öldüğünde, **ana işleme dönüş kodunu** (eğer ana işlem öldüyse, yeni ana işlem PID 1'dir) `SIGCHLD` sinyali ile gönderir. Ana işlem bu değeri `wait4()` veya `waitid()` çağrısını yaparak almalıdır ve bu gerçekleşene kadar çocuk bir zombi durumunda kalır; hala listelenir ancak kaynak tüketmez.

### PID'ler

PID'ler, işlem tanımlayıcıları, benzersiz bir işlemi tanımlar. XNU'da **PID'ler** **64bit** olup monotonik olarak artar ve **asla sarılmaz** (suistimalleri önlemek için).

### İşlem Grupları, Oturumlar & Koalisyonlar

**İşlemler**, onları yönetmeyi kolaylaştırmak için **gruplara** yerleştirilebilir. Örneğin, bir kabuk betiğindeki komutlar aynı işlem grubunda olacaktır, böylece örneğin kill kullanarak **birlikte sinyal göndermek** mümkündür.\
Ayrıca, işlemleri **oturumlarda gruplamak** da mümkündür. Bir işlem bir oturum başlattığında (`setsid(2)`), çocuk işlemler oturum içinde ayarlanır, aksi takdirde kendi oturumlarını başlatmadıkları sürece.

Koalisyon, Darwin'de işlemleri gruplamanın başka bir yoludur. Bir koalisyona katılan bir işlem, havuz kaynaklarına erişim sağlar, bir defter paylaşır veya Jetsam ile karşılaşır. Koalisyonların farklı rolleri vardır: Lider, XPC hizmeti, Uzantı.

### Kimlik Bilgileri & Kişilikler

Her işlem, sistemdeki **ayrıcalıklarını tanımlayan** **kimlik bilgilerini** taşır. Her işlem birincil bir `uid` ve birincil bir `gid`'ye sahip olacaktır (ancak birden fazla gruba ait olabilir).\
Eğer ikili dosya `setuid/setgid` bitine sahipse, kullanıcı ve grup kimliğini değiştirmek de mümkündür.\
Yeni `uid`/`gid` ayarlamak için birkaç işlev vardır.

Syscall **`persona`**, **alternatif** bir **kimlik bilgileri** seti sağlar. Bir kişiliği benimsemek, onun `uid`, `gid` ve grup üyeliklerini **bir anda** üstlenmeyi gerektirir. [**kaynak kodunda**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h) yapı bulunabilir:
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
## Thread'lerin Temel Bilgileri

1. **POSIX Thread'leri (pthreads):** macOS, C/C++ için standart bir threading API'sinin parçası olan POSIX thread'lerini (`pthreads`) destekler. macOS'taki pthread'lerin implementasyonu `/usr/lib/system/libsystem_pthread.dylib` dosyasında bulunur ve bu, kamuya açık `libpthread` projesinden gelmektedir. Bu kütüphane, thread'leri oluşturmak ve yönetmek için gerekli fonksiyonları sağlar.
2. **Thread Oluşturma:** Yeni thread'ler oluşturmak için `pthread_create()` fonksiyonu kullanılır. İçsel olarak, bu fonksiyon XNU çekirdeğine özgü daha düşük seviyeli bir sistem çağrısı olan `bsdthread_create()`'i çağırır. Bu sistem çağrısı, thread davranışını belirten, planlama politikaları ve yığın boyutu dahil olmak üzere `pthread_attr`'dan (özellikler) türetilen çeşitli bayraklar alır.
* **Varsayılan Yığın Boyutu:** Yeni thread'ler için varsayılan yığın boyutu 512 KB'dir, bu tipik işlemler için yeterlidir ancak daha fazla veya daha az alana ihtiyaç varsa thread özellikleri aracılığıyla ayarlanabilir.
3. **Thread Başlatma:** `__pthread_init()` fonksiyonu, thread kurulumu sırasında kritik öneme sahiptir ve yığın konumu ve boyutu hakkında ayrıntıları içerebilecek ortam değişkenlerini ayrıştırmak için `env[]` argümanını kullanır.

#### macOS'ta Thread Sonlandırma

1. **Thread'leri Sonlandırma:** Thread'ler genellikle `pthread_exit()` çağrılarak sonlandırılır. Bu fonksiyon, bir thread'in temiz bir şekilde çıkmasına, gerekli temizliği yapmasına ve thread'in herhangi bir katılımcıya geri değer göndermesine olanak tanır.
2. **Thread Temizliği:** `pthread_exit()` çağrıldığında, tüm ilişkili thread yapılarının kaldırılmasını yöneten `pthread_terminate()` fonksiyonu çağrılır. Bu, Mach thread portlarını (Mach, XNU çekirdeğindeki iletişim alt sistemidir) serbest bırakır ve thread ile ilişkili çekirdek düzeyindeki yapıları kaldıran bir sistem çağrısı olan `bsdthread_terminate`'i çağırır.

#### Senkronizasyon Mekanizmaları

Paylaşılan kaynaklara erişimi yönetmek ve yarış koşullarını önlemek için macOS, birkaç senkronizasyon ilkesini sağlar. Bu, çoklu thread ortamlarında veri bütünlüğünü ve sistem kararlılığını sağlamak için kritik öneme sahiptir:

1. **Mutex'ler:**
* **Normal Mutex (İmza: 0x4D555458):** 60 baytlık bellek ayak izi olan standart mutex (56 bayt mutex için ve 4 bayt imza için).
* **Hızlı Mutex (İmza: 0x4d55545A):** Normal mutex'e benzer ancak daha hızlı işlemler için optimize edilmiştir, boyutu da 60 bayttır.
2. **Koşul Değişkenleri:**
* Belirli koşulların gerçekleşmesini beklemek için kullanılır, boyutu 44 bayttır (40 bayt artı 4 bayt imza).
* **Koşul Değişkeni Özellikleri (İmza: 0x434e4441):** Koşul değişkenleri için yapılandırma özellikleri, boyutu 12 bayttır.
3. **Bir Kez Değişkeni (İmza: 0x4f4e4345):**
* Bir başlangıç kodunun yalnızca bir kez çalıştırılmasını sağlar. Boyutu 12 bayttır.
4. **Okuma-Yazma Kilitleri:**
* Aynı anda birden fazla okuyucu veya bir yazar olmasına izin verir, paylaşılan verilere verimli erişimi kolaylaştırır.
* **Okuma Yazma Kilidi (İmza: 0x52574c4b):** Boyutu 196 bayttır.
* **Okuma Yazma Kilidi Özellikleri (İmza: 0x52574c41):** Okuma-yazma kilitleri için özellikler, boyutu 20 bayttır.

{% hint style="success" %}
Bu nesnelerin son 4 baytı taşmaları tespit etmek için kullanılır.
{% endhint %}

### Thread Yerel Değişkenler (TLV)

**Thread Yerel Değişkenler (TLV)**, Mach-O dosyaları (macOS'taki yürütülebilir dosyaların formatı) bağlamında, çoklu thread'li bir uygulamada **her thread'e özgü** değişkenleri tanımlamak için kullanılır. Bu, her thread'in bir değişkenin kendi ayrı örneğine sahip olmasını sağlar ve çakışmaları önleyerek veri bütünlüğünü koruma imkanı sunar, mutex'ler gibi açık senkronizasyon mekanizmalarına ihtiyaç duymadan.

C ve ilgili dillerde, bir thread-yerli değişkeni **`__thread`** anahtar kelimesi kullanılarak tanımlanabilir. İşte örneğinizde nasıl çalıştığı:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
Bu kesit, `tlv_var`'ı bir thread-local değişken olarak tanımlar. Bu kodu çalıştıran her thread'in kendi `tlv_var`'ı olacak ve bir thread'in `tlv_var`'da yaptığı değişiklikler diğer bir thread'teki `tlv_var`'ı etkilemeyecektir.

Mach-O ikili dosyasında, thread local değişkenlerle ilgili veriler belirli bölümlere organize edilmiştir:

* **`__DATA.__thread_vars`**: Bu bölüm, thread-local değişkenler hakkında türleri ve başlatma durumları gibi meta verileri içerir.
* **`__DATA.__thread_bss`**: Bu bölüm, açıkça başlatılmamış thread-local değişkenler için kullanılır. Sıfır ile başlatılmış veriler için ayrılmış bir bellek parçasıdır.

Mach-O ayrıca bir thread çıkarken thread-local değişkenleri yönetmek için **`tlv_atexit`** adlı özel bir API sağlar. Bu API, bir thread sona erdiğinde thread-local verileri temizleyen **yıkıcıları** kaydetmenize olanak tanır.

### Thread Öncelikleri

Thread önceliklerini anlamak, işletim sisteminin hangi thread'lerin ne zaman çalıştırılacağına nasıl karar verdiğine bakmayı içerir. Bu karar, her thread'e atanan öncelik seviyesi tarafından etkilenir. macOS ve Unix benzeri sistemlerde, bu `nice`, `renice` ve Hizmet Kalitesi (QoS) sınıfları gibi kavramlar kullanılarak yönetilir.

#### Nice ve Renice

1. **Nice:**
* Bir sürecin `nice` değeri, önceliğini etkileyen bir sayıdır. Her sürecin -20 (en yüksek öncelik) ile 19 (en düşük öncelik) arasında bir nice değeri vardır. Bir süreç oluşturulduğunda varsayılan nice değeri genellikle 0'dır.
* Daha düşük bir nice değeri ( -20'ye daha yakın) bir süreci daha "bencil" hale getirir ve diğer daha yüksek nice değerine sahip süreçlere kıyasla daha fazla CPU süresi alır.
2. **Renice:**
* `renice`, zaten çalışan bir sürecin nice değerini değiştirmek için kullanılan bir komuttur. Bu, süreçlerin önceliğini dinamik olarak ayarlamak için kullanılabilir; yeni nice değerlerine göre CPU zaman tahsisatını artırabilir veya azaltabilir.
* Örneğin, bir sürecin geçici olarak daha fazla CPU kaynağına ihtiyacı varsa, `renice` kullanarak nice değerini düşürebilirsiniz.

#### Hizmet Kalitesi (QoS) Sınıfları

QoS sınıfları, özellikle **Grand Central Dispatch (GCD)**'yi destekleyen macOS gibi sistemlerde thread önceliklerini yönetmenin daha modern bir yaklaşımıdır. QoS sınıfları, geliştiricilerin işleri önem veya aciliyetlerine göre farklı seviyelere **kategorize** etmelerine olanak tanır. macOS, bu QoS sınıflarına dayalı olarak thread önceliklendirmesini otomatik olarak yönetir:

1. **Kullanıcı Etkileşimli:**
* Bu sınıf, şu anda kullanıcı ile etkileşimde bulunan veya iyi bir kullanıcı deneyimi sağlamak için hemen sonuçlar gerektiren görevler içindir. Bu görevler, arayüzün yanıt vermesini sağlamak için en yüksek önceliği alır (örneğin, animasyonlar veya olay işleme).
2. **Kullanıcı Başlatmalı:**
* Kullanıcının başlattığı ve hemen sonuç beklediği görevler, örneğin bir belge açma veya hesaplamalar gerektiren bir düğmeye tıklama gibi. Bunlar yüksek önceliklidir ancak kullanıcı etkileşimli olanların altındadır.
3. **Yardımcı:**
* Bu görevler uzun süreli olup genellikle bir ilerleme göstergesi gösterir (örneğin, dosya indirme, veri içe aktarma). Kullanıcı başlatmalı görevlerden daha düşük önceliğe sahiptirler ve hemen bitmeleri gerekmez.
4. **Arka Plan:**
* Bu sınıf, arka planda çalışan ve kullanıcıya görünmeyen görevler içindir. Bunlar, dizin oluşturma, senkronizasyon veya yedekleme gibi görevler olabilir. En düşük önceliğe sahiptirler ve sistem performansı üzerinde minimal etki yaparlar.

QoS sınıflarını kullanarak, geliştiricilerin tam öncelik numaralarını yönetmeleri gerekmez, bunun yerine görevin doğasına odaklanırlar ve sistem buna göre CPU kaynaklarını optimize eder.

Ayrıca, zamanlayıcının dikkate alacağı bir dizi zamanlama parametresi belirlemek için farklı **thread zamanlama politikaları** vardır. Bu, `thread_policy_[set/get]` kullanılarak yapılabilir. Bu, yarış durumu saldırılarında faydalı olabilir.

## MacOS Süreç İstismarı

MacOS, diğer işletim sistemleri gibi, **süreçlerin etkileşimde bulunması, iletişim kurması ve veri paylaşması** için çeşitli yöntemler ve mekanizmalar sağlar. Bu teknikler, sistemin verimli çalışması için gerekli olsa da, tehdit aktörleri tarafından **kötü niyetli faaliyetler** gerçekleştirmek için de istismar edilebilir.

### Kütüphane Enjeksiyonu

Kütüphane Enjeksiyonu, bir saldırganın **bir süreci kötü niyetli bir kütüphaneyi yüklemeye zorladığı** bir tekniktir. Enjekte edildikten sonra, kütüphane hedef sürecin bağlamında çalışır ve saldırgana sürecin sahip olduğu izinler ve erişim ile aynı hakları sağlar.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Fonksiyon Hooking

Fonksiyon Hooking, bir yazılım kodu içindeki **fonksiyon çağrılarını** veya mesajları **yakalamayı** içerir. Fonksiyonları hooklayarak, bir saldırgan bir sürecin **davranışını değiştirebilir**, hassas verileri gözlemleyebilir veya hatta yürütme akışını kontrol edebilir.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Süreçler Arası İletişim

Süreçler Arası İletişim (IPC), ayrı süreçlerin **veri paylaşma ve değiştirme** yöntemlerini ifade eder. IPC, birçok meşru uygulama için temel olsa da, süreç izolasyonunu altüst etmek, hassas bilgileri sızdırmak veya yetkisiz eylemler gerçekleştirmek için de kötüye kullanılabilir.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Electron Uygulamaları Enjeksiyonu

Belirli çevresel değişkenlerle çalıştırılan Electron uygulamaları, süreç enjeksiyonuna karşı savunmasız olabilir:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Chromium Enjeksiyonu

`--load-extension` ve `--use-fake-ui-for-media-stream` bayraklarını kullanarak **tarayıcıda adam saldırısı** gerçekleştirmek mümkündür; bu, tuş vuruşlarını, trafiği, çerezleri çalmaya, sayfalara script enjekte etmeye olanak tanır:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Kirli NIB

NIB dosyaları, bir uygulama içindeki **kullanıcı arayüzü (UI) öğelerini** ve etkileşimlerini tanımlar. Ancak, **keyfi komutlar çalıştırabilirler** ve **Gatekeeper**, bir **NIB dosyası değiştirildiğinde** zaten çalıştırılmış bir uygulamanın çalışmasını durdurmaz. Bu nedenle, keyfi programların keyfi komutlar çalıştırmasını sağlamak için kullanılabilirler:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Java Uygulamaları Enjeksiyonu

Belirli java yeteneklerini (örneğin, **`_JAVA_OPTS`** çevresel değişkeni) kötüye kullanarak bir java uygulamasının **keyfi kod/komutlar** çalıştırmasını sağlamak mümkündür.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Net Uygulamaları Enjeksiyonu

**.Net hata ayıklama işlevselliğini** (macOS korumaları gibi çalışma zamanı sertleştirmesi tarafından korunmayan) kötüye kullanarak .Net uygulamalarına kod enjekte etmek mümkündür.

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Perl Enjeksiyonu

Bir Perl scriptinin keyfi kod çalıştırmasını sağlamak için farklı seçenekleri kontrol edin:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Ruby Enjeksiyonu

Keyfi scriptlerin keyfi kod çalıştırmasını sağlamak için ruby çevresel değişkenlerini kötüye kullanmak da mümkündür:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Python Enjeksiyonu

Eğer çevresel değişken **`PYTHONINSPECT`** ayarlanmışsa, python süreci tamamlandığında bir python cli'ye geçecektir. Ayrıca, etkileşimli bir oturumun başında çalıştırılacak bir python scriptini belirtmek için **`PYTHONSTARTUP`** kullanmak da mümkündür.\
Ancak, **`PYTHONSTARTUP`** scripti, **`PYTHONINSPECT`** etkileşimli oturumu oluşturduğunda çalıştırılmayacaktır.

**`PYTHONPATH`** ve **`PYTHONHOME`** gibi diğer çevresel değişkenler de bir python komutunun keyfi kod çalıştırmasını sağlamak için faydalı olabilir.

**`pyinstaller`** ile derlenmiş yürütülebilir dosyaların, gömülü bir python kullanıyor olsalar bile bu çevresel değişkenleri kullanmayacağını unutmayın.

{% hint style="danger" %}
Genel olarak, çevresel değişkenleri kötüye kullanarak python'un keyfi kod çalıştırmasını sağlamak için bir yol bulamadım.\
Ancak, çoğu insan **Hombrew** kullanarak python'u kurar; bu, python'u varsayılan admin kullanıcı için **yazılabilir bir konuma** kurar. Bunu şöyle ele geçirebilirsiniz:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Even **root** will run this code when running python.
{% endhint %}

## Tespit

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) **proses enjeksiyonunu** tespit edip engelleyebilen açık kaynak bir uygulamadır:

* **Çevresel Değişkenler** Kullanarak: Aşağıdaki çevresel değişkenlerin varlığını izler: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** ve **`ELECTRON_RUN_AS_NODE`**
* **`task_for_pid`** çağrıları kullanarak: Bir sürecin başka birinin **görev portunu** almak istediğini bulmak için, bu da kodun sürece enjekte edilmesine olanak tanır.
* **Electron uygulama parametreleri**: Birisi **`--inspect`**, **`--inspect-brk`** ve **`--remote-debugging-port`** komut satırı argümanlarını kullanarak bir Electron uygulamasını hata ayıklama modunda başlatabilir ve böylece ona kod enjekte edebilir.
* **Sembolik bağlantılar** veya **sert bağlantılar** kullanarak: Genellikle en yaygın kötüye kullanım, **kendi kullanıcı ayrıcalıklarımızla bir bağlantı koymak** ve **bunu daha yüksek bir ayrıcalık** konumuna yönlendirmektir. Hem sert bağlantılar hem de sembolik bağlantılar için tespit çok basittir. Bağlantıyı oluşturan sürecin **hedef dosyadan farklı bir ayrıcalık seviyesi** varsa, bir **uyarı** oluştururuz. Ne yazık ki, sembolik bağlantılar durumunda engelleme mümkün değildir, çünkü bağlantının oluşturulmasından önce hedefi hakkında bilgiye sahip değiliz. Bu, Apple’ın EndpointSecurity çerçevesinin bir sınırlamasıdır.

### Diğer süreçler tarafından yapılan çağrılar

[**bu blog yazısında**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) **`task_name_for_pid`** fonksiyonunu kullanarak diğer **süreçlerin bir sürece kod enjekte ettiğini** tespit etmenin nasıl mümkün olduğunu bulabilirsiniz ve ardından o diğer süreç hakkında bilgi alabilirsiniz.

Bu fonksiyonu çağırmak için, süreci çalıştıranla **aynı uid** olmanız veya **root** olmanız gerektiğini unutmayın (ve bu, sürecin bilgilerini döndürür, kod enjekte etme yolu değil).

## Referanslar

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

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
