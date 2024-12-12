# Detecting Phishing

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

## Introduction

फिशिंग प्रयास का पता लगाने के लिए यह महत्वपूर्ण है कि **आजकल उपयोग की जा रही फिशिंग तकनीकों को समझें**। इस पोस्ट के मुख्य पृष्ठ पर, आप यह जानकारी पा सकते हैं, इसलिए यदि आप नहीं जानते कि आज कौन सी तकनीकें उपयोग की जा रही हैं, तो मैं आपको मुख्य पृष्ठ पर जाने और कम से कम उस अनुभाग को पढ़ने की सिफारिश करता हूं।

यह पोस्ट इस विचार पर आधारित है कि **हमलावर किसी न किसी तरह से पीड़ित के डोमेन नाम की नकल करने या उसका उपयोग करने की कोशिश करेंगे**। यदि आपका डोमेन `example.com` है और आपको किसी कारण से पूरी तरह से अलग डोमेन नाम जैसे `youwonthelottery.com` का उपयोग करके फिश किया गया है, तो ये तकनीकें इसे उजागर नहीं करेंगी।

## Domain name variations

ईमेल के अंदर **समान डोमेन** नाम का उपयोग करने वाले उन **फिशिंग** प्रयासों को **खोलना** काफी **आसान** है।\
यह **हमलावर द्वारा उपयोग किए जाने वाले सबसे संभावित फिशिंग नामों की एक सूची बनाने के लिए पर्याप्त है** और **जांचें** कि क्या यह **पंजीकृत** है या बस यह जांचें कि क्या इसका कोई **IP** है।

### Finding suspicious domains

इसके लिए, आप निम्नलिखित उपकरणों में से किसी का उपयोग कर सकते हैं। ध्यान दें कि ये उपकरण स्वचालित रूप से DNS अनुरोध भी करेंगे यह जांचने के लिए कि क्या डोमेन के लिए कोई IP असाइन किया गया है:

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

### Bitflipping

**आप इस तकनीक का संक्षिप्त विवरण मुख्य पृष्ठ पर पा सकते हैं। या पढ़ें मूल शोध में** [**https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/**](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

उदाहरण के लिए, डोमेन microsoft.com में 1 बिट संशोधन इसे _windnws.com_ में बदल सकता है।\
**हमलावर पीड़ित से संबंधित जितने संभव हो सके बिट-फ्लिपिंग डोमेन पंजीकृत कर सकते हैं ताकि वैध उपयोगकर्ताओं को अपनी अवसंरचना की ओर पुनर्निर्देशित किया जा सके**।

**सभी संभावित बिट-फ्लिपिंग डोमेन नामों की भी निगरानी की जानी चाहिए।**

### Basic checks

एक बार जब आपके पास संभावित संदिग्ध डोमेन नामों की एक सूची हो, तो आपको उन्हें **जांचना चाहिए** (मुख्य रूप से HTTP और HTTPS पोर्ट) यह देखने के लिए कि क्या वे किसी पीड़ित के डोमेन के समान लॉगिन फॉर्म का उपयोग कर रहे हैं।\
आप पोर्ट 3333 की भी जांच कर सकते हैं कि क्या यह खुला है और `gophish` का एक उदाहरण चला रहा है।\
यह जानना भी दिलचस्प है कि **प्रत्येक खोजे गए संदिग्ध डोमेन की उम्र कितनी है**, जितना नया होगा उतना ही जोखिम भरा होगा।\
आप संदिग्ध वेब पृष्ठ के HTTP और/या HTTPS के **स्क्रीनशॉट** भी ले सकते हैं यह देखने के लिए कि क्या यह संदिग्ध है और इस मामले में **गहराई से देखने के लिए इसे एक्सेस करें**।

### Advanced checks

यदि आप एक कदम आगे बढ़ना चाहते हैं, तो मैं आपको **उन संदिग्ध डोमेन की निगरानी करने और समय-समय पर अधिक खोजने** की सिफारिश करूंगा (हर दिन? इसमें केवल कुछ सेकंड/मिनट लगते हैं)। आपको संबंधित IPs के खुले **पोर्ट** की भी **जांच करनी चाहिए** और **`gophish` या समान उपकरणों के उदाहरणों की खोज करनी चाहिए** (हाँ, हमलावर भी गलतियाँ करते हैं) और **संदिग्ध डोमेन और उपडोमेन के HTTP और HTTPS वेब पृष्ठों की निगरानी करनी चाहिए** यह देखने के लिए कि क्या उन्होंने पीड़ित के वेब पृष्ठों से कोई लॉगिन फॉर्म कॉपी किया है।\
इसको **स्वचालित करने** के लिए, मैं आपको पीड़ित के डोमेन के लॉगिन फॉर्म की एक सूची रखने, संदिग्ध वेब पृष्ठों को स्पाइडर करने और संदिग्ध डोमेन के अंदर पाए गए प्रत्येक लॉगिन फॉर्म की तुलना पीड़ित के डोमेन के प्रत्येक लॉगिन फॉर्म के साथ करने की सिफारिश करूंगा, जैसे कि `ssdeep` का उपयोग करना।\
यदि आपने संदिग्ध डोमेन के लॉगिन फॉर्म का पता लगा लिया है, तो आप **जंक क्रेडेंशियल्स भेजने** और **जांचने** की कोशिश कर सकते हैं कि क्या यह आपको पीड़ित के डोमेन पर पुनर्निर्देशित कर रहा है।

## Domain names using keywords

मुख्य पृष्ठ पर एक डोमेन नाम भिन्नता तकनीक का भी उल्लेख किया गया है जिसमें **पीड़ित के डोमेन नाम को एक बड़े डोमेन के अंदर रखा जाता है** (जैसे paypal-financial.com के लिए paypal.com)।

### Certificate Transparency

पिछले "ब्रूट-फोर्स" दृष्टिकोण को लेना संभव नहीं है, लेकिन वास्तव में **ऐसे फिशिंग प्रयासों को उजागर करना संभव है** जो प्रमाणपत्र पारदर्शिता के लिए भी धन्यवाद। हर बार जब एक CA द्वारा एक प्रमाणपत्र जारी किया जाता है, तो विवरण सार्वजनिक किए जाते हैं। इसका मतलब है कि प्रमाणपत्र पारदर्शिता को पढ़कर या यहां तक कि इसकी निगरानी करके, यह **संभव है कि ऐसे डोमेन खोजें जो अपने नाम के अंदर एक कीवर्ड का उपयोग कर रहे हैं**। उदाहरण के लिए, यदि एक हमलावर [https://paypal-financial.com](https://paypal-financial.com) का एक प्रमाणपत्र उत्पन्न करता है, तो प्रमाणपत्र को देखकर "paypal" कीवर्ड को ढूंढना और यह जानना संभव है कि संदिग्ध ईमेल का उपयोग किया जा रहा है।

पोस्ट [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/) का सुझाव है कि आप Censys का उपयोग करके एक विशिष्ट कीवर्ड को प्रभावित करने वाले प्रमाणपत्रों की खोज कर सकते हैं और तिथि (केवल "नए" प्रमाणपत्र) और CA जारीकर्ता "Let's Encrypt" द्वारा फ़िल्टर कर सकते हैं:

![https://0xpatrik.com/content/images/2018/07/cert\_listing.png](<../../.gitbook/assets/image (1115).png>)

हालांकि, आप मुफ्त वेब [**crt.sh**](https://crt.sh) का उपयोग करके "वही" कर सकते हैं। आप **कीवर्ड** के लिए **खोज** कर सकते हैं और यदि आप चाहें तो **तिथि और CA** द्वारा परिणामों को **फ़िल्टर** कर सकते हैं।

![](<../../.gitbook/assets/image (519).png>)

इस अंतिम विकल्प का उपयोग करके, आप यहां तक कि फ़ील्ड मिलान पहचानियों का उपयोग कर सकते हैं यह देखने के लिए कि क्या वास्तविक डोमेन से कोई पहचान संदिग्ध डोमेन में मेल खाती है (ध्यान दें कि एक संदिग्ध डोमेन एक झूठी सकारात्मक हो सकती है)।

**एक और विकल्प** शानदार परियोजना है जिसे [**CertStream**](https://medium.com/cali-dog-security/introducing-certstream-3fc13bb98067) कहा जाता है। CertStream नए उत्पन्न प्रमाणपत्रों का एक वास्तविक समय का प्रवाह प्रदान करता है जिसका उपयोग आप (नज़दीकी) वास्तविक समय में निर्दिष्ट कीवर्ड का पता लगाने के लिए कर सकते हैं। वास्तव में, एक परियोजना है जिसे [**phishing\_catcher**](https://github.com/x0rz/phishing\_catcher) कहा जाता है जो यही करती है।

### **New domains**

**एक अंतिम विकल्प** कुछ TLDs के लिए **नए पंजीकृत डोमेन** की एक सूची एकत्र करना है ([Whoxy](https://www.whoxy.com/newly-registered-domains/) ऐसी सेवा प्रदान करता है) और **इन डोमेन में कीवर्ड की जांच करना**। हालांकि, लंबे डोमेन आमतौर पर एक या अधिक उपडोमेन का उपयोग करते हैं, इसलिए कीवर्ड FLD के अंदर नहीं दिखाई देगा और आप फिशिंग उपडोमेन नहीं ढूंढ पाएंगे।

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
