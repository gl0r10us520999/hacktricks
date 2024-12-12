# Pickle Rick

## Pickle Rick

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो बनाने के साथ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

![](../../.gitbook/assets/picklerick.gif)

यह मशीन आसान श्रेणी में थी और यह काफी आसान थी।

## Enumeration

मैंने **मेरे उपकरण Legion का उपयोग करके मशीन का जाँच करना शुरू किया**:

![](<../../.gitbook/assets/image (79) (2).png>)

जैसा कि आप देख सकते हैं, 2 पोर्ट खुले हैं: 80 (**HTTP**) और 22 (**SSH**)

इसलिए, मैंने HTTP सेवा का जाँच करने के लिए legion चालू किया:

![](<../../.gitbook/assets/image (234).png>)

छवि में देखें कि `robots.txt` में `Wubbalubbadubdub` स्ट्रिंग है

कुछ सेकंड के बाद मैंने देखा कि `disearch` ने पहले से क्या खोज लिया है:

![](<../../.gitbook/assets/image (235).png>)

![](<../../.gitbook/assets/image (236).png>)

और जैसा कि आप आखिरी छवि में देख सकते हैं, एक **लॉगिन** पृष्ठ खोजा गया था।

मूल पृष्ठ के स्रोत को जाँचते समय, एक उपयोगकर्ता खोजा गया: `R1ckRul3s`

![](<../../.gitbook/assets/image (237) (1).png>)

इसलिए, आप `R1ckRul3s:Wubbalubbadubdub` क्रेडेंशियल का उपयोग करके लॉगिन कर सकते हैं लॉगिन पृष्ठ पर

## User

इन क्रेडेंशियल का उपयोग करके आप एक पोर्टल तक पहुँचेंगे जहाँ आप कमांड चला सकते हैं:

![](<../../.gitbook/assets/image (241).png>)

कुछ कमांड जैसे cat अनुमति नहीं हैं लेकिन आप grep का उपयोग करके पहला इंग्रीडिएंट (फ्लैग) पढ़ सकते हैं:

![](<../../.gitbook/assets/image (242).png>)

फिर मैंने इस्तेमाल किया:

![](<../../.gitbook/assets/image (243) (1).png>)

एक रिवर्स शैल जानने के लिए:

![](<../../.gitbook/assets/image (239) (1).png>)

**दूसरा इंग्रीडिएंट** `/home/rick` में पाया जा सकता है

![](<../../.gitbook/assets/image (240).png>)

## Root

उपयोगकर्ता **www-data sudo के रूप में कुछ भी चला सकता है**:

![](<../../.gitbook/assets/image (238).png>)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो बनाने के साथ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
