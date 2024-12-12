# File/Data Carving & Recovery Tools

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

## Carving & Recovery tools

Daha fazla araç için [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

### Autopsy

Görüntülerden dosya çıkarmak için adli bilimlerde en yaygın kullanılan araç [**Autopsy**](https://www.autopsy.com/download/)'dir. İndirin, kurun ve "gizli" dosyaları bulmak için dosyayı içe aktarmasını sağlayın. Autopsy'nin disk görüntüleri ve diğer türdeki görüntüleri desteklemek için tasarlandığını, ancak basit dosyaları desteklemediğini unutmayın.

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk**, gömülü içeriği bulmak için ikili dosyaları analiz etmek için kullanılan bir araçtır. `apt` ile kurulabilir ve kaynak kodu [GitHub](https://github.com/ReFirmLabs/binwalk)'ta bulunmaktadır.

**Kullanışlı komutlar**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

Gizli dosyaları bulmak için başka bir yaygın araç **foremost**'tur. Foremost'un yapılandırma dosyasını `/etc/foremost.conf` içinde bulabilirsiniz. Eğer sadece belirli dosyaları aramak istiyorsanız, bunların yorumunu kaldırın. Eğer hiçbir şeyi yorumdan çıkarmazsanız, foremost varsayılan olarak yapılandırılmış dosya türlerini arayacaktır.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Scalpel**

**Scalpel**, bir dosya içinde gömülü **dosyaları** bulmak ve çıkarmak için kullanılabilecek bir başka araçtır. Bu durumda, çıkarmak istediğiniz dosya türlerini yapılandırma dosyasından (_/etc/scalpel/scalpel.conf_) yorumdan çıkarmanız gerekecektir.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

Bu araç kali içinde gelir ama burada bulabilirsiniz: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

Bu araç bir görüntüyü tarayabilir ve içindeki **pcap'leri** **çıkartabilir**, **ağ bilgilerini (URL'ler, alan adları, IP'ler, MAC'ler, mailler)** ve daha fazla **dosyayı** alabilir. Yapmanız gereken tek şey:
```
bulk_extractor memory.img -o out_folder
```
Navigate through **tüm bilgileri** that the tool has gathered (şifreler?), **analiz et** the **paketleri** (oku[ **Pcaps analizi**](../pcap-inspection/)), search for **garip alan adları** (malware ile ilgili veya **mevcut olmayan** alan adları).

### PhotoRec

You can find it in [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download)

It comes with GUI and CLI versions. You can select the **dosya türleri** you want PhotoRec to search for.

![](<../../../.gitbook/assets/image (242).png>)

### binvis

Check the [kod](https://code.google.com/archive/p/binvis/) and the [web sayfası aracı](https://binvis.io/#/).

#### BinVis Özellikleri

* Görsel ve aktif **yapı görüntüleyici**
* Farklı odak noktaları için birden fazla grafik
* Bir örneğin bölümlerine odaklanma
* PE veya ELF yürütülebilir dosyalarda **dize ve kaynakları görme**
* Dosyalar üzerinde kriptoanaliz için **desenler** elde etme
* **Paketleyici** veya kodlayıcı algoritmaları **belirleme**
* Desenler ile Steganografi **tanımlama**
* **Görsel** ikili fark analizi

BinVis, bir kara kutu senaryosunda bilinmeyen bir hedefle tanışmak için harika bir **başlangıç noktasıdır**.

## Özel Veri Karıştırma Araçları

### FindAES

AES anahtarlarını anahtar programlarını arayarak bulur. TrueCrypt ve BitLocker gibi 128, 192 ve 256 bit anahtarları bulabilir.

Download [buradan](https://sourceforge.net/projects/findaes/).

## Tamamlayıcı araçlar

You can use [**viu** ](https://github.com/atanunq/viu) to see images from the terminal.\
You can use the linux command line tool **pdftotext** to transform a pdf into text and read it.

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**abonelik planları**](https://github.com/sponsors/carlospolop)!
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya **bizi takip edin** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşın,** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR göndererek.

</details>
{% endhint %}
