# FZ - 125kHz RFID

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## परिचय

125kHz टैग कैसे काम करते हैं, इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## क्रियाएँ

इन प्रकार के टैग के बारे में अधिक जानकारी के लिए [**यह परिचय पढ़ें**](../pentesting-rfid.md#low-frequency-rfid-tags-125khz).

### पढ़ें

कार्ड की जानकारी को **पढ़ने** की कोशिश करता है। फिर इसे **अनुकरण** कर सकता है।

{% hint style="warning" %}
ध्यान दें कि कुछ इंटरकॉम कुंजी डुप्लिकेशन से खुद को बचाने के लिए पढ़ने से पहले एक लिखने का आदेश भेजने की कोशिश करते हैं। यदि लिखना सफल होता है, तो उस टैग को नकली माना जाता है। जब Flipper RFID का अनुकरण करता है, तो रीडर के लिए इसे मूल से अलग करना संभव नहीं होता, इसलिए ऐसी समस्याएँ नहीं होती हैं।
{% endhint %}

### मैन्युअल रूप से जोड़ें

आप **फ्लिपर ज़ीरो में डेटा** को मैन्युअल रूप से इंगित करके **नकली कार्ड बना सकते हैं** और फिर इसका अनुकरण कर सकते हैं।

#### कार्ड पर आईडी

कभी-कभी, जब आप एक कार्ड प्राप्त करते हैं, तो आप कार्ड पर आईडी (या उसका भाग) लिखी हुई पाएंगे।

* **EM Marin**

उदाहरण के लिए, इस EM-Marin कार्ड में भौतिक कार्ड में **स्पष्ट रूप से 5 बाइट में से अंतिम 3 पढ़ना संभव है**।\
यदि आप उन्हें कार्ड से नहीं पढ़ सकते हैं, तो अन्य 2 को ब्रूट-फोर्स किया जा सकता है।

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

* **HID**

इसी तरह इस HID कार्ड में केवल 3 बाइट में से 2 बाइट कार्ड में प्रिंट की गई मिलती हैं।

<figure><img src="../../../.gitbook/assets/image (1014).png" alt=""><figcaption></figcaption></figure>

### अनुकरण/लिखें

एक कार्ड को **कॉपी करने** या **मैन्युअल रूप से** आईडी **दाखिल करने** के बाद इसे Flipper Zero के साथ **अनुकरण** करना या इसे एक असली कार्ड में **लिखना** संभव है।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
