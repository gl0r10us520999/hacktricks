# macOS Dosyaları, Klasörler, İkili Dosyalar ve Bellek

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## Dosya hiyerarşisi düzeni

* **/Applications**: Yüklenmiş uygulamalar burada olmalıdır. Tüm kullanıcılar bunlara erişebilir.
* **/bin**: Komut satırı ikili dosyaları
* **/cores**: Varsa, çekirdek döküm dosyalarını saklamak için kullanılır
* **/dev**: Her şey bir dosya olarak kabul edilir, bu yüzden burada donanım aygıtlarını görebilirsiniz.
* **/etc**: Yapılandırma dosyaları
* **/Library**: Tercihler, önbellekler ve günlüklerle ilgili birçok alt dizin ve dosya burada bulunabilir. Kök dizinde ve her kullanıcının dizininde bir Library klasörü vardır.
* **/private**: Belgelendirilmemiştir ama bahsedilen birçok klasör özel dizine sembolik bağlantılardır.
* **/sbin**: Temel sistem ikili dosyaları (yönetimle ilgili)
* **/System**: OS X'in çalışmasını sağlayan dosya. Burada çoğunlukla yalnızca Apple'a özgü dosyalar bulmalısınız (üçüncü taraf değil).
* **/tmp**: Dosyalar 3 gün sonra silinir (bu, /private/tmp'e yumuşak bir bağlantıdır)
* **/Users**: Kullanıcılar için ana dizin.
* **/usr**: Yapılandırma ve sistem ikili dosyaları
* **/var**: Günlük dosyaları
* **/Volumes**: Bağlı sürücüler burada görünecektir.
* **/.vol**: `stat a.txt` komutunu çalıştırdığınızda `16777223 7545753 -rw-r--r-- 1 username wheel ...` gibi bir şey elde edersiniz; burada ilk sayı dosyanın bulunduğu hacmin kimlik numarası ve ikincisi inode numarasıdır. Bu bilgiyi kullanarak `cat /.vol/16777223/7545753` komutunu çalıştırarak bu dosyanın içeriğine erişebilirsiniz.

### Uygulama Klasörleri

* **Sistem uygulamaları** `/System/Applications` altında bulunur
* **Yüklenmiş** uygulamalar genellikle `/Applications` veya `~/Applications` dizininde yüklenir
* **Uygulama verileri**, root olarak çalışan uygulamalar için `/Library/Application Support` ve kullanıcı olarak çalışan uygulamalar için `~/Library/Application Support` dizininde bulunabilir.
* Üçüncü taraf uygulamaların **root olarak çalışması gereken** **daemon'ları** genellikle `/Library/PrivilegedHelperTools/` dizininde bulunur.
* **Sandboxed** uygulamalar `~/Library/Containers` klasörüne haritalanır. Her uygulamanın, uygulamanın paket kimliğine göre adlandırılmış bir klasörü vardır (`com.apple.Safari`).
* **Kernel** `/System/Library/Kernels/kernel` dizinindedir
* **Apple'ın kernel uzantıları** `/System/Library/Extensions` dizinindedir
* **Üçüncü taraf kernel uzantıları** `/Library/Extensions` dizininde saklanır

### Hassas Bilgiler İçeren Dosyalar

MacOS, şifreler gibi bilgileri birkaç yerde saklar:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Savunmasız pkg yükleyicileri

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X Özel Uzantıları

* **`.dmg`**: Apple Disk Image dosyaları yükleyiciler için çok yaygındır.
* **`.kext`**: Belirli bir yapıyı takip etmelidir ve OS X sürümünde bir sürücüdür. (bu bir pakettir)
* **`.plist`**: XML veya ikili formatta bilgi saklayan özellik listesi olarak da bilinir.
* XML veya ikili olabilir. İkili olanlar şu komutlarla okunabilir:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Dizin yapısını takip eden Apple uygulamaları (bu bir pakettir).
* **`.dylib`**: Dinamik kütüphaneler (Windows DLL dosyaları gibi)
* **`.pkg`**: xar (eXtensible Archive format) ile aynıdır. Yükleyici komutu, bu dosyaların içeriğini yüklemek için kullanılabilir.
* **`.DS_Store`**: Bu dosya her dizinde bulunur, dizinin özelliklerini ve özelleştirmelerini kaydeder.
* **`.Spotlight-V100`**: Bu klasör, sistemdeki her hacmin kök dizininde görünür.
* **`.metadata_never_index`**: Bu dosya bir hacmin kökünde bulunuyorsa, Spotlight o hacmi dizinlemez.
* **`.noindex`**: Bu uzantıya sahip dosya ve klasörler Spotlight tarafından dizinlenmeyecektir.
* **`.sdef`**: Paketler içindeki dosyalar, bir AppleScript ile uygulama ile nasıl etkileşim kurulabileceğini belirtir.

### macOS Paketleri

Bir paket, **Finder'da bir nesne gibi görünen** bir **dizindir** (bir Paket örneği `*.app` dosyalarıdır).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Paylaşılan Kütüphane Önbelleği (SLC)

macOS'ta (ve iOS'ta) tüm sistem paylaşılan kütüphaneleri, çerçeveler ve dylib'ler gibi, **tek bir dosyada birleştirilmiştir**, buna **dyld paylaşılan önbelleği** denir. Bu, kodun daha hızlı yüklenmesini sağladığı için performansı artırır.

Bu, macOS'ta `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` dizininde bulunur ve eski sürümlerde **paylaşılan önbelleği** **`/System/Library/dyld/`** dizininde bulabilirsiniz.\
iOS'ta bunları **`/System/Library/Caches/com.apple.dyld/`** dizininde bulabilirsiniz.

Dyld paylaşılan önbelleğine benzer şekilde, kernel ve kernel uzantıları da bir kernel önbelleğine derlenir ve bu, önyükleme sırasında yüklenir.

Tek dosya dylib paylaşılan önbelleğinden kütüphaneleri çıkarmak için, günümüzde çalışmayabilecek olan [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) ikilisini kullanmak mümkündü ama ayrıca [**dyldextractor**](https://github.com/arandomdev/dyldextractor) kullanabilirsiniz:

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
`dyld_shared_cache_util` aracının çalışmaması durumunda, **paylaşılan dyld ikili dosyasını Hopper'a** iletebileceğinizi ve Hopper'ın tüm kütüphaneleri tanımlayıp **hangi kütüphaneyi** incelemek istediğinizi **seçmenize** izin vereceğini unutmayın:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Bazı çıkarıcılar çalışmayabilir çünkü dylib'ler, bilinmeyen adreslere atlama yapabilecekleri için sabit kodlanmış adreslerle önceden bağlantılıdır.

{% hint style="success" %}
Xcode'da bir emülatör kullanarak macos'ta diğer \*OS cihazlarının Paylaşılan Kütüphane Önbelleğini indirmenin de mümkün olduğunu unutmayın. Bunlar şu dizinde indirilecektir: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, örneğin: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### SLC Haritalama

**`dyld`**, SLC'nin haritalanıp haritalanmadığını bilmek için **`shared_region_check_np`** sistem çağrısını (adres döndürür) ve SLC'yi haritalamak için **`shared_region_map_and_slide_np`** kullanır.

SLC ilk kullanımda kaydırılsa bile, tüm **işlemler** **aynı kopyayı** kullanır, bu da saldırganın sistemde işlemleri çalıştırabilmesi durumunda **ASLR** korumasını **ortadan kaldırır**. Bu geçmişte gerçekten istismar edildi ve paylaşılan bölge sayfası ile düzeltildi.

Dal havuzları, görüntü haritalamaları arasında küçük alanlar oluşturan küçük Mach-O dylib'lerdir ve bu da işlevlerin araya girmesini imkansız hale getirir.

### SLC'leri Geçersiz Kılma

Aşağıdaki çevre değişkenlerini kullanarak:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Bu, yeni bir paylaşılan kütüphane önbelleği yüklemeye izin verecektir.
* **`DYLD_SHARED_CACHE_DIR=avoid`** ve kütüphaneleri gerçek olanlarla paylaşılan önbelleğe sembolik bağlantılarla manuel olarak değiştirmek (bunları çıkarmanız gerekecek).

## Özel Dosya İzinleri

### Klasör izinleri

Bir **klasörde**, **okuma** onu **listelemeye** izin verir, **yazma** dosyaları **silme** ve **yazma** izni verir, ve **çalıştırma** dizinde **gezinmeye** izin verir. Örneğin, bir kullanıcı bir dizinde **çalıştırma** iznine sahip olmadığı bir dosya üzerinde **okuma iznine** sahip olsa bile, dosyayı **okuyamaz**.

### Bayrak değiştiricileri

Dosyalarda ayarlanabilecek bazı bayraklar vardır, bu bayraklar dosyanın farklı davranmasına neden olur. Bir dizindeki dosyaların **bayraklarını kontrol edebilirsiniz** `ls -lO /path/directory` ile.

* **`uchg`**: **uchange** bayrağı olarak bilinir ve **dosyanın** değiştirilmesini veya silinmesini **önler**. Ayarlamak için: `chflags uchg file.txt`
* Root kullanıcısı **bayrağı kaldırabilir** ve dosyayı değiştirebilir.
* **`restricted`**: Bu bayrak dosyanın **SIP tarafından korunmasını** sağlar (bu bayrağı bir dosyaya ekleyemezsiniz).
* **`Sticky bit`**: Eğer bir dizin sticky bit'e sahipse, **yalnızca** **dizin sahibi veya root dosyaları yeniden adlandırabilir veya silebilir**. Genellikle bu, sıradan kullanıcıların diğer kullanıcıların dosyalarını silmesini veya taşınmasını önlemek için /tmp dizininde ayarlanır.

Tüm bayraklar `sys/stat.h` dosyasında bulunabilir (bunu `mdfind stat.h | grep stat.h` kullanarak bulabilirsiniz) ve şunlardır:

* `UF_SETTABLE` 0x0000ffff: Sahip değiştirilebilir bayrakların maskesi.
* `UF_NODUMP` 0x00000001: Dosyayı dökme.
* `UF_IMMUTABLE` 0x00000002: Dosya değiştirilemez.
* `UF_APPEND` 0x00000004: Dosyaya yazma yalnızca ekleme yapabilir.
* `UF_OPAQUE` 0x00000008: Dizin, birleşim açısından opaktır.
* `UF_COMPRESSED` 0x00000020: Dosya sıkıştırılmıştır (bazı dosya sistemleri).
* `UF_TRACKED` 0x00000040: Bu ayar için silme/yeniden adlandırma bildirimleri yoktur.
* `UF_DATAVAULT` 0x00000080: Okuma ve yazma için yetki gereklidir.
* `UF_HIDDEN` 0x00008000: Bu öğenin bir GUI'de görüntülenmemesi gerektiğini belirten ipucu.
* `SF_SUPPORTED` 0x009f0000: Süper kullanıcı destekli bayrakların maskesi.
* `SF_SETTABLE` 0x3fff0000: Süper kullanıcı değiştirilebilir bayrakların maskesi.
* `SF_SYNTHETIC` 0xc0000000: Sistem salt okunur sentetik bayrakların maskesi.
* `SF_ARCHIVED` 0x00010000: Dosya arşivlenmiştir.
* `SF_IMMUTABLE` 0x00020000: Dosya değiştirilemez.
* `SF_APPEND` 0x00040000: Dosyaya yazma yalnızca ekleme yapabilir.
* `SF_RESTRICTED` 0x00080000: Yazma için yetki gereklidir.
* `SF_NOUNLINK` 0x00100000: Öğe kaldırılmayabilir, yeniden adlandırılamaz veya bağlanamaz.
* `SF_FIRMLINK` 0x00800000: Dosya bir firmlink'tir.
* `SF_DATALESS` 0x40000000: Dosya dataless nesnedir.

### **Dosya ACL'leri**

Dosya **ACL'leri**, farklı kullanıcılara daha **ince izinler** atamak için **ACE** (Erişim Kontrol Girişleri) içerir.

Bir **dizin** için bu izinler verilebilir: `listele`, `arama`, `dosya_ekle`, `alt_dizin_ekle`, `çocuk_sil`, `çocuk_sil`.\
Ve bir **dosya** için: `okuma`, `yazma`, `ekleme`, `çalıştırma`.

Dosya ACL'ler içeriyorsa, izinleri listelediğinizde **"+" bulacaksınız**:
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
Tüm **ACL'lere sahip dosyaları** (bu çok yavaş) bulabilirsiniz:
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Genişletilmiş Nitelikler

Genişletilmiş nitelikler bir adı ve istenen herhangi bir değeri vardır ve `ls -@` kullanılarak görülebilir ve `xattr` komutu ile manipüle edilebilir. Bazı yaygın genişletilmiş nitelikler şunlardır:

* `com.apple.resourceFork`: Kaynak fork uyumluluğu. Ayrıca `filename/..namedfork/rsrc` olarak görünür
* `com.apple.quarantine`: MacOS: Gatekeeper karantina mekanizması (III/6)
* `metadata:*`: MacOS: `_backup_excludeItem` veya `kMD*` gibi çeşitli meta veriler
* `com.apple.lastuseddate` (#PS): Son dosya kullanım tarihi
* `com.apple.FinderInfo`: MacOS: Finder bilgisi (örneğin, renk Etiketleri)
* `com.apple.TextEncoding`: ASCII metin dosyalarının metin kodlamasını belirtir
* `com.apple.logd.metadata`: `/var/db/diagnostics` içindeki dosyalar için logd tarafından kullanılır
* `com.apple.genstore.*`: Nesil depolama (`/.DocumentRevisions-V100` dosya sisteminin kökünde)
* `com.apple.rootless`: MacOS: Sistem Bütünlüğü Koruması tarafından dosyayı etiketlemek için kullanılır (III/10)
* `com.apple.uuidb.boot-uuid`: benzersiz UUID ile önyükleme dönemlerinin logd işaretlemeleri
* `com.apple.decmpfs`: MacOS: Şeffaf dosya sıkıştırması (II/7)
* `com.apple.cprotect`: \*OS: Dosya başına şifreleme verileri (III/11)
* `com.apple.installd.*`: \*OS: installd tarafından kullanılan meta veriler, örneğin, `installType`, `uniqueInstallID`

### Kaynak Forkları | macOS ADS

Bu, **MacOS** makinelerinde **Alternatif Veri Akışları** elde etmenin bir yoludur. İçeriği, bir dosya içinde **com.apple.ResourceFork** adlı bir genişletilmiş nitelik içinde kaydederek **file/..namedfork/rsrc** içinde saklayabilirsiniz.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
You can **bulunan tüm dosyaları bu genişletilmiş niteliği içeren** ile: 

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Genişletilmiş özellik `com.apple.decmpfs`, dosyanın şifreli olarak saklandığını gösterir, `ls -l` **boyutunun 0** olduğunu bildirecek ve sıkıştırılmış veriler bu özellik içinde yer alır. Dosya her erişildiğinde, bellek içinde şifresi çözülecektir.

Bu özellik `ls -lO` ile sıkıştırılmış olarak görülebilir çünkü sıkıştırılmış dosyalar `UF_COMPRESSED` bayrağı ile etiketlenir. Eğer bir sıkıştırılmış dosya `chflags nocompressed </path/to/file>` ile kaldırılırsa, sistem dosyanın sıkıştırıldığını bilmeyecek ve bu nedenle veriyi açıp erişemeyecektir (gerçekte boş olduğunu düşünecektir).

Afscexpand aracı, bir dosyayı zorla açmak için kullanılabilir.

## **Evrensel ikililer &** Mach-o Formatı

Mac OS ikilileri genellikle **evrensel ikililer** olarak derlenir. Bir **evrensel ikili**, **aynı dosyada birden fazla mimariyi destekleyebilir**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS Süreç Belleği

## macOS bellek dökümü

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Risk Kategorisi Dosyaları Mac OS

`/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` dizini, **farklı dosya uzantılarıyla ilişkili risk hakkında bilgilerin saklandığı yerdir**. Bu dizin, dosyaları çeşitli risk seviyelerine ayırarak, Safari'nin bu dosyaları indirdikten sonra nasıl işleyeceğini etkiler. Kategoriler şunlardır:

* **LSRiskCategorySafe**: Bu kategorideki dosyalar **tamamen güvenli** olarak kabul edilir. Safari, bu dosyaları indirdikten sonra otomatik olarak açacaktır.
* **LSRiskCategoryNeutral**: Bu dosyalar uyarı içermez ve Safari tarafından **otomatik olarak açılmaz**.
* **LSRiskCategoryUnsafeExecutable**: Bu kategori altındaki dosyalar, dosyanın bir uygulama olduğunu belirten **bir uyarı tetikler**. Bu, kullanıcıyı uyarmak için bir güvenlik önlemidir.
* **LSRiskCategoryMayContainUnsafeExecutable**: Bu kategori, bir yürütülebilir dosya içerebilecek arşivler gibi dosyalar içindir. Safari, tüm içeriklerin güvenli veya nötr olduğunu doğrulayamazsa **bir uyarı tetikler**.

## Log dosyaları

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: İndirilen dosyalar hakkında, nereden indirildikleri gibi bilgileri içerir.
* **`/var/log/system.log`**: OSX sistemlerinin ana günlüğü. com.apple.syslogd.plist, syslogging'in yürütülmesinden sorumludur (devre dışı olup olmadığını kontrol etmek için `launchctl list` içinde "com.apple.syslogd" arayabilirsiniz).
* **`/private/var/log/asl/*.asl`**: İlginç bilgiler içerebilecek Apple Sistem Günlükleridir.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "Finder" aracılığıyla en son erişilen dosyaları ve uygulamaları saklar.
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Sistem başlangıcında başlatılacak öğeleri saklar.
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility Uygulaması için günlük dosyası (USB'ler dahil sürücüler hakkında bilgi).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Kablosuz erişim noktaları hakkında veri.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Devre dışı bırakılan daemonların listesi.

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
