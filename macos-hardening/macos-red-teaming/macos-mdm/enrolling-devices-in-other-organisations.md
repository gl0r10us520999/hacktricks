# Diğer Kuruluşlarda Cihaz Kaydı

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Giriş

Daha önce belirtildiği gibi, bir cihazı bir kuruluşa kaydetmek için **sadece o Kuruluşa ait bir Seri Numarası gereklidir**. Cihaz kaydedildikten sonra, birçok kuruluş yeni cihaza hassas veriler yükleyecektir: sertifikalar, uygulamalar, WiFi şifreleri, VPN yapılandırmaları [ve benzeri](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Bu nedenle, kayıt süreci doğru bir şekilde korunmazsa, bu saldırganlar için tehlikeli bir giriş noktası olabilir.

**Aşağıda, araştırmanın bir özeti bulunmaktadır [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe). Daha fazla teknik detay için kontrol edin!**

## DEP ve MDM İkili Analizi Genel Görünümü

Bu araştırma, macOS'taki Cihaz Kaydı Programı (DEP) ve Mobil Cihaz Yönetimi (MDM) ile ilişkili ikililere dalmaktadır. Ana bileşenler şunlardır:

- **`mdmclient`**: MDM sunucularıyla iletişim kurar ve macOS 10.13.4 öncesi sürümlerde DEP kontrol noktalarını tetikler.
- **`profiles`**: Yapılandırma Profillerini yönetir ve macOS 10.13.4 ve sonraki sürümlerde DEP kontrol noktalarını tetikler.
- **`cloudconfigurationd`**: DEP API iletişimlerini yönetir ve Cihaz Kaydı profillerini alır.

DEP kontrol noktaları, Aktivasyon Kaydını almak için özel Yapılandırma Profilleri çerçevesinden `CPFetchActivationRecord` ve `CPGetActivationRecord` işlevlerini kullanır; `CPFetchActivationRecord`, `cloudconfigurationd` ile XPC üzerinden koordine eder.

## Tesla Protokolü ve Absinthe Şeması Tersine Mühendislik

DEP kontrol noktası, `cloudconfigurationd`'nin _iprofiles.apple.com/macProfile_ adresine şifrelenmiş, imzalı bir JSON yükü göndermesini içerir. Yük, cihazın seri numarasını ve "RequestProfileConfiguration" eylemini içerir. Kullanılan şifreleme şeması dahili olarak "Absinthe" olarak adlandırılmaktadır. Bu şemanın çözülmesi karmaşık olup birçok adım içermektedir; bu da Aktivasyon Kaydı isteğine keyfi seri numaraları eklemek için alternatif yöntemlerin araştırılmasına yol açmıştır.

## DEP İsteklerini Proxyleme

_iprofiles.apple.com_ adresine yapılan DEP isteklerini Charles Proxy gibi araçlarla kesmeye ve değiştirmeye yönelik girişimler, yük şifrelemesi ve SSL/TLS güvenlik önlemleri nedeniyle engellenmiştir. Ancak, `MCCloudConfigAcceptAnyHTTPSCertificate` yapılandırmasını etkinleştirmek, sunucu sertifikası doğrulamasını atlamaya olanak tanır; ancak yükün şifreli doğası, şifre çözme anahtarı olmadan seri numarasının değiştirilmesini engellemektedir.

## DEP ile Etkileşime Giren Sistem İkili Dosyalarını Enstrümante Etme

`cloudconfigurationd` gibi sistem ikili dosyalarını enstrümante etmek, macOS'ta Sistem Bütünlüğü Koruması'nın (SIP) devre dışı bırakılmasını gerektirir. SIP devre dışı bırakıldığında, LLDB gibi araçlar sistem süreçlerine bağlanmak ve DEP API etkileşimlerinde kullanılan seri numarasını potansiyel olarak değiştirmek için kullanılabilir. Bu yöntem, yetkilendirmeler ve kod imzalama karmaşıklıklarından kaçındığı için tercih edilmektedir.

**İkili Enstrümantasyonun Sömürülmesi:**
`cloudconfigurationd`'de JSON serileştirmeden önce DEP istek yükünü değiştirmek etkili olmuştur. Süreç şunları içermektedir:

1. LLDB'yi `cloudconfigurationd`'ye bağlamak.
2. Sistem seri numarasının alındığı noktayı bulmak.
3. Yük şifrelenmeden ve gönderilmeden önce belleğe keyfi bir seri numarası enjekte etmek.

Bu yöntem, keyfi seri numaraları için tam DEP profillerinin alınmasını sağladı ve potansiyel bir zafiyeti gösterdi.

### Python ile Enstrümantasyonu Otomatikleştirme

Sömürü süreci, LLDB API'si ile Python kullanılarak otomatikleştirildi ve keyfi seri numaralarının programatik olarak enjekte edilmesi ve karşılık gelen DEP profillerinin alınması mümkün hale getirildi.

### DEP ve MDM Zafiyetlerinin Potansiyel Etkileri

Araştırma, önemli güvenlik endişelerini vurgulamıştır:

1. **Bilgi Sızdırma**: DEP'e kayıtlı bir seri numarası sağlayarak, DEP profilinde bulunan hassas kurumsal bilgilere erişim sağlanabilir.
