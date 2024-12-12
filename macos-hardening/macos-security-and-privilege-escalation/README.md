# macOS सुरक्षा और विशेषाधिकार वृद्धि

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों ताकि अनुभवी हैकर्स और बग बाउंटी शिकारियों के साथ संवाद कर सकें!

**हैकिंग अंतर्दृष्टि**\
हैकिंग के रोमांच और चुनौतियों में गहराई से जाने वाली सामग्री के साथ संलग्न हों

**वास्तविक समय हैक समाचार**\
वास्तविक समय की समाचार और अंतर्दृष्टि के माध्यम से तेज़-तर्रार हैकिंग दुनिया के साथ अद्यतित रहें

**नवीनतम घोषणाएँ**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफ़ॉर्म अपडेट के साथ सूचित रहें

**हमारे साथ जुड़ें** [**Discord**](https://discord.com/invite/N3FrSbmwdy) पर और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## बुनियादी MacOS

यदि आप macOS से परिचित नहीं हैं, तो आपको macOS के मूल बातें सीखना शुरू करना चाहिए:

* विशेष macOS **फाइलें और अनुमतियाँ:**

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* सामान्य macOS **उपयोगकर्ता**

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* **कर्नेल** की **संरचना**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* सामान्य macOS n**etwork सेवाएँ और प्रोटोकॉल**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **ओपनसोर्स** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* `tar.gz` डाउनलोड करने के लिए एक URL को बदलें जैसे [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) को [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz) में

### MacOS MDM

कंपनियों में **macOS** सिस्टम उच्च संभावना के साथ **MDM के साथ प्रबंधित** होने वाले हैं। इसलिए, एक हमलावर के दृष्टिकोण से यह जानना दिलचस्प है **कि यह कैसे काम करता है**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - निरीक्षण, डिबगिंग और फज़िंग

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## MacOS सुरक्षा सुरक्षा

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## हमले की सतह

### फ़ाइल अनुमतियाँ

यदि एक **प्रक्रिया जो रूट के रूप में चल रही है** एक फ़ाइल लिखती है जिसे एक उपयोगकर्ता द्वारा नियंत्रित किया जा सकता है, तो उपयोगकर्ता इसका दुरुपयोग करके **विशेषाधिकार बढ़ा सकता है**।\
यह निम्नलिखित स्थितियों में हो सकता है:

* उपयोगकर्ता द्वारा पहले से बनाई गई फ़ाइल का उपयोग किया गया (उपयोगकर्ता द्वारा स्वामित्व)
* फ़ाइल का उपयोग उपयोगकर्ता द्वारा समूह के कारण लिखने योग्य है
* फ़ाइल का उपयोग एक निर्देशिका के अंदर है जो उपयोगकर्ता के स्वामित्व में है (उपयोगकर्ता फ़ाइल बना सकता है)
* फ़ाइल का उपयोग एक निर्देशिका के अंदर है जो रूट के स्वामित्व में है लेकिन उपयोगकर्ता को समूह के कारण उस पर लिखने की अनुमति है (उपयोगकर्ता फ़ाइल बना सकता है)

एक **फ़ाइल बनाने में सक्षम होना** जो **रूट द्वारा उपयोग की जाने वाली है**, एक उपयोगकर्ता को **इसके सामग्री का लाभ उठाने** या यहां तक कि **सिंबलिंक/हार्डलिंक** बनाने की अनुमति देता है ताकि इसे किसी अन्य स्थान पर इंगित किया जा सके।

इस प्रकार की कमजोरियों के लिए **कमजोर `.pkg` इंस्टॉलर** की जांच करना न भूलें:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### फ़ाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर

फाइल एक्सटेंशन द्वारा पंजीकृत अजीब ऐप्स का दुरुपयोग किया जा सकता है और विभिन्न अनुप्रयोगों को विशिष्ट प्रोटोकॉल खोलने के लिए पंजीकृत किया जा सकता है

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP विशेषाधिकार वृद्धि

macOS में **अनुप्रयोगों और बाइनरीज़ के पास फ़ोल्डरों या सेटिंग्स तक पहुँचने के लिए अनुमतियाँ हो सकती हैं** जो उन्हें दूसरों की तुलना में अधिक विशेषाधिकार प्राप्त बनाती हैं।

इसलिए, एक हमलावर जो macOS मशीन को सफलतापूर्वक समझौता करना चाहता है, उसे **अपने TCC विशेषाधिकार बढ़ाने** की आवश्यकता होगी (या यहां तक कि **SIP को बायपास करना**, उसकी आवश्यकताओं के आधार पर)।

ये विशेषाधिकार आमतौर पर **अधिकारों** के रूप में दिए जाते हैं जिनसे अनुप्रयोग पर हस्ताक्षर किए जाते हैं, या अनुप्रयोग कुछ पहुँचों का अनुरोध कर सकता है और **उपयोगकर्ता द्वारा उन्हें अनुमोदित करने** के बाद उन्हें **TCC डेटाबेस** में पाया जा सकता है। एक प्रक्रिया इन विशेषाधिकारों को प्राप्त करने का एक और तरीका है कि वह उन विशेषाधिकारों वाली प्रक्रिया की **संतान** हो क्योंकि ये आमतौर पर **विरासत में मिलते हैं**।

[**TCC में विशेषाधिकार बढ़ाने**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), [**TCC को बायपास करने**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) और अतीत में [**SIP को बायपास करने**](macos-security-protections/macos-sip.md#sip-bypasses) के लिए विभिन्न तरीकों को खोजने के लिए इन लिंक का पालन करें।

## macOS पारंपरिक विशेषाधिकार वृद्धि

बेशक, एक रेड टीम के दृष्टिकोण से आपको रूट तक बढ़ाने में भी रुचि होनी चाहिए। कुछ संकेतों के लिए निम्नलिखित पोस्ट देखें:

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}

## macOS अनुपालन

* [https://github.com/usnistgov/macos\_security](https://github.com/usnistgov/macos_security)

## संदर्भ

* [**OS X घटना प्रतिक्रिया: स्क्रिप्टिंग और विश्लेषण**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों ताकि अनुभवी हैकर्स और बग बाउंटी शिकारियों के साथ संवाद कर सकें!

**हैकिंग अंतर्दृष्टि**\
हैकिंग के रोमांच और चुनौतियों में गहराई से जाने वाली सामग्री के साथ संलग्न हों

**वास्तविक समय हैक समाचार**\
वास्तविक समय की समाचार और अंतर्दृष्टि के माध्यम से तेज़-तर्रार हैकिंग दुनिया के साथ अद्यतित रहें

**नवीनतम घोषणाएँ**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफ़ॉर्म अपडेट के साथ सूचित रहें

**हमारे साथ जुड़ें** [**Discord**](https://discord.com/invite/N3FrSbmwdy) पर और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
