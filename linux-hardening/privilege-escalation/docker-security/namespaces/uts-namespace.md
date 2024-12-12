# UTS Namespace

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Basic Information

एक UTS (UNIX टाइम-शेयरिंग सिस्टम) नामस्थान एक Linux कर्नेल विशेषता है जो **दो सिस्टम पहचानकर्ताओं** का **अलगाव** प्रदान करता है: **होस्टनाम** और **NIS** (नेटवर्क सूचना सेवा) डोमेन नाम। यह अलगाव प्रत्येक UTS नामस्थान को अपना **स्वतंत्र होस्टनाम और NIS डोमेन नाम** रखने की अनुमति देता है, जो कंटेनरीकरण परिदृश्यों में विशेष रूप से उपयोगी है जहां प्रत्येक कंटेनर को अपने स्वयं के होस्टनाम के साथ एक अलग सिस्टम के रूप में दिखाई देना चाहिए।

### How it works:

1. जब एक नया UTS नामस्थान बनाया जाता है, तो यह अपने माता-पिता नामस्थान से **होस्टनाम और NIS डोमेन नाम की एक प्रति** के साथ शुरू होता है। इसका मतलब है कि, निर्माण के समय, नया नामस्थान अपने माता-पिता के समान पहचानकर्ताओं को **साझा करता है**। हालाँकि, नामस्थान के भीतर होस्टनाम या NIS डोमेन नाम में बाद में किए गए किसी भी परिवर्तन का अन्य नामस्थानों पर कोई प्रभाव नहीं पड़ेगा।
2. UTS नामस्थान के भीतर प्रक्रियाएँ `sethostname()` और `setdomainname()` सिस्टम कॉल का उपयोग करके **होस्टनाम और NIS डोमेन नाम** बदल सकती हैं। ये परिवर्तन नामस्थान के लिए स्थानीय होते हैं और अन्य नामस्थानों या होस्ट सिस्टम को प्रभावित नहीं करते हैं।
3. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नामस्थानों के बीच स्थानांतरित हो सकती हैं या `CLONE_NEWUTS` ध्वज के साथ `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नामस्थान बना सकती हैं। जब एक प्रक्रिया नए नामस्थान में जाती है या एक बनाती है, तो यह उस नामस्थान से जुड़े होस्टनाम और NIS डोमेन नाम का उपयोग करना शुरू कर देगी।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -u [--mount-proc] /bin/bash
```
By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **सटीक और अलग दृष्टिकोण उस namespace के लिए विशिष्ट प्रक्रिया जानकारी**.

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो Linux द्वारा नए PID (Process ID) namespaces को संभालने के तरीके के कारण एक त्रुटि उत्पन्न होती है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या का विवरण**:
- Linux कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए namespaces बनाने की अनुमति देता है। हालाँकि, नए PID namespace (जिसे "unshare" प्रक्रिया कहा जाता है) के निर्माण की शुरुआत करने वाली प्रक्रिया नए namespace में नहीं जाती; केवल इसकी बाल प्रक्रियाएँ जाती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` उसी प्रक्रिया में शुरू होता है जैसे `unshare`। परिणामस्वरूप, `/bin/bash` और इसकी बाल प्रक्रियाएँ मूल PID namespace में होती हैं।
- नए namespace में `/bin/bash` की पहली बाल प्रक्रिया PID 1 बन जाती है। जब यह प्रक्रिया समाप्त होती है, तो यह namespace की सफाई को ट्रिगर करती है यदि कोई अन्य प्रक्रियाएँ नहीं हैं, क्योंकि PID 1 का विशेष कार्य अनाथ प्रक्रियाओं को अपनाना है। Linux कर्नेल तब उस namespace में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए namespace में PID 1 का समाप्त होना `PIDNS_HASH_ADDING` ध्वज की सफाई की ओर ले जाता है। इसके परिणामस्वरूप, नए प्रक्रिया बनाने के दौरान `alloc_pid` फ़ंक्शन नए PID आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" त्रुटि उत्पन्न होती है।

3. **समाधान**:
- समस्या को `unshare` के साथ `-f` विकल्प का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID namespace बनाने के बाद एक नई प्रक्रिया बनाने के लिए फोर्क करता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से यह सुनिश्चित होता है कि `unshare` कमांड स्वयं नए namespace में PID 1 बन जाता है। `/bin/bash` और इसकी बाल प्रक्रियाएँ फिर इस नए namespace में सुरक्षित रूप से समाहित होती हैं, PID 1 के पूर्ववर्ती समाप्त होने को रोकती हैं और सामान्य PID आवंटन की अनुमति देती हैं।

यह सुनिश्चित करके कि `unshare` `-f` ध्वज के साथ चलता है, नया PID namespace सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और इसकी उप-प्रक्रियाएँ बिना मेमोरी आवंटन त्रुटि का सामना किए कार्य कर सकती हैं।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपका प्रोसेस किस नामस्थान में है
```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```
### सभी UTS नामस्थान खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### UTS नामस्थान के अंदर प्रवेश करें
```bash
nsenter -u TARGET_PID --pid /bin/bash
```
{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे साथ जुड़ें** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो करें**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें.

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
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
