# macOS Dosyaları, Klasörleri, İkili Dosyalar ve Bellek

{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitimi AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitimi GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 **Discord grubuna** [**katılın**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking püf noktalarını paylaşarak PR'ler göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>
{% endhint %}

## Dosya hiyerarşisi düzeni

* **/Applications**: Yüklenen uygulamalar burada olmalıdır. Tüm kullanıcılar bunlara erişebilmelidir.
* **/bin**: Komut satırı ikilileri
* **/cores**: Var ise, çekirdek dökümlerini depolamak için kullanılır
* **/dev**: Her şey bir dosya olarak işlendiği için burada donanım cihazları bulunabilir.
* **/etc**: Yapılandırma dosyaları
* **/Library**: Tercihler, önbellekler ve günlüklerle ilgili birçok alt dizin ve dosya burada bulunabilir. Bir Library klasörü kök dizinde ve her kullanıcının dizininde bulunur.
* **/private**: Belgelenmemiş ancak bahsedilen birçok klasör özel dizinine sembolik bağlantılardır.
* **/sbin**: Temel sistem ikilileri (yönetimle ilgili)
* **/System**: OS X'in çalışmasını sağlayan dosya. Burada genellikle yalnızca Apple'a özgü dosyalar bulmalısınız (üçüncü taraf değil).
* **/tmp**: Dosyalar 3 gün sonra silinir (bu, /private/tmp'ye bir sembolik bağlantıdır)
* **/Users**: Kullanıcıların ana dizini.
* **/usr**: Yapılandırma ve sistem ikilileri
* **/var**: Günlük dosyaları
* **/Volumes**: Bağlanan sürücüler burada görünecektir.
* **/.vol**: `stat a.txt` komutunu çalıştırarak `16777223 7545753 -rw-r--r-- 1 kullanıcıadı wheel ...` gibi bir şey elde edersiniz, burada ilk sayı dosyanın bulunduğu birim numarası ve ikinci sayı inode numarasıdır. Bu bilgiyi kullanarak bu dosyanın içeriğine /.vol/ üzerinden erişebilirsiniz, `cat /.vol/16777223/7545753` komutunu çalıştırarak.

### Uygulamaların Klasörleri

* **Sistem uygulamaları**, `/System/Applications` altında bulunur.
* **Yüklü** uygulamalar genellikle `/Applications` veya `~/Applications` içinde yüklenir.
* **Uygulama verileri**, kök olarak çalışan uygulamalar için `/Library/Application Support` ve kullanıcı olarak çalışan uygulamalar için `~/Library/Application Support` içinde bulunabilir.
* **Root olarak çalışması gereken üçüncü taraf uygulama hizmetleri**, genellikle `/Library/PrivilegedHelperTools/` içinde bulunur.
* **Kumlanmış** uygulamalar, `~/Library/Containers` klasörüne eşlenir. Her uygulamanın, uygulamanın paket kimliğine (`com.apple.Safari`) göre adlandırılmış bir klasörü vardır.
* **Çekirdek**, `/System/Library/Kernels/kernel` içinde bulunur.
* **Apple'ın çekirdek uzantıları**, `/System/Library/Extensions` içinde bulunur.
* **Üçüncü taraf çekirdek uzantıları**, `/Library/Extensions` içinde saklanır.

### Hassas Bilgiler İçeren Dosyalar

MacOS, şifreler gibi bilgileri çeşitli yerlerde saklar:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Güvenlik Açıklı Paket Yükleyicileri

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X Özel Uzantılar

* **`.dmg`**: Apple Disk Image dosyaları sıkça kullanılır.
* **`.kext`**: Belirli bir yapıyı takip etmelidir ve sürücünün OS X sürümüdür. (bir paket)
* **`.plist`**: XML veya ikili biçimde bilgi saklayan özellik listesi olarak da bilinir.
* XML veya ikili olabilir. İkili olanlar şu şekilde okunabilir:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Dizin yapısını takip eden Apple uygulamaları (bir paket).
* **`.dylib`**: Dinamik kütüphaneler (Windows DLL dosyaları gibi)
* **`.pkg`**: xar (Genişletilebilir Arşiv biçimi) ile aynıdır. İçeriğini yüklemek için installer komutu kullanılabilir.
* **`.DS_Store`**: Bu dosya her dizinde bulunur, dizinin özelliklerini ve özelleştirmelerini kaydeder.
* **`.Spotlight-V100`**: Bu klasör, sistemdeki her birim kök dizininde görünür.
* **`.metadata_never_index`**: Bu dosya bir birimin kökünde bulunursa Spotlight o birimi dizine eklemeyecektir.
* **`.noindex`**: Bu uzantıya sahip dosya ve klasörler Spotlight tarafından dizine eklenmeyecektir.
* **`.sdef`**: Bir uygulamayla AppleScript'ten nasıl etkileşim kurulabileceğini belirten paketler içindeki dosyalar.

### macOS Paketleri

Bir paket, Finder'da bir nesne gibi görünen bir **dizin**dir (Bir Paket örneği `*.app` dosyalarıdır).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Paylaşılan Kütüphane Önbelleği (SLC)

MacOS'ta (ve iOS'ta) tüm sistem paylaşılan kütüphaneleri, çerçeveler ve dylib'ler gibi, **dyld paylaşılan önbelleğe** tek bir dosyada birleştirilir. Bu, kodun daha hızlı yüklenebilmesi nedeniyle performansı artırır.

Bu, macOS'ta `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` içinde bulunur ve eski sürümlerde **paylaşılan önbelleği** **`/System/Library/dyld/`** içinde bulabilirsiniz.\
iOS'ta bunları **`/System/Library/Caches/com.apple.dyld/`** içinde bulabilirsiniz.

Dyld paylaşılan önbelleğinden kütüphaneleri çıkarmak için kullanılabilecek ikili [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) olabilir, ancak günümüzde çalışmayabilir, bu nedenle [**dyldextractor**](https://github.com/arandomdev/dyldextractor) kullanabilirsiniz:

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
`dyld_shared_cache_util` aracının çalışmaması durumunda bile, **paylaşılan dyld ikilisini Hopper'a geçirebilir** ve Hopper tüm kütüphaneleri tanımlayabilir ve incelemek istediğiniz **hangi kütüphaneyi seçmek istediğinizi belirlemenize olanak tanır**:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Bazı çıkartıcılar, dylib'lerin sabit adreslerle önceden bağlantılı olduğu için çalışmayabilir, bu nedenle bilinmeyen adreslere atlayabilirler.

{% hint style="success" %}
Başka bir \*OS cihazının Paylaşılan Kütüphane Önbelleğini Xcode'da bir emülatör kullanarak macOS'ta indirmek de mümkündür. Bunlar şuraya indirilecektir: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, örneğin:`$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### SLC Haritalama

**`dyld`**, SLC'nin eşlenip eşlenmediğini bilmek için **`shared_region_check_np`** sistem çağrısını kullanır (adresi döndürür) ve SLC'yi eşlerken **`shared_region_map_and_slide_np`** sistem çağrısını kullanır.

SLC ilk kullanımda kaydırılsa bile, **tüm işlemler** aynı kopyayı kullanır, bu da saldırganın sistemde işlemleri çalıştırabilmesi durumunda **ASLR korumasını ortadan kaldırır**. Bu aslında geçmişte istismar edilmiş ve paylaşılan bölge sayfa belleği ile düzeltilmiştir.

Şube havuzları, görüntü eşlemeleri arasında küçük boşluklar oluşturan küçük Mach-O dylib'leridir, bu da işlevleri araya girmeyi imkansız hale getirir.

### SLC'leri Geçersiz Kılma

Çevresel değişkenleri kullanarak:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Bu, yeni bir paylaşılan kütüphane önbelleğini yüklemeyi sağlar
* **`DYLD_SHARED_CACHE_DIR=avoid`** ve kütüphaneleri paylaşılan önbelleğe gerçek olanlarla sembollerle değiştirerek manuel olarak (onları çıkartmanız gerekecektir)

## Özel Dosya İzinleri

### Klasör izinleri

Bir **klasörde**, **okuma** onu **listelemenize**, **yazma** onu **silmeye** ve üzerinde dosya **yazmaya** izin verir, ve **yürütme** dizini **gezmeye** izin verir. Örneğin, bir kullanıcının **yürütme izni olmayan bir dizinde** bulunan bir dosya üzerinde **okuma izni varsa**, dosyayı **okuyamaz**.

### Bayrak değiştiricileri

Dosyalara ayarlanabilecek bazı bayraklar vardır ve dosyanın farklı davranmasını sağlar. Bir dizindeki dosyaların bayraklarını `ls -lO /path/directory` ile kontrol edebilirsiniz.

* **`uchg`**: **uchange** bayrağı olarak bilinen bu bayrak, **dosyayı değiştiren veya silen herhangi bir eylemi engeller**. Ayarlamak için: `chflags uchg file.txt`
* Kök kullanıcı bayrağı **kaldırabilir** ve dosyayı değiştirebilir
* **`restricted`**: Bu bayrak dosyanın **SIP tarafından korunmasını sağlar** (bu bayrağı bir dosyaya ekleyemezsiniz).
* **`Sticky bit`**: Bir dizinin yapışkan biti varsa, **yalnızca** dizinin **sahibi veya kök** dosyaları **yeniden adlandırabilir veya silebilir**. Genellikle bu, /tmp dizininde ayarlanır ve sıradan kullanıcıların diğer kullanıcıların dosyalarını silmesini veya taşımasını engeller.

Tüm bayraklar `sys/stat.h` dosyasında bulunabilir (bunu `mdfind stat.h | grep stat.h` kullanarak bulun) ve şunlardır:

* `UF_SETTABLE` 0x0000ffff: Sahibin değiştirebileceği bayraklar maskesi.
* `UF_NODUMP` 0x00000001: Dosyayı dökme.
* `UF_IMMUTABLE` 0x00000002: Dosya değiştirilemez.
* `UF_APPEND` 0x00000004: Dosyalara yalnızca ekleme yapılabilir.
* `UF_OPAQUE` 0x00000008: Birleşim açısından dizin opaktır.
* `UF_COMPRESSED` 0x00000020: Dosya sıkıştırılmıştır (bazı dosya sistemleri).
* `UF_TRACKED` 0x00000040: Bu ayarlı dosyalar için silme/yeniden adlandırma bildirimleri yok.
* `UF_DATAVAULT` 0x00000080: Okuma ve yazma için yetki gereklidir.
* `UF_HIDDEN` 0x00008000: Bu öğenin bir GUI'de gösterilmemesi gerektiğine dair ipucu.
* `SF_SUPPORTED` 0x009f0000: Süper kullanıcı tarafından desteklenen bayraklar maskesi.
* `SF_SETTABLE` 0x3fff0000: Süper kullanıcı tarafından değiştirilebilen bayraklar maskesi.
* `SF_SYNTHETIC` 0xc0000000: Sistem salt okunur sentetik bayraklar maskesi.
* `SF_ARCHIVED` 0x00010000: Dosya arşivlenmiştir.
* `SF_IMMUTABLE` 0x00020000: Dosya değiştirilemez.
* `SF_APPEND` 0x00040000: Dosyalara yalnızca ekleme yapılabilir.
* `SF_RESTRICTED` 0x00080000: Yazma için yetki gereklidir.
* `SF_NOUNLINK` 0x00100000: Öğe kaldırılamaz, yeniden adlandırılamaz veya üzerine monte edilemez.
* `SF_FIRMLINK` 0x00800000: Dosya bir firmlink'tir.
* `SF_DATALESS` 0x40000000: Dosya verisiz nesnedir.

### **Dosya ACL'leri**

Dosya **ACL'leri**, farklı kullanıcılara daha **aşırı izinler** atayabileceğiniz **ACE** (Erişim Kontrol Girişleri) içerir.

Bir **dizine** bu izinleri verebilirsiniz: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
Ve bir **dosyaya**: `read`, `write`, `append`, `execute`.

Dosya ACL'leri içerdiğinde, izinleri listelerken **izinlerin yanında "+" işaretini bulacaksınız** gibi:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Dosyanın **ACL'lerini** şu şekilde okuyabilirsiniz:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Tüm dosyaları **ACL'leriyle birlikte** bulabilirsiniz (bu çok yavaştır):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Genişletilmiş Öznitelikler

Genişletilmiş özniteliklerin bir adı ve istenen bir değeri vardır ve `ls -@` kullanılarak görüntülenebilir ve `xattr` komutu kullanılarak manipüle edilebilir. Bazı yaygın genişletilmiş öznitelikler şunlardır:

- `com.apple.resourceFork`: Kaynak çatalı uyumluluğu. Ayrıca `filename/..namedfork/rsrc` olarak da görülebilir.
- `com.apple.quarantine`: MacOS: Gatekeeper karantina mekanizması (III/6)
- `metadata:*`: MacOS: `_backup_excludeItem` gibi çeşitli meta veriler, veya `kMD*`
- `com.apple.lastuseddate` (#PS): Son dosya kullanım tarihi
- `com.apple.FinderInfo`: MacOS: Finder bilgileri (örneğin, renk Etiketleri)
- `com.apple.TextEncoding`: ASCII metin dosyalarının metin kodlamasını belirtir
- `com.apple.logd.metadata`: `/var/db/diagnostics` dizinindeki dosyalarda logd tarafından kullanılır
- `com.apple.genstore.*`: Nesil depolama (`/.DocumentRevisions-V100` dosya sisteminin kökünde)
- `com.apple.rootless`: MacOS: Dosyayı etiketlemek için Sistem Bütünlüğü Koruma tarafından kullanılır (III/10)
- `com.apple.uuidb.boot-uuid`: Benzersiz UUID ile önyükleme dönemlerinin logd işaretlemeleri
- `com.apple.decmpfs`: MacOS: Şeffaf dosya sıkıştırması (II/7)
- `com.apple.cprotect`: \*OS: Dosya başına şifreleme verileri (III/11)
- `com.apple.installd.*`: \*OS: installd tarafından kullanılan meta veriler, örneğin, `installType`, `uniqueInstallID`

### Kaynak Çatallar | macOS ADS

Bu, **MacOS makinelerinde Alternatif Veri Akışları (ADS)** elde etmenin bir yoludur. Bir dosyanın içine içerik kaydedebilirsiniz, bunu **com.apple.ResourceFork** adlı genişletilmiş bir öznitelik içine kaydederek dosyanın içinde **file/..namedfork/rsrc** olarak kaydedebilirsiniz.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Aşağıdaki komutla bu genişletilmiş özniteliği içeren tüm dosyaları bulabilirsiniz:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
### decmpfs

Genişletilmiş öznitelik `com.apple.decmpfs`, dosyanın şifreli olarak depolandığını gösterir, `ls -l` **0 boyutunu** rapor edecektir ve sıkıştırılmış veri bu özniteliğin içindedir. Dosyaya erişildiğinde bellekte şifre çözülecektir.

Bu öznitelik `ls -lO` ile görülebilir, sıkıştırılmış dosyalar da `UF_COMPRESSED` bayrağıyla etiketlenir. Sıkıştırılmış bir dosya kaldırıldığında bu bayrakla `chflags nocompressed </dosyanın/yoluna>` komutuyla, sistem dosyanın sıkıştırıldığını bilmeyecek ve dolayısıyla verilere erişemeyecektir (aslında boş olduğunu düşünecektir).

Araç afscexpand, bir dosyayı zorla açmak için kullanılabilir.

## **Evrensel ikili &** Mach-o Formatı

Mac OS ikilileri genellikle **evrensel ikili** olarak derlenir. **Evrensel ikili**, **aynı dosyada birden fazla mimariyi destekleyebilir**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS İşlem Belleği

## macOS bellek dökme

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Risk Kategorisi Dosyaları Mac OS

`/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` dizini, farklı dosya uzantılarıyla ilişkilendirilen **riskle ilgili bilgilerin depolandığı** yerdir. Bu dizin dosyaları çeşitli risk seviyelerine ayırır ve Safari'nin bu dosyaları indirme sonrasında nasıl işleyeceğini etkiler. Kategoriler şunlardır:

* **LSRiskCategorySafe**: Bu kategorideki dosyalar **tamamen güvenli** olarak kabul edilir. Safari, bu dosyaları otomatik olarak indirdikten sonra açacaktır.
* **LSRiskCategoryNeutral**: Bu dosyalar herhangi bir uyarı ile gelmez ve Safari tarafından **otomatik olarak açılmaz**.
* **LSRiskCategoryUnsafeExecutable**: Bu kategoriye giren dosyalar, dosyanın bir uygulama olduğunu belirten bir uyarı **tetikler**. Bu, kullanıcıyı uyarmak için bir güvenlik önlemi olarak hizmet verir.
* **LSRiskCategoryMayContainUnsafeExecutable**: Bu kategori, uygulama içerebilecek arşivler gibi dosyalar içindir. Safari, tüm içeriğin güvenli veya tarafsız olduğunu doğrulayamadığı sürece **bir uyarı tetikler**.

## Günlük dosyaları

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: İndirilen dosyalar hakkında bilgiler içerir, indirildikleri URL gibi.
* **`/var/log/system.log`**: OSX sistemlerinin ana günlüğüdür. com.apple.syslogd.plist, sistem günlükleme işleminin yürütülmesinden sorumludur (devre dışı bırakılıp bırakılmadığını `launchctl list` içinde "com.apple.syslogd" arayarak kontrol edebilirsiniz).
* **`/private/var/log/asl/*.asl`**: Bunlar, ilginç bilgiler içerebilecek Apple Sistem Günlükleridir.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "Finder" aracılığıyla son erişilen dosyaları ve uygulamaları saklar.
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Sistemin başlangıcında başlatılacak öğeleri saklar.
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility Uygulaması için günlük dosyası (USB'ler dahil sürücüler hakkında bilgi).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Kablosuz erişim noktaları hakkında veri.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Devre dışı bırakılan daemonların listesi.

{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarını göndererek HackTricks ve HackTricks Cloud** github depolarına PR göndererek **paylaşın**.

</details>
{% endhint %}
