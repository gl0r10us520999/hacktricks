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


**The original post is** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## Summary

दो रजिस्ट्री कुंजी वर्तमान उपयोगकर्ता द्वारा लिखी जा सकती हैं:

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

यह सुझाव दिया गया कि **RpcEptMapper** सेवा की अनुमतियों की जांच **regedit GUI** का उपयोग करके की जाए, विशेष रूप से **Advanced Security Settings** विंडो के **Effective Permissions** टैब का। यह दृष्टिकोण विशिष्ट उपयोगकर्ताओं या समूहों को दी गई अनुमतियों का आकलन करने की अनुमति देता है बिना प्रत्येक एक्सेस कंट्रोल एंट्री (ACE) की व्यक्तिगत रूप से जांच किए।

एक स्क्रीनशॉट में एक कम विशेषाधिकार प्राप्त उपयोगकर्ता को दी गई अनुमतियों को दिखाया गया, जिसमें **Create Subkey** अनुमति उल्लेखनीय थी। इस अनुमति को **AppendData/AddSubdirectory** के रूप में भी जाना जाता है, जो स्क्रिप्ट के निष्कर्षों के साथ मेल खाता है।

कुछ मानों को सीधे संशोधित करने में असमर्थता, फिर भी नए सबकीज़ बनाने की क्षमता को नोट किया गया। एक उदाहरण में **ImagePath** मान को बदलने का प्रयास शामिल था, जिसके परिणामस्वरूप एक एक्सेस अस्वीकृत संदेश मिला।

इन सीमाओं के बावजूद, **RpcEptMapper** सेवा की रजिस्ट्री संरचना के भीतर **Performance** सबकीज़ का लाभ उठाने की संभावना के माध्यम से विशेषाधिकार वृद्धि की संभावना की पहचान की गई, जो डिफ़ॉल्ट रूप से मौजूद नहीं है। इससे DLL पंजीकरण और प्रदर्शन निगरानी की अनुमति मिल सकती है।

**Performance** सबकीज़ और इसके प्रदर्शन निगरानी के लिए उपयोग पर दस्तावेज़ीकरण की समीक्षा की गई, जिससे एक प्रमाण-को-धारक DLL का विकास हुआ। इस DLL ने **OpenPerfData**, **CollectPerfData**, और **ClosePerfData** कार्यों के कार्यान्वयन को प्रदर्शित किया, जिसे **rundll32** के माध्यम से परीक्षण किया गया, जिससे इसकी संचालन सफलता की पुष्टि हुई।

लक्ष्य **RPC Endpoint Mapper सेवा** को तैयार किए गए प्रदर्शन DLL को लोड करने के लिए मजबूर करना था। अवलोकनों ने दिखाया कि प्रदर्शन डेटा से संबंधित WMI वर्ग प्रश्नों को PowerShell के माध्यम से निष्पादित करने से एक लॉग फ़ाइल का निर्माण हुआ, जिससे **LOCAL SYSTEM** संदर्भ के तहत मनमाने कोड को निष्पादित करने की अनुमति मिली, इस प्रकार उच्च विशेषाधिकार प्राप्त हुए।

इस भेद्यता की स्थिरता और संभावित प्रभावों को रेखांकित किया गया, इसके पोस्ट-एक्सप्लोइटेशन रणनीतियों, पार्श्व आंदोलन, और एंटीवायरस/EDR सिस्टम से बचने की प्रासंगिकता को उजागर किया गया।

हालांकि भेद्यता को प्रारंभ में स्क्रिप्ट के माध्यम से अनजाने में प्रकट किया गया था, यह जोर दिया गया कि इसका शोषण पुराने Windows संस्करणों (जैसे, **Windows 7 / Server 2008 R2**) तक सीमित है और स्थानीय पहुंच की आवश्यकता है।

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
