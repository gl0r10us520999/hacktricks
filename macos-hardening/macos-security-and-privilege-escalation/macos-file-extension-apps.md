# macOS फ़ाइल एक्सटेंशन और URL स्कीम ऐप हैंडलर्स

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो करें**.**
* हैकिंग ट्रिक्स साझा करें और [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

## LaunchServices डेटाबेस

यह macOS में सभी स्थापित अनुप्रयोगों का एक डेटाबेस है जिसे प्रत्येक स्थापित अनुप्रयोग के बारे में जानकारी प्राप्त करने के लिए क्वेरी किया जा सकता है जैसे कि यह किन URL स्कीमों का समर्थन करता है और MIME प्रकार।

इस डेटाबेस को डंप करना संभव है:

{% code overflow="wrap" %}
```
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -dump
```
{% endcode %}

या उपकरण का उपयोग करते हुए [**lsdtrip**](https://newosxbook.com/tools/lsdtrip.html)।

**`/usr/libexec/lsd`** डेटाबेस का मस्तिष्क है। यह **कई XPC सेवाएँ** प्रदान करता है जैसे कि `.lsd.installation`, `.lsd.open`, `.lsd.openurl`, और अधिक। लेकिन यह **कुछ अधिकारों** की भी आवश्यकता होती है ताकि एप्लिकेशन एक्सपोज़ किए गए XPC कार्यात्मकताओं का उपयोग कर सकें, जैसे कि `.launchservices.changedefaulthandler` या `.launchservices.changeurlschemehandler` डिफ़ॉल्ट ऐप्स को माइम प्रकारों या यूआरएल स्कीमों के लिए बदलने के लिए और अन्य।

**`/System/Library/CoreServices/launchservicesd`** सेवा `com.apple.coreservices.launchservicesd` का दावा करता है और चल रहे एप्लिकेशनों के बारे में जानकारी प्राप्त करने के लिए क्वेरी किया जा सकता है। इसे सिस्टम उपकरण /**`usr/bin/lsappinfo`** के साथ या [**lsdtrip**](https://newosxbook.com/tools/lsdtrip.html) के साथ क्वेरी किया जा सकता है।

## फ़ाइल एक्सटेंशन और यूआरएल स्कीम ऐप हैंडलर

निम्नलिखित पंक्ति उन एप्लिकेशनों को खोजने के लिए उपयोगी हो सकती है जो एक्सटेंशन के आधार पर फ़ाइलें खोल सकती हैं:

{% code overflow="wrap" %}
```bash
/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister -dump | grep -E "path:|bindings:|name:"
```
{% endcode %}

या [**SwiftDefaultApps**](https://github.com/Lord-Kamina/SwiftDefaultApps) जैसी कुछ का उपयोग करें:
```bash
./swda getSchemes #Get all the available schemes
./swda getApps #Get all the apps declared
./swda getUTIs #Get all the UTIs
./swda getHandler --URL ftp #Get ftp handler
```
आप एक एप्लिकेशन द्वारा समर्थित एक्सटेंशन की जांच भी कर सकते हैं:
```
cd /Applications/Safari.app/Contents
grep -A3 CFBundleTypeExtensions Info.plist  | grep string
<string>css</string>
<string>pdf</string>
<string>webarchive</string>
<string>webbookmark</string>
<string>webhistory</string>
<string>webloc</string>
<string>download</string>
<string>safariextz</string>
<string>gif</string>
<string>html</string>
<string>htm</string>
<string>js</string>
<string>jpg</string>
<string>jpeg</string>
<string>jp2</string>
<string>txt</string>
<string>text</string>
<string>png</string>
<string>tiff</string>
<string>tif</string>
<string>url</string>
<string>ico</string>
<string>xhtml</string>
<string>xht</string>
<string>xml</string>
<string>xbl</string>
<string>svg</string>
```
{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
