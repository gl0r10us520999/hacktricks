# macOS Sistem Uzantıları

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **Bize katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya **bizi** **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## Sistem Uzantıları / Uç Nokta Güvenlik Çerçevesi

Kernel Uzantılarından farklı olarak, **Sistem Uzantıları kullanıcı alanında çalışır** ve bu da uzantı arızası nedeniyle sistem çökmesi riskini azaltır.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Üç tür sistem uzantısı vardır: **DriverKit** Uzantıları, **Ağ** Uzantıları ve **Uç Nokta Güvenliği** Uzantıları.

### **DriverKit Uzantıları**

DriverKit, **donanım desteği sağlayan** kernel uzantılarının yerini alır. Cihaz sürücülerinin (USB, Seri, NIC ve HID sürücüleri gibi) kernel alanında değil, kullanıcı alanında çalışmasına olanak tanır. DriverKit çerçevesi, **belirli I/O Kit sınıflarının kullanıcı alanı sürümlerini** içerir ve kernel, normal I/O Kit olaylarını kullanıcı alanına ileterek bu sürücülerin çalışması için daha güvenli bir ortam sunar.

### **Ağ Uzantıları**

Ağ Uzantıları, ağ davranışlarını özelleştirme yeteneği sağlar. Birkaç tür Ağ Uzantısı vardır:

* **Uygulama Proxy**: Akış odaklı, özel bir VPN protokolü uygulayan bir VPN istemcisi oluşturmak için kullanılır. Bu, ağ trafiğini bağlantılara (veya akışlara) göre yönetir, bireysel paketler yerine.
* **Paket Tüneli**: Bireysel paketlere dayalı, özel bir VPN protokolü uygulayan bir VPN istemcisi oluşturmak için kullanılır. Bu, ağ trafiğini bireysel paketler bazında yönetir.
* **Veri Filtreleme**: Ağ "akışlarını" filtrelemek için kullanılır. Akış düzeyinde ağ verilerini izleyebilir veya değiştirebilir.
* **Paket Filtreleme**: Bireysel ağ paketlerini filtrelemek için kullanılır. Paket düzeyinde ağ verilerini izleyebilir veya değiştirebilir.
* **DNS Proxy**: Özel bir DNS sağlayıcısı oluşturmak için kullanılır. DNS isteklerini ve yanıtlarını izlemek veya değiştirmek için kullanılabilir.

## Uç Nokta Güvenlik Çerçevesi

Uç Nokta Güvenliği, Apple tarafından macOS'ta sağlanan bir çerçevedir ve sistem güvenliği için bir dizi API sunar. **Kötü niyetli etkinlikleri tanımlamak ve korumak için sistem etkinliğini izleyip kontrol edebilen ürünler geliştirmek üzere güvenlik satıcıları ve geliştiriciler tarafından kullanılmak üzere tasarlanmıştır.**

Bu çerçeve, **sistem etkinliğini izlemek ve kontrol etmek için bir dizi API sağlar**, örneğin süreç yürütmeleri, dosya sistemi olayları, ağ ve kernel olayları gibi.

Bu çerçevenin temeli, **`/System/Library/Extensions/EndpointSecurity.kext`** konumunda bulunan bir Kernel Uzantısı (KEXT) olarak kernel'de uygulanmıştır. Bu KEXT, birkaç ana bileşenden oluşur:

* **EndpointSecurityDriver**: Bu, kernel uzantısı için "giriş noktası" olarak işlev görür. OS ile Uç Nokta Güvenlik çerçevesi arasındaki ana etkileşim noktasıdır.
* **EndpointSecurityEventManager**: Bu bileşen, kernel kancalarını uygulamaktan sorumludur. Kernel kancaları, çerçevenin sistem çağrılarını keserek sistem olaylarını izlemesine olanak tanır.
* **EndpointSecurityClientManager**: Bu, kullanıcı alanı istemcileriyle iletişimi yönetir, hangi istemcilerin bağlı olduğunu ve olay bildirimlerini alması gerektiğini takip eder.
* **EndpointSecurityMessageManager**: Bu, kullanıcı alanı istemcilerine mesajlar ve olay bildirimleri gönderir.

Uç Nokta Güvenlik çerçevesinin izleyebileceği olaylar şunlara ayrılır:

* Dosya olayları
* Süreç olayları
* Soket olayları
* Kernel olayları (örneğin, bir kernel uzantısının yüklenmesi/boşaltılması veya bir I/O Kit cihazının açılması)

### Uç Nokta Güvenlik Çerçevesi Mimarisi

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Kullanıcı alanı iletişimi**, IOUserClient sınıfı aracılığıyla Uç Nokta Güvenlik çerçevesi ile gerçekleşir. İki farklı alt sınıf, çağıran türüne bağlı olarak kullanılır:

* **EndpointSecurityDriverClient**: Bu, yalnızca sistem süreci `endpointsecurityd` tarafından tutulan `com.apple.private.endpoint-security.manager` yetkisini gerektirir.
* **EndpointSecurityExternalClient**: Bu, `com.apple.developer.endpoint-security.client` yetkisini gerektirir. Bu genellikle Uç Nokta Güvenlik çerçevesi ile etkileşimde bulunması gereken üçüncü taraf güvenlik yazılımları tarafından kullanılır.

Uç Nokta Güvenlik Uzantıları:**`libEndpointSecurity.dylib`** sistem uzantılarının kernel ile iletişim kurmak için kullandığı C kütüphanesidir. Bu kütüphane, Uç Nokta Güvenlik KEXT ile iletişim kurmak için I/O Kit (`IOKit`) kullanır.

**`endpointsecurityd`** uç nokta güvenlik sistem uzantılarını yönetmek ve başlatmakla ilgili önemli bir sistem daemon'udur, özellikle erken önyükleme sürecinde. **Sadece sistem uzantıları**, `Info.plist` dosyalarında **`NSEndpointSecurityEarlyBoot`** ile işaretlenmiş olanlar bu erken önyükleme muamelesini alır.

Başka bir sistem daemon'u, **`sysextd`**, **sistem uzantılarını doğrular** ve bunları uygun sistem konumlarına taşır. Ardından ilgili daemon'dan uzantıyı yüklemesini ister. **`SystemExtensions.framework`**, sistem uzantılarını etkinleştirmek ve devre dışı bırakmaktan sorumludur.

## ESF'yi Aşmak

ESF, bir kırmızı takımcıyı tespit etmeye çalışan güvenlik araçları tarafından kullanılır, bu nedenle bunun nasıl önlenebileceğine dair herhangi bir bilgi ilginçtir.

### CVE-2021-30965

Sorun, güvenlik uygulamasının **Tam Disk Erişimi izinlerine** sahip olması gerektiğidir. Yani, bir saldırgan bunu kaldırabilirse, yazılımın çalışmasını engelleyebilir:
```bash
tccutil reset All
```
Daha fazla bilgi için bu bypass ve ilgili olanlar hakkında [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI) konuşmasına bakın.

Sonunda, bu, **`tccd`** tarafından yönetilen güvenlik uygulamasına yeni izin **`kTCCServiceEndpointSecurityClient`** verilerek düzeltildi, böylece `tccutil` izinlerini temizlemeyecek ve çalışmasını engellemeyecek.

## Referanslar

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
