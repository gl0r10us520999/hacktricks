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


# Temel

Bir temel, bir sistemin belirli bölümlerinin anlık görüntüsünü almayı içerir ve **değişiklikleri vurgulamak için gelecekteki bir durumla karşılaştırılır**.

Örneğin, hangi dosyaların değiştirildiğini bulabilmek için dosya sisteminin her dosyasının hash'ini hesaplayabilir ve saklayabilirsiniz.\
Bu, oluşturulan kullanıcı hesapları, çalışan süreçler, çalışan hizmetler ve çok fazla değişmemesi gereken veya hiç değişmemesi gereken diğer şeyler için de yapılabilir.

## Dosya Bütünlüğü İzleme

Dosya Bütünlüğü İzleme (FIM), dosyalardaki değişiklikleri izleyerek BT ortamlarını ve verileri koruyan kritik bir güvenlik tekniğidir. İki ana adımı içerir:

1. **Temel Karşılaştırması:** Değişiklikleri tespit etmek için gelecekteki karşılaştırmalar için dosya özellikleri veya kriptografik kontrol toplamları (MD5 veya SHA-2 gibi) kullanarak bir temel oluşturun.
2. **Gerçek Zamanlı Değişiklik Bildirimi:** Dosyalar erişildiğinde veya değiştirildiğinde, genellikle işletim sistemi çekirdek uzantıları aracılığıyla anında bildirim alın.

## Araçlar

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

## Referanslar

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


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
