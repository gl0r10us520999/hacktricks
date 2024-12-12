# Cryptographic/Compression Algorithms

## Cryptographic/Compression Algorithms

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

## Identifying Algorithms

यदि आप एक कोड में **shift rights और lefts, xors और कई अंकगणितीय संचालन** का उपयोग करते हैं, तो यह बहुत संभव है कि यह एक **cryptographic algorithm** का कार्यान्वयन है। यहाँ कुछ तरीके दिखाए जाएंगे **बिना प्रत्येक चरण को उलटने की आवश्यकता के एल्गोरिदम की पहचान करने के लिए**।

### API functions

**CryptDeriveKey**

यदि यह फ़ंक्शन उपयोग किया गया है, तो आप दूसरे पैरामीटर के मान की जांच करके यह पता लगा सकते हैं कि **कौन सा एल्गोरिदम उपयोग किया जा रहा है**:

![](<../../.gitbook/assets/image (375) (1) (1) (1) (1).png>)

संभावित एल्गोरिदम और उनके असाइन किए गए मानों की तालिका यहाँ देखें: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

एक दिए गए डेटा के बफर को संकुचित और अनसंकुचित करता है।

**CryptAcquireContext**

[दस्तावेज़ों से](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta): **CryptAcquireContext** फ़ंक्शन का उपयोग एक विशेष कुंजी कंटेनर के लिए हैंडल प्राप्त करने के लिए किया जाता है जो एक विशेष क्रिप्टोग्राफिक सेवा प्रदाता (CSP) के भीतर है। **यह लौटाया गया हैंडल उन CryptoAPI** फ़ंक्शनों में कॉल करने के लिए उपयोग किया जाता है जो चयनित CSP का उपयोग करते हैं।

**CryptCreateHash**

डेटा के एक स्ट्रीम का हैशिंग शुरू करता है। यदि यह फ़ंक्शन उपयोग किया गया है, तो आप दूसरे पैरामीटर के मान की जांच करके यह पता लगा सकते हैं कि **कौन सा एल्गोरिदम उपयोग किया जा रहा है**:

![](<../../.gitbook/assets/image (376).png>)

\
संभावित एल्गोरिदम और उनके असाइन किए गए मानों की तालिका यहाँ देखें: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### Code constants

कभी-कभी एक एल्गोरिदम की पहचान करना वास्तव में आसान होता है क्योंकि इसे एक विशेष और अद्वितीय मान का उपयोग करने की आवश्यकता होती है।

![](<../../.gitbook/assets/image (370).png>)

यदि आप Google में पहले स्थिरांक की खोज करते हैं, तो आपको यह मिलता है:

![](<../../.gitbook/assets/image (371).png>)

इसलिए, आप मान सकते हैं कि डिकंपाइल किया गया फ़ंक्शन एक **sha256 कैलकुलेटर है।**\
आप अन्य स्थिरांकों में से किसी की भी खोज कर सकते हैं और आपको (संभवतः) वही परिणाम प्राप्त होगा।

### data info

यदि कोड में कोई महत्वपूर्ण स्थिरांक नहीं है, तो यह **.data सेक्शन से जानकारी लोड कर रहा हो सकता है**।\
आप उस डेटा तक पहुँच सकते हैं, **पहले dword को समूहित करें** और इसे Google में खोजें जैसा कि हमने पिछले अनुभाग में किया था:

![](<../../.gitbook/assets/image (372).png>)

इस मामले में, यदि आप **0xA56363C6** की खोज करते हैं, तो आप देख सकते हैं कि यह **AES एल्गोरिदम की तालिकाओं** से संबंधित है।

## RC4 **(Symmetric Crypt)**

### Characteristics

यह 3 मुख्य भागों में विभाजित है:

* **Initialization stage/**: 0x00 से 0xFF तक के **मानों की एक तालिका बनाता है** (कुल 256bytes, 0x100)। इस तालिका को आमतौर पर **Substitution Box** (या SBox) कहा जाता है।
* **Scrambling stage**: पहले बनाई गई तालिका के माध्यम से **लूप करेगा** (0x100 पुनरावृत्तियों का लूप, फिर से) प्रत्येक मान को **सेमी-रैंडम** बाइट्स के साथ संशोधित करेगा। इन सेमी-रैंडम बाइट्स को बनाने के लिए, RC4 **कुंजी का उपयोग किया जाता है**। RC4 **कुंजी** की लंबाई **1 से 256 बाइट्स** के बीच हो सकती है, हालाँकि आमतौर पर इसे 5 बाइट्स से अधिक होना अनुशंसित है। आमतौर पर, RC4 कुंजी 16 बाइट्स की लंबाई की होती है।
* **XOR stage**: अंततः, प्लेन-टेक्स्ट या सिफरटेक्स को **पहले बनाए गए मानों के साथ XOR किया जाता है**। एन्क्रिप्ट और डिक्रिप्ट करने के लिए फ़ंक्शन वही होता है। इसके लिए, **बनाए गए 256 बाइट्स के माध्यम से लूप किया जाएगा** जितनी बार आवश्यक हो। इसे आमतौर पर डिकंपाइल किए गए कोड में **%256 (mod 256)** के साथ पहचाना जाता है।

{% hint style="info" %}
**RC4 की पहचान करने के लिए एक डिस्सेम्बली/डिकंपाइल किए गए कोड में आप 0x100 के आकार के 2 लूप की जांच कर सकते हैं (कुंजी के उपयोग के साथ) और फिर इन 2 लूप में पहले बनाए गए 256 मानों के साथ इनपुट डेटा का XOR संभवतः %256 (mod 256) का उपयोग करते हुए।**
{% endhint %}

### **Initialization stage/Substitution Box:** (गिनती के रूप में उपयोग किए गए 256 संख्या और 256 वर्णों के प्रत्येक स्थान पर 0 कैसे लिखा गया है, इसे नोट करें)

![](<../../.gitbook/assets/image (377).png>)

### **Scrambling Stage:**

![](<../../.gitbook/assets/image (378).png>)

### **XOR Stage:**

![](<../../.gitbook/assets/image (379).png>)

## **AES (Symmetric Crypt)**

### **Characteristics**

* **substitution boxes और lookup tables** का उपयोग
* यह **विशिष्ट lookup table मानों** (स्थिरांक) के उपयोग के कारण **AES को पहचानना संभव है**। _ध्यान दें कि **स्थिरांक** को **बाइनरी में** _या_ _**गतिशील रूप से**_ _**स्टोर** किया जा सकता है।_
* **एन्क्रिप्शन कुंजी** को **16** से **भाग दिया जा सकता है** (आमतौर पर 32B) और आमतौर पर 16B का **IV** उपयोग किया जाता है।

### SBox constants

![](<../../.gitbook/assets/image (380).png>)

## Serpent **(Symmetric Crypt)**

### Characteristics

* इसे उपयोग करने वाले कुछ मैलवेयर को खोजना दुर्लभ है लेकिन इसके उदाहरण हैं (Ursnif)
* इसकी लंबाई (अत्यधिक लंबा फ़ंक्शन) के आधार पर यह निर्धारित करना सरल है कि एल्गोरिदम Serpent है या नहीं।

### Identifying

अगली छवि में ध्यान दें कि स्थिरांक **0x9E3779B9** का उपयोग किया गया है (ध्यान दें कि यह स्थिरांक अन्य क्रिप्टो एल्गोरिदम जैसे **TEA** -Tiny Encryption Algorithm द्वारा भी उपयोग किया जाता है)।\
लूप के **आकार** (**132**) और **डिस्सेम्बली** निर्देशों और **कोड** उदाहरण में XOR संचालन की **संख्या** को भी नोट करें:

![](<../../.gitbook/assets/image (381).png>)

जैसा कि पहले उल्लेख किया गया था, इस कोड को किसी भी डिकंपाइलर के अंदर एक **बहुत लंबे फ़ंक्शन** के रूप में देखा जा सकता है क्योंकि इसके अंदर **कोई कूद नहीं है**। डिकंपाइल किया गया कोड निम्नलिखित की तरह दिख सकता है:

![](<../../.gitbook/assets/image (382).png>)

इसलिए, इस एल्गोरिदम की पहचान करना संभव है **जादुई संख्या** और **प्रारंभिक XORs** की जांच करके, एक **बहुत लंबे फ़ंक्शन** को देखकर और **कुछ निर्देशों** की तुलना करके **एक कार्यान्वयन** (जैसे 7 द्वारा बाईं ओर शिफ्ट और 22 द्वारा बाईं ओर घुमाना)।

## RSA **(Asymmetric Crypt)**

### Characteristics

* सममित एल्गोरिदम की तुलना में अधिक जटिल
* कोई स्थिरांक नहीं! (कस्टम कार्यान्वयन निर्धारित करना कठिन है)
* KANAL (एक क्रिप्टो विश्लेषक) RSA पर संकेत दिखाने में विफल रहता है और यह स्थिरांक पर निर्भर करता है।

### Identifying by comparisons

![](<../../.gitbook/assets/image (383).png>)

* लाइन 11 (बाईं ओर) में `+7) >> 3` है जो लाइन 35 (दाईं ओर) में समान है: `+7) / 8`
* लाइन 12 (बाईं ओर) यह जांच रही है कि `modulus_len < 0x040` और लाइन 36 (दाईं ओर) यह जांच रही है कि `inputLen+11 > modulusLen`

## MD5 & SHA (hash)

### Characteristics

* 3 फ़ंक्शन: Init, Update, Final
* समान प्रारंभिक फ़ंक्शन

### Identify

**Init**

आप स्थिरांकों की जांच करके दोनों की पहचान कर सकते हैं। ध्यान दें कि sha\_init में 1 स्थिरांक है जो MD5 में नहीं है:

![](<../../.gitbook/assets/image (385).png>)

**MD5 Transform**

अधिक स्थिरांकों के उपयोग को नोट करें

![](<../../.gitbook/assets/image (253) (1) (1) (1).png>)

## CRC (hash)

* छोटा और अधिक कुशल क्योंकि इसका कार्य डेटा में आकस्मिक परिवर्तनों को खोजना है
* स्थिरांक की पहचान करने के लिए lookup tables का उपयोग करता है

### Identify

**lookup table constants** की जांच करें:

![](<../../.gitbook/assets/image (387).png>)

एक CRC हैश एल्गोरिदम इस तरह दिखता है:

![](<../../.gitbook/assets/image (386).png>)

## APLib (Compression)

### Characteristics

* पहचानने योग्य स्थिरांक नहीं
* आप एल्गोरिदम को पायथन में लिखने और ऑनलाइन समान चीजों की खोज करने का प्रयास कर सकते हैं

### Identify

ग्राफ काफी बड़ा है:

![](<../../.gitbook/assets/image (207) (2) (1).png>)

इसे पहचानने के लिए **3 तुलना** की जांच करें:

![](<../../.gitbook/assets/image (384).png>)

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
