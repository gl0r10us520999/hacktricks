# Diğer Web Hileleri

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking hilelerini paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

### Host başlığı

Birçok kez arka uç, bazı işlemleri gerçekleştirmek için **Host başlığını** güvenilir olarak kabul eder. Örneğin, bu değeri **şifre sıfırlama için kullanılacak alan adı** olarak kullanabilir. Yani, şifrenizi sıfırlamak için bir bağlantı içeren bir e-posta aldığınızda, kullanılan alan adı Host başlığında belirttiğiniz alandır. Sonra, diğer kullanıcıların şifre sıfırlama taleplerini yapabilir ve alan adını kontrolünüz altındaki bir alan adıyla değiştirerek şifre sıfırlama kodlarını çalabilirsiniz. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
Kullanıcının şifre sıfırlama bağlantısına tıklamasını beklemenize gerek olmadığını unutmayın, çünkü belki de **spam filtreleri veya diğer ara cihazlar/botlar bunu analiz etmek için tıklayabilir**.
{% endhint %}

### Oturum boolean'ları

Bazen bazı doğrulamaları doğru bir şekilde tamamladığınızda arka uç, **oturumunuza bir güvenlik niteliğine "True" değeri ekler**. Sonra, farklı bir uç nokta bu kontrolü başarıyla geçip geçmediğinizi bilecektir.\
Ancak, eğer **kontrolü geçerseniz** ve oturumunuza güvenlik niteliğinde "True" değeri verilirse, **aynı niteliğe bağlı olan diğer kaynaklara erişmeyi** deneyebilirsiniz, ancak **erişim izniniz olmamalıdır**. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Kayıt işlevselliği

Zaten mevcut bir kullanıcı olarak kaydolmayı deneyin. Eşdeğer karakterler (nokta, çok sayıda boşluk ve Unicode) kullanmayı da deneyin.

### E-posta ele geçirme

Bir e-posta kaydedin, onaylamadan önce e-postayı değiştirin, ardından, yeni onay e-postası ilk kaydedilen e-postaya gönderilirse, herhangi bir e-postayı ele geçirebilirsiniz. Ya da ikinci e-postayı birincisini onaylayacak şekilde etkinleştirebilirseniz, herhangi bir hesabı da ele geçirebilirsiniz.

### Atlassian kullanan şirketlerin İç Servis Masasına Erişim

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACE yöntemi

Geliştiriciler, üretim ortamında çeşitli hata ayıklama seçeneklerini devre dışı bırakmayı unutabilir. Örneğin, HTTP `TRACE` yöntemi tanısal amaçlar için tasarlanmıştır. Eğer etkinse, web sunucusu `TRACE` yöntemini kullanan isteklere, alınan isteği yanıtında yankılayarak yanıt verecektir. Bu davranış genellikle zararsızdır, ancak bazen, ters proxyler tarafından isteklere eklenebilecek dahili kimlik doğrulama başlıklarının adları gibi bilgi sızmasına yol açabilir.![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking hilelerini paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
