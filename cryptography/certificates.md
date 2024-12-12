# Certificates

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

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## What is a Certificate

एक **पब्लिक की सर्टिफिकेट** एक डिजिटल आईडी है जिसका उपयोग क्रिप्टोग्राफी में यह साबित करने के लिए किया जाता है कि कोई व्यक्ति एक पब्लिक की का मालिक है। इसमें की के विवरण, मालिक की पहचान (विषय), और एक विश्वसनीय प्राधिकरण (जारीकर्ता) से डिजिटल हस्ताक्षर शामिल होता है। यदि सॉफ़्टवेयर जारीकर्ता पर भरोसा करता है और हस्ताक्षर मान्य है, तो की के मालिक के साथ सुरक्षित संचार संभव है।

सर्टिफिकेट ज्यादातर [सर्टिफिकेट प्राधिकरणों](https://en.wikipedia.org/wiki/Certificate\_authority) (CAs) द्वारा [पब्लिक-की इन्फ्रास्ट्रक्चर](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI) सेटअप में जारी किए जाते हैं। एक अन्य विधि [वेब ऑफ ट्रस्ट](https://en.wikipedia.org/wiki/Web\_of\_trust) है, जहां उपयोगकर्ता सीधे एक-दूसरे की की की पुष्टि करते हैं। सर्टिफिकेट के लिए सामान्य प्रारूप [X.509](https://en.wikipedia.org/wiki/X.509) है, जिसे RFC 5280 में वर्णित विशिष्ट आवश्यकताओं के अनुसार अनुकूलित किया जा सकता है।

## x509 Common Fields

### **Common Fields in x509 Certificates**

x509 सर्टिफिकेट में, कई **फील्ड** सर्टिफिकेट की वैधता और सुरक्षा सुनिश्चित करने में महत्वपूर्ण भूमिका निभाते हैं। इन फील्ड का विवरण इस प्रकार है:

* **संस्करण संख्या** x509 प्रारूप के संस्करण को दर्शाती है।
* **अनुक्रम संख्या** सर्टिफिकेट को एक सर्टिफिकेट प्राधिकरण (CA) प्रणाली के भीतर अद्वितीय रूप से पहचानती है, मुख्य रूप से रद्दीकरण ट्रैकिंग के लिए।
* **विषय** फ़ील्ड सर्टिफिकेट के मालिक का प्रतिनिधित्व करती है, जो एक मशीन, एक व्यक्ति, या एक संगठन हो सकता है। इसमें विस्तृत पहचान शामिल है जैसे:
* **कॉमन नाम (CN)**: सर्टिफिकेट द्वारा कवर किए गए डोमेन।
* **देश (C)**, **स्थानीयता (L)**, **राज्य या प्रांत (ST, S, या P)**, **संगठन (O)**, और **संगठनात्मक इकाई (OU)** भौगोलिक और संगठनात्मक विवरण प्रदान करते हैं।
* **विशिष्ट नाम (DN)** पूर्ण विषय पहचान को संक्षिप्त करता है।
* **जारीकर्ता** विवरण बताता है कि किसने सर्टिफिकेट को सत्यापित और हस्ताक्षरित किया, जिसमें CA के लिए विषय के समान उपफील्ड शामिल हैं।
* **वैधता अवधि** **Not Before** और **Not After** टाइमस्टैम्प द्वारा चिह्नित होती है, यह सुनिश्चित करते हुए कि सर्टिफिकेट को किसी निश्चित तिथि से पहले या बाद में उपयोग नहीं किया जाता है।
* **पब्लिक की** अनुभाग, जो सर्टिफिकेट की सुरक्षा के लिए महत्वपूर्ण है, पब्लिक की के एल्गोरिदम, आकार, और अन्य तकनीकी विवरण निर्दिष्ट करता है।
* **x509v3 एक्सटेंशन** सर्टिफिकेट की कार्यक्षमता को बढ़ाते हैं, **की उपयोग**, **विस्तारित की उपयोग**, **विषय वैकल्पिक नाम**, और अन्य गुणों को निर्दिष्ट करते हैं ताकि सर्टिफिकेट के आवेदन को ठीक से समायोजित किया जा सके।

#### **Key Usage and Extensions**

* **की उपयोग** पब्लिक की के क्रिप्टोग्राफिक अनुप्रयोगों की पहचान करता है, जैसे डिजिटल हस्ताक्षर या की एन्क्रिप्शन।
* **विस्तारित की उपयोग** सर्टिफिकेट के उपयोग के मामलों को और संकीर्ण करता है, जैसे कि TLS सर्वर प्रमाणीकरण के लिए।
* **विषय वैकल्पिक नाम** और **बेसिक कंस्ट्रेंट** सर्टिफिकेट द्वारा कवर किए गए अतिरिक्त होस्ट नामों और यह कि यह एक CA या अंत-इकाई सर्टिफिकेट है, को परिभाषित करते हैं।
* **विषय की पहचानकर्ता** और **प्राधिकरण की पहचानकर्ता** की की अद्वितीयता और ट्रेसबिलिटी सुनिश्चित करते हैं।
* **प्राधिकरण जानकारी पहुंच** और **CRL वितरण बिंदु** जारीकर्ता CA को सत्यापित करने और सर्टिफिकेट रद्दीकरण स्थिति की जांच करने के लिए पथ प्रदान करते हैं।
* **CT प्री सर्टिफिकेट SCTs** पारदर्शिता लॉग प्रदान करते हैं, जो सर्टिफिकेट में सार्वजनिक विश्वास के लिए महत्वपूर्ण हैं।
```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```
### **OCSP और CRL वितरण बिंदुओं के बीच का अंतर**

**OCSP** (**RFC 2560**) में एक क्लाइंट और एक रिस्पॉन्डर मिलकर काम करते हैं यह जांचने के लिए कि क्या एक डिजिटल पब्लिक-की सर्टिफिकेट को रद्द किया गया है, बिना पूर्ण **CRL** डाउनलोड किए। यह विधि पारंपरिक **CRL** की तुलना में अधिक कुशल है, जो रद्द किए गए सर्टिफिकेट के सीरियल नंबरों की एक सूची प्रदान करती है लेकिन एक संभावित बड़े फ़ाइल को डाउनलोड करने की आवश्यकता होती है। CRLs में 512 प्रविष्टियाँ तक हो सकती हैं। अधिक विवरण [यहाँ](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm) उपलब्ध हैं।

### **सर्टिफिकेट ट्रांसपेरेंसी क्या है**

सर्टिफिकेट ट्रांसपेरेंसी सर्टिफिकेट से संबंधित खतरों से लड़ने में मदद करती है यह सुनिश्चित करके कि SSL सर्टिफिकेट का जारी होना और अस्तित्व डोमेन मालिकों, CAs, और उपयोगकर्ताओं के लिए दृश्य है। इसके उद्देश्य हैं:

* CAs को डोमेन मालिक की जानकारी के बिना एक डोमेन के लिए SSL सर्टिफिकेट जारी करने से रोकना।
* गलती से या दुर्भावनापूर्ण तरीके से जारी किए गए सर्टिफिकेट को ट्रैक करने के लिए एक खुला ऑडिटिंग सिस्टम स्थापित करना।
* उपयोगकर्ताओं को धोखाधड़ी वाले सर्टिफिकेट से सुरक्षित रखना।

#### **सर्टिफिकेट लॉग्स**

सर्टिफिकेट लॉग्स सार्वजनिक रूप से ऑडिट करने योग्य, केवल जोड़ने वाले रिकॉर्ड होते हैं, जो नेटवर्क सेवाओं द्वारा बनाए रखे जाते हैं। ये लॉग ऑडिटिंग उद्देश्यों के लिए क्रिप्टोग्राफिक प्रमाण प्रदान करते हैं। जारी करने वाली प्राधिकरण और जनता दोनों इन लॉग्स में सर्टिफिकेट जमा कर सकते हैं या सत्यापन के लिए उन्हें क्वेरी कर सकते हैं। जबकि लॉग सर्वरों की सटीक संख्या निश्चित नहीं है, यह अपेक्षित है कि यह वैश्विक स्तर पर एक हजार से कम हो। ये सर्वर CAs, ISPs, या किसी भी इच्छुक इकाई द्वारा स्वतंत्र रूप से प्रबंधित किए जा सकते हैं।

#### **क्वेरी**

किसी भी डोमेन के लिए सर्टिफिकेट ट्रांसपेरेंसी लॉग्स का अन्वेषण करने के लिए, [https://crt.sh/](https://crt.sh) पर जाएँ।

सर्टिफिकेट को स्टोर करने के लिए विभिन्न प्रारूप मौजूद हैं, प्रत्येक के अपने उपयोग के मामले और संगतता होती है। यह सारांश मुख्य प्रारूपों को कवर करता है और उनके बीच रूपांतरण के लिए मार्गदर्शन प्रदान करता है।

## **प्रारूप**

### **PEM प्रारूप**

* सर्टिफिकेट के लिए सबसे व्यापक रूप से उपयोग किया जाने वाला प्रारूप।
* सर्टिफिकेट और प्राइवेट की के लिए अलग फ़ाइलों की आवश्यकता होती है, जो Base64 ASCII में एन्कोडेड होती हैं।
* सामान्य एक्सटेंशन: .cer, .crt, .pem, .key।
* मुख्य रूप से Apache और समान सर्वरों द्वारा उपयोग किया जाता है।

### **DER प्रारूप**

* सर्टिफिकेट का एक बाइनरी प्रारूप।
* PEM फ़ाइलों में पाए जाने वाले "BEGIN/END CERTIFICATE" बयानों की कमी है।
* सामान्य एक्सटेंशन: .cer, .der।
* अक्सर Java प्लेटफार्मों के साथ उपयोग किया जाता है।

### **P7B/PKCS#7 प्रारूप**

* Base64 ASCII में स्टोर किया गया, एक्सटेंशन .p7b या .p7c के साथ।
* केवल सर्टिफिकेट और चेन सर्टिफिकेट शामिल होते हैं, प्राइवेट की को छोड़कर।
* Microsoft Windows और Java Tomcat द्वारा समर्थित।

### **PFX/P12/PKCS#12 प्रारूप**

* एक बाइनरी प्रारूप जो सर्वर सर्टिफिकेट, मध्यवर्ती सर्टिफिकेट, और प्राइवेट की को एक फ़ाइल में संलग्न करता है।
* एक्सटेंशन: .pfx, .p12।
* मुख्य रूप से Windows पर सर्टिफिकेट आयात और निर्यात के लिए उपयोग किया जाता है।

### **प्रारूपों का रूपांतरण**

**PEM रूपांतरण** संगतता के लिए आवश्यक हैं:

* **x509 से PEM**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEM से DER**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DER से PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEM से P7B**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7 से PEM**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX परिवर्तनों** का प्रबंधन करने के लिए विंडोज़ पर प्रमाणपत्रों के लिए महत्वपूर्ण है:

* **PFX से PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX से PKCS#8** में दो चरण शामिल हैं:
1. PFX को PEM में परिवर्तित करें
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. PEM को PKCS8 में परिवर्तित करें
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7B से PFX** के लिए भी दो कमांड की आवश्यकता होती है:
1. P7B को CER में परिवर्तित करें
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. CER और प्राइवेट की को PFX में परिवर्तित करें
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें ताकि आप दुनिया के **सबसे उन्नत** सामुदायिक उपकरणों द्वारा संचालित **कार्यप्रवाहों** को आसानी से बना और **स्वचालित** कर सकें।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें। 

</details>
{% endhint %}
