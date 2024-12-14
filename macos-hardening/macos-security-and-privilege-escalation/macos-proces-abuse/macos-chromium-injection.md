# macOS Chromium Injection

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

## Basic Information

Chromium-आधारित ब्राउज़र जैसे Google Chrome, Microsoft Edge, Brave, और अन्य। ये ब्राउज़र Chromium ओपन-सोर्स प्रोजेक्ट पर आधारित हैं, जिसका मतलब है कि वे एक सामान्य आधार साझा करते हैं और इसलिए, उनकी कार्यक्षमताएँ और डेवलपर विकल्प समान होते हैं।

#### `--load-extension` Flag

`--load-extension` फ्लैग का उपयोग कमांड लाइन या स्क्रिप्ट से Chromium-आधारित ब्राउज़र शुरू करते समय किया जाता है। यह फ्लैग ब्राउज़र के स्टार्टअप पर **एक या अधिक एक्सटेंशन को स्वचालित रूप से लोड करने** की अनुमति देता है।

#### `--use-fake-ui-for-media-stream` Flag

`--use-fake-ui-for-media-stream` फ्लैग एक और कमांड-लाइन विकल्प है जिसका उपयोग Chromium-आधारित ब्राउज़र शुरू करने के लिए किया जा सकता है। यह फ्लैग **कैमरा और माइक्रोफोन से मीडिया स्ट्रीम तक पहुँचने के लिए अनुमति मांगने वाले सामान्य उपयोगकर्ता प्रॉम्प्ट को बायपास करने** के लिए डिज़ाइन किया गया है। जब इस फ्लैग का उपयोग किया जाता है, तो ब्राउज़र किसी भी वेबसाइट या एप्लिकेशन को स्वचालित रूप से अनुमति देता है जो कैमरा या माइक्रोफोन तक पहुँचने का अनुरोध करता है।

### Tools

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### Example
```bash
# Intercept traffic
voodoo intercept -b chrome
```
Find more examples in the tools links

## References

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
