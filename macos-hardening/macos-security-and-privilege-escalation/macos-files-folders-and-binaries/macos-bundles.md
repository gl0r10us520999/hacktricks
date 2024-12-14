# macOS Bundles

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

## Temel Bilgiler

macOS'taki paketler, uygulamalar, kütüphaneler ve diğer gerekli dosyalar dahil olmak üzere çeşitli kaynaklar için konteyner görevi görür ve Finder'da tanıdık `*.app` dosyaları gibi tek nesneler olarak görünmelerini sağlar. En yaygın karşılaşılan paket `.app` paketidir, ancak `.framework`, `.systemextension` ve `.kext` gibi diğer türler de yaygındır.

### Bir Paketin Temel Bileşenleri

Bir paket içinde, özellikle `<application>.app/Contents/` dizininde, çeşitli önemli kaynaklar bulunmaktadır:

* **\_CodeSignature**: Bu dizin, uygulamanın bütünlüğünü doğrulamak için hayati öneme sahip kod imzalama ayrıntılarını saklar. Kod imzalama bilgilerini incelemek için şu komutları kullanabilirsiniz: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Kullanıcı etkileşimi sırasında çalışan uygulamanın yürütülebilir ikili dosyasını içerir.
* **Resources**: Uygulamanın kullanıcı arayüzü bileşenleri için bir depo, resimler, belgeler ve arayüz tanımları (nib/xib dosyaları) içerir.
* **Info.plist**: Uygulamanın ana yapılandırma dosyası olarak işlev görür, sistemin uygulamayı uygun şekilde tanıması ve etkileşimde bulunması için kritik öneme sahiptir.

#### Info.plist İçindeki Önemli Anahtarlar

`Info.plist` dosyası, uygulama yapılandırması için bir köşe taşıdır ve aşağıdaki gibi anahtarlar içerir:

* **CFBundleExecutable**: `Contents/MacOS` dizininde bulunan ana yürütülebilir dosyanın adını belirtir.
* **CFBundleIdentifier**: Uygulama için küresel bir tanımlayıcı sağlar, macOS tarafından uygulama yönetimi için yaygın olarak kullanılır.
* **LSMinimumSystemVersion**: Uygulamanın çalışması için gereken minimum macOS sürümünü belirtir.

### Paketleri Keşfetmek

Bir paketin içeriğini keşfetmek için, örneğin `Safari.app`, şu komut kullanılabilir: `bash ls -lR /Applications/Safari.app/Contents`

Bu keşif, uygulamanın güvenliğinden kullanıcı arayüzünü ve operasyonel parametrelerini tanımlamaya kadar her biri benzersiz bir amaca hizmet eden `_CodeSignature`, `MacOS`, `Resources` gibi dizinler ve `Info.plist` gibi dosyaları ortaya çıkarır.

#### Ek Paket Dizinleri

Yaygın dizinlerin ötesinde, paketler ayrıca şunları içerebilir:

* **Frameworks**: Uygulama tarafından kullanılan paketlenmiş çerçeveleri içerir. Çerçeveler, ekstra kaynaklarla birlikte dylib gibidir.
* **PlugIns**: Uygulamanın yeteneklerini artıran eklentiler ve uzantılar için bir dizin.
* **XPCServices**: Uygulama tarafından dış süreç iletişimi için kullanılan XPC hizmetlerini barındırır.

Bu yapı, gerekli tüm bileşenlerin paket içinde kapsüllenmesini sağlar ve modüler ve güvenli bir uygulama ortamını kolaylaştırır.

`Info.plist` anahtarları ve anlamları hakkında daha ayrıntılı bilgi için, Apple geliştirici belgeleri kapsamlı kaynaklar sunar: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
