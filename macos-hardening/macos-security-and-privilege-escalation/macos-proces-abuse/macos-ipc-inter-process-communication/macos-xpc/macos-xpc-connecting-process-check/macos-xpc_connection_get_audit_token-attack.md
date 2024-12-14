# macOS xpc\_connection\_get\_audit\_token Saldırısı

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

**Daha fazla bilgi için orijinal gönderiyi kontrol edin:** [**https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/**](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/). Bu bir özet:

## Mach Mesajları Temel Bilgiler

Mach Mesajlarının ne olduğunu bilmiyorsanız bu sayfayı kontrol edin:

{% content-ref url="../../" %}
[..](../../)
{% endcontent-ref %}

Şu an için hatırlamanız gereken ([buradan tanım](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing)):\
Mach mesajları, mach çekirdeğine entegre edilmiş **tek alıcı, çok gönderici iletişim** kanalı olan bir _mach portu_ üzerinden gönderilir. **Birden fazla işlem, bir mach portuna mesaj gönderebilir**, ancak herhangi bir anda **yalnızca bir işlem ondan okuyabilir**. Dosya tanımlayıcıları ve soketler gibi, mach portları çekirdek tarafından tahsis edilir ve yönetilir ve işlemler yalnızca hangi mach portlarını kullanmak istediklerini belirtmek için kullanabilecekleri bir tamsayı görürler.

## XPC Bağlantısı

XPC bağlantısının nasıl kurulduğunu bilmiyorsanız kontrol edin:

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## Açıklar Özeti

Bilmeniz gereken ilginç bir şey, **XPC'nin soyutlamasının bire bir bağlantı** olmasıdır, ancak bu, **birden fazla göndericiye sahip olabilen bir teknoloji üzerine kuruludur, bu nedenle:**

* Mach portları tek alıcı, **çok gönderici**.
* Bir XPC bağlantısının denetim belirteci, **en son alınan mesajdan kopyalanan** denetim belirtecidir.
* Bir XPC bağlantısının **denetim belirtecini** elde etmek, birçok **güvenlik kontrolü** için kritik öneme sahiptir.

Önceki durum umut verici görünse de, bunun sorun yaratmayacağı bazı senaryolar vardır ([buradan](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing)):

* Denetim belirteçleri genellikle bir bağlantıyı kabul edip etmeyeceğine karar vermek için bir yetkilendirme kontrolü için kullanılır. Bu, hizmet portuna bir mesaj gönderilerek gerçekleştiğinden, **henüz bir bağlantı kurulmamıştır**. Bu porttaki daha fazla mesaj, yalnızca ek bağlantı talepleri olarak işlenecektir. Bu nedenle, bir bağlantıyı kabul etmeden önceki **kontroller savunmasız değildir** (bu, `-listener:shouldAcceptNewConnection:` içinde denetim belirtecinin güvenli olduğu anlamına gelir). Bu nedenle, **belirli eylemleri doğrulayan XPC bağlantılarını arıyoruz**.
* XPC olay işleyicileri senkron olarak işlenir. Bu, bir mesaj için olay işleyicisinin, bir sonraki mesaj için çağrılmadan önce tamamlanması gerektiği anlamına gelir, hatta eşzamanlı dağıtım kuyruklarında bile. Bu nedenle, bir **XPC olay işleyicisinde denetim belirteci diğer normal (yanıt vermeyen!) mesajlar tarafından üzerine yazılamaz**.

İki farklı yöntem bu şekilde sömürülebilir:

1. Varyant 1:
* **Sömürü** **hizmet A'ya** ve **hizmet B'ye** **bağlanır**.
* Hizmet **B**, kullanıcının yapamayacağı hizmet A'da bir **ayrılmış işlevselliği** çağırabilir.
* Hizmet **A**, bir **`dispatch_async`** içinde bir bağlantının **olay işleyicisi** içinde _**değil**_ iken **`xpc_connection_get_audit_token`** çağrısını yapar.
* Bu nedenle, **farklı** bir mesaj **Denetim Belirteci'ni** **üzerine yazabilir** çünkü olay işleyicisinin dışında asenkron olarak dağıtılmaktadır.
* Sömürü, **hizmet B'ye hizmet A'ya gönderme hakkını** verir.
* Böylece hizmet **B**, aslında **hizmet A'ya** **mesajlar** gönderecektir.
* **Sömürü**, **ayrılmış eylemi** **çağırmaya** çalışır. Bir RC hizmet **A**, bu **eylemin** yetkilendirmesini **kontrol ederken**, **hizmet B denetim belirtecini** **üzerine yazar** (sömürüye, ayrıcalıklı eylemi çağırma erişimi verir).
2. Varyant 2:
* Hizmet **B**, kullanıcının yapamayacağı hizmet A'da bir **ayrılmış işlevselliği** çağırabilir.
* Sömürü, **hizmet A** ile bağlantı kurar ve **sömürüye** belirli bir **yanıt** **portunda** bir **mesaj** gönderir.
* Sömürü, **hizmet B'ye** o **yanıt portunu** geçerek bir mesaj gönderir.
* Hizmet **B yanıt verdiğinde**, **hizmet A'ya** mesaj gönderir, **bu sırada** **sömürü**, **hizmet A'ya** farklı bir **mesaj** göndererek **ayrılmış işlevselliğe** ulaşmaya çalışır ve hizmet B'den gelen yanıtın denetim belirtecini tam zamanında üzerine yazmasını bekler (Race Condition).

## Varyant 1: xpc\_connection\_get\_audit\_token'ı bir olay işleyicisi dışında çağırma <a href="#variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler" id="variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler"></a>

Senaryo:

* Bağlanabileceğimiz iki mach hizmeti **`A`** ve **`B`** (sandbox profiline ve bağlantıyı kabul etmeden önceki yetkilendirme kontrollerine dayanarak).
* _**A**_ belirli bir eylem için bir **yetkilendirme kontrolüne** sahip olmalıdır ki **`B`** bunu geçebilir (ancak uygulamamız geçemez).
* Örneğin, B bazı **yetkilere** sahipse veya **root** olarak çalışıyorsa, A'dan ayrıcalıklı bir eylemi gerçekleştirmesini istemesine izin verebilir.
* Bu yetkilendirme kontrolü için, **`A`**, örneğin `dispatch_async`'dan **`xpc_connection_get_audit_token`** çağrısı yaparak denetim belirtecini asenkron olarak alır.

{% hint style="danger" %}
Bu durumda bir saldırgan, **A'dan bir eylem gerçekleştirmesini** istemek için **sömürü** yaparak **Race Condition** tetikleyebilir ve **B'nin `A'ya` mesajlar göndermesini** sağlarken **A** tarafından **sömürünün** isteği **işlenmektedir**. RC **başarılı olduğunda**, **B'nin denetim belirteci** bellek içinde **kopyalanır** **ve** **sömürünün** isteği **A** tarafından **işlenmektedir**, bu da ona **yalnızca B'nin talep edebileceği ayrıcalıklı eyleme erişim** sağlar.
{% endhint %}

Bu, **`A`** olarak `smd` ve **`B`** olarak `diagnosticd` ile gerçekleşti. [`SMJobBless`](https://developer.apple.com/documentation/servicemanagement/1431078-smjobbless?language=objc) işlevi, yeni bir ayrıcalıklı yardımcı araç yüklemek için kullanılabilir (root olarak). Eğer **root olarak çalışan bir işlem** **smd** ile iletişime geçerse, başka kontroller yapılmayacaktır.

Bu nedenle, hizmet **B** **`diagnosticd`**'dir çünkü **root** olarak çalışır ve bir işlemi **izlemek** için kullanılabilir, bu nedenle izleme başladıktan sonra, **saniyede birden fazla mesaj gönderir.**

Saldırıyı gerçekleştirmek için:

1. Standart XPC protokolünü kullanarak `smd` adlı hizmete bir **bağlantı** başlatın.
2. `diagnosticd`'ye ikincil bir **bağlantı** oluşturun. Normal prosedürün aksine, iki yeni mach portu oluşturmak ve göndermek yerine, istemci portu gönderme hakkı, `smd` bağlantısıyla ilişkili **gönderme hakkının** bir kopyasıyla değiştirilir.
3. Sonuç olarak, XPC mesajları `diagnosticd`'ye dağıtılabilir, ancak `diagnosticd`'en gelen yanıtlar `smd`'ye yönlendirilir. `smd` için, hem kullanıcıdan hem de `diagnosticd`'den gelen mesajların aynı bağlantıdan geldiği görünmektedir.

![Saldırı sürecini gösteren bir resim](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/exploit.png)

4. Bir sonraki adım, `diagnosticd`'ye seçilen bir işlemi izlemeye başlaması talimatını vermektir (potansiyel olarak kullanıcının kendi işlemi). Aynı anda, `smd`'ye rutin 1004 mesajlarının bir seli gönderilir. Buradaki amaç, ayrıcalıklı yetkilere sahip bir aracı yüklemektir.
5. Bu eylem, `handle_bless` işlevinde bir yarış koşulunu tetikler. Zamanlama kritik öneme sahiptir: `xpc_connection_get_pid` işlev çağrısının, kullanıcının işleminin PID'sini döndürmesi gerekir (çünkü ayrıcalıklı araç kullanıcının uygulama paketinde bulunur). Ancak, `xpc_connection_get_audit_token` işlevi, özellikle `connection_is_authorized` alt yordamında, `diagnosticd`'ye ait denetim belirtecini referans almalıdır.

## Varyant 2: yanıt yönlendirme

XPC (Çapraz İşlem İletişimi) ortamında, olay işleyicileri eşzamanlı olarak çalışmasa da, yanıt mesajlarının işlenmesinin kendine özgü bir davranışı vardır. Özellikle, yanıt bekleyen mesajlar göndermenin iki farklı yöntemi vardır:

1. **`xpc_connection_send_message_with_reply`**: Burada, XPC mesajı belirlenen bir kuyrukta alınır ve işlenir.
2. **`xpc_connection_send_message_with_reply_sync`**: Tersine, bu yöntemde, XPC mesajı mevcut dağıtım kuyruğunda alınır ve işlenir.

Bu ayrım, **yanıt paketlerinin bir XPC olay işleyicisinin yürütülmesiyle eşzamanlı olarak ayrıştırılma olasılığını** sağladığı için kritik öneme sahiptir. Özellikle, `_xpc_connection_set_creds` denetim belirtecinin kısmi olarak üzerine yazılmasını önlemek için kilitleme uygulasa da, bu korumayı tüm bağlantı nesnesine genişletmez. Sonuç olarak, bu, bir paketin ayrıştırılması ile olay işleyicisinin yürütülmesi arasındaki aralıkta denetim belirtecinin değiştirilmesine olanak tanıyan bir zayıflık yaratır.

Bu zayıflığı sömürmek için aşağıdaki kurulum gereklidir:

* **`A`** ve **`B`** olarak adlandırılan iki mach hizmeti, her ikisi de bir bağlantı kurabilir.
* Hizmet **`A`**, yalnızca **`B`**'nin gerçekleştirebileceği belirli bir eylem için bir yetkilendirme kontrolü içermelidir (kullanıcının uygulaması bunu yapamaz).
* Hizmet **`A`**, bir yanıt bekleyen bir mesaj göndermelidir.
* Kullanıcı, **`B`**'ye yanıt vereceği bir mesaj gönderebilir.

Sömürü süreci aşağıdaki adımları içerir:

1. Hizmet **`A`**'nın yanıt bekleyen bir mesaj göndermesini bekleyin.
2. **`A`**'ya doğrudan yanıt vermek yerine, yanıt portu ele geçirilir ve hizmet **`B`**'ye bir mesaj göndermek için kullanılır.
3. Ardından, yasaklı eylemi içeren bir mesaj gönderilir ve bunun **`B`**'den gelen yanıtla eşzamanlı olarak işleneceği beklenir.

Aşağıda, tanımlanan saldırı senaryosunun görsel bir temsili bulunmaktadır:

!\[https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/variant2.png]\(../../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png)

<figure><img src="../../../../../../.gitbook/assets/image (33).png" alt="https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/variant2.png" width="563"><figcaption></figcaption></figure>

## Keşif Problemleri

* **Örneklerin Bulunmasındaki Zorluklar**: `xpc_connection_get_audit_token` kullanım örneklerini bulmak, hem statik hem de dinamik olarak zordu.
* **Metodoloji**: `xpc_connection_get_audit_token` işlevini yakalamak için Frida kullanıldı, olay işleyicilerinden gelmeyen çağrıları filtreledi. Ancak, bu yöntem yalnızca yakalanan işlem için sınırlıydı ve aktif kullanım gerektiriyordu.
* **Analiz Araçları**: Ulaşılabilir mach hizmetlerini incelemek için IDA/Ghidra gibi araçlar kullanıldı, ancak süreç zaman alıcıydı ve dyld paylaşılan önbelleği ile ilgili çağrılarla karmaşık hale geldi.
* **Betik Sınırlamaları**: `dispatch_async` bloklarından `xpc_connection_get_audit_token` çağrılarını analiz etmek için betik yazma girişimleri, blokların ayrıştırılması ve dyld paylaşılan önbelleği ile etkileşimdeki karmaşıklıklar nedeniyle engellendi.

## Çözüm <a href="#the-fix" id="the-fix"></a>

* **Rapor Edilen Sorunlar**: `smd` içinde bulunan genel ve özel sorunları detaylandıran bir rapor Apple'a gönderildi.
* **Apple'ın Yanıtı**: Apple, `smd` içindeki sorunu `xpc_connection_get_audit_token`'ı `xpc_dictionary_get_audit_token` ile değiştirerek ele aldı.
* **Düzeltmenin Doğası**: `xpc_dictionary_get_audit_token` işlevi, alınan XPC mesajına bağlı mach mesajından doğrudan denetim belirtecini alması nedeniyle güvenli kabul edilir. Ancak, `xpc_connection_get_audit_token` gibi kamu API'sinin bir parçası değildir.
* **Daha Kapsamlı Bir Düzeltmenin Yokluğu**: Apple'ın, bağlantının kaydedilen denetim belirteciyle uyumlu olmayan mesajları atma gibi daha kapsamlı bir düzeltme uygulamamasının nedeninin ne olduğu belirsizdir. Belirli senaryolarda (örneğin, `setuid` kullanımı) meşru denetim belirteci değişikliklerinin olabileceği bir faktör olabilir.
* **Mevcut Durum**: Sorun, iOS 17 ve macOS 14'te devam etmekte olup, bunu tanımlayıp anlamaya çalışanlar için bir zorluk teşkil etmektedir.

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter**'da **bizi takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
