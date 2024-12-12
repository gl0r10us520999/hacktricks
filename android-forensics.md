# एंड्रॉइड फोरेंसिक्स

{% hint style="success" %}
**AWS हैकिंग सीखें और प्रैक्टिस करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और प्रैक्टिस करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## लॉक किया गया डिवाइस

एक एंड्रॉइड डिवाइस से डेटा निकालने के लिए इसे अनलॉक किया जाना चाहिए। अगर यह लॉक है तो आप कर सकते हैं:

* जांचें कि डिवाइस में USB के माध्यम से डीबगिंग सक्षम है।
* [स्मज अटैक](https://www.usenix.org/legacy/event/woot10/tech/full\_papers/Aviv.pdf) के लिए जांच करें
* [ब्रूट-फोर्स](https://www.cultofmac.com/316532/this-brute-force-device-can-crack-any-iphones-pin-code/) के साथ प्रयास करें

## डेटा अधिग्रहण

[adb का उपयोग करके एंड्रॉइड बैकअप बनाएं](mobile-pentesting/android-app-pentesting/adb-commands.md#backup) और इसे [एंड्रॉइड बैकअप एक्सट्रैक्टर](https://sourceforge.net/projects/adbextractor/) का उपयोग करके निकालें: `java -jar abe.jar unpack file.backup file.tar`

### यदि रूट एक्सेस या JTAG इंटरफेस के लिए फिजिकल कनेक्शन है

* `cat /proc/partitions` (फ्लैश मेमोरी का पथ खोजें, सामान्य रूप से पहला एंट्री _mmcblk0_ होती है और पूरी फ्लैश मेमोरी के लिए होती है)।
* `df /data` (सिस्टम का ब्लॉक साइज़ खोजें)।
* dd if=/dev/block/mmcblk0 of=/sdcard/blk0.img bs=4096 (इसे ब्लॉक साइज़ से जुटे जानकारी के साथ चलाएं)।

### मेमोरी

लिनक्स मेमोरी एक्सट्रैक्टर (LiME) का उपयोग करें रैम जानकारी निकालने के लिए। यह एक कर्नेल एक्सटेंशन है जो adb के माध्यम से लोड किया जाना चाहिए।

{% hint style="success" %}
**AWS हैकिंग सीखें और प्रैक्टिस करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और प्रैक्टिस करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
