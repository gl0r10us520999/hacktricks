{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}


# मूलाधार

एक मूलाधार किसी सिस्टम के कुछ हिस्सों का एक स्नैपशॉट लेने से बनता है ताकि **भविष्य की स्थिति की तुलना करने के लिए परिवर्तनों को हाइलाइट किया जा सके**।

उदाहरण के लिए, आप फाइल सिस्टम की प्रत्येक फाइल का हैश गणना और स्टोर करना कर सकते हैं ताकि आप पता लगा सकें कि कौन सी फाइलें संशोधित की गई थीं।\
यह भी उपयोगकर्ता खाते बनाए गए, प्रक्रियाएँ चल रही हैं, सेवाएँ चल रही हैं और किसी भी अन्य चीज़ के साथ किया जा सकता है जो बहुत अधिक या पूरी तरह से बदलना नहीं चाहिए।

## फाइल अखंडता मॉनिटरिंग

फाइल अखंडता मॉनिटरिंग (FIM) एक महत्वपूर्ण सुरक्षा तकनीक है जो फ़ाइलों में परिवर्तनों का ट्रैक करके IT परिवेश और डेटा को सुरक्षित रखती है। इसमें दो मुख्य चरण शामिल हैं:

1. **मूलाधार तुलना:** भविष्य की तुलना के लिए फ़ाइल गुणांक या एन्क्रिप्टेड चेकसम्स (जैसे MD5 या SHA-2) का उपयोग करके मूलाधार स्थापित करें ताकि संशोधनों का पता लगा सकें।
2. **रियल-टाइम चेंज नोटिफिकेशन:** फ़ाइलों तक पहुंचने या संशोधन किए जाने पर तुरंत अलर्ट प्राप्त करें, आम तौर पर OS कर्नल एक्सटेंशन के माध्यम से।

## उपकरण

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

## संदर्भ

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
