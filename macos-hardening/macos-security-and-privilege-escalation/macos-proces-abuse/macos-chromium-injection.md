# macOS Chromium Injection

{% hint style="success" %}
**AWS हैकिंग सीखें और अभ्यास करें:** [**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)\
**GCP हैकिंग सीखें और अभ्यास करें:** [**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 पर हमें **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## मूल जानकारी

Google Chrome, Microsoft Edge, Brave और अन्य च्रोमियम-आधारित ब्राउज़र। ये ब्राउज़र च्रोमियम मुक्त स्रोत परियोजना पर निर्मित हैं, जिसका मतलब है कि उनके पास एक समान आधार है और इसलिए उनमें समान कार्यक्षमताएँ और डेवलपर विकल्प होते हैं।

#### `--load-extension` ध्वज

`--load-extension` ध्वज का उपयोग जब चालू किया जाता है तो एक च्रोमियम-आधारित ब्राउज़र को कमांड लाइन या स्क्रिप्ट से शुरू करने के लिए किया जाता है। यह ध्वज **ब्राउज़र में एक या एक से अधिक एक्सटेंशन को स्वचालित रूप से लोड करने की अनुमति देता है**।

#### `--use-fake-ui-for-media-stream` ध्वज

`--use-fake-ui-for-media-stream` ध्वज एक और कमांड लाइन विकल्प है जो चालू करने के लिए च्रोमियम-आधारित ब्राउज़र का उपयोग किया जा सकता है। यह ध्वज **सामान्य उपयोगकर्ता प्रॉम्प्ट को छलने के लिए डिज़ाइन किया गया है जो कैमरा और माइक्रोफोन से मीडिया स्ट्रीम तक पहुँचने की अनुमति के लिए पूछता है**। जब यह ध्वज प्रयोग किया जाता है, तो ब्राउज़र स्वचालित रूप से किसी भी वेबसाइट या एप्लिकेशन को अनुमति प्रदान करता है जो कैमरा या माइक्रोफोन तक पहुँचने का अनुरोध करता है।

### उपकरण

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### उदाहरण
```bash
# Intercept traffic
voodoo intercept -b chrome
```
## संदर्भ

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 पर हमें **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
