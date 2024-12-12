# वेब से संवेदनशील जानकारी फासवन

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) **और** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **github रेपो में।**

</details>
{% endhint %}

यदि किसी समय आपको एक **वेब पृष्ठ मिलता है जो आपकी सत्र पर आधारित संवेदनशील जानकारी प्रस्तुत करता है**: शायद यह कुकीज को प्रतिबिंबित कर रहा है, या डिटेल्स या किसी अन्य संवेदनशील जानकारी को प्रिंट कर रहा है, तो आप इसे चुरा सकते हैं।\
यहाँ मैं आपको इसे प्राप्त करने की प्रमुख तरीके प्रस्तुत करता हूँ:

* [**CORS बाईपास**](pentesting-web/cors-bypass.md): यदि आप CORS हेडर्स को बाईपास कर सकते हैं तो आप एक दुरुपयोगी पृष्ठ के लिए एजेक्स अनुरोध करके जानकारी चुरा सकते हैं।
* [**XSS**](pentesting-web/xss-cross-site-scripting/): यदि आप पृष्ठ पर XSS दोष पाते हैं तो आप इसका दुरुपयोग करके जानकारी चुरा सकते हैं।
* [**डैंगलिंग मार्कअप**](pentesting-web/dangling-markup-html-scriptless-injection/): यदि आप XSS टैग नहीं इंजेक्ट कर सकते हैं तो आप अन्य साधारण HTML टैग का उपयोग करके जानकारी चुरा सकते हैं।
* [**क्लिकजैकिंग**](pentesting-web/clickjacking.md): यदि इस हमले के खिलाफ कोई सुरक्षा नहीं है, तो आप उपयोगकर्ता को धोखा देकर संवेदनशील डेटा भेजने में सक्षम हो सकते हैं (एक उदाहरण [यहाँ](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20))।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) **और** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **github रेपो में।**

</details>
{% endhint %}
