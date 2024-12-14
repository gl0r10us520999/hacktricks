# ZIPs tricks

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** bizi takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

**Komut satırı araçları**, **zip dosyalarını** yönetmek için, zip dosyalarını teşhis etmek, onarmak ve kırmak için gereklidir. İşte bazı anahtar yardımcı programlar:

- **`unzip`**: Bir zip dosyasının neden açılmadığını gösterir.
- **`zipdetails -v`**: Zip dosyası format alanlarının ayrıntılı analizini sunar.
- **`zipinfo`**: Bir zip dosyasının içeriğini çıkarmadan listeler.
- **`zip -F input.zip --out output.zip`** ve **`zip -FF input.zip --out output.zip`**: Bozulmuş zip dosyalarını onarmaya çalışır.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Zip şifrelerini brute-force ile kırmak için bir araç, yaklaşık 7 karaktere kadar olan şifreler için etkilidir.

[Zip dosya formatı spesifikasyonu](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT), zip dosyalarının yapısı ve standartları hakkında kapsamlı bilgiler sunar.

Şunu belirtmek önemlidir ki, şifre korumalı zip dosyaları **içindeki dosya adlarını veya dosya boyutlarını şifrelemez**, bu, RAR veya 7z dosyalarıyla paylaşılmayan bir güvenlik açığıdır; bu dosyalar bu bilgileri şifreler. Ayrıca, daha eski ZipCrypto yöntemiyle şifrelenmiş zip dosyaları, sıkıştırılmış bir dosyanın şifrelenmemiş bir kopyası mevcutsa **düz metin saldırısına** karşı savunmasızdır. Bu saldırı, zip'in şifresini kırmak için bilinen içeriği kullanır; bu zayıflık [HackThis makalesinde](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) ve [bu akademik makalede](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) daha ayrıntılı olarak açıklanmıştır. Ancak, **AES-256** şifrelemesiyle güvence altına alınmış zip dosyaları bu düz metin saldırısına karşı bağışıklık gösterir; bu, hassas veriler için güvenli şifreleme yöntemlerinin seçilmesinin önemini göstermektedir.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** bizi takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
