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

* The **User Keychain** (`~/Library/Keychains/login.keychain-db`), which is used to store **उपयोगकर्ता-विशिष्ट क्रेडेंशियल्स** like application passwords, internet passwords, user-generated certificates, network passwords, and user-generated public/private keys.
* The **System Keychain** (`/Library/Keychains/System.keychain`), which stores **सिस्टम-व्यापी क्रेडेंशियल्स** such as WiFi passwords, system root certificates, system private keys, and system application passwords.
* It's possible to find other components like certificates in `/System/Library/Keychains/*`
* In **iOS** there is only one **Keychain** located in `/private/var/Keychains/`. This folder also contains databases for the `TrustStore`, certificates authorities (`caissuercache`) and OSCP entries (`ocspache`).
* Apps will be restricted in the keychain only to their private area based on their application identifier.

### Password Keychain Access

These files, while they do not have inherent protection and can be **downloaded**, are encrypted and require the **उपयोगकर्ता का प्लेनटेक्स्ट पासवर्ड को डिक्रिप्ट करने के लिए**. A tool like [**Chainbreaker**](https://github.com/n0fate/chainbreaker) could be used for decryption.

## Keychain Entries Protections

### ACLs

Each entry in the keychain is governed by **Access Control Lists (ACLs)** which dictate who can perform various actions on the keychain entry, including:

* **ACLAuhtorizationExportClear**: Allows the holder to get the clear text of the secret.
* **ACLAuhtorizationExportWrapped**: Allows the holder to get the clear text encrypted with another provided password.
* **ACLAuhtorizationAny**: Allows the holder to perform any action.

The ACLs are further accompanied by a **list of trusted applications** that can perform these actions without prompting. This could be:

* **N`il`** (no authorization required, **everyone is trusted**)
* An **empty** list (**nobody** is trusted)
* **List** of specific **applications**.

Also the entry might contain the key **`ACLAuthorizationPartitionID`,** which is use to identify the **teamid, apple,** and **cdhash.**

* If the **teamid** is specified, then in order to **access the entry** value **withuot** a **prompt** the used application must have the **same teamid**.
* If the **apple** is specified, then the app needs to be **signed** by **Apple**.
* If the **cdhash** is indicated, then **app** must have the specific **cdhash**.

### Creating a Keychain Entry

When a **new** **entry** is created using **`Keychain Access.app`**, the following rules apply:

* All apps can encrypt.
* **No apps** can export/decrypt (without prompting the user).
* All apps can see the integrity check.
* No apps can change ACLs.
* The **partitionID** is set to **`apple`**.

When an **application creates an entry in the keychain**, the rules are slightly different:

* All apps can encrypt.
* Only the **creating application** (or any other apps explicitly added) can export/decrypt (without prompting the user).
* All apps can see the integrity check.
* No apps can change the ACLs.
* The **partitionID** is set to **`teamid:[teamID here]`**.

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
**कीचेन अनुक्रमण और रहस्यों का डंपिंग** जो **प्रॉम्प्ट उत्पन्न नहीं करेगा** उसे [**LockSmith**](https://github.com/its-a-feature/LockSmith) टूल के साथ किया जा सकता है।

अन्य API एंडपॉइंट्स [**SecKeyChain.h**](https://opensource.apple.com/source/libsecurity\_keychain/libsecurity\_keychain-55017/lib/SecKeychain.h.auto.html) स्रोत कोड में पाए जा सकते हैं।
{% endhint %}

**सुरक्षा ढांचे** का उपयोग करके प्रत्येक कीचेन प्रविष्टि की **सूची** और **जानकारी** प्राप्त करें या आप एप्पल के ओपन-सोर्स CLI टूल [**security**](https://opensource.apple.com/source/Security/Security-59306.61.1/SecurityTool/macOS/security.c.auto.html)** को भी देख सकते हैं।** कुछ API उदाहरण:

* API **`SecItemCopyMatching`** प्रत्येक प्रविष्टि के बारे में जानकारी देता है और इसका उपयोग करते समय कुछ विशेषताएँ सेट की जा सकती हैं:
* **`kSecReturnData`**: यदि सत्य है, तो यह डेटा को डिक्रिप्ट करने की कोशिश करेगा (संभावित पॉप-अप से बचने के लिए इसे गलत सेट करें)
* **`kSecReturnRef`**: कीचेन आइटम के लिए संदर्भ भी प्राप्त करें (यदि बाद में आप देखते हैं कि आप बिना पॉप-अप के डिक्रिप्ट कर सकते हैं तो इसे सत्य पर सेट करें)
* **`kSecReturnAttributes`**: प्रविष्टियों के बारे में मेटाडेटा प्राप्त करें
* **`kSecMatchLimit`**: कितने परिणाम लौटाने हैं
* **`kSecClass`**: कीचेन प्रविष्टि का प्रकार

प्रत्येक प्रविष्टि के **ACLs** प्राप्त करें:

* API **`SecAccessCopyACLList`** के साथ आप **कीचेन आइटम के लिए ACL** प्राप्त कर सकते हैं, और यह ACLs की एक सूची लौटाएगा (जैसे `ACLAuhtorizationExportClear` और अन्य पहले उल्लेखित) जहां प्रत्येक सूची में:
* विवरण
* **विश्वसनीय एप्लिकेशन सूची**। यह हो सकता है:
* एक ऐप: /Applications/Slack.app
* एक बाइनरी: /usr/libexec/airportd
* एक समूह: group://AirPort

डेटा निर्यात करें:

* API **`SecKeychainItemCopyContent`** स्पष्ट पाठ प्राप्त करता है
* API **`SecItemExport`** कुंजी और प्रमाणपत्रों को निर्यात करता है लेकिन सामग्री को एन्क्रिप्टेड निर्यात करने के लिए पासवर्ड सेट करना पड़ सकता है

और ये हैं **एक रहस्य को बिना प्रॉम्प्ट के निर्यात करने की आवश्यकताएँ**:

* यदि **1+ विश्वसनीय** ऐप्स सूचीबद्ध हैं:
* उचित **अधिकार** की आवश्यकता है (**`Nil`**, या रहस्य जानकारी तक पहुँचने के लिए अनुमति सूची में **भाग** होना)
* **PartitionID** से मेल खाने के लिए कोड हस्ताक्षर की आवश्यकता है
* एक **विश्वसनीय ऐप** का कोड हस्ताक्षर मेल खाने की आवश्यकता है (या सही KeychainAccessGroup का सदस्य होना)
* यदि **सभी एप्लिकेशन विश्वसनीय** हैं:
* उचित **अधिकार** की आवश्यकता है
* **PartitionID** से मेल खाने के लिए कोड हस्ताक्षर की आवश्यकता है
* यदि **कोई PartitionID नहीं है**, तो यह आवश्यक नहीं है

{% hint style="danger" %}
इसलिए, यदि **1 एप्लिकेशन सूचीबद्ध है**, तो आपको **उस एप्लिकेशन में कोड इंजेक्ट करने** की आवश्यकता है।

यदि **apple** **partitionID** में इंगित है, तो आप इसे **`osascript`** के साथ एक्सेस कर सकते हैं इसलिए कुछ भी जो partitionID में apple के साथ सभी एप्लिकेशनों पर भरोसा कर रहा है। इसके लिए **`Python`** का भी उपयोग किया जा सकता है।
{% endhint %}

### दो अतिरिक्त विशेषताएँ

* **अदृश्य**: यह **UI** कीचेन ऐप से प्रविष्टि को **छिपाने** के लिए एक बूलियन ध्वज है
* **सामान्य**: यह **मेटाडेटा** संग्रहीत करने के लिए है (इसलिए यह एन्क्रिप्टेड नहीं है)
* माइक्रोसॉफ्ट संवेदनशील एंडपॉइंट तक पहुँचने के लिए सभी रिफ्रेश टोकन को स्पष्ट पाठ में संग्रहीत कर रहा था।

## संदर्भ

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर हमें **फॉलो करें** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
