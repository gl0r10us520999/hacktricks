# macOS बंडल्स

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रिपॉसिटरी में।

</details>
{% endhint %}

## मैकओएस बंडल्स

मैकओएस में बंडल्स एप्लिकेशन, लाइब्रेरीज और अन्य आवश्यक फ़ाइलों जैसे संसाधनों के लिए एक कंटेनर के रूप में काम करते हैं, जिन्हें फाइंडर में एकल वस्तु के रूप में प्रदर्शित किया जाता है, जैसे जाने माने `*.app` फ़ाइलें। सबसे आम बंडल `.app` बंडल है, हालांकि `.framework`, `.systemextension`, और `.kext` जैसे अन्य प्रकार भी प्रसारित हैं।

### बंडल के महत्वपूर्ण घटक

एक बंडल के भीतर, विशेष रूप से `<application>.app/Contents/` निर्देशिका के भीतर, कई महत्वपूर्ण संसाधन संग्रहीत होते हैं:

* **\_CodeSignature**: इस निर्देशिका में एप्लिकेशन की अखंडता की पुष्टि के लिए महत्वपूर्ण कोड-साइनिंग विवरण संग्रहीत है। आप कोड-साइनिंग जानकारी की जांच कर सकते हैं उपयुक्त आदेशों का उपयोग करके: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: एप्लिकेशन के कार्यात्मक बाइनरी को संचालित करने वाला निर्देशिका।
* **Resources**: एप्लिकेशन के उपयोगकर्ता इंटरफ़ेस संघटन के लिए एक भंडार है जिसमें छवियाँ, दस्तावेज़ और इंटरफ़ेस विवरण (nib/xib फ़ाइलें) शामिल हैं।
* **Info.plist**: एप्लिकेशन का मुख्य कॉन्फ़िगरेशन फ़ाइल के रूप में काम करता है, जिसे प्रणाली को एप्लिकेशन को सही ढंग से पहचानने और इससे बातचीत करने के लिए महत्वपूर्ण माना जाता है।

#### Info.plist में महत्वपूर्ण कुंजी

`Info.plist` फ़ाइल एप्लिकेशन कॉन्फ़िगरेशन के लिए एक मूलभूत है, जिसमें CFBundleExecutable जैसी कुंजियाँ शामिल हैं:

* **CFBundleExecutable**: `Contents/MacOS` निर्देशिका में स्थित मुख्य कार्यात्मक फ़ाइल का नाम निर्दिष्ट करता है।
* **CFBundleIdentifier**: एप्लिकेशन के लिए एक वैश्विक पहचानकर्ता प्रदान करता है, जिसे मैकओएस द्वारा एप्लिकेशन प्रबंधन के लिए व्यापक रूप से उपयोग किया जाता है।
* **LSMinimumSystemVersion**: एप्लिकेशन को चलाने के लिए आवश्यक मैकओएस के न्यूनतम संस्करण को दर्शाता है।

### बंडल की खोज

`Safari.app` जैसे बंडल की सामग्री की खोज करने के लिए निम्नलिखित आदेश का उपयोग किया जा सकता है: `bash ls -lR /Applications/Safari.app/Contents`

यह खोज निर्देशिकाओं जैसे `_CodeSignature`, `MacOS`, `Resources`, और फ़ाइलों जैसे `Info.plist` को उजागर करती है, प्रत्येक एक अपने उपयोगकर्ता इंटरफ़ेस और संचालन पैरामीटर को परिभाषित करने के लिए एक अद्वितीय उद्देश्य से सेवा करती है।

#### अतिरिक्त बंडल निर्देशिकाएँ

सामान्य निर्देशिकाओं के अतिरिक्त, बंडल्स में निम्नलिखित भी शामिल हो सकते हैं:

* **Frameworks**: एप्लिकेशन द्वारा उपयोग किए जाने वाले बंडल फ्रेमवर्क्स को संग्रहीत करता है। फ्रेमवर्क्स डाइलिब्स की तरह होते हैं जिनमें अतिरिक्त संसाधन होते हैं।
* **PlugIns**: एप्लिकेशन की क्षमताएँ बढ़ाने वाले प्लगइन्स और एक्सटेंशन्स के लिए एक निर्देशिका।
* **XPCServices**: एप्लिकेशन द्वारा प्रक्रिया के बाहर संचार के लिए उपयोग किए जाने वाले XPC सेवाएँ रखता है।

यह संरचना सुनिश्चित करती है कि सभी आवश्यक घटक बंडल के भीतर समाहित हों, जो एक मॉड्यूलर और सुरक्षित एप्लिकेशन वातावरण को सुविधाजनक बनाती है।

`Info.plist` कुंजियों और उनके अर्थों पर अधिक विस्तृत जानकारी के लिए, Apple डेवलपर दस्तावेज़ीकरण व्यापक संसाधन प्रदान करता है: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रिपॉसिटरी में।

</details>
{% endhint %}
