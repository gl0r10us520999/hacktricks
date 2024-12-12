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


इंटरनेट पर कई ब्लॉग हैं जो **डिफ़ॉल्ट/कमजोर** लॉगिन क्रेडेंशियल्स के साथ LDAP के साथ कॉन्फ़िगर की गई प्रिंटरों के खतरों को **उजागर करते हैं**।\
यह इसलिए है क्योंकि एक हमलावर **प्रिंटर को एक धोखेबाज़ LDAP सर्वर के खिलाफ प्रमाणित करने के लिए धोखा दे सकता है** (आमतौर पर एक `nc -vv -l -p 444` पर्याप्त है) और प्रिंटर के **क्रेडेंशियल्स को स्पष्ट पाठ में कैप्चर कर सकता है**।

इसके अलावा, कई प्रिंटर **उपयोगकर्ता नामों के साथ लॉग** रखेंगे या यहां तक कि **डोमेन कंट्रोलर से सभी उपयोगकर्ता नामों को डाउनलोड करने में सक्षम हो सकते हैं**।

यह सब **संवेदनशील जानकारी** और सामान्य **सुरक्षा की कमी** प्रिंटरों को हमलावरों के लिए बहुत दिलचस्प बनाती है।

इस विषय पर कुछ ब्लॉग:

* [https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/](https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/)
* [https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)

## प्रिंटर कॉन्फ़िगरेशन
- **स्थान**: LDAP सर्वर सूची यहाँ पाई जाती है: `Network > LDAP Setting > Setting Up LDAP`।
- **व्यवहार**: इंटरफ़ेस LDAP सर्वर संशोधनों की अनुमति देता है बिना क्रेडेंशियल्स फिर से दर्ज किए, उपयोगकर्ता की सुविधा के लिए लेकिन सुरक्षा जोखिम पैदा करता है।
- **शोषण**: शोषण में LDAP सर्वर पते को नियंत्रित मशीन पर पुनर्निर्देशित करना और क्रेडेंशियल्स कैप्चर करने के लिए "Test Connection" सुविधा का लाभ उठाना शामिल है।

## क्रेडेंशियल्स कैप्चर करना

**अधिक विस्तृत चरणों के लिए, मूल [स्रोत](https://grimhacker.com/2018/03/09/just-a-printer/) को देखें।**

### विधि 1: नेटकैट लिस्नर
एक साधारण नेटकैट लिस्नर पर्याप्त हो सकता है:
```bash
sudo nc -k -v -l -p 386
```
हालांकि, इस विधि की सफलता भिन्न होती है।

### विधि 2: पूर्ण LDAP सर्वर के साथ Slapd
एक अधिक विश्वसनीय दृष्टिकोण एक पूर्ण LDAP सर्वर स्थापित करना है क्योंकि प्रिंटर एक शून्य बाइंड करता है उसके बाद क्रेडेंशियल बाइंडिंग का प्रयास करने से पहले एक क्वेरी करता है।

1. **LDAP सर्वर सेटअप**: गाइड [इस स्रोत](https://www.server-world.info/en/note?os=Fedora_26&p=openldap) से चरणों का पालन करता है।
2. **मुख्य चरण**:
- OpenLDAP स्थापित करें।
- व्यवस्थापक पासवर्ड कॉन्फ़िगर करें।
- बुनियादी स्कीमा आयात करें।
- LDAP DB पर डोमेन नाम सेट करें।
- LDAP TLS कॉन्फ़िगर करें।
3. **LDAP सेवा निष्पादन**: एक बार सेटअप हो जाने के बाद, LDAP सेवा को निम्नलिखित का उपयोग करके चलाया जा सकता है:
```bash
slapd -d 2
```
## संदर्भ
* [https://grimhacker.com/2018/03/09/just-a-printer/](https://grimhacker.com/2018/03/09/just-a-printer/)


{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
