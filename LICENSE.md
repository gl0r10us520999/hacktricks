{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **Bize katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya **bizi** **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}


<a rel="license" href="https://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://licensebuttons.net/l/by-nc/4.0/88x31.png" /></a><br>Copyright © Carlos Polop 2021.  Aksi belirtilmedikçe (kitaba kopyalanan dış bilgiler orijinal yazarlara aittir), Carlos Polop'un <a href="https://github.com/carlospolop/hacktricks">HACK TRICKS</a> üzerindeki metni <a href="https://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Atıf-Gayri Ticari 4.0 Uluslararası (CC BY-NC 4.0)</a> lisansı altındadır.

Lisans: Atıf-Gayri Ticari 4.0 Uluslararası (CC BY-NC 4.0)<br>
İnsan Okunabilir Lisans: https://creativecommons.org/licenses/by-nc/4.0/<br>
Tam Yasal Şartlar: https://creativecommons.org/licenses/by-nc/4.0/legalcode<br>
Biçimlendirme: https://github.com/jmatsushita/Creative-Commons-4.0-Markdown/blob/master/licenses/by-nc.markdown<br>

# creative commons

# Atıf-Gayri Ticari 4.0 Uluslararası

Creative Commons Corporation (“Creative Commons”) bir hukuk firması değildir ve hukuki hizmetler veya hukuki tavsiye vermez. Creative Commons kamu lisanslarının dağıtımı, avukat-müvekkil veya başka bir ilişki oluşturmaz. Creative Commons, lisanslarını ve ilgili bilgileri "olduğu gibi" sunar. Creative Commons, lisansları, bu şartlar ve koşullar altında lisanslanan herhangi bir materyal veya ilgili bilgiler hakkında hiçbir garanti vermez. Creative Commons, bunların kullanımından kaynaklanan zararlara karşı tüm sorumluluğu mümkün olan en geniş ölçüde reddeder.

## Creative Commons Kamu Lisanslarının Kullanımı

Creative Commons kamu lisansları, yaratıcıların ve diğer hak sahiplerinin telif hakkına ve aşağıdaki kamu lisansında belirtilen belirli diğer haklara tabi orijinal eserleri ve diğer materyalleri paylaşmak için kullanabileceği standart bir terim ve koşul seti sağlar. Aşağıdaki hususlar yalnızca bilgilendirme amaçlıdır, kapsamlı değildir ve lisanslarımızın bir parçasını oluşturmaz.

* __Lisans verenler için hususlar:__ Kamu lisanslarımız, kamuya telif hakkı ve belirli diğer haklar tarafından kısıtlanmış olan materyali kullanma izni verme yetkisine sahip olanlar tarafından kullanılmak üzere tasarlanmıştır. Lisanslarımız geri alınamaz. Lisans verenler, seçtikleri lisansın şartlarını ve koşullarını okumalı ve anlamalıdır. Lisans verenler, kamuya materyali beklenildiği gibi yeniden kullanabilmesi için gerekli tüm hakları güvence altına almalıdır. Lisans verenler, lisansa tabi olmayan herhangi bir materyali açıkça işaret etmelidir. Bu, diğer CC lisanslı materyalleri veya telif hakkına istisna veya sınırlama altında kullanılan materyali içerir. [Lisans verenler için daha fazla husus](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensors).

* __Kamu için hususlar:__ Kamu lisanslarımızdan birini kullanarak, bir lisans veren, kamuya lisanslı materyali belirtilen şartlar ve koşullar altında kullanma izni verir. Lisans verenin izni herhangi bir nedenle gerekli değilse - örneğin, telif hakkına ilişkin herhangi bir geçerli istisna veya sınırlama nedeniyle - o kullanım lisansla düzenlenmez. Lisanslarımız yalnızca, bir lisans verenin yetkisi dahilinde verdiği telif hakkı ve belirli diğer haklar altında izinler verir. Lisanslı materyalin kullanımı, diğer nedenlerden dolayı hala kısıtlanabilir; bu, başkalarının materyal üzerindeki telif hakkı veya diğer hakları nedeniyle olabilir. Bir lisans veren, tüm değişikliklerin işaretlenmesini veya tanımlanmasını talep edebilir. Lisanslarımız tarafından zorunlu olmasa da, makul olduğunda bu taleplere saygı göstermeniz teşvik edilir. [Kamu için daha fazla husus](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensees).

# Creative Commons Atıf-Gayri Ticari 4.0 Uluslararası Kamu Lisansı

Lisanslı Hakları (aşağıda tanımlanmıştır) kullanarak, bu Creative Commons Atıf-Gayri Ticari 4.0 Uluslararası Kamu Lisansı ("Kamu Lisansı") şartlarına ve koşullarına bağlı kalmayı kabul edersiniz. Bu Kamu Lisansı bir sözleşme olarak yorumlanabilirse, bu şartlar ve koşulları kabul etmeniz karşılığında Lisanslı Haklar size verilir ve Lisans veren, bu şartlar ve koşullar altında Lisanslı Materyali sunmaktan elde ettiği faydalar karşılığında bu hakları size verir.

## Bölüm 1 – Tanımlar.

a. __Uyarlanmış Materyal__ , Telif Hakkı ve Benzer Haklara tabi olan ve Lisanslı Materyalden türetilen veya ona dayanan, Lisanslı Materyalin çevrildiği, değiştirildiği, düzenlendiği, dönüştürüldüğü veya Lisans verenin sahip olduğu Telif Hakkı ve Benzer Haklar altında izin gerektiren bir şekilde başka şekilde değiştirildiği materyaldir. Bu Kamu Lisansı açısından, Lisanslı Materyal bir müzik eseri, performans veya ses kaydı olduğunda, Uyarlanmış Materyal her zaman Lisanslı Materyalin hareketli bir görüntü ile zamanlı ilişki içinde senkronize edildiği durumlarda üretilir.

b. __Adaptör Lisansı__ , Uyarlanmış Materyale katkılarınızda Telif Hakkı ve Benzer Haklarınızı bu Kamu Lisansının şartlarına ve koşullarına uygun olarak uyguladığınız lisans anlamına gelir.

c. __Telif Hakkı ve Benzer Haklar__ , telif hakkı ve/veya telif hakkına yakın olan benzer haklar anlamına gelir; bunlar arasında, sınırlama olmaksızın, performans, yayın, ses kaydı ve Sui Generis Veri Tabanı Hakları bulunur; hakların nasıl etiketlendiği veya kategorize edildiği dikkate alınmaksızın. Bu Kamu Lisansı açısından, Bölüm 2(b)(1)-(2)'de belirtilen haklar Telif Hakkı ve Benzer Haklar değildir.

d. __Etkin Teknolojik Önlemler__ , uygun yetki olmaksızın, 20 Aralık 1996'da kabul edilen WIPO Telif Hakkı Antlaşması'nın 11. Maddesi uyarınca yükümlülükleri yerine getiren yasalar altında aşılmaması gereken önlemleri ifade eder ve/veya benzer uluslararası anlaşmaları.

e. __İstisnalar ve Sınırlamalar__ , adil kullanım, adil işlem ve/veya Lisanslı Materyalin kullanımınıza uygulanan Telif Hakkı ve Benzer Haklar için herhangi bir diğer istisna veya sınırlama anlamına gelir.

f. __Lisanslı Materyal__ , Lisans verenin bu Kamu Lisansını uyguladığı sanatsal veya edebi eser, veri tabanı veya diğer materyaldir.

g. __Lisanslı Haklar__ , bu Kamu Lisansının şartlarına ve koşullarına tabi olarak size verilen haklardır; bu haklar, Lisanslı Materyali kullanmanıza uygulanan tüm Telif Hakkı ve Benzer Haklarla sınırlıdır ve Lisans verenin lisans verme yetkisine sahiptir.

h. __Lisans Veren__ , bu Kamu Lisansı altında hakları veren birey(ler) veya varlık(lar) anlamına gelir.

i. __Gayri Ticari__ , esasen ticari avantaj veya maddi tazminat amacıyla tasarlanmamış veya yönlendirilmemiştir. Bu Kamu Lisansı açısından, Lisanslı Materyalin, Telif Hakkı ve Benzer Haklar altında başka bir materyal ile dijital dosya paylaşımı veya benzeri yollarla değişimi Gayri Ticari'dir; bu değişimle bağlantılı olarak maddi tazminat ödemesi yoksa.

j. __Paylaşmak__ , Lisanslı Haklar altında izin gerektiren herhangi bir yöntem veya süreçle materyali kamuya sağlamak, örneğin, çoğaltma, kamuya gösterim, kamu performansı, dağıtım, yayma, iletişim veya ithalat ve materyali kamuya sunmak, kamu üyelerinin materyale kendi seçtikleri bir yerden ve zamanda erişebileceği yollar dahil.

k. __Sui Generis Veri Tabanı Hakları__ , 11 Mart 1996 tarihli Avrupa Parlamentosu ve Konseyi'nin 96/9/EC sayılı Direktifi'nden kaynaklanan telif hakkı dışındaki haklar anlamına gelir; bu direktif, değişiklikler ve/veya halefiyetler ile birlikte, dünyanın herhangi bir yerinde esasen eşdeğer diğer hakları içerir.

l. __Siz__ , bu Kamu Lisansı altında Lisanslı Hakları kullanan birey veya varlık anlamına gelir. Sizin, karşılık gelen bir anlamı vardır.

## Bölüm 2 – Kapsam.

a. ___Lisans verme.___

1. Bu Kamu Lisansının şartlarına ve koşullarına tabi olarak, Lisans veren, size Lisanslı Materyalde Lisanslı Hakları kullanma hakkını dünya çapında, telif ücreti ödemeksizin, alt lisans verilemez, münhasır olmayan, geri alınamaz bir lisans verir:

A. Lisanslı Materyali, tamamen veya kısmen, yalnızca Gayri Ticari amaçlar için çoğaltmak ve Paylaşmak; ve

B. Uyarlanmış Materyali yalnızca Gayri Ticari amaçlar için üretmek, çoğaltmak ve Paylaşmak.

2. __İstisnalar ve Sınırlamalar.__ Şüpheyi ortadan kaldırmak için, İstisnalar ve Sınırlamalar sizin kullanımınıza uygulandığında, bu Kamu Lisansı geçerli değildir ve şartlarına ve koşullarına uymanız gerekmez.

3. __Süre.__ Bu Kamu Lisansının süresi Bölüm 6(a)'da belirtilmiştir.

4. __Medya ve formatlar; teknik değişikliklere izin verilir.__ Lisans veren, Lisanslı Hakları, şu anda bilinen veya daha sonra oluşturulacak tüm medya ve formatlarda kullanmanıza izin verir ve bunu yapmak için gerekli teknik değişiklikleri yapmanıza izin verir. Lisans veren, Lisanslı Hakları kullanmak için gerekli teknik değişiklikleri yapmanızı engelleme hakkından feragat eder ve/veya bu konuda herhangi bir hak veya yetkiyi ileri sürmeyeceğini kabul eder; bu, Etkin Teknolojik Önlemleri aşmak için gerekli teknik değişiklikleri de içerir. Bu Kamu Lisansı açısından, yalnızca bu Bölüm 2(a)(4) tarafından yetkilendirilmiş değişiklikler asla Uyarlanmış Materyal üretmez.

5. __Aşağı akış alıcıları.__

A. __Lisans verenin teklifi – Lisanslı Materyal.__ Lisanslı Materyalin her alıcısı, otomatik olarak, bu Kamu Lisansının şartlarına ve koşullarına tabi olarak Lisanslı Hakları kullanma teklifi alır.

B. __Aşağı akış kısıtlamaları yoktur.__ Lisanslı Materyale, Lisanslı Hakları kullanan herhangi bir alıcının kullanımını kısıtlayacak şekilde ek veya farklı şartlar veya koşullar sunamaz veya uygulayamazsınız.

6. __Onay yoktur.__ Bu Kamu Lisansında hiçbir şey, sizin veya Lisanslı Materyali kullanımınızın, Lisans veren veya atıf almak üzere belirlenen diğerleri tarafından onaylandığı veya desteklendiği veya resmi bir statü verildiği anlamına gelmez veya böyle yorumlanamaz.

b. ___Diğer haklar.___

1. Ahlaki haklar, bütünlük hakkı gibi, bu Kamu Lisansı altında lisanslanmamıştır; ayrıca tanıtım, gizlilik ve/veya diğer benzer kişilik hakları da yoktur; ancak, mümkün olduğunca, Lisans veren, Lisanslı Hakları kullanabilmeniz için gerekli sınırlı ölçüde, Lisans verenin sahip olduğu bu tür hakları ileri sürmemeyi kabul eder.

2. Patent ve ticari marka hakları bu Kamu Lisansı altında lisanslanmamıştır.

3. Mümkün olduğunca, Lisans veren, Lisanslı Hakları kullanmanız için sizden telif ücreti talep etme hakkından feragat eder; bu, doğrudan veya herhangi bir gönüllü veya feragat edilebilir yasal veya zorunlu lisanslama düzeni aracılığıyla olabilir. Diğer tüm durumlarda, Lisans veren, bu tür telif ücretlerini toplama hakkını açıkça saklı tutar; bu, Lisanslı Materyalin Gayri Ticari amaçlar dışında kullanılması durumunda da geçerlidir.

## Bölüm 3 – Lisans Şartları.

Lisanslı Hakları kullanmanız, aşağıdaki şartlara tabi olarak açıkça yapılır.

a. ___Atıf.___

1. Lisanslı Materyali (değiştirilmiş formda dahil) Paylaşırsanız, şunları yapmalısınız:

A. Lisans veren tarafından Lisanslı Materyal ile sağlanmışsa, aşağıdakileri korumalısınız:

i. Lisanslı Materyalin yaratıcısının ve atıf alması gereken diğerlerinin kimliğini, Lisans verenin talep ettiği makul bir şekilde belirtmek (belirtilmişse takma adla dahil);

ii. bir telif hakkı bildirimi;

iii. bu Kamu Lisansına atıfta bulunan bir bildirim;

iv. garantilerin reddine atıfta bulunan bir bildirim;

v. makul ölçüde uygulanabilir olduğu ölçüde, Lisanslı Materyale bir URI veya köprü.

B. Lisanslı Materyali değiştirdiyseniz belirtin ve önceki değişikliklerin bir göstergesini koruyun; ve

C. Lisanslı Materyalin bu Kamu Lisansı altında lisanslandığını belirtin ve bu Kamu Lisansının metnini veya URI'sini veya köprüsünü ekleyin.

2. Bölüm 3(a)(1) şartlarını, Lisanslı Materyali Paylaştığınız ortam, araçlar ve bağlama dayalı olarak makul bir şekilde yerine getirebilirsiniz. Örneğin, gerekli bilgileri içeren bir kaynağa bir URI veya köprü sağlayarak şartları yerine getirmek makul olabilir.

3. Lisans veren tarafından talep edilirse, Bölüm 3(a)(1)(A) tarafından gerekli olan bilgilerin makul ölçüde uygulanabilir olduğu ölçüde kaldırılmasını sağlamalısınız.

4. Ürettiğiniz Uyarlanmış Materyali Paylaşırsanız, uyguladığınız Adaptör Lisansı, Uyarlanmış Materyalin alıcılarının bu Kamu Lisansına uymasını engellememelidir.

## Bölüm 4 – Sui Generis Veri Tabanı Hakları.

Lisanslı Haklar, Lisanslı Materyali kullanımınıza uygulanan Sui Generis Veri Tabanı Haklarını içeriyorsa:

a. Şüpheyi ortadan kaldırmak için, Bölüm 2(a)(1) size, veri tabanının tümünü veya önemli bir kısmını Gayri Ticari amaçlar için çıkarma, yeniden kullanma, çoğaltma ve Paylaşma hakkı verir;

b. Eğer Sui Generis Veri Tabanı Haklarına sahip olduğunuz bir veri tabanında veri tabanının tümünü veya önemli bir kısmını dahil ederseniz, o veri tabanı (ancak bireysel içerikleri değil) Uyarlanmış Materyaldir; ve

c. Veri tabanının tümünü veya önemli bir kısmını Paylaşırsanız, Bölüm 3(a) şartlarına uymalısınız.

Şüpheyi ortadan kaldırmak için, bu Bölüm 4, Lisanslı Hakların diğer Telif Hakkı ve Benzer Haklar altında yükümlülüklerinizi değiştirmez.

## Bölüm 5 – Garantilerin Reddedilmesi ve Sorumluluğun Sınırlandırılması.

a. __Aksi takdirde Lisans veren tarafından ayrı olarak üstlenilmedikçe, mümkün olduğunca, Lisans veren, Lisanslı Materyali olduğu gibi ve mevcut olduğu gibi sunar ve Lisanslı Materyal hakkında hiçbir türde, açık, örtük, yasal veya diğer herhangi bir temsil veya garanti vermez. Bu, başlık, ticari uygunluk, belirli bir amaca uygunluk, ihlal etmeme, gizli veya diğer kusurların yokluğu, doğruluk veya hata varlığı veya yokluğu ile ilgili garantileri içerir; bunlar biliniyor veya keşfedilebilir olup olmadığına bakılmaksızın. Garantilerin reddedilmesine izin verilmediği durumlarda, bu red sizin için geçerli olmayabilir.__

b. __Mümkün olduğunca, Lisans veren, bu Kamu Lisansı veya Lisanslı Materyalin kullanımı nedeniyle ortaya çıkan herhangi bir doğrudan, özel, dolaylı, tesadüfi, sonuçsal, cezai, örnek niteliğinde veya diğer kayıplar, maliyetler, harcamalar veya zararlar için herhangi bir yasal teori (ihmal dahil, sınırlama olmaksızın) veya başka bir şekilde size karşı sorumlu olmayacaktır; bu, Lisans verenin bu tür kayıpların, maliyetlerin, harcamaların veya zararların olasılığı hakkında bilgilendirilmiş olması durumunda bile geçerlidir. Sorumluluğun sınırlandırılmasına izin verilmediği durumlarda, bu sınırlama sizin için geçerli olmayabilir.__

c. Yukarıda sağlanan garantilerin reddi ve sorumluluğun sınırlandırılması, mümkün olduğunca, tüm sorumluluğun mutlak bir red ve feragat olarak en yakın şekilde yorumlanacaktır.

## Bölüm 6 – Süre ve Fesih.

a. Bu Kamu Lisansı, burada lisanslanan Telif Hakkı ve Benzer Hakların süresi için geçerlidir. Ancak, bu Kamu Lisansına uymadığınız takdirde, bu Kamu Lisansı altındaki haklarınız otomatik olarak sona erer.

b. Eğer Bölüm 6(a) uyarınca Lisanslı Materyali kullanma hakkınız sona erdiyse, bu hak, aşağıdaki durumlarda yeniden kazanılır:

1. ihlalin düzeltildiği tarihten itibaren otomatik olarak, eğer ihlal, ihlali keşfettiğiniz tarihten itibaren 30 gün içinde düzeltilirse; veya

2. Lisans veren tarafından açıkça yeniden kazanılması durumunda.

Şüpheyi ortadan kaldırmak için, bu Bölüm 6(b), Lisans verenin bu Kamu Lisansına uymadığınız durumlarda herhangi bir çözüm arama hakkını etkilemez.

c. Şüpheyi ortadan kaldırmak için, Lisans veren ayrıca Lisanslı Materyali ayrı şartlar veya koşullar altında sunabilir veya Lisanslı Materyali herhangi bir zamanda dağıtmayı durdurabilir; ancak, bunu yapmak bu Kamu Lisansını sona erdirmeyecektir.

d. Bölüm 1, 5, 6, 7 ve 8, bu Kamu Lisansının sona ermesinden sonra geçerliliğini korur.

## Bölüm 7 – Diğer Şartlar ve Koşullar.

a. Lisans veren, sizin tarafınızdan iletilen herhangi bir ek veya farklı şart veya koşula bağlı olmayacaktır; aksi açıkça kabul edilmedikçe.

b. Burada belirtilmeyen Lisanslı Materyal ile ilgili herhangi bir düzenleme, anlayış veya anlaşma, bu Kamu Lisansının şart ve koşullarından ayrı ve bağımsızdır.

## Bölüm 8 – Yorumlama.

a. Şüpheyi ortadan kaldırmak için, bu Kamu Lisansı, Lisanslı Materyalin, bu Kamu Lisansı altında izin olmaksızın yasal olarak yapılabilecek herhangi bir kullanımını azaltmaz, sınırlamaz, kısıtlamaz veya koşul getirmez.

b. Mümkün olduğunca, bu Kamu Lisansının herhangi bir hükmü uygulanamaz olarak kabul edilirse, uygulanabilir hale getirmek için gerekli en az ölçüde otomatik olarak yeniden düzenlenecektir. Eğer hüküm yeniden düzenlenemezse, bu Kamu Lisansından ayrılacak ve kalan şart ve koşulların uygulanabilirliğini etkilemeyecektir.

c. Bu Kamu Lisansının hiçbir şart veya koşulu feragat edilmeyecek ve uyumsuzluk durumunda hiçbir onay verilmeyecektir; bu, Lisans veren tarafından açıkça kabul edilmedikçe.

d. Bu Kamu Lisansı altında hiçbir şey, Lisans veren veya sizin için geçerli olan herhangi bir ayrıcalık ve dokunulmazlık üzerinde bir sınırlama veya feragat olarak yorumlanamaz; bu, herhangi bir yargı veya otoritenin yasal süreçlerinden muafiyet dahil.
```
Creative Commons is not a party to its public licenses. Notwithstanding, Creative Commons may elect to apply one of its public licenses to material it publishes and in those instances will be considered the “Licensor.” Except for the limited purpose of indicating that material is shared under a Creative Commons public license or as otherwise permitted by the Creative Commons policies published at [creativecommons.org/policies](http://creativecommons.org/policies), Creative Commons does not authorize the use of the trademark “Creative Commons” or any other trademark or logo of Creative Commons without its prior written consent including, without limitation, in connection with any unauthorized modifications to any of its public licenses or any other arrangements, understandings, or agreements concerning use of licensed material. For the avoidance of doubt, this paragraph does not form part of the public licenses.

Creative Commons may be contacted at [creativecommons.org](http://creativecommons.org/).
```
{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
