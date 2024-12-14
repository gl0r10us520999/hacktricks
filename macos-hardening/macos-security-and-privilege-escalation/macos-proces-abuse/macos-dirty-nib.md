# macOS Dirty NIB

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

**Teknik hakkında daha fazla bilgi için orijinal gönderiyi kontrol edin:** [**https://blog.xpnsec.com/dirtynib/**](https://blog.xpnsec.com/dirtynib/) ve [**https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/**](https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/) **gönderisini.** İşte bir özet:

### Nib dosyaları nedir

Nib (NeXT Interface Builder'ın kısaltması) dosyaları, Apple'ın geliştirme ekosisteminin bir parçası olarak, uygulamalardaki **UI öğelerini** ve etkileşimlerini tanımlamak için tasarlanmıştır. Pencereler ve düğmeler gibi serileştirilmiş nesneleri kapsar ve çalışma zamanında yüklenir. Sürekli kullanımlarına rağmen, Apple artık daha kapsamlı UI akış görselleştirmesi için Storyboard'ları önermektedir.

Ana Nib dosyası, uygulamanın `Info.plist` dosyasında **`NSMainNibFile`** değerinde referans alınır ve uygulamanın `main` fonksiyonunda yürütülen **`NSApplicationMain`** fonksiyonu tarafından yüklenir.

### Kirli Nib Enjeksiyon Süreci

#### NIB Dosyası Oluşturma ve Ayarlama

1. **Başlangıç Ayarı**:
* XCode kullanarak yeni bir NIB dosyası oluşturun.
* Arayüze bir Nesne ekleyin, sınıfını `NSAppleScript` olarak ayarlayın.
* Kullanıcı Tanımlı Çalışma Zamanı Özellikleri aracılığıyla başlangıç `source` özelliğini yapılandırın.
2. **Kod Yürütme Aleti**:
* Kurulum, AppleScript'in talep üzerine çalıştırılmasını sağlar.
* `Apple Script` nesnesini etkinleştirmek için bir düğme entegre edin, özellikle `executeAndReturnError:` seçicisini tetikleyin.
3. **Test Etme**:
* Test amaçlı basit bir Apple Script:

```bash
set theDialogText to "PWND"
display dialog theDialogText
```
* XCode hata ayıklayıcısında çalıştırarak ve düğmeye tıklayarak test edin.

#### Bir Uygulamayı Hedefleme (Örnek: Pages)

1. **Hazırlık**:
* Hedef uygulamayı (örneğin, Pages) ayrı bir dizine (örneğin, `/tmp/`) kopyalayın.
* Gatekeeper sorunlarını aşmak ve önbelleğe almak için uygulamayı başlatın.
2. **NIB Dosyasını Üzerine Yazma**:
* Mevcut bir NIB dosyasını (örneğin, Hakkında Panel NIB) oluşturulan DirtyNIB dosyasıyla değiştirin.
3. **Yürütme**:
* Uygulama ile etkileşimde bulunarak yürütmeyi tetikleyin (örneğin, `Hakkında` menü öğesini seçerek).

#### Kavramsal Kanıt: Kullanıcı Verilerine Erişim

* Kullanıcı izni olmadan fotoğraflar gibi kullanıcı verilerine erişmek ve çıkarmak için AppleScript'i değiştirin.

### Kod Örneği: Kötü Amaçlı .xib Dosyası

* Rastgele kod yürütmeyi gösteren [**kötü amaçlı bir .xib dosyası örneğini**](https://gist.github.com/xpn/16bfbe5a3f64fedfcc1822d0562636b4) erişin ve inceleyin.

### Diğer Örnek

[https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/](https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/) gönderisinde kirli nib oluşturma hakkında bir eğitim bulabilirsiniz.&#x20;

### Başlatma Kısıtlamalarını Ele Alma

* Başlatma Kısıtlamaları, uygulama yürütmesini beklenmedik yerlerden (örneğin, `/tmp`) engeller.
* Başlatma Kısıtlamaları ile korunmayan uygulamaları tanımlamak ve NIB dosyası enjeksiyonu için hedeflemek mümkündür.

### Ek macOS Koruma Önlemleri

macOS Sonoma'dan itibaren, Uygulama paketleri içindeki değişiklikler kısıtlanmıştır. Ancak, önceki yöntemler şunları içeriyordu:

1. Uygulamayı farklı bir konuma (örneğin, `/tmp/`) kopyalamak.
2. İlk korumaları aşmak için uygulama paketindeki dizinleri yeniden adlandırmak.
3. Uygulamayı Gatekeeper ile kaydetmek için çalıştırdıktan sonra, uygulama paketini değiştirmek (örneğin, MainMenu.nib'i Dirty.nib ile değiştirmek).
4. Dizinleri geri yeniden adlandırmak ve enjeksiyon yapılan NIB dosyasını yürütmek için uygulamayı yeniden çalıştırmak.

**Not**: Son macOS güncellemeleri, Gatekeeper önbelleklemesinden sonra uygulama paketleri içinde dosya değişikliklerini engelleyerek bu istismarı etkisiz hale getirmiştir.

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
