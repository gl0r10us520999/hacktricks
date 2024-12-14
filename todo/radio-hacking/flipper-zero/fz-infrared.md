# FZ - इन्फ्रारेड

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

## परिचय <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

इन्फ्रारेड कैसे काम करता है, इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../infrared.md" %}
[infrared.md](../infrared.md)
{% endcontent-ref %}

## Flipper Zero में IR सिग्नल रिसीवर <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Flipper एक डिजिटल IR सिग्नल रिसीवर TSOP का उपयोग करता है, जो **IR रिमोट से सिग्नल इंटरसेप्ट करने की अनुमति देता है**। कुछ **स्मार्टफोन** जैसे Xiaomi में भी IR पोर्ट होता है, लेकिन ध्यान रखें कि **उनमें से अधिकांश केवल सिग्नल भेज सकते हैं** और **उन्हें प्राप्त नहीं कर सकते**।

Flipper का इन्फ्रारेड **रिसीवर काफी संवेदनशील है**। आप **सिग्नल पकड़ सकते हैं** जबकि आप **रिमोट और टीवी के बीच कहीं** हैं। रिमोट को सीधे Flipper के IR पोर्ट की ओर इंगित करना आवश्यक नहीं है। यह तब सहायक होता है जब कोई टीवी के पास खड़े होकर चैनल बदल रहा हो, और आप और Flipper दोनों कुछ दूरी पर हों।

चूंकि **इन्फ्रारेड** सिग्नल का **डिकोडिंग** **सॉफ़्टवेयर** पक्ष पर होता है, Flipper Zero संभावित रूप से **किसी भी IR रिमोट कोड का रिसेप्शन और ट्रांसमिशन** का समर्थन करता है। **अज्ञात** प्रोटोकॉल के मामले में जिन्हें पहचाना नहीं जा सका - यह **कच्चे सिग्नल को रिकॉर्ड और प्ले बैक** करता है जैसे कि इसे प्राप्त किया गया था।

## क्रियाएँ

### यूनिवर्सल रिमोट

Flipper Zero को **किसी भी टीवी, एयर कंडीशनर, या मीडिया सेंटर को नियंत्रित करने के लिए एक यूनिवर्सल रिमोट के रूप में उपयोग किया जा सकता है**। इस मोड में, Flipper **सभी समर्थित निर्माताओं के सभी **ज्ञात कोड** को **SD कार्ड से शब्दकोश के अनुसार** ब्रूटफोर्स करता है। आपको किसी विशेष रिमोट को बंद करने के लिए नहीं चुनना है।

यूनिवर्सल रिमोट मोड में पावर बटन दबाना काफी है, और Flipper **सभी टीवी के "पावर ऑफ"** कमांड को क्रमबद्ध रूप से भेजेगा जो इसे पता है: Sony, Samsung, Panasonic... और इसी तरह। जब टीवी अपने सिग्नल को प्राप्त करता है, तो यह प्रतिक्रिया देगा और बंद हो जाएगा।

इस तरह का ब्रूट-फोर्स समय लेता है। जितना बड़ा शब्दकोश होगा, इसे पूरा करने में उतना ही अधिक समय लगेगा। यह पता लगाना असंभव है कि टीवी ने किस सिग्नल को ठीक से पहचाना क्योंकि टीवी से कोई फीडबैक नहीं होता।

### नया रिमोट सीखें

Flipper Zero के साथ **इन्फ्रारेड सिग्नल कैप्चर करना** संभव है। यदि यह **डेटाबेस में सिग्नल खोजता है** तो Flipper स्वचालित रूप से **जान जाएगा कि यह कौन सा डिवाइस है** और आपको इसके साथ इंटरैक्ट करने देगा।\
यदि नहीं, तो Flipper **सिग्नल को स्टोर** कर सकता है और आपको इसे **प्ले बैक** करने की अनुमति देगा।

## संदर्भ

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

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
