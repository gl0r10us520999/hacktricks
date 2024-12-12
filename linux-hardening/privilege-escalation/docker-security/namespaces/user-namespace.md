# User Namespace

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

## Basic Information

एक उपयोगकर्ता नामस्थान एक Linux कर्नेल सुविधा है जो **उपयोगकर्ता और समूह आईडी मैपिंग का पृथक्करण प्रदान करती है**, जिससे प्रत्येक उपयोगकर्ता नामस्थान को **अपने स्वयं के उपयोगकर्ता और समूह आईडी का सेट** रखने की अनुमति मिलती है। यह पृथक्करण विभिन्न उपयोगकर्ता नामस्थानों में चलने वाली प्रक्रियाओं को **विभिन्न विशेषाधिकार और स्वामित्व** रखने की अनुमति देता है, भले ही वे संख्यात्मक रूप से समान उपयोगकर्ता और समूह आईडी साझा करते हों।

उपयोगकर्ता नामस्थान विशेष रूप से कंटेनरीकरण में उपयोगी होते हैं, जहां प्रत्येक कंटेनर को अपने स्वयं के स्वतंत्र उपयोगकर्ता और समूह आईडी का सेट होना चाहिए, जिससे कंटेनरों और होस्ट सिस्टम के बीच बेहतर सुरक्षा और पृथक्करण की अनुमति मिलती है।

### How it works:

1. जब एक नया उपयोगकर्ता नामस्थान बनाया जाता है, तो यह **उपयोगकर्ता और समूह आईडी मैपिंग का एक खाली सेट के साथ शुरू होता है**। इसका मतलब है कि नए उपयोगकर्ता नामस्थान में चलने वाली कोई भी प्रक्रिया **शुरुआत में नामस्थान के बाहर कोई विशेषाधिकार नहीं रखेगी**।
2. नए नामस्थान में उपयोगकर्ता और समूह आईडी और माता-पिता (या होस्ट) नामस्थान में आईडी के बीच मैपिंग स्थापित की जा सकती है। यह **नए नामस्थान में प्रक्रियाओं को माता-पिता नामस्थान में उपयोगकर्ता और समूह आईडी के अनुसार विशेषाधिकार और स्वामित्व रखने की अनुमति देता है**। हालाँकि, आईडी मैपिंग को विशिष्ट रेंज और आईडी के उपसमुच्चयों तक सीमित किया जा सकता है, जिससे नए नामस्थान में प्रक्रियाओं को दिए गए विशेषाधिकार पर बारीक नियंत्रण की अनुमति मिलती है।
3. एक उपयोगकर्ता नामस्थान के भीतर, **प्रक्रियाओं के पास नामस्थान के भीतर संचालन के लिए पूर्ण रूट विशेषाधिकार (UID 0) हो सकते हैं**, जबकि नामस्थान के बाहर सीमित विशेषाधिकार भी हो सकते हैं। यह **कंटेनरों को अपने स्वयं के नामस्थान के भीतर रूट-जैसे क्षमताओं के साथ चलाने की अनुमति देता है बिना होस्ट सिस्टम पर पूर्ण रूट विशेषाधिकार के**।
4. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नामस्थानों के बीच स्थानांतरित हो सकती हैं या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नामस्थान बना सकती हैं जिसमें `CLONE_NEWUSER` ध्वज होता है। जब कोई प्रक्रिया नए नामस्थान में जाती है या एक बनाती है, तो यह उस नामस्थान से संबंधित उपयोगकर्ता और समूह आईडी मैपिंग का उपयोग करना शुरू कर देगी।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -U [--mount-proc] /bin/bash
```
By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **सटीक और अलग दृष्टिकोण उस नामस्थान के लिए विशिष्ट प्रक्रिया जानकारी**.

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो एक त्रुटि उत्पन्न होती है क्योंकि Linux नए PID (प्रक्रिया आईडी) नामस्थान को संभालने के तरीके के कारण। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या का विवरण**:
- Linux कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नामस्थान बनाने की अनुमति देता है। हालाँकि, नए PID नामस्थान के निर्माण की शुरुआत करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नामस्थान में प्रवेश नहीं करती है; केवल इसकी बाल प्रक्रियाएँ करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` उसी प्रक्रिया में शुरू होता है जैसे `unshare`। परिणामस्वरूप, `/bin/bash` और इसकी बाल प्रक्रियाएँ मूल PID नामस्थान में होती हैं।
- नए नामस्थान में `/bin/bash` की पहली बाल प्रक्रिया PID 1 बन जाती है। जब यह प्रक्रिया समाप्त होती है, तो यह नामस्थान की सफाई को ट्रिगर करती है यदि कोई अन्य प्रक्रियाएँ नहीं हैं, क्योंकि PID 1 का विशेष कार्य अनाथ प्रक्रियाओं को अपनाना है। Linux कर्नेल तब उस नामस्थान में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नामस्थान में PID 1 का समाप्त होना `PIDNS_HASH_ADDING` ध्वज की सफाई की ओर ले जाता है। इसका परिणाम यह होता है कि `alloc_pid` फ़ंक्शन नए प्रक्रिया बनाने के समय नया PID आवंटित करने में विफल रहता है, जिससे "Cannot allocate memory" त्रुटि उत्पन्न होती है।

3. **समाधान**:
- इस समस्या को `unshare` के साथ `-f` विकल्प का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नामस्थान बनाने के बाद एक नई प्रक्रिया बनाने के लिए फोर्क करता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से यह सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नामस्थान में PID 1 बन जाता है। `/bin/bash` और इसकी बाल प्रक्रियाएँ फिर इस नए नामस्थान में सुरक्षित रूप से समाहित होती हैं, PID 1 के पूर्ववर्ती समाप्त होने को रोकती हैं और सामान्य PID आवंटन की अनुमति देती हैं।

यह सुनिश्चित करके कि `unshare` `-f` ध्वज के साथ चलता है, नया PID नामस्थान सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और इसकी उप-प्रक्रियाएँ बिना मेमोरी आवंटन त्रुटि का सामना किए कार्य कर सकें।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
To use user namespace, Docker daemon needs to be started with **`--userns-remap=default`**(Ubuntu 14.04 में, इसे `/etc/default/docker` को संशोधित करके और फिर `sudo service docker restart` चलाकर किया जा सकता है)

### &#x20;Check which namespace is your process in
```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```
यह संभव है कि आप docker कंटेनर से उपयोगकर्ता मानचित्र की जांच कर सकें:
```bash
cat /proc/self/uid_map
0          0 4294967295  --> Root is root in host
0     231072      65536  --> Root is 231072 userid in host
```
या होस्ट से:
```bash
cat /proc/<pid>/uid_map
```
### सभी उपयोगकर्ता नामस्थान खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### एक उपयोगकर्ता नामस्थान के अंदर प्रवेश करें
```bash
nsenter -U TARGET_PID --pid /bin/bash
```
आप केवल **दूसरे प्रक्रिया नामस्थान में प्रवेश कर सकते हैं यदि आप रूट हैं**। और आप **दूसरे नामस्थान में प्रवेश नहीं कर सकते** **बिना एक वर्णनकर्ता** जो इसे इंगित करता है (जैसे `/proc/self/ns/user`)।

### नया उपयोगकर्ता नामस्थान बनाएं (मैपिंग के साथ)

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
{% endcode %}
```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```
### Recovering Capabilities

In the case of user namespaces, **जब एक नया उपयोगकर्ता नामस्थान बनाया जाता है, तो उस नामस्थान में प्रवेश करने वाली प्रक्रिया को उस नामस्थान के भीतर पूर्ण क्षमताओं का एक सेट दिया जाता है**। ये क्षमताएँ प्रक्रिया को विशेषाधिकार प्राप्त संचालन करने की अनुमति देती हैं जैसे कि **फाइल सिस्टम को माउंट करना**, उपकरण बनाना, या फ़ाइलों के स्वामित्व को बदलना, लेकिन **केवल इसके उपयोगकर्ता नामस्थान के संदर्भ में**।

उदाहरण के लिए, जब आपके पास एक उपयोगकर्ता नामस्थान के भीतर `CAP_SYS_ADMIN` क्षमता होती है, तो आप ऐसे संचालन कर सकते हैं जो आमतौर पर इस क्षमता की आवश्यकता होती है, जैसे कि फाइल सिस्टम को माउंट करना, लेकिन केवल आपके उपयोगकर्ता नामस्थान के संदर्भ में। इस क्षमता के साथ किए गए कोई भी संचालन मेज़बान प्रणाली या अन्य नामस्थानों को प्रभावित नहीं करेंगे।

{% hint style="warning" %}
इसलिए, भले ही एक नए उपयोगकर्ता नामस्थान के भीतर एक नई प्रक्रिया प्राप्त करना **आपको सभी क्षमताएँ वापस देगा** (CapEff: 000001ffffffffff), आप वास्तव में **केवल नामस्थान से संबंधित क्षमताओं का उपयोग कर सकते हैं** (उदाहरण के लिए माउंट) लेकिन हर एक का नहीं। इसलिए, यह अपने आप में एक Docker कंटेनर से भागने के लिए पर्याप्त नहीं है।
{% endhint %}
```bash
# There are the syscalls that are filtered after changing User namespace with:
unshare -UmCpf  bash

Probando: 0x067 . . . Error
Probando: 0x070 . . . Error
Probando: 0x074 . . . Error
Probando: 0x09b . . . Error
Probando: 0x0a3 . . . Error
Probando: 0x0a4 . . . Error
Probando: 0x0a7 . . . Error
Probando: 0x0a8 . . . Error
Probando: 0x0aa . . . Error
Probando: 0x0ab . . . Error
Probando: 0x0af . . . Error
Probando: 0x0b0 . . . Error
Probando: 0x0f6 . . . Error
Probando: 0x12c . . . Error
Probando: 0x130 . . . Error
Probando: 0x139 . . . Error
Probando: 0x140 . . . Error
Probando: 0x141 . . . Error
```
{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमारा अनुसरण करें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PRs सबमिट करें।

</details>
{% endhint %}hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

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
