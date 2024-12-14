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


# Zana za kuchora

## Autopsy

Zana inayotumika sana katika uchunguzi wa kidijitali kutoa faili kutoka kwa picha ni [**Autopsy**](https://www.autopsy.com/download/). Pakua, sakinisha na fanya iweze kuchukua faili ili kupata faili "zilizofichwa". Kumbuka kwamba Autopsy imejengwa kusaidia picha za diski na aina nyingine za picha, lakini si faili rahisi.

## Binwalk <a id="binwalk"></a>

**Binwalk** ni zana ya kutafuta faili za binary kama picha na faili za sauti kwa faili na data zilizojumuishwa.
Inaweza kusakinishwa kwa kutumia `apt` hata hivyo [chanzo](https://github.com/ReFirmLabs/binwalk) kinaweza kupatikana kwenye github.
**Amri muhimu**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

Zana lingine la kawaida la kutafuta faili zilizofichwa ni **foremost**. Unaweza kupata faili ya usanidi ya foremost katika `/etc/foremost.conf`. Ikiwa unataka tu kutafuta faili fulani, ondoa alama ya maoni. Ikiwa huondoi alama ya maoni, foremost itatafuta aina za faili zilizowekwa kama chaguo-msingi.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** ni chombo kingine ambacho kinaweza kutumika kutafuta na kutoa **faili zilizojumuishwa katika faili**. Katika kesi hii, utahitaji kuondoa maoni kutoka kwa faili ya usanidi \(_/etc/scalpel/scalpel.conf_\) aina za faili unazotaka ikatoe.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

Chombo hiki kinapatikana ndani ya kali lakini unaweza kukipata hapa: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

Chombo hiki kinaweza kuskan picha na **kutoa pcaps** ndani yake, **taarifa za mtandao\(URLs, domains, IPs, MACs, mails\)** na zaidi **faili**. Unahitaji tu kufanya:
```text
bulk_extractor memory.img -o out_folder
```
Navigate through **maelezo yote** that the tool has gathered \(passwords?\), **analyze** the **paket** \(read[ **Pcaps analysis**](../pcap-inspection/)\), search for **domeni za ajabu** \(domains related to **malware** or **isiyokuwepo**\).

## PhotoRec

You can find it in [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)

It comes with GUI and CLI version. You can select the **aina za faili** you want PhotoRec to search for.

![](../../../.gitbook/assets/image%20%28524%29.png)

# Specific Data Carving Tools

## FindAES

Searches for AES keys by searching for their key schedules. Able to find 128. 192, and 256 bit keys, such as those used by TrueCrypt and BitLocker.

Download [hapa](https://sourceforge.net/projects/findaes/).

# Complementary tools

You can use [**viu** ](https://github.com/atanunq/viu)to see images form the terminal.
You can use the linux command line tool **pdftotext** to transform a pdf into text and read it.



{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**mpango ya usajili**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) or the [**kikundi cha telegram**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
