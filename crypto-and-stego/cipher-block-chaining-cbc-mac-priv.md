{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}


# CBC

अगर **कुकी** **केवल** **उपयोगकर्ता नाम** है (या कुकी का पहला हिस्सा उपयोगकर्ता नाम है) और आप उपयोगकर्ता "**एडमिन**" का अनुकरण करना चाहते हैं। तो, आप उपयोगकर्ता नाम **"बीडमिन"** बना सकते हैं और कुकी के **पहले बाइट** को **ब्रूटफोर्स** कर सकते हैं।

# CBC-MAC

**साइफर ब्लॉक चेनिंग मैसेज ऑथेंटिकेशन कोड** (**CBC-MAC**) एक विधि है जो गुप्तशास्त्र में प्रयोग की जाती है। यह एक संदेश लेकर उसे ब्लॉक द्वारा एन्क्रिप्ट करने के द्वारा काम करता है, जहां प्रत्येक ब्लॉक की एन्क्रिप्शन पिछले ब्लॉक से जुड़ी होती है। यह प्रक्रिया एक **ब्लॉकों की श्रृंखला** बनाती है, जिससे सुनिश्चित होता है कि मूल संदेश के एक भी बिट को बदलने से एन्क्रिप्ट किए गए डेटा के अंतिम ब्लॉक में एक अप्रत्याशित परिवर्तन होगा। इस परिवर्तन को बनाने या उलटाने के लिए, एन्क्रिप्शन कुंजी की आवश्यकता होती है, जो सुरक्षा सुनिश्चित करता है।

संदेश m का CBC-MAC निर्धारित करने के लिए, किसी भी शून्य प्रारंभीक वेक्टर के साथ m को CBC मोड में एन्क्रिप्ट किया जाता है और अंतिम ब्लॉक को रखा जाता है। निम्न चित्र में एक गुप्त कुंजी k और एक ब्लॉक साइफर E का उपयोग करके संदेश का CBC-MAC की गणना की गई है![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5):

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# सुरक्षा दोष

CBC-MAC के साथ आम तौर पर **इस्तेमाल किया जाने वाला IV 0 होता है**।\
यह एक समस्या है क्योंकि 2 ज्ञात संदेश (`m1` और `m2`) स्वतंत्र रूप से 2 हस्ताक्षर (`s1` और `s2`) उत्पन्न करेंगे। इसलिए:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

फिर m1 और m2 को जोड़कर बनाया गया संदेश (m3) 2 हस्ताक्षर (s31 और s32) उत्पन्न करेगा:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**जिसे एन्क्रिप्शन की कुंजी को नहीं जानते हुए भी गणना की जा सकती है।**

सोचिए आप **8 बाइट** ब्लॉक में नाम **एडमिनिस्ट्रेटर** को एन्क्रिप्ट कर रहे हैं:

* `एडमिनिस्ट`
* `रेटर\00\00\00`

आप उपयोगकर्ता नाम **एडमिनिस्ट** (m1) का हस्ताक्षर (s1) प्राप्त कर सकते हैं।\
फिर, आप `रेटर\00\00\00 XOR s1` का परिणाम वाला उपयोगकर्ता नाम बना सकते हैं। यह `E(m2 XOR s1 XOR 0)` उत्पन्न करेगा जो s32 है।\
अब, आप s32 का उपयोग करके पूरे नाम **एडमिनिस्ट्रेटर** का हस्ताक्षर उपयोग कर सकते हैं।

### सारांश

1. उपयोगकर्ता नाम **एडमिनिस्ट** (m1) का हस्ताक्षर प्राप्त करें जो s1 है
2. उपयोगकर्ता नाम **रेटर\x00\x00\x00 XOR s1 XOR 0** का हस्ताक्षर प्राप्त करें जो s32 है**।**
3. कुकी को s32 पर सेट करें और यह उपयोगकर्ता **एडमिनिस्ट्रेटर** के लिए एक मान्य कुकी होगी।

# हमला नियंत्रण IV

यदि आप उपयोग किए गए IV को नियंत्रित कर सकते हैं तो हमला बहुत आसान हो सकता है।\
यदि कुकी केवल उपयोगकर्ता नाम एन्क्रिप्ट किया गया है, तो उपयोगकर्ता "**एडमिनिस्ट्रेटर**" का अनुकरण करने के लिए आप उपयोगकर्ता "**एडमिनिस्ट्रेटर**" बना सकते हैं और आप इसकी कुकी प्राप्त करेंगे।\
अब, यदि आप IV को नियंत्रित कर सकते हैं, तो आप IV का पहला बाइट बदल सकते हैं ताकि **IV\[0] XOR "A" == IV'\[0] XOR "a"** और उपयोगकर्ता **एडमिनिस्ट्रेटर** के लिए कुकी पुनः उत्पन्न कर सकें। यह कुकी **उपयोगकर्ता एडमिनिस्ट्रेटर** का अनुकरण करने के लिए मान्य होगी।

## संदर्भ

अधिक जानकारी [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
