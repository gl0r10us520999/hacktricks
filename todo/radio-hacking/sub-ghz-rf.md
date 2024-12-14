# Sub-GHz RF

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## Garage Doors

Garaj kapısı açıcıları genellikle 300-190 MHz aralığında çalışır ve en yaygın frekanslar 300 MHz, 310 MHz, 315 MHz ve 390 MHz'dir. Bu frekans aralığı, diğer frekans bantlarına göre daha az kalabalık olduğu ve diğer cihazlardan gelen parazitlerden daha az etkilenme olasılığı olduğu için garaj kapısı açıcıları için yaygın olarak kullanılır.

## Car Doors

Çoğu araba anahtar uzaktan kumandası ya **315 MHz ya da 433 MHz** üzerinde çalışır. Bu ikisi de radyo frekanslarıdır ve çeşitli uygulamalarda kullanılır. İki frekans arasındaki ana fark, 433 MHz'nin 315 MHz'den daha uzun bir menzil sunmasıdır. Bu, 433 MHz'nin uzaktan anahtarsız giriş gibi daha uzun menzil gerektiren uygulamalar için daha iyi olduğu anlamına gelir.\
Avrupa'da 433.92MHz yaygın olarak kullanılırken, ABD ve Japonya'da 315MHz kullanılmaktadır.

## **Brute-force Attack**

<figure><img src="../../.gitbook/assets/image (1084).png" alt=""><figcaption></figcaption></figure>

Her kodu 5 kez göndermek yerine (alıcıya ulaşmasını sağlamak için böyle gönderilir) sadece bir kez gönderirseniz, süre 6 dakikaya düşer:

<figure><img src="../../.gitbook/assets/image (622).png" alt=""><figcaption></figcaption></figure>

Ve eğer sinyaller arasındaki 2 ms bekleme süresini **kaldırırsanız, süreyi 3 dakikaya** düşürebilirsiniz.

Ayrıca, De Bruijn Dizisi'ni kullanarak (tüm potansiyel ikili sayıları brute force ile göndermek için gereken bit sayısını azaltmanın bir yolu) bu **süre sadece 8 saniyeye** düşer:

<figure><img src="../../.gitbook/assets/image (583).png" alt=""><figcaption></figcaption></figure>

Bu saldırının örneği [https://github.com/samyk/opensesame](https://github.com/samyk/opensesame) adresinde uygulanmıştır.

**Bir önsöz gerektirmek, De Bruijn Dizisi** optimizasyonunu engelleyecek ve **dönüşümlü kodlar bu saldırıyı önleyecektir** (kodun brute force edilemeyecek kadar uzun olduğunu varsayarsak).

## Sub-GHz Attack

Bu sinyalleri Flipper Zero ile saldırmak için kontrol edin:

{% content-ref url="flipper-zero/fz-sub-ghz.md" %}
[fz-sub-ghz.md](flipper-zero/fz-sub-ghz.md)
{% endcontent-ref %}

## Rolling Codes Protection

Otomatik garaj kapısı açıcıları genellikle garaj kapısını açmak ve kapatmak için kablosuz bir uzaktan kumanda kullanır. Uzaktan kumanda, garaj kapısı açıcıya **bir radyo frekansı (RF) sinyali** gönderir ve bu, motoru kapıyı açmak veya kapatmak için etkinleştirir.

Birinin, RF sinyalini kesmek ve daha sonra kullanmak üzere kaydetmek için bir kod yakalayıcı cihazı kullanması mümkündür. Bu, **tekrar saldırısı** olarak bilinir. Bu tür bir saldırıyı önlemek için, birçok modern garaj kapısı açıcı daha güvenli bir şifreleme yöntemi olan **dönüşümlü kod** sistemini kullanır.

**RF sinyali genellikle dönüşümlü kod kullanılarak iletilir**, bu da kodun her kullanımda değiştiği anlamına gelir. Bu, birinin sinyali **yakalamayı** ve garaja **yetkisiz** erişim sağlamayı **zorlaştırır**.

Dönüşümlü kod sisteminde, uzaktan kumanda ve garaj kapısı açıcı, uzaktan kumanda her kullanıldığında **yeni bir kod üreten** **paylaşılan bir algoritmaya** sahiptir. Garaj kapısı açıcı yalnızca **doğru koda** yanıt verecek, bu da birinin yalnızca bir kodu yakalayarak garaja yetkisiz erişim sağlamasını çok daha zor hale getirecektir.

### **Missing Link Attack**

Temelde, düğmeyi dinlersiniz ve **uzaktan kumanda cihazın menzilinden çıktığında sinyali yakalarsınız** (örneğin araba veya garaj). Daha sonra cihaza geçer ve **yakalanan kodu kullanarak açarsınız**.

### Full Link Jamming Attack

Bir saldırgan, **sinyali aracın veya alıcının yakınında kesebilir** böylece **alıcı kodu gerçekten ‘duyamaz** ve bu olduğunda, sadece **kodu yakalayıp tekrar gönderebilirsiniz**.

Kurban bir noktada **anahtarları kullanarak arabayı kilitleyecektir**, ancak o zaman saldırgan **yeterince "kapıyı kapat" kodunu kaydetmiş olacaktır** ki umarım bu kodlar kapıyı açmak için yeniden gönderilebilir (bir **frekans değişikliği gerekebilir** çünkü bazı arabalar kapıyı açmak ve kapatmak için aynı kodları kullanır ama her iki komutu farklı frekanslarda dinler).

{% hint style="warning" %}
**Sinyal kesme çalışır**, ancak dikkat çekicidir çünkü **aracın kilitlenmesini test eden kişi** kapıların kilitli olduğunu doğrulamak için kapıları kontrol ederse, arabanın kilitli olmadığını fark eder. Ayrıca, böyle saldırılardan haberdar iseler, kapıların kilit **sesini** yapmadığını veya arabanın **ışıklarının** ‘kilit’ düğmesine bastıklarında hiç yanmadığını dinleyebilirler.
{% endhint %}

### **Code Grabbing Attack ( aka ‘RollJam’ )**

Bu daha **gizli bir Sinyal Kesme tekniğidir**. Saldırgan sinyali keser, böylece kurban kapıyı kilitlemeye çalıştığında çalışmaz, ancak saldırgan bu kodu **kaydeder**. Daha sonra, kurban düğmeye basarak arabayı **tekrar kilitlemeye çalışır** ve araba **bu ikinci kodu kaydeder**.\
Bundan hemen sonra **saldırgan ilk kodu gönderebilir** ve **araba kilitlenecektir** (kurban ikinci basışın kapattığını düşünecektir). Ardından, saldırgan **ç stolen kodu göndererek arabayı açabilir** (bir **"kapalı araba" kodunun da açmak için kullanılabileceğini varsayarsak**). Bir frekans değişikliği gerekebilir (çünkü bazı arabalar kapıyı açmak ve kapatmak için aynı kodları kullanır ama her iki komutu farklı frekanslarda dinler).

Saldırgan, **araba alıcısını kesebilir ve kendi alıcısını kesmeyebilir** çünkü eğer araba alıcısı örneğin 1MHz geniş bantta dinliyorsa, saldırgan uzaktan kumandanın kullandığı tam frekansı **kesmeyecek** ama **o spektrumda yakın bir frekansta** kesme yaparken **saldırganın alıcısı daha küçük bir aralıkta dinleyecektir** ve uzaktan kumanda sinyalini **kesme sinyali olmadan** dinleyebilir.

{% hint style="warning" %}
Diğer spesifikasyonlarda görülen uygulamalar, **dönüşümlü kodun gönderilen toplam kodun bir kısmı** olduğunu göstermektedir. Yani gönderilen kod bir **24 bit anahtardır**; ilk **12'si dönüşümlü kod**, **ikinci 8'i komut** (kilitleme veya açma gibi) ve son 4'ü **kontrol toplamıdır**. Bu tür bir uygulama yapan araçlar da doğal olarak savunmasızdır çünkü saldırgan yalnızca dönüşümlü kod segmentini değiştirerek **her iki frekansta da herhangi bir dönüşümlü kodu kullanabilir**.
{% endhint %}

{% hint style="danger" %}
Kurban, saldırgan ilk kodu gönderirken üçüncü bir kod gönderirse, birinci ve ikinci kod geçersiz hale gelecektir.
{% endhint %}

### Alarm Sounding Jamming Attack

Bir arabada kurulu bir aftermarket dönüşümlü kod sistemine karşı test yaparken, **aynı kodu iki kez göndermek** hemen **alarmı** ve immobilizeri etkinleştirdi ve benzersiz bir **hizmet reddi** fırsatı sağladı. Ironik olarak, **alarmı** ve immobilizeri **devre dışı bırakmanın** yolu **uzaktan kumandayı** **basmaktı**, bu da bir saldırgana **sürekli DoS saldırısı** yapma yeteneği sağladı. Ya da bu saldırıyı **önceki saldırıyla birleştirerek daha fazla kod elde edebilir** çünkü kurban saldırıyı bir an önce durdurmak isteyecektir.

## References

* [https://www.americanradioarchives.com/what-radio-frequency-does-car-key-fobs-run-on/](https://www.americanradioarchives.com/what-radio-frequency-does-car-key-fobs-run-on/)
* [https://www.andrewmohawk.com/2016/02/05/bypassing-rolling-code-systems/](https://www.andrewmohawk.com/2016/02/05/bypassing-rolling-code-systems/)
* [https://samy.pl/defcon2015/](https://samy.pl/defcon2015/)
* [https://hackaday.io/project/164566-how-to-hack-a-car/details](https://hackaday.io/project/164566-how-to-hack-a-car/details)

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
