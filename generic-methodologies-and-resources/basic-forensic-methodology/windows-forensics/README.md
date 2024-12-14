# Windows Artifacts

## Windows Artifacts

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

## Genel Windows Artefaktları

### Windows 10 Bildirimleri

`C:\Users\<kullanıcı_adı>\AppData\Local\Microsoft\Windows\Notifications` yolunda `appdb.dat` (Windows yıldönümünden önce) veya `wpndatabase.db` (Windows Yıldönümünden sonra) veritabanını bulabilirsiniz.

Bu SQLite veritabanının içinde, ilginç veriler içerebilecek tüm bildirimlerin (XML formatında) bulunduğu `Notification` tablosunu bulabilirsiniz.

### Zaman Çizelgesi

Zaman çizelgesi, ziyaret edilen web sayfalarının, düzenlenen belgelerin ve çalıştırılan uygulamaların **kronolojik geçmişini** sağlayan bir Windows özelliğidir.

Veritabanı `C:\Users\<kullanıcı_adı>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db` yolunda bulunur. Bu veritabanı, bir SQLite aracı veya [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) aracı ile açılabilir **ve bu araç 2 dosya oluşturur, bu dosyalar** [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md) aracı ile açılabilir. 

### ADS (Alternatif Veri Akışları)

İndirilen dosyalar, intranet, internet vb. üzerinden **nasıl** **indirildiğini** gösteren **ADS Zone.Identifier** içerebilir. Bazı yazılımlar (tarayıcılar gibi) genellikle dosyanın indirildiği **URL** gibi **daha fazla** **bilgi** de ekler.

## **Dosya Yedekleri**

### Geri Dönüşüm Kutusu

Vista/Win7/Win8/Win10'da **Geri Dönüşüm Kutusu**, sürücünün kökünde **`$Recycle.bin`** klasöründe bulunabilir (`C:\$Recycle.bin`).\
Bu klasörde bir dosya silindiğinde 2 özel dosya oluşturulur:

* `$I{id}`: Dosya bilgileri (silindiği tarih)
* `$R{id}`: Dosyanın içeriği

![](<../../../.gitbook/assets/image (1029).png>)

Bu dosyalara sahip olduğunuzda, silinen dosyaların orijinal adresini ve silindiği tarihi almak için [**Rifiuti**](https://github.com/abelcheung/rifiuti2) aracını kullanabilirsiniz (Vista – Win10 için `rifiuti-vista.exe` kullanın).
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### Hacim Gölgesi Kopyaları

Gölge Kopyası, Microsoft Windows'ta yer alan bir teknolojidir ve bilgisayar dosyalarının veya hacimlerinin **yedek kopyalarını** veya anlık görüntülerini, kullanıldıkları sırada bile oluşturabilir.

Bu yedekler genellikle dosya sisteminin kökünde bulunan `\System Volume Information` dizininde yer alır ve isimleri aşağıdaki resimde gösterilen **UID'lerden** oluşur:

![](<../../../.gitbook/assets/image (94).png>)

**ArsenalImageMounter** ile adli görüntüyü monte ederek, [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) aracı, bir gölge kopyasını incelemek ve hatta gölge kopyası yedeklerinden **dosyaları çıkarmak** için kullanılabilir.

![](<../../../.gitbook/assets/image (576).png>)

Kayıt defteri girişi `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore`, **yedeklenmeyecek** dosyaları ve anahtarları içerir:

![](<../../../.gitbook/assets/image (254).png>)

Kayıt defteri `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` ayrıca `Hacim Gölgesi Kopyaları` hakkında yapılandırma bilgilerini içerir.

### Ofis Otomatik Kaydedilen Dosyaları

Ofis otomatik kaydedilen dosyalarını şurada bulabilirsiniz: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## Shell Öğeleri

Bir shell öğesi, başka bir dosyaya nasıl erişileceği hakkında bilgi içeren bir öğedir.

### Son Belgeler (LNK)

Windows, kullanıcı bir dosyayı **açtığında, kullandığında veya oluşturduğunda** bu **kısayolları** **otomatik olarak** **oluşturur**:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Ofis: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

Bir klasör oluşturulduğunda, klasöre, üst klasöre ve büyük üst klasöre bir bağlantı da oluşturulur.

Bu otomatik olarak oluşturulan bağlantı dosyaları, **bir dosya** **veya** **klasör** olup olmadığını, o dosyanın **MAC** **zamanlarını**, dosyanın nerede saklandığına dair **hacim bilgilerini** ve **hedef dosyanın klasörünü** içeren **kaynak hakkında bilgi** **barındırır**. Bu bilgiler, dosyalar silinirse kurtarmak için faydalı olabilir.

Ayrıca, bağlantı dosyasının **oluşturulma tarihi**, orijinal dosyanın **ilk** **kullanıldığı** **zamandır** ve bağlantı dosyasının **değiştirilme tarihi**, kaynak dosyanın en son **kullanıldığı** **zamandır**.

Bu dosyaları incelemek için [**LinkParser**](http://4discovery.com/our-tools/) aracını kullanabilirsiniz.

Bu araçta **2 set** zaman damgası bulacaksınız:

* **İlk Set:**
1. FileModifiedDate
2. FileAccessDate
3. FileCreationDate
* **İkinci Set:**
1. LinkModifiedDate
2. LinkAccessDate
3. LinkCreationDate.

İlk zaman damgası seti, **dosyanın kendisine ait zaman damgalarını** referans alır. İkinci set, **bağlantılı dosyanın zaman damgalarını** referans alır.

Aynı bilgiyi Windows CLI aracı olan [**LECmd.exe**](https://github.com/EricZimmerman/LECmd) ile de alabilirsiniz.
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
In this case, the information is going to be saved inside a CSV file.

### Jumplists

Bunlar, her uygulama için belirtilen son dosyalardır. Her uygulamada erişebileceğiniz **bir uygulama tarafından kullanılan son dosyaların listesi**dir. **Otomatik olarak veya özel olarak** oluşturulabilirler.

Otomatik olarak oluşturulan **jumplists**, `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` dizininde saklanır. Jumplists, `{id}.autmaticDestinations-ms` formatına göre adlandırılır; burada başlangıç ID'si uygulamanın ID'sidir.

Özel jumplists, `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` dizininde saklanır ve genellikle dosya ile ilgili **önemli** bir şey olduğunda uygulama tarafından oluşturulurlar (belki favori olarak işaretlenmiştir).

Her jumplist'in **oluşturulma zamanı**, dosyanın **ilk kez erişildiği zamanı** ve **değiştirilme zamanı** son erişim zamanını gösterir.

Jumplists'i [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md) kullanarak inceleyebilirsiniz.

![](<../../../.gitbook/assets/image (168).png>)

(_JumplistExplorer tarafından sağlanan zaman damgalarının jumplist dosyasının kendisiyle ilgili olduğunu unutmayın_)

### Shellbags

[**Shellbags nedir öğrenmek için bu bağlantıyı takip edin.**](interesting-windows-registry-keys.md#shellbags)

## Windows USB'lerinin Kullanımı

Bir USB cihazının kullanıldığını belirlemek mümkündür, çünkü:

* Windows Son Klasörü
* Microsoft Office Son Klasörü
* Jumplists

Bazı LNK dosyalarının orijinal yolu göstermek yerine WPDNSE klasörüne işaret ettiğini unutmayın:

![](<../../../.gitbook/assets/image (218).png>)

WPDNSE klasöründeki dosyalar, orijinal dosyaların bir kopyasıdır, bu nedenle PC'nin yeniden başlatılmasında hayatta kalmazlar ve GUID bir shellbag'den alınır.

### Kayıt Bilgileri

[USB bağlı cihazlar hakkında ilginç bilgileri içeren kayıt anahtarlarını öğrenmek için bu sayfayı kontrol edin](interesting-windows-registry-keys.md#usb-information).

### setupapi

USB bağlantısının ne zaman yapıldığına dair zaman damgalarını almak için `C:\Windows\inf\setupapi.dev.log` dosyasını kontrol edin ( `Section start` için arama yapın).

![](<../../../.gitbook/assets/image (477) (2) (2) (2) (2) (2) (2) (2) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (10) (14) (2).png>)

### USB Dedektifi

[**USBDetective**](https://usbdetective.com) bir görüntüye bağlı USB cihazları hakkında bilgi almak için kullanılabilir.

![](<../../../.gitbook/assets/image (452).png>)

### Tak ve Çalıştır Temizleme

'Tak ve Çalıştır Temizleme' olarak bilinen planlı görev, esasen eski sürücü sürümlerinin kaldırılması için tasarlanmıştır. En son sürücü paket sürümünü koruma amacıyla belirtilmiş olmasına rağmen, çevrimiçi kaynaklar bunun 30 gündür etkin olmayan sürücüleri de hedef aldığını önermektedir. Sonuç olarak, son 30 günde bağlanmamış çıkarılabilir cihazların sürücüleri silinme riski altındadır.

Görev aşağıdaki yolda bulunmaktadır: `C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup`.

Görevin içeriğini gösteren bir ekran görüntüsü sağlanmıştır: ![](https://2.bp.blogspot.com/-wqYubtuR\_W8/W19bV5S9XyI/AAAAAAAANhU/OHsBDEvjqmg9ayzdNwJ4y2DKZnhCdwSMgCLcBGAs/s1600/xml.png)

**Görevin Ana Bileşenleri ve Ayarları:**

* **pnpclean.dll**: Bu DLL, gerçek temizleme işlemini gerçekleştirir.
* **UseUnifiedSchedulingEngine**: `TRUE` olarak ayarlanmıştır, genel görev zamanlama motorunun kullanıldığını gösterir.
* **MaintenanceSettings**:
* **Period ('P1M')**: Görev Zamanlayıcısına, temizleme görevini her ay düzenli Otomatik bakım sırasında başlatmasını yönlendirir.
* **Deadline ('P2M')**: Görev Zamanlayıcısına, görev iki ardışık ay boyunca başarısız olursa, acil Otomatik bakım sırasında görevi yürütmesini talimat verir.

Bu yapılandırma, sürücülerin düzenli bakımını ve temizliğini sağlar ve ardışık hatalar durumunda görevi yeniden denemek için önlemler içerir.

**Daha fazla bilgi için kontrol edin:** [**https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html**](https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html)

## E-postalar

E-postalar **2 ilginç bölüm içerir: Başlıklar ve e-postanın içeriği**. **Başlıklarda** aşağıdaki bilgileri bulabilirsiniz:

* **Kim** e-postaları gönderdi (e-posta adresi, IP, e-postayı yönlendiren mail sunucuları)
* **Ne zaman** e-posta gönderildi

Ayrıca, `References` ve `In-Reply-To` başlıkları içinde mesajların ID'sini bulabilirsiniz:

![](<../../../.gitbook/assets/image (593).png>)

### Windows Mail Uygulaması

Bu uygulama e-postaları HTML veya metin olarak kaydeder. E-postaları `\Users\<username>\AppData\Local\Comms\Unistore\data\3\` içindeki alt klasörlerde bulabilirsiniz. E-postalar `.dat` uzantısıyla kaydedilir.

E-postaların **meta verileri** ve **kişiler** **EDB veritabanında** bulunabilir: `\Users\<username>\AppData\Local\Comms\UnistoreDB\store.vol`

**Uzantıyı** `.vol`'dan `.edb`'ye değiştirin ve [ESEDatabaseView](https://www.nirsoft.net/utils/ese_database_view.html) aracını kullanarak açabilirsiniz. `Message` tablosunda e-postaları görebilirsiniz.

### Microsoft Outlook

Exchange sunucuları veya Outlook istemcileri kullanıldığında bazı MAPI başlıkları olacaktır:

* `Mapi-Client-Submit-Time`: E-postanın gönderildiği zaman sistemin zamanı
* `Mapi-Conversation-Index`: İletişim dizisinin çocuk mesajlarının sayısı ve dizinin her mesajının zaman damgası
* `Mapi-Entry-ID`: Mesaj tanımlayıcısı.
* `Mappi-Message-Flags` ve `Pr_last_Verb-Executed`: MAPI istemcisi hakkında bilgi (mesaj okundu mu? okunmadı mı? yanıtlandı mı? yönlendirildi mi? ofis dışında mı?)

Microsoft Outlook istemcisinde, gönderilen/alınan tüm mesajlar, kişi verileri ve takvim verileri bir PST dosyasında saklanır:

* `%USERPROFILE%\Local Settings\Application Data\Microsoft\Outlook` (WinXP)
* `%USERPROFILE%\AppData\Local\Microsoft\Outlook`

Kayıt yolu `HKEY_CURRENT_USER\Software\Microsoft\WindowsNT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook`, kullanılan dosyayı gösterir.

PST dosyasını [**Kernel PST Viewer**](https://www.nucleustechnologies.com/es/visor-de-pst.html) aracıyla açabilirsiniz.

![](<../../../.gitbook/assets/image (498).png>)

### Microsoft Outlook OST Dosyaları

Bir **OST dosyası**, Microsoft Outlook'un **IMAP** veya bir **Exchange** sunucusuyla yapılandırıldığında oluşturulur ve PST dosyasına benzer bilgileri saklar. Bu dosya, sunucu ile senkronize edilir, **son 12 ay** için veri tutar ve **maksimum 50GB** boyutundadır ve PST dosyasıyla aynı dizinde bulunur. Bir OST dosyasını görüntülemek için [**Kernel OST viewer**](https://www.nucleustechnologies.com/ost-viewer.html) kullanılabilir.

### Ekleri Kurtarma

Kaybolan ekler şunlardan kurtarılabilir:

* **IE10 için**: `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook`
* **IE11 ve üzeri için**: `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook`

### Thunderbird MBOX Dosyaları

**Thunderbird**, verileri saklamak için **MBOX dosyaları** kullanır ve bu dosyalar `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles` dizininde bulunur.

### Görüntü Küçültmeleri

* **Windows XP ve 8-8.1**: Küçültme içeren bir klasöre erişmek, silinmiş olsa bile görüntü önizlemelerini saklayan bir `thumbs.db` dosyası oluşturur.
* **Windows 7/10**: `thumbs.db`, UNC yolu üzerinden erişildiğinde oluşturulur.
* **Windows Vista ve daha yeni**: Küçültme önizlemeleri, `%userprofile%\AppData\Local\Microsoft\Windows\Explorer` dizininde **thumbcache\_xxx.db** olarak adlandırılan dosyalarda merkezi olarak saklanır. Bu dosyaları görüntülemek için [**Thumbsviewer**](https://thumbsviewer.github.io) ve [**ThumbCache Viewer**](https://thumbcacheviewer.github.io) araçları kullanılabilir.

### Windows Kayıt Bilgileri

Windows Kayıt Defteri, kapsamlı sistem ve kullanıcı etkinlik verilerini saklar ve aşağıdaki dosyalarda bulunur:

* Çeşitli `HKEY_LOCAL_MACHINE` alt anahtarları için `%windir%\System32\Config`.
* `HKEY_CURRENT_USER` için `%UserProfile%{User}\NTUSER.DAT`.
* Windows Vista ve sonraki sürümler, `HKEY_LOCAL_MACHINE` kayıt dosyalarını `%Windir%\System32\Config\RegBack\` dizininde yedekler.
* Ayrıca, program yürütme bilgileri, Windows Vista ve Windows 2008 Server'dan itibaren `%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT` içinde saklanır.

### Araçlar

Kayıt dosyalarını analiz etmek için bazı araçlar faydalıdır:

* **Kayıt Defteri Düzenleyici**: Windows'ta yüklüdür. Mevcut oturumun Windows kayıt defterinde gezinmek için bir GUI'dir.
* [**Registry Explorer**](https://ericzimmerman.github.io/#!index.md): Kayıt dosyasını yüklemenizi ve bunlar arasında gezinmenizi sağlayan bir GUI içerir. Ayrıca ilginç bilgiler içeren anahtarları vurgulayan Yer İmleri içerir.
* [**RegRipper**](https://github.com/keydet89/RegRipper3.0): Yine, yüklenen kayıt defterinde gezinmenizi sağlayan bir GUI içerir ve ayrıca yüklenen kayıt defterinde ilginç bilgileri vurgulayan eklentiler içerir.
* [**Windows Kayıt Kurtarma**](https://www.mitec.cz/wrr.html): Yüklenen kayıt defterinden önemli bilgileri çıkarmak için başka bir GUI uygulamasıdır.

### Silinen Elemanı Kurtarma

Bir anahtar silindiğinde, böyle işaretlenir, ancak kapladığı alan gerekli olana kadar kaldırılmaz. Bu nedenle, **Registry Explorer** gibi araçlar kullanarak bu silinmiş anahtarları kurtarmak mümkündür.

### Son Yazma Zamanı

Her Anahtar-Değer, en son ne zaman değiştirildiğini gösteren bir **zaman damgası** içerir.

### SAM

**SAM** dosyası/hive, sistemin **kullanıcılar, gruplar ve kullanıcı parolası** hash'lerini içerir.

`SAM\Domains\Account\Users` içinde kullanıcı adını, RID'yi, son giriş zamanını, son başarısız oturumu, giriş sayacını, parola politikasını ve hesabın ne zaman oluşturulduğunu elde edebilirsiniz. **Hash'leri** almak için ayrıca **SYSTEM** dosyasını/hive'yi de **gerekir**.

### Windows Kayıt Defterindeki İlginç Girişler

{% content-ref url="interesting-windows-registry-keys.md" %}
[interesting-windows-registry-keys.md](interesting-windows-registry-keys.md)
{% endcontent-ref %}

## Çalıştırılan Programlar

### Temel Windows Süreçleri

[Bu yazıda](https://jonahacks.medium.com/investigating-common-windows-processes-18dee5f97c1d) şüpheli davranışları tespit etmek için yaygın Windows süreçleri hakkında bilgi edinebilirsiniz.

### Windows Son Uygulamaları

Kayıt defteri `NTUSER.DAT` içinde `Software\Microsoft\Current Version\Search\RecentApps` yolunda, **çalıştırılan uygulama**, **son çalıştırma zamanı** ve **kaç kez** başlatıldığına dair bilgiler içeren alt anahtarlar bulabilirsiniz.

### BAM (Arka Plan Etkinlik Modaratörü)

`SYSTEM` dosyasını bir kayıt defteri düzenleyici ile açabilir ve `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}` yolunda **her kullanıcı tarafından çalıştırılan uygulamalar** hakkında bilgi bulabilirsiniz (yoldaki `{SID}`'yi not edin) ve **ne zaman** çalıştırıldıklarını (zaman, kayıt defterinin Veri değerinin içinde) görebilirsiniz.

### Windows Prefetch

Önceden alma, bir bilgisayarın kullanıcının **yakın gelecekte erişebileceği içeriği görüntülemek için gerekli kaynakları sessizce almasına** olanak tanıyan bir tekniktir, böylece kaynaklara daha hızlı erişilebilir.

Windows önceden alma, **çalıştırılan programların önbelleklerini** oluşturarak daha hızlı yüklenmelerini sağlar. Bu önbellekler, `C:\Windows\Prefetch` yolunda `.pf` dosyaları olarak oluşturulur. XP/VISTA/WIN7'de 128 dosya ve Win8/Win10'da 1024 dosya sınırı vardır.

Dosya adı `{program_name}-{hash}.pf` şeklinde oluşturulur (hash, yürütülebilir dosyanın yolu ve argümanlarına dayanır). W10'da bu dosyalar sıkıştırılmıştır. Dosyanın varlığı, **programın bir noktada çalıştırıldığını** gösterir.

`C:\Windows\Prefetch\Layout.ini` dosyası, **önceden alınan dosyaların klasörlerinin adlarını** içerir. Bu dosya, **çalıştırma sayısı**, **çalıştırma tarihleri** ve program tarafından **açılan dosyalar** hakkında **bilgi** içerir.

Bu dosyaları incelemek için [**PEcmd.exe**](https://github.com/EricZimmerman/PECmd) aracını kullanabilirsiniz:
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
![](<../../../.gitbook/assets/image (315).png>)

### Superprefetch

**Superprefetch**, önceden yükleme ile aynı amaca sahiptir, **programları daha hızlı yüklemek** için neyin yükleneceğini tahmin eder. Ancak, önceden yükleme hizmetinin yerini almaz.\
Bu hizmet, `C:\Windows\Prefetch\Ag*.db` konumunda veritabanı dosyaları oluşturur.

Bu veritabanlarında **programın adı**, **çalıştırma sayısı**, **açılan dosyalar**, **erişilen hacim**, **tam yol**, **zaman dilimleri** ve **zaman damgaları** bulabilirsiniz.

Bu bilgilere [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/) aracıyla erişebilirsiniz.

### SRUM

**Sistem Kaynak Kullanım İzleyici** (SRUM), **bir süreç tarafından tüketilen kaynakları** **izler**. W8'de ortaya çıkmıştır ve verileri `C:\Windows\System32\sru\SRUDB.dat` konumunda bir ESE veritabanında saklar.

Aşağıdaki bilgileri sağlar:

* Uygulama Kimliği ve Yol
* Süreci çalıştıran kullanıcı
* Gönderilen Bayt
* Alınan Bayt
* Ağ Arayüzü
* Bağlantı süresi
* Süreç süresi

Bu bilgiler her 60 dakikada bir güncellenir.

Bu dosyadan tarihi [**srum\_dump**](https://github.com/MarkBaggett/srum-dump) aracıyla elde edebilirsiniz.
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**AppCompatCache**, ayrıca **ShimCache** olarak da bilinir, **Microsoft** tarafından uygulama uyumluluğu sorunlarını ele almak için geliştirilen **Uygulama Uyumluluk Veritabanı**nın bir parçasını oluşturur. Bu sistem bileşeni, aşağıdaki dosya meta verilerinin çeşitli parçalarını kaydeder:

* Dosyanın tam yolu
* Dosyanın boyutu
* **$Standard\_Information** (SI) altında Son Değiştirilme zamanı
* ShimCache'in Son Güncellenme zamanı
* İşlem Çalıştırma Bayrağı

Bu tür veriler, işletim sisteminin sürümüne bağlı olarak belirli konumlarda kayıt defterinde saklanır:

* XP için, veriler `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache` altında 96 giriş kapasitesi ile saklanır.
* Server 2003 için, ayrıca Windows sürümleri 2008, 2012, 2016, 7, 8 ve 10 için, depolama yolu `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache` olup, sırasıyla 512 ve 1024 giriş kapasitesine sahiptir.

Saklanan bilgileri ayrıştırmak için, [**AppCompatCacheParser** aracı](https://github.com/EricZimmerman/AppCompatCacheParser) kullanılması önerilir.

![](<../../../.gitbook/assets/image (75).png>)

### Amcache

**Amcache.hve** dosyası, bir sistemde yürütülen uygulamalar hakkında ayrıntıları kaydeden temel bir kayıt defteri hivesidir. Genellikle `C:\Windows\AppCompat\Programas\Amcache.hve` konumunda bulunur.

Bu dosya, yürütülen süreçlerin kayıtlarını, yürütülebilir dosyaların yollarını ve SHA1 hash'lerini saklamasıyla dikkat çeker. Bu bilgi, bir sistemdeki uygulamaların etkinliğini izlemek için çok değerlidir.

**Amcache.hve** dosyasından veri çıkarmak ve analiz etmek için, [**AmcacheParser**](https://github.com/EricZimmerman/AmcacheParser) aracı kullanılabilir. Aşağıdaki komut, AmcacheParser'ı **Amcache.hve** dosyasının içeriğini ayrıştırmak ve sonuçları CSV formatında çıkarmak için nasıl kullanacağınıza dair bir örnektir:
```bash
AmcacheParser.exe -f C:\Users\genericUser\Desktop\Amcache.hve --csv C:\Users\genericUser\Desktop\outputFolder
```
Among the generated CSV files, the `Amcache_Unassociated file entries` is particularly noteworthy due to the rich information it provides about unassociated file entries.

Oluşturulan CSV dosyaları arasında, `Amcache_Unassociated file entries` özellikle dikkate değerdir çünkü ilişkilendirilmemiş dosya girişleri hakkında zengin bilgiler sunar.

The most interesting CVS file generated is the `Amcache_Unassociated file entries`.

Oluşturulan en ilginç CVS dosyası `Amcache_Unassociated file entries`dir.

### RecentFileCache

This artifact can only be found in W7 in `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` and it contains information about the recent execution of some binaries.

Bu artefakt yalnızca W7'de `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` içinde bulunabilir ve bazı ikili dosyaların son çalıştırılması hakkında bilgi içerir.

You can use the tool [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) to parse the file.

Dosyayı ayrıştırmak için [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) aracını kullanabilirsiniz.

### Scheduled tasks

You can extract them from `C:\Windows\Tasks` or `C:\Windows\System32\Tasks` and read them as XML.

Bunları `C:\Windows\Tasks` veya `C:\Windows\System32\Tasks`'dan çıkarabilir ve XML olarak okuyabilirsiniz.

### Services

You can find them in the registry under `SYSTEM\ControlSet001\Services`. You can see what is going to be executed and when.

Bunları `SYSTEM\ControlSet001\Services` altında kayıt defterinde bulabilirsiniz. Ne zaman ve neyin çalıştırılacağını görebilirsiniz.

### **Windows Store**

The installed applications can be found in `\ProgramData\Microsoft\Windows\AppRepository\`\
This repository has a **log** with **each application installed** in the system inside the database **`StateRepository-Machine.srd`**.

Yüklenen uygulamalar `\ProgramData\Microsoft\Windows\AppRepository\` içinde bulunabilir. Bu depo, sistemdeki **her yüklü uygulama** ile ilgili bir **log** içerir ve bu log, **`StateRepository-Machine.srd`** veritabanındadır.

Inside the Application table of this database, it's possible to find the columns: "Application ID", "PackageNumber", and "Display Name". These columns have information about pre-installed and installed applications and it can be found if some applications were uninstalled because the IDs of installed applications should be sequential.

Bu veritabanının Uygulama tablosunun içinde "Application ID", "PackageNumber" ve "Display Name" sütunlarını bulmak mümkündür. Bu sütunlar, önceden yüklenmiş ve yüklenmiş uygulamalar hakkında bilgi içerir ve bazı uygulamaların kaldırılıp kaldırılmadığı bulunabilir çünkü yüklü uygulamaların kimlikleri sıralı olmalıdır.

It's also possible to **find installed application** inside the registry path: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
And **uninstalled** **applications** in: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

Ayrıca, kayıt defteri yolunda **yüklü uygulamaları** bulmak mümkündür: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
Ve **kaldırılmış** **uygulamaları**: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\` içinde bulabilirsiniz.

## Windows Events

Information that appears inside Windows events are:

Windows olayları içinde görünen bilgiler şunlardır:

* What happened
* Timestamp (UTC + 0)
* Users involved
* Hosts involved (hostname, IP)
* Assets accessed (files, folder, printer, services)

Neler olduğu
Zaman damgası (UTC + 0)
İlgili kullanıcılar
İlgili ana bilgisayarlar (ana bilgisayar adı, IP)
Erişilen varlıklar (dosyalar, klasör, yazıcı, hizmetler)

The logs are located in `C:\Windows\System32\config` before Windows Vista and in `C:\Windows\System32\winevt\Logs` after Windows Vista. Before Windows Vista, the event logs were in binary format and after it, they are in **XML format** and use the **.evtx** extension.

Loglar, Windows Vista'dan önce `C:\Windows\System32\config` içinde ve Windows Vista'dan sonra `C:\Windows\System32\winevt\Logs` içinde bulunmaktadır. Windows Vista'dan önce, olay günlükleri ikili formatta ve sonrasında **XML formatında** ve **.evtx** uzantısını kullanmaktadır.

The location of the event files can be found in the SYSTEM registry in **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

Olay dosyalarının yeri, **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`** kayıt defterinde bulunabilir.

They can be visualized from the Windows Event Viewer (**`eventvwr.msc`**) or with other tools like [**Event Log Explorer**](https://eventlogxp.com) **or** [**Evtx Explorer/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)**.**

Windows Olay Görüntüleyicisi (**`eventvwr.msc`**) veya [**Event Log Explorer**](https://eventlogxp.com) **veya** [**Evtx Explorer/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)** gibi diğer araçlarla görselleştirilebilirler.

## Understanding Windows Security Event Logging

Access events are recorded in the security configuration file located at `C:\Windows\System32\winevt\Security.evtx`. This file's size is adjustable, and when its capacity is reached, older events are overwritten. Recorded events include user logins and logoffs, user actions, and changes to security settings, as well as file, folder, and shared asset access.

Erişim olayları, `C:\Windows\System32\winevt\Security.evtx` konumundaki güvenlik yapılandırma dosyasına kaydedilir. Bu dosyanın boyutu ayarlanabilir ve kapasitesi dolduğunda, daha eski olaylar üzerine yazılır. Kaydedilen olaylar, kullanıcı oturum açma ve kapama, kullanıcı eylemleri ve güvenlik ayarlarında yapılan değişiklikler ile dosya, klasör ve paylaşılan varlık erişimini içerir.

### Key Event IDs for User Authentication:

Kullanıcı Kimlik Doğrulaması için Ana Olay Kimlikleri:

* **EventID 4624**: Indicates a user successfully authenticated.
* **EventID 4625**: Signals an authentication failure.
* **EventIDs 4634/4647**: Represent user logoff events.
* **EventID 4672**: Denotes login with administrative privileges.

* **EventID 4624**: Bir kullanıcının başarılı bir şekilde kimlik doğruladığını gösterir.
* **EventID 4625**: Bir kimlik doğrulama hatasını belirtir.
* **EventIDs 4634/4647**: Kullanıcı oturum kapama olaylarını temsil eder.
* **EventID 4672**: Yönetici ayrıcalıklarıyla oturum açıldığını belirtir.

#### Sub-types within EventID 4634/4647:

EventID 4634/4647 içindeki alt türler:

* **Interactive (2)**: Direct user login.
* **Network (3)**: Access to shared folders.
* **Batch (4)**: Execution of batch processes.
* **Service (5)**: Service launches.
* **Proxy (6)**: Proxy authentication.
* **Unlock (7)**: Screen unlocked with a password.
* **Network Cleartext (8)**: Clear text password transmission, often from IIS.
* **New Credentials (9)**: Usage of different credentials for access.
* **Remote Interactive (10)**: Remote desktop or terminal services login.
* **Cache Interactive (11)**: Login with cached credentials without domain controller contact.
* **Cache Remote Interactive (12)**: Remote login with cached credentials.
* **Cached Unlock (13)**: Unlocking with cached credentials.

* **Etkileşimli (2)**: Doğrudan kullanıcı oturumu.
* **Ağ (3)**: Paylaşılan klasörlere erişim.
* **Toplu (4)**: Toplu işlemlerin yürütülmesi.
* **Hizmet (5)**: Hizmet başlatmaları.
* **Proxy (6)**: Proxy kimlik doğrulaması.
* **Kilidi Açma (7)**: Şifre ile ekranın kilidinin açılması.
* **Ağ Düz Metin (8)**: Düz metin şifre iletimi, genellikle IIS'den.
* **Yeni Kimlik Bilgileri (9)**: Erişim için farklı kimlik bilgileri kullanımı.
* **Uzaktan Etkileşimli (10)**: Uzaktan masaüstü veya terminal hizmetleri oturumu.
* **Önbellek Etkileşimli (11)**: Alan denetleyicisi ile iletişim olmadan önbelleklenmiş kimlik bilgileri ile oturum açma.
* **Önbellek Uzaktan Etkileşimli (12)**: Önbelleklenmiş kimlik bilgileri ile uzaktan oturum açma.
* **Önbellekli Kilidi Açma (13)**: Önbelleklenmiş kimlik bilgileri ile kilidin açılması.

#### Status and Sub Status Codes for EventID 4625:

EventID 4625 için Durum ve Alt Durum Kodları:

* **0xC0000064**: User name does not exist - Could indicate a username enumeration attack.
* **0xC000006A**: Correct user name but wrong password - Possible password guessing or brute-force attempt.
* **0xC0000234**: User account locked out - May follow a brute-force attack resulting in multiple failed logins.
* **0xC0000072**: Account disabled - Unauthorized attempts to access disabled accounts.
* **0xC000006F**: Logon outside allowed time - Indicates attempts to access outside of set login hours, a possible sign of unauthorized access.
* **0xC0000070**: Violation of workstation restrictions - Could be an attempt to login from an unauthorized location.
* **0xC0000193**: Account expiration - Access attempts with expired user accounts.
* **0xC0000071**: Expired password - Login attempts with outdated passwords.
* **0xC0000133**: Time sync issues - Large time discrepancies between client and server may be indicative of more sophisticated attacks like pass-the-ticket.
* **0xC0000224**: Mandatory password change required - Frequent mandatory changes might suggest an attempt to destabilize account security.
* **0xC0000225**: Indicates a system bug rather than a security issue.
* **0xC000015b**: Denied logon type - Access attempt with unauthorized logon type, such as a user trying to execute a service logon.

* **0xC0000064**: Kullanıcı adı mevcut değil - Bir kullanıcı adı tahmin saldırısını gösterebilir.
* **0xC000006A**: Doğru kullanıcı adı ama yanlış şifre - Olası şifre tahmini veya kaba kuvvet denemesi.
* **0xC0000234**: Kullanıcı hesabı kilitlendi - Birden fazla başarısız oturum açma ile sonuçlanan bir kaba kuvvet saldırısını takip edebilir.
* **0xC0000072**: Hesap devre dışı - Devre dışı bırakılmış hesaplara yetkisiz erişim girişimleri.
* **0xC000006F**: İzin verilen zaman dışında oturum açma - Belirlenen oturum açma saatleri dışında erişim girişimlerini gösterir, yetkisiz erişim belirtisi olabilir.
* **0xC0000070**: İş istasyonu kısıtlamalarının ihlali - Yetkisiz bir yerden oturum açma girişimi olabilir.
* **0xC0000193**: Hesap süresi dolmuş - Süresi dolmuş kullanıcı hesapları ile erişim girişimleri.
* **0xC0000071**: Süresi dolmuş şifre - Eski şifreler ile oturum açma girişimleri.
* **0xC0000133**: Zaman senkronizasyon sorunları - İstemci ve sunucu arasında büyük zaman farklılıkları, pass-the-ticket gibi daha karmaşık saldırıların göstergesi olabilir.
* **0xC0000224**: Zorunlu şifre değişikliği gereklidir - Sık zorunlu değişiklikler, hesap güvenliğini bozma girişimini gösterebilir.
* **0xC0000225**: Bir sistem hatasını gösterir, güvenlik sorunu değil.
* **0xC000015b**: Reddedilen oturum açma türü - Yetkisiz oturum açma türü ile erişim girişimi, örneğin bir kullanıcının bir hizmet oturumu başlatmaya çalışması.

#### EventID 4616:

* **Time Change**: Modification of the system time, could obscure the timeline of events.

* **Zaman Değişikliği**: Sistem zamanının değiştirilmesi, olayların zaman çizelgesini belirsizleştirebilir.

#### EventID 6005 and 6006:

* **System Startup and Shutdown**: EventID 6005 indicates the system starting up, while EventID 6006 marks it shutting down.

* **Sistem Başlangıcı ve Kapatılması**: EventID 6005, sistemin başlatıldığını gösterirken, EventID 6006 kapanışını işaret eder.

#### EventID 1102:

* **Log Deletion**: Security logs being cleared, which is often a red flag for covering up illicit activities.

* **Günlük Silme**: Güvenlik günlüklerinin temizlenmesi, genellikle yasadışı faaliyetleri örtbas etme için bir kırmızı bayraktır.

#### EventIDs for USB Device Tracking:

* **20001 / 20003 / 10000**: USB device first connection.
* **10100**: USB driver update.
* **EventID 112**: Time of USB device insertion.

USB Cihaz Takibi için Olay Kimlikleri:

* **20001 / 20003 / 10000**: USB cihazının ilk bağlantısı.
* **10100**: USB sürücü güncellemesi.
* **EventID 112**: USB cihazının takılma zamanı.

For practical examples on simulating these login types and credential dumping opportunities, refer to [Altered Security's detailed guide](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them).

Bu oturum açma türlerini simüle etme ve kimlik bilgisi dökme fırsatları hakkında pratik örnekler için [Altered Security'nin detaylı kılavuzuna](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them) başvurun.

Event details, including status and sub-status codes, provide further insights into event causes, particularly notable in Event ID 4625.

Olay detayları, durum ve alt durum kodları dahil, olay nedenleri hakkında daha fazla bilgi sağlar, özellikle Event ID 4625'te dikkate değerdir.

### Recovering Windows Events

To enhance the chances of recovering deleted Windows Events, it's advisable to power down the suspect computer by directly unplugging it. **Bulk\_extractor**, a recovery tool specifying the `.evtx` extension, is recommended for attempting to recover such events.

### Windows Olaylarını Kurtarma

Silinmiş Windows Olaylarını kurtarma şansını artırmak için, şüpheli bilgisayarı doğrudan fişini çekerek kapatmak önerilir. **Bulk\_extractor**, `.evtx` uzantısını belirten bir kurtarma aracı, bu tür olayları kurtarmak için önerilir.

### Identifying Common Attacks via Windows Events

For a comprehensive guide on utilizing Windows Event IDs in identifying common cyber attacks, visit [Red Team Recipe](https://redteamrecipe.com/event-codes/).

### Windows Olayları Aracılığıyla Yaygın Saldırıları Tanımlama

Yaygın siber saldırıları tanımlamak için Windows Olay Kimliklerini kullanma konusunda kapsamlı bir kılavuz için [Red Team Recipe](https://redteamrecipe.com/event-codes/) adresini ziyaret edin.

#### Brute Force Attacks

Identifiable by multiple EventID 4625 records, followed by an EventID 4624 if the attack succeeds.

#### Kaba Kuvvet Saldırıları

Birden fazla EventID 4625 kaydı ile tanımlanabilir, saldırı başarılı olursa ardından bir EventID 4624 gelir.

#### Time Change

Recorded by EventID 4616, changes to system time can complicate forensic analysis.

#### Zaman Değişikliği

EventID 4616 ile kaydedilen sistem zamanındaki değişiklikler, adli analizleri karmaşıklaştırabilir.

#### USB Device Tracking

Useful System EventIDs for USB device tracking include 20001/20003/10000 for initial use, 10100 for driver updates, and EventID 112 from DeviceSetupManager for insertion timestamps.

#### USB Cihaz Takibi

USB cihaz takibi için yararlı Sistem Olay Kimlikleri, başlangıç kullanımı için 20001/20003/10000, sürücü güncellemeleri için 10100 ve takılma zaman damgaları için DeviceSetupManager'dan EventID 112'dir.

#### System Power Events

EventID 6005 indicates system startup, while EventID 6006 marks shutdown.

#### Sistem Güç Olayları

EventID 6005, sistemin başlatıldığını gösterirken, EventID 6006 kapanışını işaret eder.

#### Log Deletion

Security EventID 1102 signals the deletion of logs, a critical event for forensic analysis.

#### Günlük Silme

Güvenlik EventID 1102, günlüklerin silinmesini işaret eder, bu da adli analiz için kritik bir olaydır.

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
