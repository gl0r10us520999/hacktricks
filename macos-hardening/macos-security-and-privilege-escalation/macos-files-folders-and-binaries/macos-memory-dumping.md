# macOS Memory Dumping

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


## Memory Artifacts

### Swap Files

स्वैप फ़ाइलें, जैसे कि `/private/var/vm/swapfile0`, **भौतिक मेमोरी भर जाने पर कैश के रूप में कार्य करती हैं**। जब भौतिक मेमोरी में और जगह नहीं होती है, तो इसका डेटा एक स्वैप फ़ाइल में स्थानांतरित किया जाता है और फिर आवश्यकतानुसार भौतिक मेमोरी में वापस लाया जाता है। कई स्वैप फ़ाइलें हो सकती हैं, जिनके नाम जैसे swapfile0, swapfile1, आदि हो सकते हैं।

### Hibernate Image

फ़ाइल जो `/private/var/vm/sleepimage` पर स्थित है, **हाइबरनेशन मोड** के दौरान महत्वपूर्ण है। **जब OS X हाइबरनेट करता है, तो मेमोरी से डेटा इस फ़ाइल में संग्रहीत किया जाता है**। कंप्यूटर को जगाने पर, सिस्टम इस फ़ाइल से मेमोरी डेटा पुनः प्राप्त करता है, जिससे उपयोगकर्ता वहीं से जारी रख सकता है जहाँ उसने छोड़ा था।

यह ध्यान देने योग्य है कि आधुनिक MacOS सिस्टम पर, यह फ़ाइल आमतौर पर सुरक्षा कारणों से एन्क्रिप्टेड होती है, जिससे पुनर्प्राप्ति कठिन हो जाती है।

* यह जांचने के लिए कि क्या sleepimage के लिए एन्क्रिप्शन सक्षम है, कमांड `sysctl vm.swapusage` चलाया जा सकता है। यह दिखाएगा कि क्या फ़ाइल एन्क्रिप्टेड है।

### Memory Pressure Logs

MacOS सिस्टम में एक और महत्वपूर्ण मेमोरी-संबंधित फ़ाइल **मेमोरी प्रेशर लॉग** है। ये लॉग `/var/log` में स्थित होते हैं और सिस्टम की मेमोरी उपयोग और प्रेशर घटनाओं के बारे में विस्तृत जानकारी प्रदान करते हैं। ये मेमोरी-संबंधित समस्याओं का निदान करने या यह समझने के लिए विशेष रूप से उपयोगी हो सकते हैं कि सिस्टम समय के साथ मेमोरी का प्रबंधन कैसे करता है।

## Dumping memory with osxpmem

MacOS मशीन में मेमोरी को डंप करने के लिए आप [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip) का उपयोग कर सकते हैं।

**Note**: निम्नलिखित निर्देश केवल Intel आर्किटेक्चर वाले Macs के लिए काम करेंगे। यह उपकरण अब आर्काइव किया गया है और अंतिम रिलीज 2017 में थी। नीचे दिए गए निर्देशों का उपयोग करके डाउनलोड की गई बाइनरी Intel चिप्स को लक्षित करती है क्योंकि 2017 में Apple Silicon नहीं था। यह arm64 आर्किटेक्चर के लिए बाइनरी को संकलित करना संभव हो सकता है लेकिन आपको स्वयं प्रयास करना होगा।
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
यदि आप यह त्रुटि पाते हैं: `osxpmem.app/MacPmem.kext failed to load - (libkern/kext) authentication failure (file ownership/permissions); check the system/kernel logs for errors or try kextutil(8)` आप इसे ठीक कर सकते हैं:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**अन्य त्रुटियाँ** को **kext के लोड की अनुमति देकर** ठीक किया जा सकता है "Security & Privacy --> General" में, बस **अनुमति दें**।

आप इस **oneliner** का उपयोग करके एप्लिकेशन डाउनलोड कर सकते हैं, kext लोड कर सकते हैं और मेमोरी डंप कर सकते हैं:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}


{% hint style="success" %}
सीखें और अभ्यास करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और अभ्यास करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो करें**.**
* **हैकिंग ट्रिक्स साझा करें PRs को** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रिपोजिटरी में सबमिट करके.

</details>
{% endhint %}
