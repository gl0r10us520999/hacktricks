# macOS Keychain

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Main Keychains

* The **User Keychain** (`~/Library/Keychains/login.keychain-db`), जो **उपयोगकर्ता-विशिष्ट प्रमाणपत्रों** को स्टोर करने के लिए उपयोग किया जाता है जैसे कि एप्लिकेशन पासवर्ड, इंटरनेट पासवर्ड, उपयोगकर्ता-निर्मित प्रमाणपत्र, नेटवर्क पासवर्ड, और उपयोगकर्ता-निर्मित सार्वजनिक/निजी कुंजी।
* The **System Keychain** (`/Library/Keychains/System.keychain`), जो **सिस्टम-व्यापी प्रमाणपत्रों** को स्टोर करता है जैसे कि WiFi पासवर्ड, सिस्टम रूट प्रमाणपत्र, सिस्टम निजी कुंजी, और सिस्टम एप्लिकेशन पासवर्ड।
* अन्य घटक जैसे कि प्रमाणपत्र `/System/Library/Keychains/*` में पाए जा सकते हैं।
* **iOS** में केवल एक **Keychain** है जो `/private/var/Keychains/` में स्थित है। इस फ़ोल्डर में `TrustStore`, प्रमाणपत्र प्राधिकरण (`caissuercache`) और OSCP प्रविष्टियों (`ocspache`) के लिए डेटाबेस भी होते हैं।
* ऐप्स को उनके एप्लिकेशन पहचानकर्ता के आधार पर कीचेन में केवल उनके निजी क्षेत्र में प्रतिबंधित किया जाएगा।

### Password Keychain Access

ये फ़ाइलें, जबकि इनमें अंतर्निहित सुरक्षा नहीं है और इन्हें **डाउनलोड** किया जा सकता है, एन्क्रिप्टेड हैं और इन्हें **डिक्रिप्ट करने के लिए उपयोगकर्ता का स्पष्ट पासवर्ड** आवश्यक है। डिक्रिप्शन के लिए [**Chainbreaker**](https://github.com/n0fate/chainbreaker) जैसे टूल का उपयोग किया जा सकता है।

## Keychain Entries Protections

### ACLs

कीचेन में प्रत्येक प्रविष्टि **Access Control Lists (ACLs)** द्वारा शासित होती है जो यह निर्धारित करती है कि कौन कीचेन प्रविष्टि पर विभिन्न क्रियाएँ कर सकता है, जिसमें शामिल हैं:

* **ACLAuhtorizationExportClear**: धारक को रहस्य का स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationExportWrapped**: धारक को दूसरे प्रदान किए गए पासवर्ड के साथ एन्क्रिप्टेड स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationAny**: धारक को कोई भी क्रिया करने की अनुमति देता है।

ACLs के साथ एक **विश्वसनीय अनुप्रयोगों की सूची** होती है जो बिना प्रॉम्प्ट के ये क्रियाएँ कर सकती हैं। यह हो सकता है:

* **N`il`** (कोई प्राधिकरण आवश्यक नहीं, **सभी विश्वसनीय हैं**)
* एक **खाली** सूची (**कोई भी** विश्वसनीय नहीं है)
* **विशिष्ट** **अनुप्रयोगों** की **सूची**।

इसके अलावा, प्रविष्टि में कुंजी **`ACLAuthorizationPartitionID`** हो सकती है, जिसका उपयोग **teamid, apple,** और **cdhash** की पहचान के लिए किया जाता है।

* यदि **teamid** निर्दिष्ट है, तो **प्रविष्टि** मान **बिना** **प्रॉम्प्ट** के **एक्सेस** करने के लिए उपयोग किए जाने वाले एप्लिकेशन का **समान teamid** होना चाहिए।
* यदि **apple** निर्दिष्ट है, तो ऐप को **Apple** द्वारा **साइन** किया जाना चाहिए।
* यदि **cdhash** निर्दिष्ट है, तो **app** को विशिष्ट **cdhash** होना चाहिए।

### Creating a Keychain Entry

जब **`Keychain Access.app`** का उपयोग करके एक **नया** **प्रविष्टि** बनाई जाती है, तो निम्नलिखित नियम लागू होते हैं:

* सभी ऐप्स एन्क्रिप्ट कर सकते हैं।
* **कोई ऐप्स** निर्यात/डिक्रिप्ट नहीं कर सकते (उपयोगकर्ता को प्रॉम्प्ट किए बिना)।
* सभी ऐप्स अखंडता जांच देख सकते हैं।
* कोई ऐप्स ACLs को नहीं बदल सकते।
* **partitionID** को **`apple`** पर सेट किया गया है।

जब एक **अनुप्रयोग कीचेन में एक प्रविष्टि बनाता है**, तो नियम थोड़े अलग होते हैं:

* सभी ऐप्स एन्क्रिप्ट कर सकते हैं।
* केवल **बनाने वाला अनुप्रयोग** (या कोई अन्य ऐप्स जो स्पष्ट रूप से जोड़े गए हैं) निर्यात/डिक्रिप्ट कर सकते हैं (उपयोगकर्ता को प्रॉम्प्ट किए बिना)।
* सभी ऐप्स अखंडता जांच देख सकते हैं।
* कोई ऐप्स ACLs को नहीं बदल सकते।
* **partitionID** को **`teamid:[teamID here]`** पर सेट किया गया है।

## Accessing the Keychain

### `security`
```bash
# List keychains
security list-keychains

# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S

# Dump specifically the user keychain
security dump-keychain ~/Library/Keychains/login.keychain-db
```
### APIs

{% hint style="success" %}
**कीचेन एन्यूमरेशन और डंपिंग** उन रहस्यों की जो **प्रॉम्प्ट उत्पन्न नहीं करेंगे** को [**LockSmith**](https://github.com/its-a-feature/LockSmith) टूल के साथ किया जा सकता है।

अन्य API एंडपॉइंट्स [**SecKeyChain.h**](https://opensource.apple.com/source/libsecurity\_keychain/libsecurity\_keychain-55017/lib/SecKeychain.h.auto.html) स्रोत कोड में पाए जा सकते हैं।
{% endhint %}

**Security Framework** का उपयोग करके प्रत्येक कीचेन प्रविष्टि की **सूची** और **जानकारी** प्राप्त करें या आप एप्पल के ओपन-सोर्स CLI टूल [**security**](https://opensource.apple.com/source/Security/Security-59306.61.1/SecurityTool/macOS/security.c.auto.html)** को भी देख सकते हैं।** कुछ API उदाहरण:

* API **`SecItemCopyMatching`** प्रत्येक प्रविष्टि के बारे में जानकारी देता है और इसका उपयोग करते समय आप कुछ विशेषताएँ सेट कर सकते हैं:
* **`kSecReturnData`**: यदि सत्य है, तो यह डेटा को डिक्रिप्ट करने की कोशिश करेगा (संभावित पॉप-अप से बचने के लिए इसे गलत सेट करें)
* **`kSecReturnRef`**: कीचेन आइटम के लिए संदर्भ भी प्राप्त करें (यदि बाद में आप देखते हैं कि आप बिना पॉप-अप के डिक्रिप्ट कर सकते हैं तो इसे सत्य पर सेट करें)
* **`kSecReturnAttributes`**: प्रविष्टियों के बारे में मेटाडेटा प्राप्त करें
* **`kSecMatchLimit`**: कितने परिणाम लौटाने हैं
* **`kSecClass`**: कीचेन प्रविष्टि का प्रकार

प्रत्येक प्रविष्टि के **ACLs** प्राप्त करें:

* API **`SecAccessCopyACLList`** के साथ आप **कीचेन आइटम के लिए ACL** प्राप्त कर सकते हैं, और यह ACLs की एक सूची लौटाएगा (जैसे `ACLAuhtorizationExportClear` और अन्य पहले उल्लेखित) जहाँ प्रत्येक सूची में:
* विवरण
* **विश्वसनीय एप्लिकेशन सूची**। यह हो सकता है:
* एक ऐप: /Applications/Slack.app
* एक बाइनरी: /usr/libexec/airportd
* एक समूह: group://AirPort

डेटा निर्यात करें:

* API **`SecKeychainItemCopyContent`** स्पष्ट पाठ प्राप्त करता है
* API **`SecItemExport`** कुंजी और प्रमाणपत्रों को निर्यात करता है लेकिन सामग्री को एन्क्रिप्टेड निर्यात करने के लिए पासवर्ड सेट करने की आवश्यकता हो सकती है

और ये हैं **निर्धारण** बिना प्रॉम्प्ट के **रहस्य निर्यात** करने के लिए:

* यदि **1+ विश्वसनीय** ऐप्स सूचीबद्ध हैं:
* उचित **अधिकार** की आवश्यकता है (**`Nil`**, या रहस्य जानकारी तक पहुँचने के लिए अनुमति प्राप्त ऐप्स की सूची का **भाग** होना चाहिए)
* **PartitionID** से मेल खाने के लिए कोड हस्ताक्षर की आवश्यकता है
* एक **विश्वसनीय ऐप** का कोड हस्ताक्षर मेल खाने की आवश्यकता है (या सही KeychainAccessGroup का सदस्य होना चाहिए)
* यदि **सभी एप्लिकेशन विश्वसनीय** हैं:
* उचित **अधिकार** की आवश्यकता है
* **PartitionID** से मेल खाने के लिए कोड हस्ताक्षर की आवश्यकता है
* यदि **कोई PartitionID** नहीं है, तो इसकी आवश्यकता नहीं है

{% hint style="danger" %}
इसलिए, यदि **1 एप्लिकेशन सूचीबद्ध** है, तो आपको **उस एप्लिकेशन में कोड इंजेक्ट** करने की आवश्यकता है।

यदि **एप्पल** **partitionID** में इंगित है, तो आप इसे **`osascript`** के साथ एक्सेस कर सकते हैं इसलिए कुछ भी जो partitionID में एप्पल के साथ सभी एप्लिकेशनों पर भरोसा कर रहा है। इसके लिए **`Python`** का भी उपयोग किया जा सकता है।
{% endhint %}

### Two additional attributes

* **अदृश्य**: यह **UI** कीचेन ऐप से प्रविष्टि को **छिपाने** के लिए एक बूलियन ध्वज है
* **सामान्य**: यह **मेटाडेटा** संग्रहीत करने के लिए है (इसलिए यह एन्क्रिप्टेड नहीं है)
* माइक्रोसॉफ्ट संवेदनशील एंडपॉइंट तक पहुँचने के लिए सभी रिफ्रेश टोकन को स्पष्ट पाठ में संग्रहीत कर रहा था।

## References

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर हमें **फॉलो** करें** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
