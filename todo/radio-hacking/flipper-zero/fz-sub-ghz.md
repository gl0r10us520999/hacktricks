# FZ - Sub-GHz

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमारा अनुसरण करें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}


## परिचय <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero **300-928 MHz** की रेंज में रेडियो फ्रीक्वेंसी प्राप्त और प्रसारित कर सकता है, जिसमें इसका अंतर्निहित मॉड्यूल है, जो रिमोट कंट्रोल को पढ़ने, सहेजने और अनुकरण करने में सक्षम है। ये नियंत्रण गेट, बैरियर्स, रेडियो ताले, रिमोट कंट्रोल स्विच, वायरलेस डोरबेल, स्मार्ट लाइट और अधिक के साथ इंटरैक्शन के लिए उपयोग किए जाते हैं। Flipper Zero आपको यह जानने में मदद कर सकता है कि आपकी सुरक्षा से समझौता किया गया है या नहीं।

<figure><img src="../../../.gitbook/assets/image (714).png" alt=""><figcaption></figcaption></figure>

## Sub-GHz हार्डवेयर <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero में एक अंतर्निहित सब-1 GHz मॉड्यूल है जो [﻿](https://www.st.com/en/nfc/st25r3916.html#overview)﻿[CC1101 चिप](https://www.ti.com/lit/ds/symlink/cc1101.pdf) और एक रेडियो एंटीना (अधिकतम रेंज 50 मीटर है) पर आधारित है। CC1101 चिप और एंटीना दोनों को 300-348 MHz, 387-464 MHz, और 779-928 MHz बैंड में फ्रीक्वेंसी पर काम करने के लिए डिज़ाइन किया गया है।

<figure><img src="../../../.gitbook/assets/image (923).png" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

### फ्रीक्वेंसी एनालाइज़र

{% hint style="info" %}
कैसे पता करें कि कौन सी फ्रीक्वेंसी रिमोट का उपयोग कर रही है
{% endhint %}

विश्लेषण करते समय, Flipper Zero सभी उपलब्ध फ्रीक्वेंसियों पर सिग्नल स्ट्रेंथ (RSSI) को स्कैन कर रहा है। Flipper Zero सबसे उच्च RSSI मान के साथ फ्रीक्वेंसी प्रदर्शित करता है, जिसमें सिग्नल स्ट्रेंथ -90 [dBm](https://en.wikipedia.org/wiki/DBm) से अधिक है।

रिमोट की फ्रीक्वेंसी निर्धारित करने के लिए, निम्नलिखित करें:

1. रिमोट कंट्रोल को Flipper Zero के बाईं ओर बहुत करीब रखें।
2. **मुख्य मेनू** **→ Sub-GHz** पर जाएं।
3. **फ्रीक्वेंसी एनालाइज़र** का चयन करें, फिर उस रिमोट कंट्रोल पर बटन दबाएं और दबाए रखें जिसे आप विश्लेषित करना चाहते हैं।
4. स्क्रीन पर फ्रीक्वेंसी मान की समीक्षा करें।

### पढ़ें

{% hint style="info" %}
उपयोग की गई फ्रीक्वेंसी के बारे में जानकारी प्राप्त करें (कौन सी फ्रीक्वेंसी का उपयोग किया जा रहा है यह जानने का एक और तरीका)
{% endhint %}

**पढ़ें** विकल्प **निर्धारित मॉड्यूलेशन पर कॉन्फ़िगर की गई फ्रीक्वेंसी पर सुनता है**: डिफ़ॉल्ट रूप से 433.92 AM। यदि पढ़ते समय **कुछ पाया जाता है**, तो **स्क्रीन पर जानकारी दी जाती है**। इस जानकारी का उपयोग भविष्य में सिग्नल को दोहराने के लिए किया जा सकता है।

जब पढ़ना उपयोग में है, तो **बाईं बटन** को दबाना और **कॉन्फ़िगर करना** संभव है।\
इस समय इसमें **4 मॉड्यूलेशन** (AM270, AM650, FM328 और FM476) और **कई प्रासंगिक फ्रीक्वेंसियाँ** संग्रहीत हैं:

<figure><img src="../../../.gitbook/assets/image (947).png" alt=""><figcaption></figcaption></figure>

आप **कोई भी सेट कर सकते हैं जो आपको रुचिकर लगे**, हालाँकि, यदि आप **निश्चित नहीं हैं कि कौन सी फ्रीक्वेंसी** रिमोट द्वारा उपयोग की जा रही है, तो **हॉपिंग को ON सेट करें** (डिफ़ॉल्ट रूप से OFF), और बटन को कई बार दबाएं जब तक Flipper इसे कैप्चर न कर ले और आपको फ्रीक्वेंसी सेट करने के लिए आवश्यक जानकारी न दे।

{% hint style="danger" %}
फ्रीक्वेंसियों के बीच स्विच करने में कुछ समय लगता है, इसलिए स्विचिंग के समय प्रसारित सिग्नल छूट सकते हैं। बेहतर सिग्नल रिसेप्शन के लिए, फ्रीक्वेंसी एनालाइज़र द्वारा निर्धारित एक निश्चित फ्रीक्वेंसी सेट करें।
{% endhint %}

### **कच्चा पढ़ें**

{% hint style="info" %}
कॉन्फ़िगर की गई फ्रीक्वेंसी में सिग्नल चुराएं (और दोहराएं)
{% endhint %}

**कच्चा पढ़ें** विकल्प **सिग्नल रिकॉर्ड करता है** जो सुनने की फ्रीक्वेंसी में भेजे जाते हैं। इसका उपयोग **सिग्नल चुराने** और **दोहराने** के लिए किया जा सकता है।

डिफ़ॉल्ट रूप से **कच्चा पढ़ें भी 433.92 AM650 में है**, लेकिन यदि पढ़ने के विकल्प के साथ आपने पाया कि जो सिग्नल आपको रुचिकर है वह **विभिन्न फ्रीक्वेंसी/मॉड्यूलेशन में है, तो आप उसे भी संशोधित कर सकते हैं** बाईं ओर दबाकर (जब कच्चा पढ़ने के विकल्प के अंदर हों)।

### ब्रूट-फोर्स

यदि आप जानते हैं कि उदाहरण के लिए गैरेज दरवाजे के लिए कौन सा प्रोटोकॉल उपयोग किया गया है, तो आप **सभी कोड उत्पन्न कर सकते हैं और उन्हें Flipper Zero के साथ भेज सकते हैं।** यह सामान्य प्रकार के गैरेज का समर्थन करने का एक उदाहरण है: [**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)

### मैन्युअल रूप से जोड़ें

{% hint style="info" %}
कॉन्फ़िगर की गई प्रोटोकॉल की सूची से सिग्नल जोड़ें
{% endhint %}

#### [समर्थित प्रोटोकॉल](https://docs.flipperzero.one/sub-ghz/add-new-remote) की सूची <a href="#id-3iglu" id="id-3iglu"></a>

| Princeton\_433 (स्थिर कोड सिस्टम के अधिकांश के साथ काम करता है) | 433.92 | स्थिर  |
| --------------------------------------------------------------- | ------ | ------- |
| Nice Flo 12bit\_433                                             | 433.92 | स्थिर  |
| Nice Flo 24bit\_433                                             | 433.92 | स्थिर  |
| CAME 12bit\_433                                                 | 433.92 | स्थिर  |
| CAME 24bit\_433                                                 | 433.92 | स्थिर  |
| Linear\_300                                                     | 300.00 | स्थिर  |
| CAME TWEE                                                       | 433.92 | स्थिर  |
| Gate TX\_433                                                    | 433.92 | स्थिर  |
| DoorHan\_315                                                    | 315.00 | गतिशील |
| DoorHan\_433                                                    | 433.92 | गतिशील |
| LiftMaster\_315                                                 | 315.00 | गतिशील |
| LiftMaster\_390                                                 | 390.00 | गतिशील |
| Security+2.0\_310                                               | 310.00 | गतिशील |
| Security+2.0\_315                                               | 315.00 | गतिशील |
| Security+2.0\_390                                               | 390.00 | गतिशील |

### समर्थित Sub-GHz विक्रेता

सूची की जांच करें [https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors)

### क्षेत्र द्वारा समर्थित फ्रीक्वेंसियाँ

सूची की जांच करें [https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies)

### परीक्षण

{% hint style="info" %}
सहेजी गई फ्रीक्वेंसियों के dBms प्राप्त करें
{% endhint %}

## संदर्भ

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमारा अनुसरण करें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
