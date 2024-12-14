# वेब से संवेदनशील जानकारी का खुलासा चुराना

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

यदि किसी समय आप एक **वेब पृष्ठ पाते हैं जो आपके सत्र के आधार पर आपको संवेदनशील जानकारी प्रस्तुत करता है**: शायद यह कुकीज़ को दर्शा रहा है, या CC विवरण या कोई अन्य संवेदनशील जानकारी प्रिंट कर रहा है, आप इसे चुराने की कोशिश कर सकते हैं।\
यहाँ मैं आपको मुख्य तरीकों का परिचय देता हूँ जिनसे आप इसे प्राप्त करने की कोशिश कर सकते हैं:

* [**CORS बायपास**](../pentesting-web/cors-bypass.md): यदि आप CORS हेडर को बायपास कर सकते हैं तो आप एक दुर्भावनापूर्ण पृष्ठ के लिए Ajax अनुरोध करते हुए जानकारी चुराने में सक्षम होंगे।
* [**XSS**](../pentesting-web/xss-cross-site-scripting/): यदि आप पृष्ठ पर XSS भेद्यता पाते हैं तो आप इसे जानकारी चुराने के लिए दुरुपयोग कर सकते हैं।
* [**Danging Markup**](../pentesting-web/dangling-markup-html-scriptless-injection/): यदि आप XSS टैग इंजेक्ट नहीं कर सकते हैं तो भी आप अन्य सामान्य HTML टैग का उपयोग करके जानकारी चुराने में सक्षम हो सकते हैं।
* [**Clickjaking**](../pentesting-web/clickjacking.md): यदि इस हमले के खिलाफ कोई सुरक्षा नहीं है, तो आप उपयोगकर्ता को संवेदनशील डेटा भेजने के लिए धोखा देने में सक्षम हो सकते हैं (एक उदाहरण [यहाँ](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20))।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
