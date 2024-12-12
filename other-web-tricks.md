# अन्य वेब ट्रिक्स

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**अपने वेब ऐप्स, नेटवर्क और क्लाउड पर एक हैकर का दृष्टिकोण प्राप्त करें**

**महत्वपूर्ण, शोषण योग्य कमजोरियों को खोजें और रिपोर्ट करें जिनका वास्तविक व्यावसायिक प्रभाव है।** हमारे 20+ कस्टम टूल का उपयोग करके हमले की सतह का मानचित्रण करें, सुरक्षा मुद्दों को खोजें जो आपको विशेषाधिकार बढ़ाने की अनुमति देते हैं, और आवश्यक सबूत इकट्ठा करने के लिए स्वचालित शोषण का उपयोग करें, जिससे आपका कठिन काम प्रभावशाली रिपोर्टों में बदल जाए।

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

### होस्ट हेडर

कई बार बैक-एंड **होस्ट हेडर** पर कुछ क्रियाएँ करने के लिए भरोसा करता है। उदाहरण के लिए, यह इसके मान का उपयोग **पासवर्ड रीसेट भेजने के लिए डोमेन के रूप में** कर सकता है। इसलिए जब आप पासवर्ड रीसेट के लिए लिंक के साथ एक ईमेल प्राप्त करते हैं, तो उपयोग किया जाने वाला डोमेन वही है जो आपने होस्ट हेडर में डाला है। फिर, आप अन्य उपयोगकर्ताओं के पासवर्ड रीसेट का अनुरोध कर सकते हैं और डोमेन को एक ऐसा बदल सकते हैं जो आपके द्वारा नियंत्रित किया जाता है ताकि आप उनके पासवर्ड रीसेट कोड चुरा सकें। [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
ध्यान दें कि यह संभव है कि आपको टोकन प्राप्त करने के लिए उपयोगकर्ता के पासवर्ड रीसेट लिंक पर क्लिक करने की प्रतीक्षा करने की भी आवश्यकता नहीं है, क्योंकि शायद **स्पैम फ़िल्टर या अन्य मध्यवर्ती उपकरण/बॉट इसे विश्लेषण करने के लिए क्लिक करेंगे**।
{% endhint %}

### सत्र बूलियन

कुछ समय जब आप कुछ सत्यापन को सही ढंग से पूरा करते हैं, तो बैक-एंड **सुरक्षा विशेषता में "True" मान के साथ एक बूलियन जोड़ता है**। फिर, एक अलग एंडपॉइंट जानता है कि क्या आपने उस जांच को सफलतापूर्वक पास किया।\
हालांकि, यदि आप **जांच पास करते हैं** और आपके सत्र को सुरक्षा विशेषता में "True" मान दिया जाता है, तो आप **अन्य संसाधनों तक पहुँचने का प्रयास कर सकते हैं** जो **उसी विशेषता पर निर्भर करते हैं** लेकिन जिन तक आपको **पहुँचने की अनुमति नहीं होनी चाहिए**। [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### पंजीकरण कार्यक्षमता

एक पहले से मौजूद उपयोगकर्ता के रूप में पंजीकरण करने का प्रयास करें। समकक्ष वर्णों (बिंदुओं, बहुत सारे स्थानों और यूनिकोड) का उपयोग करने का प्रयास करें।

### ईमेल अधिग्रहण

एक ईमेल पंजीकृत करें, इसे पुष्टि करने से पहले ईमेल बदलें, फिर, यदि नया पुष्टि ईमेल पहले पंजीकृत ईमेल पर भेजा जाता है, तो आप किसी भी ईमेल का अधिग्रहण कर सकते हैं। या यदि आप पहले वाले को पुष्टि करने के लिए दूसरे ईमेल को सक्षम कर सकते हैं, तो आप किसी भी खाते का भी अधिग्रहण कर सकते हैं।

### कंपनियों के आंतरिक सर्विसडेस्क तक पहुँचें जो एटलसियन का उपयोग कर रही हैं

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACE विधि

डेवलपर्स उत्पादन वातावरण में विभिन्न डिबगिंग विकल्पों को बंद करना भूल सकते हैं। उदाहरण के लिए, HTTP `TRACE` विधि का उपयोग निदान उद्देश्यों के लिए किया जाता है। यदि सक्षम है, तो वेब सर्वर `TRACE` विधि का उपयोग करने वाले अनुरोधों का उत्तर देगा, जो प्राप्त अनुरोध को प्रतिक्रिया में प्रतिध्वनित करेगा। यह व्यवहार अक्सर हानिरहित होता है, लेकिन कभी-कभी जानकारी का खुलासा करता है, जैसे कि आंतरिक प्रमाणीकरण हेडर का नाम जो अनुरोधों में रिवर्स प्रॉक्सी द्वारा जोड़ा जा सकता है।![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**अपने वेब ऐप्स, नेटवर्क और क्लाउड पर एक हैकर का दृष्टिकोण प्राप्त करें**

**महत्वपूर्ण, शोषण योग्य कमजोरियों को खोजें और रिपोर्ट करें जिनका वास्तविक व्यावसायिक प्रभाव है।** हमारे 20+ कस्टम टूल का उपयोग करके हमले की सतह का मानचित्रण करें, सुरक्षा मुद्दों को खोजें जो आपको विशेषाधिकार बढ़ाने की अनुमति देते हैं, और आवश्यक सबूत इकट्ठा करने के लिए स्वचालित शोषण का उपयोग करें, जिससे आपका कठिन काम प्रभावशाली रिपोर्टों में बदल जाए।

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
