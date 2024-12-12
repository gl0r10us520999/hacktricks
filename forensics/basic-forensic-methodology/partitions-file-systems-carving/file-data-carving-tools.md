{% hint style="success" %}
एडब्ल्यूएस हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
जीसीपी हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
{% endhint %}


# कार्विंग टूल्स

## Autopsy

फोरेंसिक्स में सबसे आम उपयोग किया जाने वाला टूल [**Autopsy**](https://www.autopsy.com/download/) है। इसे डाउनलोड करें, इंस्टॉल करें और फ़ाइल को इंजेस्ट करने के लिए इसका उपयोग करें ताकि "छिपी" फ़ाइलें मिल सकें। ध्यान दें कि Autopsy को डिस्क इमेज और अन्य प्रकार की इमेज का समर्थन करने के लिए बनाया गया है, लेकिन साधारण फ़ाइलों का समर्थन नहीं किया गया है।

## Binwalk <a id="binwalk"></a>

**Binwalk** एक टूल है जो इमेज और ऑडियो फ़ाइलों जैसे बाइनरी फ़ाइलों में छिपी फ़ाइलें और डेटा की खोज के लिए उपयोग किया जाता है। इसे `apt` के साथ इंस्टॉल किया जा सकता है हालांकि [स्रोत](https://github.com/ReFirmLabs/binwalk) github पर उपलब्ध है। **उपयोगी कमांड**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपी हुई फ़ाइलें खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost का विन्यास फ़ाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप कुछ विशिष्ट फ़ाइलें खोजना चाहते हैं तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं तो foremost अपने डिफ़ॉल्ट विन्यस्त फ़ाइल प्रकारों के लिए खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **स्कैल्पल**

**स्कैल्पल** एक और उपकरण है जिसका उपयोग **एक फ़ाइल में एम्बेड किए गए फ़ाइलें खोजने और निकालने** के लिए किया जा सकता है। इस मामले में, आपको उसे निकालना चाहिए जिसे आप विन्यास फ़ाइल \(_/etc/scalpel/scalpel.conf_\) से अनटिक करना चाहेंगे।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली के अंदर आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

यह टूल एक छवि को स्कैन कर सकता है और उसमें **पीकैप्स को निकाल सकता है**, **नेटवर्क जानकारी\(URLs, डोमेन, IPs, MACs, मेल्स\)** और अधिक **फ़ाइलें**। आपको केवल यह करना है:
```text
bulk_extractor memory.img -o out_folder
```
Navigate through **सभी जानकारी** जो उपकरण ने इकट्ठा की है \(पासवर्ड?\), **विश्लेषित** करें **पैकेट्स** \(पढ़ें [**Pcaps विश्लेषण**](../pcap-inspection/)\), **अजीब डोमेन** खोजें \(मैलवेयर से संबंधित या **अस्तित्वहीन**\).

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) में पा सकते हैं।

यह GUI और CLI संस्करण के साथ आता है। आप चुन सकते हैं **फ़ाइल-प्रकार** जिनकी खोज के लिए PhotoRec को खोजना है।

![](../../../.gitbook/assets/image%20%28524%29.png)

# विशेष डेटा कार्विंग उपकरण

## FindAES

अपने कुंजी अनुसूचियों की खोज करके AES कुंजियों की खोज करता है। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम है, जैसे जो TrueCrypt और BitLocker द्वारा उपयोग की जाती हैं।

यहाँ [डाउनलोड करें](https://sourceforge.net/projects/findaes/).

# पूरक उपकरण

आप [**viu** ](https://github.com/atanunq/viu)का उपयोग कर सकते हैं ताकि आप छवियों को टर्मिनल से देख सकें।
आप लिनक्स कमांड लाइन उपकरण **pdftotext** का उपयोग करके एक पीडीएफ को पाठ में रूपांतरित करने और पढ़ने के लिए कर सकते हैं।
