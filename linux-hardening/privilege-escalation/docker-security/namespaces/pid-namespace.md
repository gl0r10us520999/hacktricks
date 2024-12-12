# PID Namespace

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

## Basic Information

PID (Process IDentifier) namespace एक विशेषता है जो Linux kernel में प्रक्रिया अलगाव प्रदान करती है, जिससे प्रक्रियाओं के एक समूह को अपने स्वयं के अद्वितीय PIDs का सेट प्राप्त होता है, जो अन्य namespaces में PIDs से अलग होता है। यह कंटेनरीकरण में विशेष रूप से उपयोगी है, जहां प्रक्रिया अलगाव सुरक्षा और संसाधन प्रबंधन के लिए आवश्यक है।

जब एक नया PID namespace बनाया जाता है, तो उस namespace में पहली प्रक्रिया को PID 1 सौंपा जाता है। यह प्रक्रिया नए namespace की "init" प्रक्रिया बन जाती है और namespace के भीतर अन्य प्रक्रियाओं का प्रबंधन करने के लिए जिम्मेदार होती है। namespace के भीतर बनाई गई प्रत्येक अगली प्रक्रिया का एक अद्वितीय PID होगा, और ये PIDs अन्य namespaces में PIDs से स्वतंत्र होंगे।

PID namespace के भीतर एक प्रक्रिया के दृष्टिकोण से, यह केवल उसी namespace में अन्य प्रक्रियाओं को देख सकती है। यह अन्य namespaces में प्रक्रियाओं के बारे में अवगत नहीं होती है, और यह पारंपरिक प्रक्रिया प्रबंधन उपकरणों (जैसे, `kill`, `wait`, आदि) का उपयोग करके उनके साथ बातचीत नहीं कर सकती। यह एक स्तर का अलगाव प्रदान करता है जो प्रक्रियाओं को एक-दूसरे में हस्तक्षेप करने से रोकने में मदद करता है।

### How it works:

1. जब एक नई प्रक्रिया बनाई जाती है (जैसे, `clone()` सिस्टम कॉल का उपयोग करके), तो प्रक्रिया को एक नए या मौजूदा PID namespace में सौंपा जा सकता है। **यदि एक नया namespace बनाया जाता है, तो प्रक्रिया उस namespace की "init" प्रक्रिया बन जाती है**।
2. **kernel** नए namespace में PIDs और संबंधित PIDs के बीच एक **मैपिंग बनाए रखता है** जो माता-पिता namespace में होते हैं (यानी, वह namespace जिससे नया namespace बनाया गया था)। यह मैपिंग **kernel को आवश्यकतानुसार PIDs का अनुवाद करने की अनुमति देती है**, जैसे कि विभिन्न namespaces में प्रक्रियाओं के बीच संकेत भेजते समय।
3. **PID namespace के भीतर प्रक्रियाएँ केवल उसी namespace में अन्य प्रक्रियाओं को देख और बातचीत कर सकती हैं**। वे अन्य namespaces में प्रक्रियाओं के बारे में अवगत नहीं होती हैं, और उनके PIDs उनके namespace के भीतर अद्वितीय होते हैं।
4. जब एक **PID namespace नष्ट किया जाता है** (जैसे, जब namespace की "init" प्रक्रिया समाप्त होती है), **तो उस namespace के भीतर सभी प्रक्रियाएँ समाप्त हो जाती हैं**। यह सुनिश्चित करता है कि namespace से संबंधित सभी संसाधनों को सही ढंग से साफ किया जाए।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -pf --mount-proc /bin/bash
```
<details>

<summary>Error: bash: fork: Cannot allocate memory</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो एक त्रुटि उत्पन्न होती है क्योंकि Linux नए PID (Process ID) namespaces को संभालने के तरीके के कारण। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या का विवरण**:
- Linux कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए namespaces बनाने की अनुमति देता है। हालाँकि, नई PID namespace (जिसे "unshare" प्रक्रिया कहा जाता है) बनाने की प्रक्रिया नए namespace में प्रवेश नहीं करती है; केवल इसकी संतान प्रक्रियाएँ करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` उसी प्रक्रिया में शुरू होता है जैसे `unshare`। परिणामस्वरूप, `/bin/bash` और इसकी संतान प्रक्रियाएँ मूल PID namespace में होती हैं।
- नए namespace में `/bin/bash` की पहली संतान प्रक्रिया PID 1 बन जाती है। जब यह प्रक्रिया समाप्त होती है, तो यह namespace की सफाई को ट्रिगर करती है यदि कोई अन्य प्रक्रियाएँ नहीं हैं, क्योंकि PID 1 का अनाथ प्रक्रियाओं को अपनाने की विशेष भूमिका होती है। Linux कर्नेल तब उस namespace में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए namespace में PID 1 का समाप्त होना `PIDNS_HASH_ADDING` ध्वज की सफाई की ओर ले जाता है। इसका परिणाम यह होता है कि `alloc_pid` फ़ंक्शन नए PID को आवंटित करने में विफल रहता है जब एक नई प्रक्रिया बनाई जाती है, जिससे "Cannot allocate memory" त्रुटि उत्पन्न होती है।

3. **समाधान**:
- इस समस्या को `unshare` के साथ `-f` विकल्प का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID namespace बनाने के बाद एक नई प्रक्रिया बनाने के लिए फोर्क करता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से यह सुनिश्चित होता है कि `unshare` कमांड स्वयं नए namespace में PID 1 बन जाता है। `/bin/bash` और इसकी संतान प्रक्रियाएँ फिर इस नए namespace में सुरक्षित रूप से समाहित होती हैं, PID 1 के पूर्ववर्ती समाप्त होने को रोकती हैं और सामान्य PID आवंटन की अनुमति देती हैं।

यह सुनिश्चित करके कि `unshare` `-f` ध्वज के साथ चलता है, नया PID namespace सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और इसकी उप-प्रक्रियाएँ बिना मेमोरी आवंटन त्रुटि का सामना किए कार्य कर सकें।

</details>

यदि आप पैरामीटर `--mount-proc` का उपयोग करके `/proc` फ़ाइल प्रणाली का एक नया उदाहरण माउंट करते हैं, तो आप सुनिश्चित करते हैं कि नए माउंट namespace में उस namespace के लिए विशिष्ट प्रक्रिया जानकारी का **सटीक और अलगावित दृश्य** है।

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपका प्रोसेस किस नामस्थान में है
```bash
ls -l /proc/self/ns/pid
lrwxrwxrwx 1 root root 0 Apr  3 18:45 /proc/self/ns/pid -> 'pid:[4026532412]'
```
### सभी PID नामस्थान खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name pid -exec readlink {} \; 2>/dev/null | sort -u
```
{% endcode %}

ध्यान दें कि प्रारंभिक (डिफ़ॉल्ट) PID नामस्थान से रूट उपयोगकर्ता सभी प्रक्रियाओं को देख सकता है, यहां तक कि नए PID नामस्थान में भी, यही कारण है कि हम सभी PID नामस्थान देख सकते हैं।

### एक PID नामस्थान के अंदर प्रवेश करें
```bash
nsenter -t TARGET_PID --pid /bin/bash
```
जब आप डिफ़ॉल्ट नामस्थान से PID नामस्थान के अंदर प्रवेश करते हैं, तो आप सभी प्रक्रियाओं को देख पाएंगे। और उस PID ns से प्रक्रिया नए bash को PID ns पर देख सकेगी।

इसके अलावा, आप केवल **दूसरे प्रक्रिया PID नामस्थान में प्रवेश कर सकते हैं यदि आप रूट हैं**। और आप **दूसरे नामस्थान में** **बिना एक वर्णनकर्ता** के **प्रवेश नहीं कर सकते** जो इसे इंगित करता है (जैसे `/proc/self/ns/pid`)

## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)
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
</details>
{% endhint %}
