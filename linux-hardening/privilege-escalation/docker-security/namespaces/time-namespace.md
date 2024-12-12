# Time Namespace

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Basic Information

Linux में टाइम नेमस्पेस सिस्टम मोनोटोनिक और बूट-टाइम घड़ियों के लिए प्रति-नेमस्पेस ऑफसेट की अनुमति देता है। इसका सामान्य उपयोग Linux कंटेनरों में कंटेनर के भीतर दिनांक/समय को बदलने और चेकपॉइंट या स्नैपशॉट से पुनर्स्थापित करने के बाद घड़ियों को समायोजित करने के लिए किया जाता है।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -T [--mount-proc] /bin/bash
```
By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **सटीक और अलग दृष्टिकोण उस नामस्थान के लिए विशिष्ट प्रक्रिया जानकारी**.

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो लिनक्स नए PID (प्रक्रिया आईडी) नामस्थान को संभालने के तरीके के कारण एक त्रुटि उत्पन्न होती है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या का विवरण**:
- लिनक्स कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नामस्थान बनाने की अनुमति देता है। हालाँकि, नए PID नामस्थान के निर्माण की शुरुआत करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नामस्थान में प्रवेश नहीं करती है; केवल इसकी बाल प्रक्रियाएँ करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` उसी प्रक्रिया में शुरू होता है जैसे `unshare`। परिणामस्वरूप, `/bin/bash` और इसकी बाल प्रक्रियाएँ मूल PID नामस्थान में होती हैं।
- नए नामस्थान में `/bin/bash` की पहली बाल प्रक्रिया PID 1 बन जाती है। जब यह प्रक्रिया समाप्त होती है, तो यह नामस्थान की सफाई को ट्रिगर करती है यदि कोई अन्य प्रक्रियाएँ नहीं हैं, क्योंकि PID 1 का विशेष कार्य अनाथ प्रक्रियाओं को अपनाना है। लिनक्स कर्नेल तब उस नामस्थान में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नामस्थान में PID 1 का समाप्त होना `PIDNS_HASH_ADDING` ध्वज की सफाई की ओर ले जाता है। इसका परिणाम यह होता है कि `alloc_pid` फ़ंक्शन नए प्रक्रिया बनाने के समय नया PID आवंटित करने में विफल रहता है, जिससे "मेमोरी आवंटित नहीं कर सकता" त्रुटि उत्पन्न होती है।

3. **समाधान**:
- समस्या को `unshare` के साथ `-f` विकल्प का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नामस्थान बनाने के बाद एक नई प्रक्रिया बनाने के लिए मजबूर करता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से यह सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नामस्थान में PID 1 बन जाता है। `/bin/bash` और इसकी बाल प्रक्रियाएँ फिर इस नए नामस्थान में सुरक्षित रूप से समाहित होती हैं, PID 1 के पूर्ववर्ती समाप्त होने को रोकती हैं और सामान्य PID आवंटन की अनुमति देती हैं।

यह सुनिश्चित करके कि `unshare` `-f` ध्वज के साथ चलता है, नया PID नामस्थान सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और इसकी उप-प्रक्रियाएँ बिना मेमोरी आवंटन त्रुटि का सामना किए कार्य कर सकें।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपका प्रक्रिया किस नामस्थान में है
```bash
ls -l /proc/self/ns/time
lrwxrwxrwx 1 root root 0 Apr  4 21:16 /proc/self/ns/time -> 'time:[4026531834]'
```
### सभी टाइम नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name time -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name time -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### टाइम नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -T TARGET_PID --pid /bin/bash
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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

{% endhint %}
</details>
{% endhint %}
