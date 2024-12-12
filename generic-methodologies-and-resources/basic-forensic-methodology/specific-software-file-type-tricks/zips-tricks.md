# ZIP hileleri

{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitimi AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitimi GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) katılın veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* Hacking püf noktalarını paylaşarak PR'ler göndererek **HackTricks** ve **HackTricks Cloud** github depolarına katkıda bulunun.

</details>
{% endhint %}

**Komut satırı araçları** zip dosyalarını yönetmek için gereklidir ve zip dosyalarını teşhis etmek, onarmak ve kırmak için kullanılır. İşte bazı temel yardımcı programlar:

- **`unzip`**: Bir zip dosyasının neden açılmadığını ortaya çıkarır.
- **`zipdetails -v`**: Zip dosyası biçim alanlarının detaylı analizini sunar.
- **`zipinfo`**: İçerikleri çıkarmadan bir zip dosyasının içeriğini listeler.
- **`zip -F input.zip --out output.zip`** ve **`zip -FF input.zip --out output.zip`**: Bozuk zip dosyalarını onarmayı deneyin.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Zip şifrelerini kaba kuvvet yöntemiyle kırmak için etkili olan, yaklaşık 7 karaktere kadar şifreler için etkili bir araç.

[Zip dosya biçimi belirtimi](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT), zip dosyalarının yapısı ve standartları hakkında kapsamlı bilgiler sağlar.

Önemli bir not olarak, şifre korumalı zip dosyalarının içindeki dosya adlarını veya dosya boyutlarını **şifrelemediğini** unutmamak önemlidir; bu, RAR veya 7z dosyalarıyla paylaşılmayan bir güvenlik açığıdır, çünkü bu bilgileri şifreler. Ayrıca, eski ZipCrypto yöntemiyle şifrelenmiş zip dosyaları, sıkıştırılmış bir dosyanın şifresiz bir kopyası mevcutsa **düz metin saldırısına** açıktır. Bu saldırı, zip dosyasının şifresini kırmak için bilinen içeriği kullanır; bu zayıflık [HackThis'in makalesinde](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) detaylı olarak açıklanmış ve [bu akademik makalede](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) daha fazla açıklanmıştır. Bununla birlikte, **AES-256** şifreleme ile korunan zip dosyaları, bu düz metin saldırısına karşı bağışıktır ve hassas veriler için güvenli şifreleme yöntemlerini seçmenin önemini gösterir.

## Referanslar
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/) 

{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitimi AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitimi GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) katılın veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* Hacking püf noktalarını paylaşarak PR'ler göndererek **HackTricks** ve **HackTricks Cloud** github depolarına katkıda bulunun.

</details>
{% endhint %}
