{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}


# ECB

(ECB) Elektronik Kod Kitabı - her **açık metin bloğunu** **şifreli metin bloğu ile** **değiştiren** simetrik şifreleme şemasıdır. En **basit** şifreleme şemasını temsil eder. Ana fikir, açık metni **N bitlik bloklara** **bölmek** (girdi verisinin blok boyutuna, şifreleme algoritmasına bağlıdır) ve ardından yalnızca anahtarı kullanarak her açık metin bloğunu şifrelemektir (şifre çözmektir).

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

ECB kullanmanın birçok güvenlik etkisi vardır:

* **Şifreli mesajdan bloklar çıkarılabilir**
* **Şifreli mesajdan bloklar yer değiştirebilir**

# Açığın Tespiti

Bir uygulamaya birkaç kez giriş yaptığınızı ve **her zaman aynı çerezi** aldığınızı hayal edin. Bu, uygulamanın çerezinin **`<kullanıcı_adı>|<şifre>`** olmasındandır.\
Sonra, her ikisi de **aynı uzun şifreye** ve **neredeyse** **aynı** **kullanıcı adına** sahip iki yeni kullanıcı oluşturursunuz.\
Her iki kullanıcının **bilgilerinin** bulunduğu **8B'lik blokların** **eşit** olduğunu keşfedersiniz. Sonra, bunun **ECB'nin kullanılıyor olabileceği** nedeniyle olabileceğini hayal edersiniz.

Aşağıdaki örnekte olduğu gibi. Bu **2 çözülmüş çerezin** nasıl birkaç kez **`\x23U\xE45K\xCB\x21\xC8`** bloğunu içerdiğine dikkat edin.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Bu, **o çerezlerin kullanıcı adı ve şifresinin birkaç kez "a" harfini içermesinden** kaynaklanmaktadır (örneğin). **Farklı** olan **bloklar**, **en az 1 farklı karakter** içeren bloklardır (belki ayırıcı "|" veya kullanıcı adındaki bazı gerekli farklılık).

Şimdi, saldırganın formatın `<username><delimiter><password>` mi yoksa `<password><delimiter><username>` mi olduğunu keşfetmesi gerekiyor. Bunu yapmak için, **formatı ve ayırıcının uzunluğunu bulana kadar benzer ve uzun kullanıcı adları ve şifreler** ile birkaç kullanıcı adı oluşturabilir: 

| Kullanıcı adı uzunluğu: | Şifre uzunluğu: | Kullanıcı adı+Şifre uzunluğu: | Çerezin uzunluğu (çözüldükten sonra): |
| ----------------------- | ---------------- | ------------------------------ | ------------------------------------- |
| 2                       | 2                | 4                              | 8                                     |
| 3                       | 3                | 6                              | 8                                     |
| 3                       | 4                | 7                              | 8                                     |
| 4                       | 4                | 8                              | 16                                    |
| 7                       | 7                | 14                             | 16                                    |

# Açığın istismarı

## Tüm blokları kaldırma

Çerezin formatını bilerek (`<username>|<password>`), `admin` kullanıcı adını taklit etmek için `aaaaaaaaadmin` adında yeni bir kullanıcı oluşturun ve çerezi alın ve çözün:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Görünüşe göre daha önce yalnızca `a` içeren kullanıcı adıyla oluşturulan `\x23U\xE45K\xCB\x21\xC8` desenini görebiliriz.\
Sonra, 8B'lik ilk bloğu kaldırabilir ve `admin` kullanıcı adı için geçerli bir çerez elde edersiniz:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Blokları Taşıma

Birçok veritabanında `WHERE username='admin';` veya `WHERE username='admin    ';` aramak aynı şeydir. _(Ekstra boşluklara dikkat edin)_

Bu nedenle, `admin` kullanıcısını taklit etmenin başka bir yolu:

* `len(<username>) + len(<delimiter) % len(block)` olan bir kullanıcı adı oluşturmak. `8B` blok boyutuyla, `username       ` adında bir kullanıcı adı oluşturabilirsiniz; ayırıcı olarak `|` kullanıldığında `<username><delimiter>` parçası 2 adet 8B blok oluşturacaktır.
* Ardından, taklit etmek istediğimiz kullanıcı adını ve boşlukları içeren tam sayıda blok dolduracak bir şifre oluşturmak, örneğin: `admin   `

Bu kullanıcının çerezi 3 bloktan oluşacak: ilk 2 blok kullanıcı adı + ayırıcı blokları ve üçüncü blok şifre (kullanıcı adını taklit eden): `username       |admin   `

**Sonra, sadece ilk bloğu son blokla değiştirin ve `admin` kullanıcısını taklit etmiş olacaksınız: `admin          |username`**

## Referanslar

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


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
