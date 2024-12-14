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


## smss.exe

**सत्र प्रबंधक**।\
सत्र 0 **csrss.exe** और **wininit.exe** (**OS** **सेवाएँ**) शुरू करता है जबकि सत्र 1 **csrss.exe** और **winlogon.exe** (**उपयोगकर्ता** **सत्र**) शुरू करता है। हालाँकि, आपको प्रक्रियाओं के पेड़ में उस **बाइनरी** की **केवल एक प्रक्रिया** देखनी चाहिए जिसमें बच्चे न हों।

इसके अलावा, 0 और 1 के अलावा सत्र का अर्थ हो सकता है कि RDP सत्र हो रहे हैं।


## csrss.exe

**क्लाइंट/सर्वर रन सबसिस्टम प्रक्रिया**।\
यह **प्रक्रियाओं** और **थ्रेड्स** का प्रबंधन करता है, अन्य प्रक्रियाओं के लिए **Windows** **API** उपलब्ध कराता है और **ड्राइव लेटर** को **मैप** करता है, **टेम्प फ़ाइलें** बनाता है, और **शटडाउन** **प्रक्रिया** को संभालता है।

सत्र 0 में एक **चल रही प्रक्रिया** और सत्र 1 में एक और है (तो प्रक्रियाओं के पेड़ में **2 प्रक्रियाएँ**)। एक नई सत्र के लिए एक और बनाई जाती है।


## winlogon.exe

**Windows लॉगिन प्रक्रिया**।\
यह उपयोगकर्ता **लॉगिन**/**लॉगऑफ** के लिए जिम्मेदार है। यह उपयोगकर्ता नाम और पासवर्ड पूछने के लिए **logonui.exe** लॉन्च करता है और फिर उन्हें सत्यापित करने के लिए **lsass.exe** को कॉल करता है।

फिर यह **userinit.exe** लॉन्च करता है जो **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** में **Userinit** कुंजी के साथ निर्दिष्ट है।

इसके अलावा, पिछले रजिस्ट्री में **explorer.exe** **Shell कुंजी** में होना चाहिए या इसे **मैलवेयर स्थिरता विधि** के रूप में दुरुपयोग किया जा सकता है।


## wininit.exe

**Windows प्रारंभिक प्रक्रिया**। \
यह सत्र 0 में **services.exe**, **lsass.exe**, और **lsm.exe** लॉन्च करता है। केवल 1 प्रक्रिया होनी चाहिए।


## userinit.exe

**Userinit लॉगिन एप्लिकेशन**।\
यह **HKCU** में **ntduser.dat** लोड करता है और **उपयोगकर्ता** **पर्यावरण** को प्रारंभ करता है और **लॉगिन** **स्क्रिप्ट** और **GPO** चलाता है।

यह **explorer.exe** लॉन्च करता है।


## lsm.exe

**स्थानीय सत्र प्रबंधक**।\
यह उपयोगकर्ता सत्रों को प्रबंधित करने के लिए smss.exe के साथ काम करता है: लॉगिन/लॉगऑफ, शेल प्रारंभ, लॉक/अनलॉक डेस्कटॉप, आदि।

W7 के बाद lsm.exe को एक सेवा (lsm.dll) में बदल दिया गया था।

W7 में केवल 1 प्रक्रिया होनी चाहिए और उनमें से एक सेवा जो DLL चला रही है।


## services.exe

**सेवा नियंत्रण प्रबंधक**।\
यह **सेवाओं** को **लोड** करता है जो **ऑटो-स्टार्ट** और **ड्राइवर** के रूप में कॉन्फ़िगर की गई हैं।

यह **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** और कई अन्य की मूल प्रक्रिया है।

सेवाएँ `HKLM\SYSTEM\CurrentControlSet\Services` में परिभाषित हैं और यह प्रक्रिया सेवा जानकारी का एक DB मेमोरी में बनाए रखती है जिसे sc.exe द्वारा क्वेरी किया जा सकता है।

ध्यान दें कि **कुछ** **सेवाएँ** एक **अपनी प्रक्रिया** में चलने जा रही हैं और अन्य **svchost.exe प्रक्रिया** साझा करने जा रही हैं।

केवल 1 प्रक्रिया होनी चाहिए।


## lsass.exe

**स्थानीय सुरक्षा प्राधिकरण उपसिस्टम**।\
यह उपयोगकर्ता **प्रमाणीकरण** के लिए जिम्मेदार है और **सुरक्षा** **टोकन** बनाता है। यह `HKLM\System\CurrentControlSet\Control\Lsa` में स्थित प्रमाणीकरण पैकेज का उपयोग करता है।

यह **सुरक्षा** **इवेंट** **लॉग** में लिखता है और केवल 1 प्रक्रिया होनी चाहिए।

ध्यान रखें कि इस प्रक्रिया पर पासवर्ड डंप करने के लिए उच्च हमले होते हैं।


## svchost.exe

**सामान्य सेवा होस्ट प्रक्रिया**।\
यह एक साझा प्रक्रिया में कई DLL सेवाओं की मेज़बानी करता है।

आमतौर पर, आप पाएंगे कि **svchost.exe** `-k` ध्वज के साथ लॉन्च किया गया है। यह रजिस्ट्री **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** पर एक क्वेरी लॉन्च करेगा जहाँ -k में उल्लेखित तर्क के साथ एक कुंजी होगी जो उसी प्रक्रिया में लॉन्च करने के लिए सेवाओं को शामिल करेगी।

उदाहरण के लिए: `-k UnistackSvcGroup` लॉन्च करेगा: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

यदि **ध्वज `-s`** का भी उपयोग किसी तर्क के साथ किया जाता है, तो svchost से **केवल निर्दिष्ट सेवा** को इस तर्क में लॉन्च करने के लिए कहा जाता है।

`svchost.exe` की कई प्रक्रियाएँ होंगी। यदि इनमें से कोई भी **`-k` ध्वज** का उपयोग नहीं कर रहा है, तो यह बहुत संदिग्ध है। यदि आप पाते हैं कि **services.exe मूल प्रक्रिया नहीं है**, तो यह भी बहुत संदिग्ध है।


## taskhost.exe

यह प्रक्रिया DLLs से चलने वाली प्रक्रियाओं के लिए एक होस्ट के रूप में कार्य करती है। यह DLLs से चलने वाली सेवाओं को भी लोड करती है।

W8 में इसे taskhostex.exe कहा जाता है और W10 में taskhostw.exe।


## explorer.exe

यह प्रक्रिया **उपयोगकर्ता के डेस्कटॉप** और फ़ाइल एक्सटेंशन के माध्यम से फ़ाइलें लॉन्च करने के लिए जिम्मेदार है।

**प्रत्येक लॉग इन उपयोगकर्ता के लिए केवल 1** प्रक्रिया उत्पन्न होनी चाहिए।

यह **userinit.exe** से चलती है जिसे समाप्त किया जाना चाहिए, इसलिए इस प्रक्रिया के लिए **कोई मूल प्रक्रिया** नहीं होनी चाहिए।


# दुर्भावनापूर्ण प्रक्रियाओं को पकड़ना

* क्या यह अपेक्षित पथ से चल रहा है? (कोई Windows बाइनरी टेम्प स्थान से नहीं चलती)
* क्या यह अजीब IPs के साथ संचार कर रहा है?
* डिजिटल हस्ताक्षरों की जांच करें (Microsoft के आर्टिफैक्ट पर हस्ताक्षर होना चाहिए)
* क्या यह सही ढंग से लिखा गया है?
* क्या यह अपेक्षित SID के तहत चल रहा है?
* क्या मूल प्रक्रिया अपेक्षित है (यदि कोई हो)?
* क्या बच्चे प्रक्रियाएँ अपेक्षित हैं? (कोई cmd.exe, wscript.exe, powershell.exe..?)
