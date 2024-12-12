# SmbExec/ScExec

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

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**अपने वेब ऐप्स, नेटवर्क और क्लाउड पर एक हैकर का दृष्टिकोण प्राप्त करें**

**महत्वपूर्ण, शोषण योग्य कमजोरियों को खोजें और रिपोर्ट करें जिनका वास्तविक व्यावसायिक प्रभाव है।** हमारे 20+ कस्टम टूल का उपयोग करके हमले की सतह का मानचित्रण करें, सुरक्षा मुद्दों को खोजें जो आपको विशेषाधिकार बढ़ाने की अनुमति देते हैं, और आवश्यक सबूत इकट्ठा करने के लिए स्वचालित शोषण का उपयोग करें, जिससे आपका कठिन काम प्रभावशाली रिपोर्टों में बदल जाए।

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

## यह कैसे काम करता है

**Smbexec** एक उपकरण है जिसका उपयोग विंडोज सिस्टम पर दूरस्थ कमांड निष्पादन के लिए किया जाता है, जो **Psexec** के समान है, लेकिन यह लक्षित प्रणाली पर कोई दुर्भावनापूर्ण फ़ाइलें नहीं रखता है।

### **SMBExec** के बारे में मुख्य बिंदु

- यह लक्षित मशीन पर एक अस्थायी सेवा (उदाहरण के लिए, "BTOBTO") बनाकर cmd.exe (%COMSPEC%) के माध्यम से कमांड निष्पादित करता है, बिना किसी बाइनरी को गिराए।
- इसके छिपे हुए दृष्टिकोण के बावजूद, यह प्रत्येक निष्पादित कमांड के लिए इवेंट लॉग उत्पन्न करता है, जो एक प्रकार के गैर-इंटरैक्टिव "शेल" की पेशकश करता है।
- **Smbexec** का उपयोग करके कनेक्ट करने के लिए कमांड इस प्रकार दिखता है:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Executing Commands Without Binaries

- **Smbexec** सीधे सेवा binPaths के माध्यम से कमांड निष्पादन की अनुमति देता है, जिससे लक्ष्य पर भौतिक बाइनरी की आवश्यकता समाप्त हो जाती है।
- यह विधि Windows लक्ष्य पर एक बार के लिए कमांड निष्पादित करने के लिए उपयोगी है। उदाहरण के लिए, इसे Metasploit के `web_delivery` मॉड्यूल के साथ जोड़ने से PowerShell-लक्षित रिवर्स मीटरप्रीटर पेलोड का निष्पादन संभव होता है।
- हमलावर की मशीन पर एक दूरस्थ सेवा बनाकर जिसमें binPath को cmd.exe के माध्यम से प्रदान किए गए कमांड को चलाने के लिए सेट किया गया है, पेलोड को सफलतापूर्वक निष्पादित करना संभव है, callback और पेलोड निष्पादन को Metasploit लिस्नर के साथ प्राप्त करना, भले ही सेवा प्रतिक्रिया त्रुटियाँ उत्पन्न हों।

### Commands Example

सेवा बनाना और शुरू करना निम्नलिखित कमांड के साथ किया जा सकता है:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
FOr further details check [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)


## References
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**अपने वेब ऐप्स, नेटवर्क और क्लाउड पर हैकर का दृष्टिकोण प्राप्त करें**

**वास्तविक व्यावसायिक प्रभाव के साथ महत्वपूर्ण, शोषण योग्य कमजोरियों को खोजें और रिपोर्ट करें।** हमारे 20+ कस्टम टूल का उपयोग करके हमले की सतह को मैप करें, सुरक्षा मुद्दों को खोजें जो आपको विशेषाधिकार बढ़ाने की अनुमति देते हैं, और आवश्यक सबूत इकट्ठा करने के लिए स्वचालित शोषण का उपयोग करें, जिससे आपका कठिन काम प्रभावशाली रिपोर्टों में बदल जाए।

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
