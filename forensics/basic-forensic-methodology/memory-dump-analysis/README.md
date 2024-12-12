# मेमोरी डंप विश्लेषण

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे प्रासंगिक साइबर सुरक्षा कार्यक्रम है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उष्णकटिबंधीय बैठक बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

## प्रारंभ

**pcap** के अंदर **malware** के लिए **खोज** करना शुरू करें। [**Malware Analysis**](../malware-analysis.md) में उल्लेखित **tools** का उपयोग करें।

## [Volatility](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)

**Volatility मेमोरी डंप विश्लेषण के लिए मुख्य ओपन-सोर्स ढांचा है**। यह Python उपकरण बाहरी स्रोतों या VMware VMs से डंप का विश्लेषण करता है, डंप के OS प्रोफ़ाइल के आधार पर प्रक्रियाओं और पासवर्ड जैसे डेटा की पहचान करता है। यह प्लगइन्स के साथ विस्तारित किया जा सकता है, जिससे यह फोरेंसिक जांच के लिए अत्यधिक बहुपरकारी बनता है।

**[यहाँ एक cheatsheet पाएं](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)**

## मिनी डंप क्रैश रिपोर्ट

जब डंप छोटा होता है (केवल कुछ KB, शायद कुछ MB) तो यह शायद एक मिनी डंप क्रैश रिपोर्ट है और मेमोरी डंप नहीं है।

![](<../../../.gitbook/assets/image (216).png>)

यदि आपके पास Visual Studio स्थापित है, तो आप इस फ़ाइल को खोल सकते हैं और प्रक्रिया का नाम, आर्किटेक्चर, अपवाद जानकारी और निष्पादित हो रहे मॉड्यूल जैसी कुछ बुनियादी जानकारी बाइंड कर सकते हैं:

![](<../../../.gitbook/assets/image (217).png>)

आप अपवाद को भी लोड कर सकते हैं और डिकंपाइल की गई निर्देशों को देख सकते हैं

![](<../../../.gitbook/assets/image (219).png>)

![](<../../../.gitbook/assets/image (218) (1).png>)

वैसे भी, Visual Studio डंप के गहरे विश्लेषण के लिए सबसे अच्छा उपकरण नहीं है।

आपको इसे **IDA** या **Radare** का उपयोग करके **गहराई** से निरीक्षण करना चाहिए।

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे प्रासंगिक साइबर सुरक्षा कार्यक्रम है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उष्णकटिबंधीय बैठक बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
