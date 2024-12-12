# अन्य संगठनों में उपकरणों को नामांकित करना

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## परिचय

जैसा कि [**पहले टिप्पणी की गई**](./#what-is-mdm-mobile-device-management)**,** किसी उपकरण को एक संगठन में नामांकित करने के लिए **केवल उस संगठन का एक सीरियल नंबर आवश्यक है**। एक बार जब उपकरण नामांकित हो जाता है, तो कई संगठन नए उपकरण पर संवेदनशील डेटा स्थापित करेंगे: प्रमाणपत्र, अनुप्रयोग, वाईफाई पासवर्ड, वीपीएन कॉन्फ़िगरेशन [और इसी तरह](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf)।\
इसलिए, यदि नामांकन प्रक्रिया को सही तरीके से सुरक्षित नहीं किया गया है, तो यह हमलावरों के लिए एक खतरनाक प्रवेश बिंदु हो सकता है।

**यह अनुसंधान का एक सारांश है [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)। आगे की तकनीकी जानकारी के लिए इसे देखें!**

## DEP और MDM बाइनरी विश्लेषण का अवलोकन

यह अनुसंधान macOS पर डिवाइस नामांकन कार्यक्रम (DEP) और मोबाइल डिवाइस प्रबंधन (MDM) से संबंधित बाइनरी में गहराई से जाता है। मुख्य घटक शामिल हैं:

- **`mdmclient`**: MDM सर्वरों के साथ संवाद करता है और macOS संस्करण 10.13.4 से पहले DEP चेक-इन को ट्रिगर करता है।
- **`profiles`**: कॉन्फ़िगरेशन प्रोफाइल का प्रबंधन करता है, और macOS संस्करण 10.13.4 और बाद में DEP चेक-इन को ट्रिगर करता है।
- **`cloudconfigurationd`**: DEP API संचार का प्रबंधन करता है और डिवाइस नामांकन प्रोफाइल को पुनः प्राप्त करता है।

DEP चेक-इन `CPFetchActivationRecord` और `CPGetActivationRecord` कार्यों का उपयोग करता है जो निजी कॉन्फ़िगरेशन प्रोफाइल ढांचे से सक्रियण रिकॉर्ड को प्राप्त करने के लिए हैं, जिसमें `CPFetchActivationRecord` XPC के माध्यम से `cloudconfigurationd` के साथ समन्वय करता है।

## टेस्ला प्रोटोकॉल और एब्सिन्थ स्कीम रिवर्स इंजीनियरिंग

DEP चेक-इन में `cloudconfigurationd` एक एन्क्रिप्टेड, साइन किया हुआ JSON पेलोड _iprofiles.apple.com/macProfile_ पर भेजता है। पेलोड में उपकरण का सीरियल नंबर और क्रिया "RequestProfileConfiguration" शामिल है। उपयोग की जाने वाली एन्क्रिप्शन योजना को आंतरिक रूप से "Absinthe" के रूप में संदर्भित किया जाता है। इस योजना को सुलझाना जटिल है और इसमें कई चरण शामिल हैं, जिसने सक्रियण रिकॉर्ड अनुरोध में मनमाने सीरियल नंबर डालने के वैकल्पिक तरीकों की खोज की।

## DEP अनुरोधों को प्रॉक्सी करना

_iprofiles.apple.com_ पर DEP अनुरोधों को इंटरसेप्ट और संशोधित करने के प्रयासों को चार्ल्स प्रॉक्सी जैसे उपकरणों द्वारा पेलोड एन्क्रिप्शन और SSL/TLS सुरक्षा उपायों ने बाधित किया। हालाँकि, `MCCloudConfigAcceptAnyHTTPSCertificate` कॉन्फ़िगरेशन को सक्षम करने से सर्वर प्रमाणपत्र मान्यता को बायपास करने की अनुमति मिलती है, हालाँकि पेलोड की एन्क्रिप्टेड प्रकृति अभी भी डिक्रिप्शन कुंजी के बिना सीरियल नंबर के संशोधन को रोकती है।

## DEP के साथ इंटरैक्ट करने वाले सिस्टम बाइनरी को इंस्ट्रूमेंट करना

`cloudconfigurationd` जैसे सिस्टम बाइनरी को इंस्ट्रूमेंट करने के लिए macOS पर सिस्टम इंटीग्रिटी प्रोटेक्शन (SIP) को अक्षम करना आवश्यक है। SIP अक्षम होने पर, LLDB जैसे उपकरणों का उपयोग सिस्टम प्रक्रियाओं से जुड़ने और DEP API इंटरैक्शन में उपयोग किए जाने वाले सीरियल नंबर को संभावित रूप से संशोधित करने के लिए किया जा सकता है। यह विधि वरीयता प्राप्त है क्योंकि यह अधिकारों और कोड साइनिंग की जटिलताओं से बचती है।

**बाइनरी इंस्ट्रूमेंटेशन का शोषण:**
`cloudconfigurationd` में JSON सीरियलाइजेशन से पहले DEP अनुरोध पेलोड को संशोधित करना प्रभावी साबित हुआ। प्रक्रिया में शामिल थे:

1. `cloudconfigurationd` से LLDB को संलग्न करना।
2. उस बिंदु को ढूंढना जहां सिस्टम सीरियल नंबर प्राप्त किया जाता है।
3. पेलोड को एन्क्रिप्ट और भेजे जाने से पहले मेमोरी में एक मनमाना सीरियल नंबर इंजेक्ट करना।

इस विधि ने मनमाने सीरियल नंबर के लिए पूर्ण DEP प्रोफाइल प्राप्त करने की अनुमति दी, जो एक संभावित भेद्यता को प्रदर्शित करती है।

### पायथन के साथ इंस्ट्रूमेंटेशन को स्वचालित करना

शोषण प्रक्रिया को LLDB API के साथ पायथन का उपयोग करके स्वचालित किया गया, जिससे मनमाने सीरियल नंबर को प्रोग्रामेटिक रूप से इंजेक्ट करना और संबंधित DEP प्रोफाइल प्राप्त करना संभव हो गया।

### DEP और MDM भेद्यताओं के संभावित प्रभाव

अनुसंधान ने महत्वपूर्ण सुरक्षा चिंताओं को उजागर किया:

1. **जानकारी का खुलासा**: DEP-रजिस्टर्ड सीरियल नंबर प्रदान करके, DEP प्रोफाइल में निहित संवेदनशील संगठनात्मक जानकारी प्राप्त की जा सकती है।
{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
