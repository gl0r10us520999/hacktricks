# संवेदनशील माउंट्स

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc` और `/sys` की अनुचित नेमस्पेस आइसोलेशन के बिना उजागर होना महत्वपूर्ण सुरक्षा जोखिम लाता है, जिसमें हमले की सतह विस्तार और सूचना फासवनी शामिल है। ये निर्देशिकाएँ संवेदनशील फ़ाइलें शामिल करती हैं जो, अगर गलत रूप से कॉन्फ़िगर की गई हैं या अनधिकृत उपयोगकर्ता द्वारा एक्सेस की गई हैं, तो कंटेनर बाहर निकल सकता है, होस्ट में संशोधन कर सकता है, या आगे के हमलों की सहायता करने वाली जानकारी प्रदान कर सकती हैं। उदाहरण के लिए, `-v /proc:/host/proc` को गलती से माउंट करना AppArmor सुरक्षा को छोड़ सकता है क्योंकि इसका पथ-आधारित प्रकृति है, `/host/proc` को सुरक्षित नहीं छोड़ता।

**आप प्रत्येक संभावित दुरुपयोग का विस्तार से विवरण** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)** में पा सकते हैं।**

## procfs दुरुपयोग

### `/proc/sys`

यह निर्देशिका कर्णल चरित्रिक चरों को संशोधित करने की अनुमति देती है, सामान्यत: `sysctl(2)` के माध्यम से, और कई चिंहित उपनिर्देशिकाएँ शामिल हैं:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) में वर्णित।
* पहले 128 बाइट के साथ कोर-फ़ाइल उत्पन्न करने पर कार्यक्रम को परिभाषित करने की अनुमति देता है। यदि फ़ाइल पाइप `|` के साथ शुरू होती है, तो यह कोड निष्पादन में ले जा सकता है।
*   **परीक्षण और दुरुपयोग उदाहरण**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # लेखन पहुंच का परीक्षण करें
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # कस्टम हैंडलर सेट करें
sleep 5 && ./crash & # हैंडलर को ट्रिगर करें
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विस्तार से वर्णित।
* कर्णल मॉड्यूल लोडर के पथ को शामिल करता है, जो कर्णल मॉड्यूल लोड करने के लिए आह्वानित किया जाता है।
*   **पहुंच की जाँच उदाहरण**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe की पहुंच की जाँच करें
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में संदर्भित।
* एक वैश्विक ध्वनि जो कर्णल पैनिक या OOM किलर को आह्वान करती है जब OOM स्थिति घटित होती है।

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) के अनुसार, फ़ाइल सिस्टम के बारे में विकल्प और जानकारी शामिल है।
* लेखन पहुंच से मेज़बान के खिलाफ विभिन्न सेवा अवरोध हमले सक्षम हो सकते हैं।

#### **`/proc/sys/fs/binfmt_misc`**

* जादू नंबर के आधार पर गैर-मूल बाइनरी प्रारूपों के लिए अनुप्रयोक्ताओं को पंजीकृत करने की अनुमति देता है।
* `/proc/sys/fs/binfmt_misc/register` लिखने योग्य है तो विशेषाधिकार या रूट शैली एक्सेस हो सकता है।
* संबंधित दुरुपयोग और स्पष्टीकरण:
* [binfmt\_misc के माध्यम से गरीब आदमी का रूटकिट](https://github.com/toffan/binfmt\_misc)
* विस्तृत ट्यूटोरियल: [वीडियो लिंक](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### `/proc` में अन्य

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC` सक्षम है तो कर्णल कॉन्फ़िगरेशन का पता लगा सकता है।
* चल रहे कर्णल में दोषों की पहचान करने के लिए उपयोगी है।

#### **`/proc/sysrq-trigger`**

* Sysrq कमांड को आह्वान करने की अनुमति देता है, स्थिति घटना या अन्य महत्वपूर्ण क्रियाएँ कर सकता है।
*   **मेज़बान को पुनरारंभ करने का उदाहरण**:

```bash
echo b > /proc/sysrq-trigger # मेज़बान को पुनरारंभ करता है
```

#### **`/proc/kmsg`**

* कर्णल रिंग बफर संदेशों को उजागर करता है।
* कर्णल दुरुपयोग, पते लीक, और संवेदनशील सिस्टम जानकारी प्रदान कर सकता है।

#### **`/proc/kallsyms`**

* कर्णल निर्यातित प्रतीक और उनके पतों की सूची देता है।
* कर्णल दुरुपयोग विकास के लिए अत्यधिक महत्वपूर्ण है, विशेषतः KASLR को पार करने के लिए।
* पता सूचना `kptr_restrict` को `1` या `2` पर सेट किया गया है।
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विवरण।

#### **`/proc/[pid]/mem`**

* कर्णल मेमोरी डिवाइस `/dev/mem` के साथ संवाद करता है।
* ऐतिहासिक रूप से विशेषाधिकार उन्नति हमलों के लिए विकल्पशील रहा है।
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) पर अधिक।

#### **`/proc/kcore`**

* ईएलएफ कोर प्रारूप में सिस्टम की भौतिक मेमोरी को प्रतिनिधित करता है।
* पढ़ने से मेज़बान सिस्टम और अन्य कंटेनरों की मेमोरी सामग्री लीक हो सकती है।
* बड़े फ़ाइल आकार से पढ़ने में समस्याएँ या सॉफ़्टवेयर क्रैश हो सकता है।
* [2019 में /proc/kcore को डंप करने का उपयोग](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) में विस्तृत उपयोग।

#### **`/proc/kmem`**

* `/dev/kmem` के लिए वैकल्पिक इंटरफेस, कर्णल वर्चुअल मेमोरी का प्रतिनिधित करता है।
* पढ़ने और लिखने की अनुमति देता है, इसलिए कर्णल मेमोरी का सीधा संशोधन कर सकता है।

#### **`/proc/mem`**

* `/dev/mem` के लिए वैकल्पिक इंटरफेस, भौतिक मेमोरी का प्रतिनिधित करता है।
* पढ़ने और लिखने की अनुमति देता है, सभी म
#### **`/sys/class/thermal`**

* तापमान सेटिंग को नियंत्रित करता है, जो डीओएस हमले या भौतिक क्षति का कारण बन सकता है।

#### **`/sys/kernel/vmcoreinfo`**

* कर्नेल पतों को लीक करता है, जो केएएसएलआर को कमजोर कर सकता है।

#### **`/sys/kernel/security`**

* `securityfs` इंटरफेस को होस्ट करता है, जो एप्पार्मर जैसे लिनक्स सुरक्षा मॉड्यूल को कॉन्फ़िगर करने की अनुमति देता है।
* पहुंच एक कंटेनर को अपनी एमएसी प्रणाली को अक्षम करने की अनुमति दे सकती है।

#### **`/sys/firmware/efi/vars` और `/sys/firmware/efi/efivars`**

* NVRAM में EFI वेरिएबल्स के साथ इंटरैक्ट करने के लिए इंटरफेस उजागर करता है।
* गलत कॉन्फ़िगरेशन या शोषण लैपटॉप को ब्रिक करने या अनबूटेबल होस्ट मशीन की ओर ले जा सकता है।

#### **`/sys/kernel/debug`**

* `debugfs` कर्नेल के लिए "कोई नियम" डीबगिंग इंटरफेस प्रदान करता है।
* इसकी असीमित प्रकृति के कारण सुरक्षा समस्याओं का इतिहास है।

### संदर्भ

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [उन्दरस्टैंडिंग एंड हार्डनिंग लिनक्स कंटेनर्स](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [अब्यूजिंग प्रिविलेज्ड और अनप्रिविलेज्ड लिनक्स कंटेनर्स](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
एडवांस्ड वेब सिक्योरिटी हैकिंग सीखें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**हैकट्रिक्स ट्रेनिंग AWS रेड टीम एक्सपर्ट (आरटीई)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
जीसीपी हैकिंग सीखें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**हैकट्रिक्स ट्रेनिंग GCP रेड टीम एक्सपर्ट (जीआरटीई)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
{% endhint %}
