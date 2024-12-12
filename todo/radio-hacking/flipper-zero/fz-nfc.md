# FZ - NFC

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

## परिचय <a href="#id-9wrzi" id="id-9wrzi"></a>

RFID और NFC के बारे में जानकारी के लिए निम्नलिखित पृष्ठ देखें:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## समर्थित NFC कार्ड <a href="#id-9wrzi" id="id-9wrzi"></a>

{% hint style="danger" %}
NFC कार्ड के अलावा, Flipper Zero **अन्य प्रकार के उच्च-आवृत्ति कार्ड** जैसे कई **Mifare** क्लासिक और अल्ट्रालाइट और **NTAG** का समर्थन करता है।
{% endhint %}

NFC कार्ड के नए प्रकारों को समर्थित कार्डों की सूची में जोड़ा जाएगा। Flipper Zero निम्नलिखित **NFC कार्ड प्रकार A** (ISO 14443A) का समर्थन करता है:

* ﻿**बैंक कार्ड (EMV)** — केवल UID, SAK, और ATQA पढ़ें, बिना सहेजे।
* ﻿**अज्ञात कार्ड** — (UID, SAK, ATQA) पढ़ें और एक UID का अनुकरण करें।

**NFC कार्ड प्रकार B, प्रकार F, और प्रकार V** के लिए, Flipper Zero UID को बिना सहेजे पढ़ने में सक्षम है।

### NFC कार्ड प्रकार A <a href="#uvusf" id="uvusf"></a>

#### बैंक कार्ड (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero केवल UID, SAK, ATQA, और बैंक कार्ड पर संग्रहीत डेटा को **बिना सहेजे** पढ़ सकता है।

बैंक कार्ड पढ़ने का स्क्रीनFlipper Zero केवल डेटा को **बिना सहेजे और अनुकरण किए** पढ़ सकता है।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### अज्ञात कार्ड <a href="#id-37eo8" id="id-37eo8"></a>

जब Flipper Zero **NFC कार्ड के प्रकार का निर्धारण करने में असमर्थ होता है**, तब केवल **UID, SAK, और ATQA** को **पढ़ा और सहेजा** जा सकता है।

अज्ञात कार्ड पढ़ने का स्क्रीनFlipper Zero केवल एक UID का अनुकरण कर सकता है।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC कार्ड प्रकार B, F, और V <a href="#wyg51" id="wyg51"></a>

**NFC कार्ड प्रकार B, F, और V** के लिए, Flipper Zero केवल **UID को पढ़ और प्रदर्शित** कर सकता है, बिना सहेजे।

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

NFC के बारे में एक परिचय के लिए [**इस पृष्ठ को पढ़ें**](../pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz).

### पढ़ें

Flipper Zero **NFC कार्ड पढ़ सकता है**, हालाँकि, यह **ISO 14443 पर आधारित सभी प्रोटोकॉल को नहीं समझता**। हालाँकि, चूंकि **UID एक निम्न-स्तरीय विशेषता है**, आप एक ऐसी स्थिति में हो सकते हैं जब **UID पहले से पढ़ा गया हो, लेकिन उच्च-स्तरीय डेटा ट्रांसफर प्रोटोकॉल अभी भी अज्ञात हो**। आप UID को पढ़ सकते हैं, अनुकरण कर सकते हैं और Flipper का उपयोग करके UID को मैन्युअल रूप से इनपुट कर सकते हैं जो UID का उपयोग करने वाले प्राथमिक रीडर्स के लिए अधिकृत है।

#### UID पढ़ना बनाम अंदर का डेटा पढ़ना <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

Flipper में, 13.56 MHz टैग को दो भागों में विभाजित किया जा सकता है:

* **निम्न-स्तरीय पढ़ाई** — केवल UID, SAK, और ATQA पढ़ता है। Flipper इस डेटा के आधार पर उच्च-स्तरीय प्रोटोकॉल का अनुमान लगाने की कोशिश करता है जो कार्ड से पढ़ा गया है। आप इसके साथ 100% निश्चित नहीं हो सकते, क्योंकि यह कुछ कारकों के आधार पर केवल एक अनुमान है।
* **उच्च-स्तरीय पढ़ाई** — एक विशिष्ट उच्च-स्तरीय प्रोटोकॉल का उपयोग करके कार्ड की मेमोरी से डेटा पढ़ता है। यह Mifare Ultralight पर डेटा पढ़ना, Mifare Classic से सेक्टर पढ़ना, या PayPass/Apple Pay से कार्ड के गुण पढ़ना होगा।

### विशिष्ट पढ़ें

यदि Flipper Zero निम्न स्तर के डेटा से कार्ड के प्रकार को खोजने में असमर्थ है, तो `Extra Actions` में आप `Read Specific Card Type` का चयन कर सकते हैं और **मैन्युअल रूप से** **उस कार्ड के प्रकार को इंगित कर सकते हैं जिसे आप पढ़ना चाहते हैं**।

#### EMV बैंक कार्ड (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

UID को केवल पढ़ने के अलावा, आप बैंक कार्ड से और भी बहुत सा डेटा निकाल सकते हैं। यह **पूर्ण कार्ड संख्या** (कार्ड के सामने के 16 अंक), **वैधता तिथि**, और कुछ मामलों में यहां तक कि **स्वामी का नाम** और **हाल के लेनदेन** की सूची प्राप्त करना संभव है।\
हालांकि, आप **इस तरह से CVV नहीं पढ़ सकते** (कार्ड के पीछे के 3 अंक)। इसके अलावा, **बैंक कार्ड पुनरावृत्ति हमलों से सुरक्षित हैं**, इसलिए Flipper के साथ इसे कॉपी करना और फिर इसे किसी चीज़ के लिए भुगतान करने के लिए अनुकरण करने की कोशिश करना काम नहीं करेगा।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
