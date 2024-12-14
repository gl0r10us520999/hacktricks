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


## smss.exe

**Oturum Yöneticisi**.\
Oturum 0, **csrss.exe** ve **wininit.exe** (**OS** **hizmetleri**) başlatırken, Oturum 1 **csrss.exe** ve **winlogon.exe** (**Kullanıcı** **oturumu**) başlatır. Ancak, süreçler ağacında **çocukları olmayan** bu **ikili** dosyadan **sadece bir süreç** görmelisiniz.

Ayrıca, 0 ve 1 dışındaki oturumlar RDP oturumlarının gerçekleştiğini gösterebilir.


## csrss.exe

**İstemci/Sunucu Çalıştırma Alt Sistemi Süreci**.\
**Süreçleri** ve **iş parçacıklarını** yönetir, diğer süreçler için **Windows** **API**'sini kullanılabilir hale getirir ve ayrıca **sürücü harflerini** haritalar, **geçici dosyalar** oluşturur ve **kapatma** **sürecini** yönetir.

Oturum 0'da bir tane ve Oturum 1'de başka bir tane **çalışıyor** (yani süreçler ağacında **2 süreç**). Her yeni oturum için **bir tane** daha oluşturulur.


## winlogon.exe

**Windows Oturum Açma Süreci**.\
Kullanıcı **oturum açma**/**oturum kapama** işlemlerinden sorumludur. Kullanıcı adı ve şifre sormak için **logonui.exe**'yi başlatır ve ardından bunları doğrulamak için **lsass.exe**'yi çağırır.

Sonra, **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`**'da **Userinit** anahtarı ile belirtilen **userinit.exe**'yi başlatır.

Ayrıca, önceki kayıt defterinde **explorer.exe** **Shell anahtarı** içinde olmalıdır veya bu, **kötü amaçlı yazılım kalıcılık yöntemi** olarak kötüye kullanılabilir.


## wininit.exe

**Windows Başlatma Süreci**. \
Oturum 0'da **services.exe**, **lsass.exe** ve **lsm.exe**'yi başlatır. Sadece 1 süreç olmalıdır.


## userinit.exe

**Userinit Oturum Açma Uygulaması**.\
**HKCU**'da **ntduser.dat**'ı yükler ve **kullanıcı** **ortamını** başlatır, **oturum açma** **betiklerini** ve **GPO**'yu çalıştırır.

**explorer.exe**'yi başlatır.


## lsm.exe

**Yerel Oturum Yöneticisi**.\
Kullanıcı oturumlarını manipüle etmek için smss.exe ile çalışır: Oturum açma/kapama, kabuk başlatma, masaüstünü kilitleme/açma vb.

W7'den sonra lsm.exe bir hizmete dönüştürüldü (lsm.dll).

W7'de yalnızca 1 süreç olmalıdır ve bunlardan bir hizmet DLL'yi çalıştırmalıdır.


## services.exe

**Hizmet Kontrol Yöneticisi**.\
**Otomatik başlatma** olarak yapılandırılan **hizmetleri** ve **sürücüleri** **yükler**.

**svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** ve daha birçok sürecin ebeveynidir.

Hizmetler `HKLM\SYSTEM\CurrentControlSet\Services` içinde tanımlanır ve bu süreç, sc.exe tarafından sorgulanabilen hizmet bilgilerini bellekte tutan bir veritabanı yönetir.

**Bazı** **hizmetlerin** kendi **süreçlerinde** çalışacağını ve diğerlerinin **bir svchost.exe sürecini paylaşacağını** not edin.

Sadece 1 süreç olmalıdır.


## lsass.exe

**Yerel Güvenlik Otoritesi Alt Sistemi**.\
Kullanıcı **kimlik doğrulaması** ve **güvenlik** **jetonları** oluşturmakla sorumludur. `HKLM\System\CurrentControlSet\Control\Lsa` içinde bulunan kimlik doğrulama paketlerini kullanır.

**Güvenlik** **olay** **kaydı**'na yazar ve yalnızca 1 süreç olmalıdır.

Bu sürecin şifreleri dökmek için yüksek oranda saldırıya uğradığını unutmayın.


## svchost.exe

**Genel Hizmet Ana Bilgisayar Süreci**.\
Birden fazla DLL hizmetini tek bir paylaşılan süreçte barındırır.

Genellikle, **svchost.exe**'nin `-k` bayrağı ile başlatıldığını göreceksiniz. Bu, kayıt defterine **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** sorgusu başlatır; burada -k'da belirtilen argümanla bir anahtar bulunur ve bu anahtar aynı süreçte başlatılacak hizmetleri içerir.

Örneğin: `-k UnistackSvcGroup` şunları başlatır: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Eğer **`-s` bayrağı** da bir argüman ile kullanılıyorsa, svchost'tan **yalnızca bu argümanda belirtilen hizmeti** başlatması istenir.

Birçok `svchost.exe` süreci olacaktır. Eğer bunlardan herhangi biri **`-k` bayrağını** kullanmıyorsa, bu çok şüphelidir. Eğer **services.exe ebeveyn değilse**, bu da çok şüphelidir.


## taskhost.exe

Bu süreç, DLL'lerden çalışan süreçler için bir ana bilgisayar görevi görür. Ayrıca, DLL'lerden çalışan hizmetleri yükler.

W8'de bu taskhostex.exe olarak adlandırılır ve W10'da taskhostw.exe'dir.


## explorer.exe

Bu, **kullanıcının masaüstünden** ve dosyaları dosya uzantıları aracılığıyla başlatmaktan sorumlu olan süreçtir.

**Her oturum açan kullanıcı için yalnızca 1** süreç oluşturulmalıdır.

Bu, **userinit.exe**'den çalıştırılır ve sonlandırılmalıdır, bu nedenle bu süreç için **ebeveyn** görünmemelidir.


# Kötü Amaçlı Süreçleri Yakalamak

* Beklenen yoldan mı çalışıyor? (Windows ikili dosyaları geçici konumdan çalışmaz)
* Garip IP'lerle mi iletişim kuruyor?
* Dijital imzaları kontrol edin (Microsoft belgeleri imzalanmış olmalıdır)
* Doğru yazılmış mı?
* Beklenen SID altında mı çalışıyor?
* Ebeveyn süreç beklenen mi (varsa)?
* Çocuk süreçler beklenenler mi? (cmd.exe, wscript.exe, powershell.exe yok mu..?)
