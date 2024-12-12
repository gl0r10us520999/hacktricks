<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

उन कमजोरियों को खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी अटैक सरफेस को ट्रैक करता है, प्रोएक्टिव थ्रेट स्कैन चलाता है, और आपके पूरे टेक स्टैक में मुद्दों को खोजता है, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक। आज ही [**मुफ्त में इसे आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks).

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

# Invoke
```text
powershell -ep bypass
. .\powerup.ps
Invoke-AllChecks
```
# जांचें

_03/2019_

* [x] वर्तमान विशेषाधिकार
* [x] अनुद्धृत सेवा पथ
* [x] सेवा निष्पादन योग्य अनुमतियाँ
* [x] सेवा अनुमतियाँ
* [x] %PATH% हाइजैक करने योग्य DLL स्थानों के लिए
* [x] AlwaysInstallElevated रजिस्ट्री कुंजी
* [x] रजिस्ट्री में Autologon क्रेडेंशियल्स
* [x] संशोधन योग्य रजिस्ट्री autoruns और कॉन्फ़िग
* [x] संशोधन योग्य schtask फ़ाइलें/कॉन्फ़िग
* [x] अनटेंडेड इंस्टॉल फ़ाइलें
* [x] एन्क्रिप्टेड web.config स्ट्रिंग्स
* [x] एन्क्रिप्टेड एप्लिकेशन पूल और वर्चुअल डायरेक्टरी पासवर्ड
* [x] McAfee SiteList.xml में प्लेनटेक्स्ट पासवर्ड
* [x] कैश्ड ग्रुप पॉलिसी प्रेफरेंसेज .xml फ़ाइलें

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण कमजोरियों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी अटैक सरफेस को ट्रैक करता है, प्रोएक्टिव थ्रेट स्कैन चलाता है, और आपके पूरे टेक स्टैक में मुद्दों को ढूंढता है, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक। आज ही [**मुफ्त में इसे आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
