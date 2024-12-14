# İlginç Windows Kayıt Defteri Anahtarları

### İlginç Windows Kayıt Defteri Anahtarları

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


### **Windows Sürümü ve Sahibi Bilgileri**
- **`Software\Microsoft\Windows NT\CurrentVersion`** altında, Windows sürümünü, Servis Paketini, kurulum zamanını ve kayıtlı sahibinin adını basit bir şekilde bulabilirsiniz.

### **Bilgisayar Adı**
- Host adı **`System\ControlSet001\Control\ComputerName\ComputerName`** altında bulunur.

### **Saat Dilimi Ayarı**
- Sistem saat dilimi **`System\ControlSet001\Control\TimeZoneInformation`** içinde saklanır.

### **Erişim Zamanı Takibi**
- Varsayılan olarak, son erişim zamanı takibi kapalıdır (**`NtfsDisableLastAccessUpdate=1`**). Bunu etkinleştirmek için:
`fsutil behavior set disablelastaccess 0` kullanın.

### Windows Sürümleri ve Servis Paketleri
- **Windows sürümü**, sürümü (örneğin, Home, Pro) ve sürümünü (örneğin, Windows 10, Windows 11) gösterirken, **Servis Paketleri** düzeltmeler ve bazen yeni özellikler içeren güncellemeleridir.

### Son Erişim Zamanını Etkinleştirme
- Son erişim zamanı takibini etkinleştirmek, dosyaların en son ne zaman açıldığını görmenizi sağlar; bu, adli analiz veya sistem izleme için kritik olabilir.

### Ağ Bilgileri Detayları
- Kayıt defteri, **ağ türleri (kablosuz, kablolu, 3G)** ve **ağ kategorileri (Halka Açık, Özel/Ev, Alan/İş)** dahil olmak üzere ağ yapılandırmaları hakkında kapsamlı veriler tutar; bu, ağ güvenlik ayarlarını ve izinlerini anlamak için hayati öneme sahiptir.

### İstemci Tarafı Önbellekleme (CSC)
- **CSC**, paylaşılan dosyaların kopyalarını önbelleğe alarak çevrimdışı dosya erişimini artırır. Farklı **CSCFlags** ayarları, hangi dosyaların ve nasıl önbelleğe alınacağını kontrol eder, bu da performansı ve kullanıcı deneyimini etkiler, özellikle kesintili bağlantıların olduğu ortamlarda.

### Otomatik Başlatılan Programlar
- Çeşitli `Run` ve `RunOnce` kayıt defteri anahtarlarında listelenen programlar, başlangıçta otomatik olarak başlatılır, bu da sistemin önyükleme süresini etkiler ve kötü amaçlı yazılım veya istenmeyen yazılımları tanımlamak için ilgi noktaları olabilir.

### Shellbags
- **Shellbags**, yalnızca klasör görünüm tercihlerini saklamakla kalmaz, aynı zamanda klasör erişiminin adli kanıtını sağlar; bu, klasör artık mevcut olmasa bile geçerlidir. Diğer yollarla belirgin olmayan kullanıcı etkinliğini ortaya çıkardığı için soruşturmalar için değerlidir.

### USB Bilgileri ve Adli Analiz
- Kayıt defterinde saklanan USB cihazlarıyla ilgili detaylar, hangi cihazların bir bilgisayara bağlandığını izlemeye yardımcı olabilir; bu, bir cihazı hassas dosya transferleri veya yetkisiz erişim olaylarıyla ilişkilendirebilir.

### Hacim Seri Numarası
- **Hacim Seri Numarası**, dosya sisteminin belirli bir örneğini izlemek için kritik olabilir; bu, dosya kökeninin farklı cihazlar arasında belirlenmesi gereken adli senaryolar için yararlıdır.

### **Kapatma Detayları**
- Kapatma zamanı ve sayısı (ikincisi yalnızca XP için) **`System\ControlSet001\Control\Windows`** ve **`System\ControlSet001\Control\Watchdog\Display`** içinde saklanır.

### **Ağ Yapılandırması**
- Ayrıntılı ağ arayüzü bilgileri için **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**'e bakın.
- İlk ve son ağ bağlantı zamanları, VPN bağlantıları dahil olmak üzere, **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`** altında çeşitli yollarla kaydedilir.

### **Paylaşılan Klasörler**
- Paylaşılan klasörler ve ayarlar **`System\ControlSet001\Services\lanmanserver\Shares`** altında bulunur. İstemci tarafı önbellekleme (CSC) ayarları çevrimdışı dosya kullanılabilirliğini belirler.

### **Otomatik Başlatılan Programlar**
- **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** gibi yollar ve `Software\Microsoft\Windows\CurrentVersion` altında benzer girişler, başlangıçta çalışacak şekilde ayarlanmış programları detaylandırır.

### **Aramalar ve Yazılan Yollar**
- Explorer aramaları ve yazılan yollar, sırasıyla **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** altında WordwheelQuery ve TypedPaths için izlenir.

### **Son Belgeler ve Ofis Dosyaları**
- Erişilen son belgeler ve Ofis dosyaları, `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` ve belirli Ofis sürüm yollarında not edilir.

### **En Son Kullanılan (MRU) Öğeler**
- Son dosya yollarını ve komutları gösteren MRU listeleri, `NTUSER.DAT` altında çeşitli `ComDlg32` ve `Explorer` alt anahtarlarında saklanır.

### **Kullanıcı Etkinliği Takibi**
- Kullanıcı Yardımcı özelliği, **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`** altında çalıştırma sayısı ve son çalıştırma zamanı dahil olmak üzere ayrıntılı uygulama kullanım istatistiklerini kaydeder.

### **Shellbags Analizi**
- Klasör erişim detaylarını ortaya çıkaran Shellbags, `USRCLASS.DAT` ve `NTUSER.DAT` altında `Software\Microsoft\Windows\Shell` içinde saklanır. Analiz için **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** kullanın.

### **USB Cihaz Geçmişi**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** ve **`HKLM\SYSTEM\ControlSet001\Enum\USB`** bağlı USB cihazları hakkında zengin detaylar içerir; bunlar arasında üretici, ürün adı ve bağlantı zaman damgaları bulunur.
- Belirli bir USB cihazıyla ilişkili kullanıcı, cihazın **{GUID}**'sini arayarak `NTUSER.DAT` hives'inde belirlenebilir.
- Son bağlanan cihaz ve hacim seri numarası, sırasıyla `System\MountedDevices` ve `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt` üzerinden izlenebilir.

Bu kılavuz, Windows sistemlerinde ayrıntılı sistem, ağ ve kullanıcı etkinliği bilgilerine erişim için kritik yolları ve yöntemleri özetlemektedir; açıklık ve kullanılabilirlik hedeflenmiştir.
