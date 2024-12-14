# ZIPs tricks

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

**Komut satırı araçları**, **zip dosyalarını** yönetmek için, zip dosyalarını teşhis etmek, onarmak ve kırmak için gereklidir. İşte bazı anahtar yardımcı programlar:

- **`unzip`**: Bir zip dosyasının neden açılmadığını gösterir.
- **`zipdetails -v`**: Zip dosyası format alanlarının ayrıntılı analizini sunar.
- **`zipinfo`**: Bir zip dosyasının içeriğini çıkarmadan listeler.
- **`zip -F input.zip --out output.zip`** ve **`zip -FF input.zip --out output.zip`**: Bozulmuş zip dosyalarını onarmaya çalışır.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Zip şifrelerini brute-force ile kırmak için bir araç, yaklaşık 7 karaktere kadar şifreler için etkilidir.

[Zip dosya formatı spesifikasyonu](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT), zip dosyalarının yapısı ve standartları hakkında kapsamlı bilgiler sunar.

Şifre korumalı zip dosyalarının **içindeki dosya adlarını veya dosya boyutlarını şifrelemediğini** belirtmek önemlidir; bu, RAR veya 7z dosyalarıyla paylaşılmayan bir güvenlik açığıdır. Ayrıca, daha eski ZipCrypto yöntemiyle şifrelenmiş zip dosyaları, sıkıştırılmış bir dosyanın şifrelenmemiş bir kopyası mevcutsa **düz metin saldırısına** karşı savunmasızdır. Bu saldırı, zip'in şifresini kırmak için bilinen içeriği kullanır; bu zayıflık [HackThis makalesinde](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) ve [bu akademik çalışmada](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) daha ayrıntılı olarak açıklanmıştır. Ancak, **AES-256** şifrelemesiyle güvence altına alınmış zip dosyaları bu düz metin saldırısına karşı bağışıklık gösterir; bu, hassas veriler için güvenli şifreleme yöntemlerinin seçilmesinin önemini göstermektedir.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

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
