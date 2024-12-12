# IPC Namespace

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

एक IPC (Inter-Process Communication) नामस्थान एक Linux कर्नेल सुविधा है जो **System V IPC वस्तुओं** जैसे संदेश कतारों, साझा मेमोरी खंडों और सेमाफोर का **अलगाव** प्रदान करती है। यह अलगाव सुनिश्चित करता है कि **विभिन्न IPC नामस्थानों में प्रक्रियाएँ एक-दूसरे की IPC वस्तुओं को सीधे एक्सेस या संशोधित नहीं कर सकतीं**, जो प्रक्रिया समूहों के बीच सुरक्षा और गोपनीयता की एक अतिरिक्त परत प्रदान करता है।

### How it works:

1. जब एक नया IPC नामस्थान बनाया जाता है, तो यह **System V IPC वस्तुओं का एक पूरी तरह से अलग सेट** के साथ शुरू होता है। इसका मतलब है कि नए IPC नामस्थान में चल रही प्रक्रियाएँ डिफ़ॉल्ट रूप से अन्य नामस्थानों या होस्ट सिस्टम में IPC वस्तुओं को एक्सेस या हस्तक्षेप नहीं कर सकतीं।
2. एक नामस्थान के भीतर बनाई गई IPC वस्तुएँ केवल **उस नामस्थान के भीतर की प्रक्रियाओं के लिए दृश्य और सुलभ होती हैं**। प्रत्येक IPC वस्तु को अपने नामस्थान के भीतर एक अद्वितीय कुंजी द्वारा पहचाना जाता है। हालांकि कुंजी विभिन्न नामस्थानों में समान हो सकती है, वस्तुएँ स्वयं अलग हैं और नामस्थानों के बीच एक्सेस नहीं की जा सकतीं।
3. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नामस्थानों के बीच स्थानांतरित हो सकती हैं या `CLONE_NEWIPC` ध्वज के साथ `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नामस्थान बना सकती हैं। जब एक प्रक्रिया नए नामस्थान में जाती है या एक बनाती है, तो यह उस नामस्थान से संबंधित IPC वस्तुओं का उपयोग करना शुरू कर देगी।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **सटीक और अलग दृष्टिकोण उस नामस्थान के लिए विशिष्ट प्रक्रिया जानकारी**.

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो लिनक्स नए PID (प्रक्रिया आईडी) नामस्थान को संभालने के तरीके के कारण एक त्रुटि उत्पन्न होती है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या का विवरण**:
- लिनक्स कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नामस्थान बनाने की अनुमति देता है। हालाँकि, नए PID नामस्थान के निर्माण की शुरुआत करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नामस्थान में प्रवेश नहीं करती है; केवल इसकी बाल प्रक्रियाएँ करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` उसी प्रक्रिया में शुरू होता है जैसे `unshare`। परिणामस्वरूप, `/bin/bash` और इसकी बाल प्रक्रियाएँ मूल PID नामस्थान में होती हैं।
- नए नामस्थान में `/bin/bash` की पहली बाल प्रक्रिया PID 1 बन जाती है। जब यह प्रक्रिया समाप्त होती है, तो यह नामस्थान की सफाई को ट्रिगर करती है यदि कोई अन्य प्रक्रियाएँ नहीं हैं, क्योंकि PID 1 का अनाथ प्रक्रियाओं को अपनाने की विशेष भूमिका होती है। लिनक्स कर्नेल तब उस नामस्थान में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नामस्थान में PID 1 का समाप्त होना `PIDNS_HASH_ADDING` ध्वज की सफाई की ओर ले जाता है। इसका परिणाम यह होता है कि `alloc_pid` फ़ंक्शन नए प्रक्रिया बनाने के दौरान नया PID आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" त्रुटि उत्पन्न होती है।

3. **समाधान**:
- इस समस्या को `unshare` के साथ `-f` विकल्प का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नामस्थान बनाने के बाद एक नई प्रक्रिया बनाने के लिए फोर्क करता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से यह सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नामस्थान में PID 1 बन जाता है। `/bin/bash` और इसकी बाल प्रक्रियाएँ फिर इस नए नामस्थान में सुरक्षित रूप से समाहित होती हैं, PID 1 के पूर्व समय में समाप्त होने को रोकती हैं और सामान्य PID आवंटन की अनुमति देती हैं।

यह सुनिश्चित करके कि `unshare` `-f` ध्वज के साथ चलता है, नया PID नामस्थान सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और इसकी उप-प्रक्रियाएँ बिना मेमोरी आवंटन त्रुटि का सामना किए कार्य कर सकें।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपका प्रक्रिया किस नामस्थान में है
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### सभी IPC नामस्थान खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### IPC नामस्थान के अंदर प्रवेश करें
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
आप केवल **दूसरे प्रक्रिया नामस्थान में प्रवेश कर सकते हैं यदि आप रूट हैं**। और आप **दूसरे नामस्थान में प्रवेश नहीं कर सकते** **बिना एक वर्णनकर्ता** जो इसे इंगित करता है (जैसे `/proc/self/ns/net`)।

### IPC ऑब्जेक्ट बनाएं
```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
```
## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
</details>
{% endhint %}
