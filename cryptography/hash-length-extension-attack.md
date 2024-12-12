{% hint style="success" %}
**एडब्ल्यूएस हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण एडब्ल्यूएस रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**जीसीपी हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण जीसीपी रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
{% endhint %}


# हमले का सारांश

एक सर्वर की कलर टेक्स्ट डेटा के साथ कुछ गोपनीय जोड़कर उस डेटा को हैश कर रहा है। यदि आप जानते हैं:

* **गोपनीय की लंबाई** (यह दी गई लंबाई सीमा से भी ब्रूटफोर्स किया जा सकता है)
* **स्पष्ट पाठ डेटा**
* **एल्गोरिथ्म (और यह हमले के लिए वंशानुक्रमित है)**
* **पैडिंग ज्ञात है**
* आम तौर पर एक डिफ़ॉल्ट प्रयोग होता है, इसलिए अगर अन्य 3 आवश्यकताएं पूरी होती हैं, तो यह भी होता है
* पैडिंग गोपनीय+डेटा की लंबाई पर निर्भर होता है, इसलिए गोपनीय की लंबाई की आवश्यकता है

तो, एक **हमलावर** के लिए संभव है कि वह **डेटा** को **जोड़ सकता है** और **पिछले डेटा + जोड़े गए डेटा** के लिए एक मान्य **सिग्नेचर** उत्पन्न कर सकता है।

## कैसे?

मूल रूप से, वंशानुक्रमित एल्गोरिथ्म पहले **डेटा ब्लॉक को हैश** करके हैश उत्पन्न करते हैं, और फिर, **पिछले** बनाए गए **हैश** (स्थिति) से, वे **अगले डेटा ब्लॉक को जोड़ते हैं** और **हैश** करते हैं।

फिर, यहां यह विचार करें कि गोपनीय "गोपनीय" है और डेटा "डेटा" है, "गोपनीयडेटा" का MD5 6036708eba0d11f6ef52ad44e8b74d5b है।\
यदि एक हमलावर "जोड़ना" स्ट्रिंग जोड़ना चाहता है तो वह:

* 64 "ए" का MD5 उत्पन्न करें
* पहले से ही आरंभित हैश की स्थिति को 6036708eba0d11f6ef52ad44e8b74d5b पर बदलें
* स्ट्रिंग "जोड़ना" जोड़ें
* हैश समाप्त करें और परिणामी हैश "गोपनीय" + "डेटा" + "पैडिंग" + "जोड़ना" के लिए एक **मान्य** होगा**

## **उपकरण**

{% embed url="https://github.com/iagox86/hash_extender" %}

## संदर्भ

आप इस हमले को अच्छी तरह से समझ सकते हैं [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)


{% hint style="success" %}
**एडब्ल्यूएस हैकिंग सीखें और अभ्यास करें:**<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण एडब्ल्यूएस रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
**जीसीपी हैकिंग सीखें और अभ्यास करें:** <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण जीसीपी रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
{% endhint %}
