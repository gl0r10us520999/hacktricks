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


# Baseline

एक बेसलाइन में सिस्टम के कुछ हिस्सों का स्नैपशॉट लेना शामिल होता है ताकि **भविष्य की स्थिति के साथ इसकी तुलना की जा सके और परिवर्तनों को उजागर किया जा सके**।

उदाहरण के लिए, आप फ़ाइल सिस्टम की प्रत्येक फ़ाइल का हैश निकाल सकते हैं और उसे स्टोर कर सकते हैं ताकि यह पता चल सके कि कौन सी फ़ाइलें संशोधित की गई थीं।\
यह उपयोगकर्ता खातों, चल रहे प्रक्रियाओं, चल रही सेवाओं और किसी भी अन्य चीज़ के साथ भी किया जा सकता है जो बहुत अधिक नहीं बदलनी चाहिए, या बिल्कुल भी नहीं।

## File Integrity Monitoring

File Integrity Monitoring (FIM) एक महत्वपूर्ण सुरक्षा तकनीक है जो फ़ाइलों में परिवर्तनों को ट्रैक करके IT वातावरण और डेटा की सुरक्षा करती है। इसमें दो प्रमुख चरण शामिल हैं:

1. **बेसलाइन तुलना:** फ़ाइल विशेषताओं या क्रिप्टोग्राफिक चेकसम (जैसे MD5 या SHA-2) का उपयोग करके एक बेसलाइन स्थापित करें ताकि संशोधनों का पता लगाने के लिए भविष्य की तुलना की जा सके।
2. **वास्तविक समय परिवर्तन सूचना:** जब फ़ाइलों को एक्सेस या संशोधित किया जाता है, तो तात्कालिक अलर्ट प्राप्त करें, आमतौर पर OS कर्नेल एक्सटेंशन के माध्यम से।

## Tools

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

## References

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


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
