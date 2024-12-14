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


# Carving tools

## Autopsy

Görüntülerden dosya çıkarmak için adli bilimlerde en yaygın kullanılan araç [**Autopsy**](https://www.autopsy.com/download/)'dir. İndirin, kurun ve "gizli" dosyaları bulmak için dosyayı içe aktarmasını sağlayın. Autopsy'nin disk görüntüleri ve diğer türdeki görüntüleri desteklemek için tasarlandığını, ancak basit dosyaları desteklemediğini unutmayın.

## Binwalk <a id="binwalk"></a>

**Binwalk**, gömülü dosyalar ve veriler için görüntüler ve ses dosaları gibi ikili dosyaları aramak için bir araçtır. `apt` ile kurulabilir, ancak [kaynağı](https://github.com/ReFirmLabs/binwalk) github'da bulabilirsiniz.  
**Kullanışlı komutlar**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

Gizli dosyaları bulmak için başka bir yaygın araç **foremost**'tur. Foremost'un yapılandırma dosyasını `/etc/foremost.conf` konumunda bulabilirsiniz. Eğer sadece belirli dosyaları aramak istiyorsanız, bunların yorumunu kaldırın. Eğer hiçbir şeyi yorumdan çıkarmazsanız, foremost varsayılan olarak yapılandırılmış dosya türlerini arayacaktır.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel**, bir dosya içinde gömülü **dosyaları** bulmak ve çıkarmak için kullanılabilecek bir diğer araçtır. Bu durumda, çıkarmak istediğiniz dosya türlerini yapılandırma dosyasından (_/etc/scalpel/scalpel.conf_) yorumdan çıkarmanız gerekecektir.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

Bu araç kali içinde gelir ama burada bulabilirsiniz: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

Bu araç bir görüntüyü tarayabilir ve içindeki **pcap'leri** **çıkartabilir**, **ağ bilgilerini (URL'ler, alan adları, IP'ler, MAC'ler, mailler)** ve daha fazla **dosyayı** alabilir. Yapmanız gereken tek şey:
```text
bulk_extractor memory.img -o out_folder
```
Tüm **bilgileri** gözden geçirin \(şifreler?\), **analiz** edin **paketleri** \(okuyun[ **Pcaps analizi**](../pcap-inspection/)\), **garip alan adlarını** arayın \(**kötü amaçlı yazılım** veya **var olmayan** alan adları ile ilgili\).

## PhotoRec

Bunu [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) adresinde bulabilirsiniz.

GUI ve CLI sürümü ile gelir. PhotoRec'in aramasını istediğiniz **dosya türlerini** seçebilirsiniz.

![](../../../.gitbook/assets/image%20%28524%29.png)

# Özel Veri Karıştırma Araçları

## FindAES

Anahtar programlarını arayarak AES anahtarlarını arar. TrueCrypt ve BitLocker gibi 128, 192 ve 256 bit anahtarları bulabilir.

[Buradan](https://sourceforge.net/projects/findaes/) indirin.

# Tamamlayıcı araçlar

Terminalden görüntüleri görmek için [**viu** ](https://github.com/atanunq/viu) kullanabilirsiniz. 
Bir pdf'yi metne dönüştürmek ve okumak için linux komut satırı aracı **pdftotext** kullanabilirsiniz.



{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* Hacking ipuçlarını [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR göndererek paylaşın.

</details>
{% endhint %}
