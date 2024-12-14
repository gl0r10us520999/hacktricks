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


# CBC

Eğer **cookie** **sadece** **kullanıcı adı** (veya cookie'nin ilk kısmı kullanıcı adıysa) ve kullanıcı adı "**admin**" olarak taklit edilmek isteniyorsa, o zaman kullanıcı adı **"bdmin"** oluşturulabilir ve **cookie'nin ilk baytı** için **bruteforce** yapılabilir.

# CBC-MAC

**Şifre blok zincirleme mesaj doğrulama kodu** (**CBC-MAC**), kriptografide kullanılan bir yöntemdir. Bu yöntem, bir mesajı blok blok şifreleyerek çalışır; her bloğun şifrelemesi, bir önceki bloğa bağlıdır. Bu süreç, **blokların bir zincirini** oluşturur ve orijinal mesajın tek bir bitinin değiştirilmesinin, şifrelenmiş verinin son bloğunda öngörülemeyen bir değişikliğe yol açacağını garanti eder. Böyle bir değişikliği yapmak veya tersine çevirmek için şifreleme anahtarı gereklidir, bu da güvenliği sağlar.

Mesaj m'nin CBC-MAC'ını hesaplamak için, m CBC modunda sıfır başlangıç vektörü ile şifrelenir ve son blok saklanır. Aşağıdaki şekil, gizli anahtar k ve bir blok şifre E kullanarak bloklardan oluşan bir mesajın CBC-MAC'ının hesaplanmasını tasvir etmektedir![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5):

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

CBC-MAC ile genellikle kullanılan **IV 0'dır**.\
Bu bir sorun çünkü 2 bilinen mesaj (`m1` ve `m2`) bağımsız olarak 2 imza (`s1` ve `s2`) üretecektir. Yani:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Sonra, m1 ve m2'nin birleştirilmesiyle oluşan bir mesaj (m3) 2 imza (s31 ve s32) üretecektir:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Bu, şifreleme anahtarını bilmeden hesaplanabilir.**

**Administrator** ismini **8bayt** bloklarda şifrelediğinizi hayal edin:

* `Administ`
* `rator\00\00\00`

**Administ** (m1) adında bir kullanıcı adı oluşturabilir ve imzayı (s1) alabilirsiniz.\
Sonra, `rator\00\00\00 XOR s1` sonucunu kullanarak bir kullanıcı adı oluşturabilirsiniz. Bu, `E(m2 XOR s1 XOR 0)` üretecektir ki bu da s32'dir.\
Artık, s32'yi **Administrator** tam adı için imza olarak kullanabilirsiniz.

### Summary

1. **Administ** (m1) kullanıcı adının imzasını alın, bu s1'dir.
2. **rator\x00\x00\x00 XOR s1 XOR 0** kullanıcı adının imzasını alın, bu s32'dir.
3. Cookie'yi s32 olarak ayarlayın ve bu, **Administrator** kullanıcısı için geçerli bir cookie olacaktır.

# Attack Controlling IV

Kullanılan IV'yi kontrol edebiliyorsanız, saldırı çok kolay olabilir.\
Eğer cookie sadece şifrelenmiş kullanıcı adıysa, "**administrator**" kullanıcısını taklit etmek için "**Administrator**" kullanıcısını oluşturabilir ve onun cookie'sini alabilirsiniz.\
Şimdi, eğer IV'yi kontrol edebiliyorsanız, IV'nin ilk Baytını değiştirebilir ve **IV\[0] XOR "A" == IV'\[0] XOR "a"** yaparak **Administrator** kullanıcısı için cookie'yi yeniden üretebilirsiniz. Bu cookie, başlangıç **IV** ile **administrator** kullanıcısını taklit etmek için geçerli olacaktır.

## References

Daha fazla bilgi için [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
