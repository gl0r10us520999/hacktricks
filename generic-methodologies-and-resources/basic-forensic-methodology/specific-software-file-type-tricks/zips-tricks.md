# ZIPs ट्रिक्स

{% hint style="success" %}
**AWS हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

**कमांड-लाइन टूल्स** जो **zip फ़ाइलें** प्रबंधित करने के लिए आवश्यक हैं, zip फ़ाइलें का निदान करने, मरम्मत करने, और क्रैक करने के लिए। यहाँ कुछ मुख्य उपयोगी उपकरण हैं:

- **`unzip`**: बताता है कि एक zip फ़ाइल को क्यों डीकंप्रेस नहीं किया जा सकता है।
- **`zipdetails -v`**: zip फ़ाइल प्रारूप के क्षेत्रों का विस्तृत विश्लेषण प्रदान करता है।
- **`zipinfo`**: उन्हें निकाले बिना zip फ़ाइल की सामग्री की सूची देता है।
- **`zip -F input.zip --out output.zip`** और **`zip -FF input.zip --out output.zip`**: क्षतिग्रस्त zip फ़ाइलों को मरम्मत करने का प्रयास करें।
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: zip पासवर्ड का ब्रूट-फ़ोर्स क्रैकिंग के लिए एक उपकरण, जो लगभग 7 अक्षरों तक के पासवर्ड के लिए प्रभावी है।

[Zip फ़ाइल प्रारूप विनिर्देशिका](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) zip फ़ाइलों के संरचना और मानकों पर विस्तृत विवरण प्रदान करती है।

महत्वपूर्ण है कि पासवर्ड से सुरक्षित zip फ़ाइलें **फ़ाइल नाम या फ़ाइल का आकार एन्क्रिप्ट नहीं करतीं**, जो एक सुरक्षा दोष है जिसे RAR या 7z फ़ाइलें नहीं साझा करतीं जो इस जानकारी को एन्क्रिप्ट करती हैं। इसके अतिरिक्त, पुरानी ZipCrypto विधि से एन्क्रिप्ट की गई zip फ़ाइलें एक **सादाक्षर हमले** के लिए विकल्पशील हैं अगर किसी अनएन्क्रिप्टेड कॉपी की एक कम्प्रेस्ड फ़ाइल उपलब्ध है। यह हमला जानी गई सामग्री का उपयोग करता है ताकि zip का पासवर्ड क्रैक किया जा सके, जो [HackThis के लेख](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) में विस्तार से वर्णित है और [इस शैक्षिक पेपर](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) में और विस्तार से समझाया गया है। हालांकि, **AES-256** एन्क्रिप्शन के साथ सुरक्षित की गई zip फ़ाइलें इस सादाक्षर हमले से मुक्त हैं, संवेदनशील डेटा के लिए सुरक्षित एन्क्रिप्शन विधियों का चयन करने की महत्वता को प्रदर्शित करती है।

## संदर्भ
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/) 

{% hint style="success" %}
**AWS हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**GCP हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
