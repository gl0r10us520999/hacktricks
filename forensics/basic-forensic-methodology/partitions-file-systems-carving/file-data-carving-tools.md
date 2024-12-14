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

फोरेंसिक्स में छवियों से फ़ाइलें निकालने के लिए सबसे सामान्य उपकरण [**Autopsy**](https://www.autopsy.com/download/) है। इसे डाउनलोड करें, इंस्टॉल करें और "छिपी हुई" फ़ाइलें खोजने के लिए इसे फ़ाइल को इनजेस्ट करने दें। ध्यान दें कि Autopsy को डिस्क इमेज और अन्य प्रकार की छवियों का समर्थन करने के लिए बनाया गया है, लेकिन साधारण फ़ाइलों के लिए नहीं।

## Binwalk <a id="binwalk"></a>

**Binwalk** एक उपकरण है जो बाइनरी फ़ाइलों जैसे छवियों और ऑडियो फ़ाइलों में अंतर्निहित फ़ाइलों और डेटा की खोज करता है। इसे `apt` के साथ स्थापित किया जा सकता है, हालाँकि [स्रोत](https://github.com/ReFirmLabs/binwalk) github पर पाया जा सकता है।  
**उपयोगी कमांड**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

एक और सामान्य उपकरण जो छिपी हुई फ़ाइलों को खोजने के लिए है वह है **foremost**। आप foremost की कॉन्फ़िगरेशन फ़ाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप केवल कुछ विशिष्ट फ़ाइलों के लिए खोज करना चाहते हैं, तो उन्हें अनकमेंट करें। यदि आप कुछ भी अनकमेंट नहीं करते हैं, तो foremost अपनी डिफ़ॉल्ट कॉन्फ़िगर की गई फ़ाइल प्रकारों के लिए खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक और उपकरण है जिसका उपयोग **फाइल में एम्बेडेड फाइलों** को खोजने और निकालने के लिए किया जा सकता है। इस मामले में, आपको कॉन्फ़िगरेशन फ़ाइल \(_/etc/scalpel/scalpel.conf_\) से उन फ़ाइल प्रकारों को अनकमेंट करना होगा जिन्हें आप निकालना चाहते हैं।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह उपकरण काली के अंदर आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

यह उपकरण एक इमेज को स्कैन कर सकता है और इसके अंदर **pcaps** को **निकालेगा**, **नेटवर्क जानकारी\(URLs, domains, IPs, MACs, mails\)** और अधिक **फाइलें**। आपको केवल यह करना है:
```text
bulk_extractor memory.img -o out_folder
```
Navigate through **सभी जानकारी** जो उपकरण ने इकट्ठा की है \(passwords?\), **विश्लेषण करें** **पैकेट्स** \(read[ **Pcaps analysis**](../pcap-inspection/)\), **अजीब डोमेन** की खोज करें \(domains related to **malware** or **non-existent**\).

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) पर पा सकते हैं।

यह GUI और CLI संस्करण के साथ आता है। आप **फाइल-प्रकार** चुन सकते हैं जिन्हें आप PhotoRec से खोजने के लिए चाहते हैं।

![](../../../.gitbook/assets/image%20%28524%29.png)

# विशिष्ट डेटा कार्विंग उपकरण

## FindAES

AES कुंजियों की खोज उनके कुंजी शेड्यूल की खोज करके करता है। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम, जैसे कि TrueCrypt और BitLocker द्वारा उपयोग की जाने वाली।

[यहाँ डाउनलोड करें](https://sourceforge.net/projects/findaes/)।

# पूरक उपकरण

आप [**viu** ](https://github.com/atanunq/viu) का उपयोग करके टर्मिनल से चित्र देख सकते हैं।  
आप लिनक्स कमांड लाइन उपकरण **pdftotext** का उपयोग करके एक pdf को टेक्स्ट में बदल सकते हैं और इसे पढ़ सकते हैं।

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
