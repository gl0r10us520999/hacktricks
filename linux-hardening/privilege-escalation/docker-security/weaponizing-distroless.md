# Weaponizing Distroless

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

## Distroless Nedir

Distroless bir konteyner, **belirli bir uygulamayı çalıştırmak için gerekli olan bağımlılıkları** içeren bir konteyner türüdür; gereksiz yazılım veya araçlar içermez. Bu konteynerler, **hafif** ve **güvenli** olmaları için tasarlanmıştır ve gereksiz bileşenleri kaldırarak **saldırı yüzeyini minimize etmeyi** hedefler.

Distroless konteynerler genellikle **güvenlik ve güvenilirliğin ön planda olduğu üretim ortamlarında** kullanılır.

**Distroless konteynerlere** bazı **örnekler** şunlardır:

* **Google** tarafından sağlanan: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* **Chainguard** tarafından sağlanan: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Distroless'ı Silahlandırma

Distroless bir konteyneri silahlandırmanın amacı, **distroless** tarafından getirilen **sınırlamalara** (sistemde yaygın ikili dosyaların eksikliği) rağmen **rastgele ikili dosyaları ve yükleri çalıştırabilmektir**; ayrıca `/dev/shm`'de bulunan **salt okunur** veya **çalıştırılamaz** gibi korumalara karşı da geçerli olmaktır.

### Bellek Üzerinden

2023'ün bir noktasında gelecek...

### Mevcut ikili dosyalar aracılığıyla

#### openssl

****[**Bu yazıda,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) **`openssl`** ikilisinin bu konteynerlerde sıkça bulunduğu, muhtemelen konteyner içinde çalışacak yazılım tarafından **gerekli** olduğu açıklanmaktadır.


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
