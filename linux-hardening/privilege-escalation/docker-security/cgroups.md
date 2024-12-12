# CGroups

{% hint style="success" %}
**AWS हैकिंग सीखें और प्रैक्टिस करें:** [**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)\
**GCP हैकिंग सीखें और प्रैक्टिस करें:** [**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## मौलिक जानकारी

**लिनक्स कंट्रोल ग्रुप्स**, या **cgroups**, लिनक्स कर्नेल की एक विशेषता है जो प्रक्रिया समूहों के बीच CPU, मेमोरी, और डिस्क I/O जैसे सिस्टम संसाधनों का आवंटन, सीमितीकरण, और प्राथमिकताकरण संभावित बनाती है। ये **प्रक्रिया संग्रहों के संसाधन उपयोग का प्रबंधन और विलयन** के लिए एक तंत्र प्रदान करते हैं, जो संसाधन सीमितीकरण, वर्कलोड विलयन, और विभिन्न प्रक्रिया समूहों के बीच संसाधन प्राथमिकता के लिए उपयोगी है।

**cgroups के दो संस्करण** हैं: संस्करण 1 और संस्करण 2। दोनों को सिस्टम पर समकालिक रूप से उपयोग किया जा सकता है। मुख्य भिन्नता यह है कि **cgroups संस्करण 2** एक **वृक्षाकार, पेड़-जैसा संरचना** पेश करता है, जो प्रक्रिया समूहों के बीच अधिक सूक्ष्म और विस्तृत संसाधन वितरण को संभावित बनाता है। साथ ही, संस्करण 2 नए संसाधन नियंत्रकों का समर्थन, पुराने अनुप्रयोगों के लिए बेहतर समर्थन, और बेहतर प्रदर्शन जैसे विभिन्न सुधार भी लाता है।

समग्र रूप से, cgroups **संस्करण 2 अधिक सुविधाएँ और बेहतर प्रदर्शन** प्रदान करता है संस्करण 1 से, लेकिन पिछला अभी भी उस स्थितियों में उपयोग किया जा सकता है जहाँ पुराने सिस्टमों के साथ संगतता की चिंता है।

आप किसी प्रक्रिया के cgroups देखकर v1 और v2 cgroups को सूचीबद्ध कर सकते हैं /proc/\<pid> में उसके cgroup फ़ाइल को देखकर। इस कमांड से अपने शैल के cgroups को देखने से आप शुरू कर सकते हैं:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
### देखना cgroups

फ़ाइल सिस्टम सामान्यत: **cgroups** तक पहुँचने के लिए उपयोग किया जाता है, जो कि कर्नेल इंटरेक्शन के लिए पारंपरिक रूप से उपयोग किया जाने वाला यूनिक्स सिस्टम कॉल इंटरफेस से भिन्न है। एक शैल के cgroup विन्यास की जांच करने के लिए, किसी भी शैल के cgroup को पता करने के लिए, व्यक्तिगत सेटिंग्स और संसाधन उपयोग सूचना को देख सकते हैं।

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

cgroups के मुख्य इंटरफेस फ़ाइलों का उपसर्ग **cgroup** के साथ होता है। **cgroup.procs** फ़ाइल, जिसे cat जैसे मानक कमांड्स के साथ देखा जा सकता है, cgroup में प्रक्रियाएँ सूचीबद्ध करती है। एक और फ़ाइल, **cgroup.threads**, धागा सूचना शामिल करती है।

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

शैल को प्रबंधित करने वाले cgroups सामान्यत: दो नियंत्रकों को सम्मिलित करते हैं जो मेमोरी उपयोग और प्रक्रिया गणना को विनियंत्रित करते हैं। नियंत्रक के उपसर्ग वाली फ़ाइलों के साथ इंटरैक्ट करने के लिए जांच की जानी चाहिए। उदाहरण के लिए, **pids.current** को संदर्भित किया जाएगा ताकि cgroup में धागों की गणना की जा सके।

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

एक मान में **max** का संकेत देता है कि cgroup के लिए कोई विशिष्ट सीमा नहीं है। हालांकि, cgroups की पुंजीय नियमिता के कारण, सीमाएँ निचले स्तर पर किसी cgroup द्वारा लगाई जा सकती हैं।

### cgroups को मानिपुलेट और बनाना

प्रक्रियाएँ cgroups को **उनके प्रक्रिया आईडी (PID) को `cgroup.procs` फ़ाइल में लिखकर** दिए जाते हैं। इसके लिए रूट विशेषाधिकार चाहिए होते हैं। उदाहरण के लिए, एक प्रक्रिया जोड़ने के लिए:
```bash
echo [pid] > cgroup.procs
```
इसी तरह, **cgroup गुणों को संशोधित करना, जैसे कि एक PID सीमा सेट करना**, चाहे तो उचित मान को संबंधित फ़ाइल में लिखकर किया जाता है। एक cgroup के लिए 3,000 PIDs की अधिकतम सीमा सेट करने के लिए:
```bash
echo 3000 > pids.max
```
**नए cgroups बनाना** cgroup विभाजन के भीतर एक नया उपनिर्देशिका बनाने का काम है, जिससे कर्नेल को स्वचालित रूप से आवश्यक इंटरफेस फ़ाइलें उत्पन्न करने के लिए प्रेरित किया जाता है। `rmdir` के साथ हटाई जा सकती हैं, लेकिन कुछ नियंत्रणों की जानकारी रखें:

* **प्रक्रियाएँ केवल पत्ते cgroups में रखी जा सकती हैं** (अर्थात, विभाजन में सबसे अंतिम वाले).
* **एक cgroup अपने माता-पिता में अनुपस्थित नियंत्रक को नहीं धारण कर सकता**।
* **बाल cgroups के लिए नियंत्रकों को स्पष्ट रूप से घोषित किया जाना चाहिए** `cgroup.subtree_control` फ़ाइल में। उदाहरण के लिए, एक बाल cgroup में CPU और PID नियंत्रकों को सक्षम करने के लिए:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
**रूट सीग्रुप** इन नियमों की एक अपवाद है, जो सीधे प्रक्रिया स्थानांतरण की अनुमति देता है। इसका उपयोग प्रक्रियाओं को सिस्टमडी प्रबंधन से हटाने के लिए किया जा सकता है।

**सीग्रुप के भीतर CPU उपयोग का मॉनिटरिंग** `cpu.stat` फ़ाइल के माध्यम से संभव है, जो कुल CPU समय का उपभोक्त होना प्रदर्शित करता है, सेवा के सबप्रोसेस के उपयोग को ट्रैक करने के लिए उपयोगी है:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>cpu.stat फ़ाइल में दिखाए गए CPU उपयोग सांख्यिकी</p></figcaption></figure>

## संदर्भ

* **पुस्तक: हाउ लिनक्स वर्क्स, 3 वीं संस्करण: ब्रायन वार्ड द्वारा हर सुपरयूजर को क्या पता होना चाहिए**

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर हमें फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
