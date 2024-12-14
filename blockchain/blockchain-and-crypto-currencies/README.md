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


## Temel Kavramlar

- **Akıllı Sözleşmeler**, belirli koşullar yerine getirildiğinde bir blockchain üzerinde çalışan programlar olarak tanımlanır ve aracılara ihtiyaç duymadan anlaşma yürütmelerini otomatikleştirir.
- **Merkeziyetsiz Uygulamalar (dApps)**, kullanıcı dostu bir ön yüz ve şeffaf, denetlenebilir bir arka uç ile akıllı sözleşmeler üzerine inşa edilir.
- **Jetonlar ve Paralar**, paraların dijital para olarak hizmet etmesi, jetonların ise belirli bağlamlarda değer veya mülkiyeti temsil etmesi ile ayrılır.
- **Yardımcı Jetonlar**, hizmetlere erişim sağlar ve **Güvenlik Jetonları** varlık mülkiyetini belirtir.
- **DeFi**, merkezi otoriteler olmadan finansal hizmetler sunan Merkeziyetsiz Finans anlamına gelir.
- **DEX** ve **DAO'lar**, sırasıyla Merkeziyetsiz Borsa Platformları ve Merkeziyetsiz Otonom Organizasyonlar anlamına gelir.

## Konsensüs Mekanizmaları

Konsensüs mekanizmaları, blockchain üzerinde güvenli ve kabul edilen işlem doğrulamalarını sağlar:
- **İş Kanıtı (PoW)**, işlem doğrulama için hesaplama gücüne dayanır.
- **Hisse Kanıtı (PoS)**, doğrulayıcıların belirli bir miktar jeton bulundurmasını gerektirir ve PoW'ye kıyasla enerji tüketimini azaltır.

## Bitcoin Temelleri

### İşlemler

Bitcoin işlemleri, adresler arasında fon transferini içerir. İşlemler, yalnızca özel anahtarın sahibi tarafından transferlerin başlatılmasını sağlamak için dijital imzalarla doğrulanır.

#### Ana Bileşenler:

- **Çok İmzalı İşlemler**, bir işlemi yetkilendirmek için birden fazla imza gerektirir.
- İşlemler, **girdiler** (fon kaynağı), **çıktılar** (hedef), **ücretler** (madencilere ödenen) ve **senaryolar** (işlem kuralları) içerir.

### Lightning Ağı

Bitcoin'in ölçeklenebilirliğini artırmayı hedefler, bir kanalda birden fazla işlemi gerçekleştirerek yalnızca nihai durumu blockchain'e yayınlar.

## Bitcoin Gizlilik Endişeleri

Gizlilik saldırıları, **Ortak Girdi Mülkiyeti** ve **UTXO Değişim Adresi Tespiti** gibi işlem kalıplarını istismar eder. **Karıştırıcılar** ve **CoinJoin** gibi stratejiler, kullanıcılar arasındaki işlem bağlantılarını gizleyerek anonimliği artırır.

## Bitcoin'leri Anonim Olarak Edinme

Yöntemler arasında nakit ticareti, madencilik ve karıştırıcılar kullanmak yer alır. **CoinJoin**, birden fazla işlemi karıştırarak izlenebilirliği karmaşıklaştırırken, **PayJoin**, CoinJoin'leri normal işlemler olarak gizleyerek gizliliği artırır.


# Bitcoin Gizlilik Saldırıları

# Bitcoin Gizlilik Saldırıları Özeti

Bitcoin dünyasında, işlemlerin gizliliği ve kullanıcıların anonimliği genellikle endişe konusudur. İşte saldırganların Bitcoin gizliliğini tehlikeye atabileceği birkaç yaygın yöntemin basitleştirilmiş bir özeti.

## **Ortak Girdi Mülkiyeti Varsayımı**

Farklı kullanıcıların girdilerinin tek bir işlemde birleştirilmesi genellikle nadirdir, çünkü bu karmaşıklık içerir. Bu nedenle, **aynı işlemdeki iki girdi adresinin genellikle aynı sahibine ait olduğu varsayılır**.

## **UTXO Değişim Adresi Tespiti**

Bir UTXO, veya **Harcanmamış İşlem Çıktısı**, bir işlemde tamamen harcanmalıdır. Eğer yalnızca bir kısmı başka bir adrese gönderilirse, geri kalan yeni bir değişim adresine gider. Gözlemciler, bu yeni adresin gönderene ait olduğunu varsayarak gizliliği tehlikeye atabilir.

### Örnek
Bunu hafifletmek için, karıştırma hizmetleri veya birden fazla adres kullanmak mülkiyeti gizlemeye yardımcı olabilir.

## **Sosyal Ağlar ve Forumlar Üzerinden Maruz Kalma**

Kullanıcılar bazen Bitcoin adreslerini çevrimiçi paylaşır, bu da **adresin sahibine bağlanmasını kolaylaştırır**.

## **İşlem Grafiği Analizi**

İşlemler, fon akışına dayalı olarak kullanıcılar arasındaki potansiyel bağlantıları ortaya çıkaran grafikler olarak görselleştirilebilir.

## **Gereksiz Girdi Heuristiği (Optimal Değişim Heuristiği)**

Bu heuristik, birden fazla girdi ve çıktı içeren işlemleri analiz ederek hangi çıktının gönderene geri dönen değişim olduğunu tahmin etmeye dayanır.

### Örnek
```bash
2 btc --> 4 btc
3 btc     1 btc
```
If adding more inputs makes the change output larger than any single input, it can confuse the heuristic.

## **Zorunlu Adres Yeniden Kullanımı**

Saldırganlar, alıcının gelecekteki işlemlerde bunları diğer girdilerle birleştirmesini umarak daha önce kullanılan adreslere küçük miktarlar gönderebilir, böylece adresleri birbirine bağlayabilirler.

### Doğru Cüzdan Davranışı
Cüzdanlar, bu gizlilik sızıntısını önlemek için daha önce kullanılmış, boş adreslerde alınan coinleri kullanmaktan kaçınmalıdır.

## **Diğer Blockchain Analiz Teknikleri**

- **Tam Ödeme Miktarları:** Değişiklik olmadan yapılan işlemler, muhtemelen aynı kullanıcıya ait iki adres arasında gerçekleşir.
- **Yuvarlak Sayılar:** Bir işlemdeki yuvarlak bir sayı, bunun bir ödeme olduğunu gösterir; yuvarlak olmayan çıktı muhtemelen değişikliktir.
- **Cüzdan Parmak İzi:** Farklı cüzdanlar, analistlerin kullanılan yazılımı ve potansiyel olarak değişim adresini tanımlamasına olanak tanıyan benzersiz işlem oluşturma desenlerine sahiptir.
- **Miktar ve Zaman Korelasyonları:** İşlem zamanlarını veya miktarlarını açıklamak, işlemlerin izlenebilir hale gelmesine neden olabilir.

## **Trafik Analizi**

Ağ trafiğini izleyerek, saldırganlar işlemleri veya blokları IP adreslerine bağlayabilir, bu da kullanıcı gizliliğini tehlikeye atabilir. Bu, bir varlığın birçok Bitcoin düğümü işletmesi durumunda özellikle doğrudur ve işlemleri izleme yeteneklerini artırır.

## Daha Fazla
Gizlilik saldırıları ve savunmaları hakkında kapsamlı bir liste için [Bitcoin Gizliliği](https://en.bitcoin.it/wiki/Privacy) sayfasını ziyaret edin.

# Anonim Bitcoin İşlemleri

## Bitcoinleri Anonim Olarak Elde Etme Yolları

- **Nakit İşlemler**: Nakit ile bitcoin edinme.
- **Nakit Alternatifleri**: Hediye kartları satın alıp bunları çevrimiçi olarak bitcoin ile değiştirme.
- **Madencilik**: Bitcoin kazanmanın en özel yöntemi madenciliktir, özellikle yalnız yapıldığında çünkü madencilik havuzları madencinin IP adresini bilebilir. [Madencilik Havuzları Bilgisi](https://en.bitcoin.it/wiki/Pooled_mining)
- **Hırsızlık**: Teorik olarak, bitcoin çalmak anonim olarak edinmenin bir başka yöntemi olabilir, ancak bu yasadışıdır ve önerilmez.

## Karıştırma Hizmetleri

Bir karıştırma hizmeti kullanarak, bir kullanıcı **bitcoin gönderebilir** ve **karşılığında farklı bitcoinler alabilir**, bu da orijinal sahibin izlenmesini zorlaştırır. Ancak, bu hizmetin kayıt tutmaması ve gerçekten bitcoinleri geri döndürmesi için güven gerektirir. Alternatif karıştırma seçenekleri arasında Bitcoin kumarhaneleri bulunmaktadır.

## CoinJoin

**CoinJoin**, farklı kullanıcılardan gelen birden fazla işlemi birleştirerek, girdileri çıktılarla eşleştirmeye çalışan herkes için süreci karmaşık hale getirir. Etkinliğine rağmen, benzersiz girdi ve çıktı boyutlarına sahip işlemler hala izlenebilir.

CoinJoin kullanmış olabilecek örnek işlemler arasında `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` ve `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238` bulunmaktadır.

Daha fazla bilgi için [CoinJoin](https://coinjoin.io/en) sayfasını ziyaret edin. Ethereum'da benzer bir hizmet için [Tornado Cash](https://tornado.cash) sayfasını kontrol edin; bu, madencilerden gelen fonlarla işlemleri anonimleştirir.

## PayJoin

CoinJoin'un bir varyantı olan **PayJoin** (veya P2EP), iki taraf (örneğin, bir müşteri ve bir satıcı) arasında işlemi, CoinJoin'un belirgin eşit çıktılar özelliği olmadan, normal bir işlem olarak gizler. Bu, tespit edilmesini son derece zorlaştırır ve işlem gözetim varlıkları tarafından kullanılan ortak-girdi-sahipliği heuristiğini geçersiz kılabilir.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transactions like the above could be PayJoin, enhancing privacy while remaining indistinguishable from standard bitcoin transactions.

**PayJoin kullanımı, geleneksel gözetim yöntemlerini önemli ölçüde bozabilir**, bu da işlem gizliliği arayışında umut verici bir gelişme haline getirir.


# Kripto Para Birimlerinde Gizlilik için En İyi Uygulamalar

## **Cüzdan Senkronizasyon Teknikleri**

Gizliliği ve güvenliği korumak için, cüzdanların blockchain ile senkronize edilmesi çok önemlidir. İki yöntem öne çıkmaktadır:

- **Tam düğüm**: Tüm blockchain'i indirerek, tam düğüm maksimum gizlilik sağlar. Yapılan tüm işlemler yerel olarak saklanır, bu da düşmanların kullanıcının hangi işlemlerle veya adreslerle ilgilendiğini belirlemesini imkansız hale getirir.
- **İstemci tarafı blok filtreleme**: Bu yöntem, blockchain'deki her blok için filtreler oluşturarak cüzdanların belirli ilgi alanlarını ağ gözlemcilerine ifşa etmeden ilgili işlemleri tanımlamasını sağlar. Hafif cüzdanlar bu filtreleri indirir, yalnızca kullanıcının adresleriyle eşleşme bulunduğunda tam blokları alır.

## **Anonimlik için Tor Kullanımı**

Bitcoin'in eşler arası bir ağda çalıştığı göz önüne alındığında, IP adresinizi gizlemek için Tor kullanılması önerilir, bu da ağla etkileşimde gizliliği artırır.

## **Adres Yeniden Kullanımını Önleme**

Gizliliği korumak için her işlem için yeni bir adres kullanmak hayati öneme sahiptir. Adreslerin yeniden kullanılması, işlemleri aynı varlıkla ilişkilendirerek gizliliği tehlikeye atabilir. Modern cüzdanlar, tasarımları aracılığıyla adres yeniden kullanımını teşvik etmez.

## **İşlem Gizliliği Stratejileri**

- **Birden fazla işlem**: Bir ödemeyi birkaç işleme bölmek, işlem miktarını belirsiz hale getirerek gizlilik saldırılarını engelleyebilir.
- **Değişimden kaçınma**: Değişim çıktısı gerektirmeyen işlemleri tercih etmek, değişim tespit yöntemlerini bozarak gizliliği artırır.
- **Birden fazla değişim çıktısı**: Değişimden kaçınmak mümkün değilse, birden fazla değişim çıktısı oluşturmak yine de gizliliği artırabilir.

# **Monero: Anonimlik Işığı**

Monero, dijital işlemlerde mutlak anonimlik ihtiyacını karşılar ve gizlilik için yüksek bir standart belirler.

# **Ethereum: Gaz ve İşlemler**

## **Gazı Anlamak**

Gaz, Ethereum'da işlemleri gerçekleştirmek için gereken hesaplama çabasını ölçer ve **gwei** cinsinden fiyatlandırılır. Örneğin, 2,310,000 gwei (veya 0.00231 ETH) maliyetli bir işlem, bir gaz limiti ve bir temel ücret içerir, ayrıca madencileri teşvik etmek için bir bahşiş vardır. Kullanıcılar, fazla ödeme yapmamalarını sağlamak için maksimum bir ücret belirleyebilir ve fazlası iade edilir.

## **İşlemleri Gerçekleştirme**

Ethereum'daki işlemler bir gönderici ve bir alıcı içerir; bu alıcı ya kullanıcı ya da akıllı sözleşme adresi olabilir. İşlemler bir ücret gerektirir ve madencilik yapılması gerekir. Bir işlemdeki temel bilgiler alıcı, göndericinin imzası, değer, isteğe bağlı veri, gaz limiti ve ücretlerdir. Özellikle, göndericinin adresi imzadan çıkarılır, bu da işlem verilerinde bulunmasına gerek kalmaz.

Bu uygulamalar ve mekanizmalar, gizlilik ve güvenliği önceliklendiren herkes için kripto para birimleriyle etkileşimde bulunmak için temeldir.


## Referanslar

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)


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
