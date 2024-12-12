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


# पैक किए गए बाइनरी की पहचान करना

* **स्ट्रिंग्स की कमी**: यह सामान्य है कि पैक किए गए बाइनरी में लगभग कोई स्ट्रिंग नहीं होती है।
* बहुत सारी **अप्रयुक्त स्ट्रिंग्स**: इसके अलावा, जब कोई मैलवेयर किसी प्रकार के व्यावसायिक पैकर का उपयोग कर रहा होता है, तो बहुत सारी स्ट्रिंग्स बिना क्रॉस-रेफरेंस के मिलना सामान्य है। भले ही ये स्ट्रिंग्स मौजूद हों, इसका मतलब यह नहीं है कि बाइनरी पैक नहीं है।
* आप यह पता लगाने के लिए कुछ टूल का उपयोग कर सकते हैं कि बाइनरी को पैक करने के लिए कौन सा पैकर उपयोग किया गया था:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# बुनियादी सिफारिशें

* पैक किए गए बाइनरी का **विश्लेषण** **नीचे से शुरू करें और ऊपर की ओर बढ़ें**। अनपैकर्स तब समाप्त होते हैं जब अनपैक किया गया कोड समाप्त होता है, इसलिए यह संभावना नहीं है कि अनपैकर शुरू में अनपैक किए गए कोड को निष्पादन पास करता है।
* **रजिस्टर** या **मेमोरी** के **क्षेत्रों** के लिए **JMP's** या **CALLs** की खोज करें। इसके अलावा, **फंक्शंस जो तर्कों और एक पते को धक्का देते हैं और फिर `retn` को कॉल करते हैं** के लिए खोजें, क्योंकि उस मामले में फंक्शन की वापसी उस पते को कॉल कर सकती है जो स्टैक में धकेला गया था।
* `VirtualAlloc` पर एक **ब्रेकपॉइंट** रखें क्योंकि यह मेमोरी में स्थान आवंटित करता है जहां प्रोग्राम अनपैक किए गए कोड को लिख सकता है। "यूजर कोड पर चलाएं" या F8 का उपयोग करें **EAX के अंदर मान प्राप्त करने के लिए** फंक्शन को निष्पादित करने के बाद और "**डंप में उस पते का पालन करें**"। आप कभी नहीं जानते कि क्या यह वह क्षेत्र है जहां अनपैक किया गया कोड सहेजा जाएगा।
* **`VirtualAlloc`** के साथ मान "**40**" एक तर्क के रूप में पढ़ें + लिखें + निष्पादित करें (कुछ कोड जिसे निष्पादन की आवश्यकता है, यहां कॉपी किया जाएगा)।
* **कोड को अनपैक करते समय** यह सामान्य है कि **गणितीय संचालन** और **`memcopy`** या **`Virtual`**`Alloc` जैसी फंक्शंस के लिए **कई कॉल** मिलें। यदि आप किसी फंक्शन में हैं जो स्पष्ट रूप से केवल गणितीय संचालन करता है और शायद कुछ `memcopy` करता है, तो सिफारिश है कि आप **फंक्शन के अंत को खोजने की कोशिश करें** (शायद किसी रजिस्टर के लिए एक JMP या कॉल) **या** कम से कम **अंतिम फंक्शन के लिए कॉल** करें और फिर चलाएं क्योंकि कोड दिलचस्प नहीं है।
* कोड को अनपैक करते समय **ध्यान दें** जब भी आप **मेमोरी क्षेत्र बदलते हैं** क्योंकि मेमोरी क्षेत्र में परिवर्तन **अनपैकिंग कोड की शुरुआत** को इंगित कर सकता है। आप प्रोसेस हैकर का उपयोग करके आसानी से मेमोरी क्षेत्र को डंप कर सकते हैं (प्रोसेस --> प्रॉपर्टीज --> मेमोरी)।
* कोड को अनपैक करने की कोशिश करते समय यह जानने का एक अच्छा तरीका है कि **क्या आप पहले से ही अनपैक किए गए कोड के साथ काम कर रहे हैं** (ताकि आप इसे केवल डंप कर सकें) यह है कि **बाइनरी की स्ट्रिंग्स की जांच करें**। यदि किसी बिंदु पर आप एक जंप करते हैं (शायद मेमोरी क्षेत्र बदलते हुए) और आप देखते हैं कि **बहुत सारी स्ट्रिंग्स जोड़ी गई हैं**, तो आप जान सकते हैं **आप अनपैक किए गए कोड के साथ काम कर रहे हैं**।\
हालांकि, यदि पैकर में पहले से ही बहुत सारी स्ट्रिंग्स हैं, तो आप देख सकते हैं कि कितनी स्ट्रिंग्स में "http" शब्द है और देखें कि क्या यह संख्या बढ़ती है।
* जब आप मेमोरी के एक क्षेत्र से एक निष्पादन योग्य डंप करते हैं, तो आप [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases) का उपयोग करके कुछ हेडर ठीक कर सकते हैं।

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
