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

More tools in [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

### Autopsy

Chombo kinachotumika sana katika uchunguzi kutoa faili kutoka kwa picha ni [**Autopsy**](https://www.autopsy.com/download/). Pakua, sakinisha na fanya iweze kuchukua faili ili kupata faili "zilizofichwa". Kumbuka kwamba Autopsy imejengwa kusaidia picha za diski na aina nyingine za picha, lakini si faili rahisi.

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** ni chombo cha kuchambua faili za binary ili kupata maudhui yaliyofichwa. Inaweza kusakinishwa kupitia `apt` na chanzo chake kiko kwenye [GitHub](https://github.com/ReFirmLabs/binwalk).

**Amri muhimu**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

Chombo kingine cha kawaida cha kutafuta faili zilizofichwa ni **foremost**. Unaweza kupata faili ya usanidi ya foremost katika `/etc/foremost.conf`. Ikiwa unataka tu kutafuta faili fulani, ondoa alama ya maoni. Ikiwa huondoi alama ya maoni, foremost itatafuta aina zake za faili zilizowekwa kama chaguo-msingi.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Scalpel**

**Scalpel** ni chombo kingine ambacho kinaweza kutumika kutafuta na kutoa **faili zilizojumuishwa ndani ya faili**. Katika kesi hii, utahitaji kuondoa maoni kutoka kwa faili ya usanidi (_/etc/scalpel/scalpel.conf_) aina za faili unazotaka ikatoe.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

Chombo hiki kinapatikana ndani ya kali lakini unaweza kukipata hapa: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

Chombo hiki kinaweza kuskan picha na **kutoa pcaps** ndani yake, **taarifa za mtandao (URLs, domains, IPs, MACs, mails)** na zaidi **faili**. Unachohitaji kufanya ni:
```
bulk_extractor memory.img -o out_folder
```
Navigate through **maelezo yote** that the tool has gathered (passwords?), **chambua** the **paket** (read[ **Pcaps analysis**](../pcap-inspection/)), search for **domeni za ajabu** (domains related to **malware** or **zisizokuwepo**).

### PhotoRec

You can find it in [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download)

It comes with GUI and CLI versions. You can select the **aina za faili** you want PhotoRec to search for.

![](<../../../.gitbook/assets/image (242).png>)

### binvis

Check the [code](https://code.google.com/archive/p/binvis/) and the [web page tool](https://binvis.io/#/).

#### Features of BinVis

* Visual and active **muonekano wa muundo**
* Multiple plots for different focus points
* Focusing on portions of a sample
* **Kuona stings na rasilimali**, in PE or ELF executables e. g.
* Getting **mifumo** for cryptanalysis on files
* **Kugundua** packer or encoder algorithms
* **Tambua** Steganography by patterns
* **Visual** binary-diffing

BinVis is a great **nukta ya kuanzia kujifunza kuhusu lengo lisilojulikana** in a black-boxing scenario.

## Specific Data Carving Tools

### FindAES

Searches for AES keys by searching for their key schedules. Able to find 128. 192, and 256 bit keys, such as those used by TrueCrypt and BitLocker.

Download [hapa](https://sourceforge.net/projects/findaes/).

## Complementary tools

You can use [**viu** ](https://github.com/atanunq/viu)to see images from the terminal.\
You can use the linux command line tool **pdftotext** to transform a pdf into text and read it.

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) or the [**kikundi cha telegram**](https://t.me/peass) or **fuata** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki hila za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
