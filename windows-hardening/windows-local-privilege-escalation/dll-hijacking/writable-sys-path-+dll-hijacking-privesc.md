# Writable Sys Path +Dll Hijacking Privesc

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

## Introduction

यदि आप पाते हैं कि आप **System Path फ़ोल्डर में लिख सकते हैं** (ध्यान दें कि यह तब काम नहीं करेगा जब आप User Path फ़ोल्डर में लिख सकते हैं) तो यह संभव है कि आप **system में privileges बढ़ा सकते हैं**।

इसके लिए आप **Dll Hijacking** का दुरुपयोग कर सकते हैं जहाँ आप एक **library को हाईजैक करने जा रहे हैं** जिसे एक सेवा या प्रक्रिया द्वारा **आपसे अधिक privileges** के साथ लोड किया जा रहा है, और क्योंकि वह सेवा एक Dll लोड कर रही है जो शायद पूरे सिस्टम में मौजूद नहीं है, यह इसे उस System Path से लोड करने की कोशिश करेगी जहाँ आप लिख सकते हैं।

**Dll Hijacking क्या है** के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Privesc with Dll Hijacking

### Finding a missing Dll

आपको सबसे पहले **एक प्रक्रिया की पहचान करनी होगी** जो **आपसे अधिक privileges** के साथ चल रही है और जो **System Path से Dll लोड करने की कोशिश कर रही है** जिसमें आप लिख सकते हैं।

इन मामलों में समस्या यह है कि शायद वे प्रक्रियाएँ पहले से ही चल रही हैं। यह जानने के लिए कि कौन सी Dlls सेवाओं में कमी है, आपको जितनी जल्दी हो सके procmon लॉन्च करना होगा (प्रक्रियाएँ लोड होने से पहले)। इसलिए, कमी वाली .dlls खोजने के लिए करें:

* **Create** the folder `C:\privesc_hijacking` and add the path `C:\privesc_hijacking` to **System Path env variable**. You can do this **manually** or with **PS**:
```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```
* **`procmon`** लॉन्च करें और **`Options`** --> **`Enable boot logging`** पर जाएं और प्रॉम्प्ट में **`OK`** दबाएं।
* फिर, **रीबूट** करें। जब कंप्यूटर फिर से शुरू होगा, **`procmon`** तुरंत घटनाओं को **रिकॉर्ड करना** शुरू कर देगा।
* एक बार जब **Windows** **शुरू हो जाए, `procmon`** को फिर से चलाएं, यह आपको बताएगा कि यह चल रहा है और **आपसे पूछेगा कि क्या आप घटनाओं को एक फ़ाइल में स्टोर करना चाहते हैं**। **हाँ** कहें और **घटनाओं को एक फ़ाइल में स्टोर करें**।
* **फाइल** **जनरेट** होने के बाद, खुले हुए **`procmon`** विंडो को **बंद** करें और **घटनाओं की फ़ाइल** खोलें।
* ये **फिल्टर** जोड़ें और आप सभी Dlls पाएंगे जो कुछ **प्रोसेस ने** लिखने योग्य सिस्टम पथ फ़ोल्डर से **लोड करने की कोशिश की**:

<figure><img src="../../../.gitbook/assets/image (945).png" alt=""><figcaption></figcaption></figure>

### मिस्ड Dlls

एक मुफ्त **वर्चुअल (vmware) Windows 11 मशीन** पर इसे चलाने पर मुझे ये परिणाम मिले:

<figure><img src="../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

इस मामले में .exe बेकार हैं इसलिए उन्हें नजरअंदाज करें, मिस्ड DLLs निम्नलिखित से थीं:

| सेवा                             | Dll                | CMD लाइन                                                             |
| ------------------------------- | ------------------ | -------------------------------------------------------------------- |
| टास्क शेड्यूलर (Schedule)      | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| डायग्नोस्टिक पॉलिसी सेवा (DPS) | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                             | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

इसका पता लगाने के बाद, मैंने एक दिलचस्प ब्लॉग पोस्ट पाया जो यह भी बताता है कि कैसे [**WptsExtensions.dll का दुरुपयोग करें privesc के लिए**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll)। जो हम **अब करने जा रहे हैं**।

### शोषण

तो, **अधिकारों को बढ़ाने** के लिए हम **WptsExtensions.dll** पुस्तकालय को हाईजैक करने जा रहे हैं। **पथ** और **नाम** होने के बाद हमें बस **दुष्ट dll** **जनरेट** करने की आवश्यकता है।

आप [**इन उदाहरणों में से किसी का उपयोग करने की कोशिश कर सकते हैं**](./#creating-and-compiling-dlls)। आप पे लोड चला सकते हैं जैसे: एक रिव शेल प्राप्त करें, एक उपयोगकर्ता जोड़ें, एक बीकन निष्पादित करें...

{% hint style="warning" %}
ध्यान दें कि **सभी सेवाएं** **`NT AUTHORITY\SYSTEM`** के साथ **नहीं चलतीं**, कुछ **`NT AUTHORITY\LOCAL SERVICE`** के साथ भी चलती हैं, जिनके पास **कम अधिकार** होते हैं और आप **नया उपयोगकर्ता नहीं बना पाएंगे** इसके अनुमतियों का दुरुपयोग करें।\
हालांकि, उस उपयोगकर्ता के पास **`seImpersonate`** विशेषाधिकार है, इसलिए आप [**पोटैटो सूट का उपयोग करके अधिकारों को बढ़ा सकते हैं**](../roguepotato-and-printspoofer.md)। इसलिए, इस मामले में एक रिव शेल एक बेहतर विकल्प है बजाय एक उपयोगकर्ता बनाने की कोशिश करने के।
{% endhint %}

लेखन के समय **टास्क शेड्यूलर** सेवा **Nt AUTHORITY\SYSTEM** के साथ चल रही है।

**दुष्ट Dll** (_मेरे मामले में मैंने x64 रिव शेल का उपयोग किया और मुझे एक शेल वापस मिला लेकिन डिफेंडर ने इसे मार दिया क्योंकि यह msfvenom से था_) जनरेट करने के बाद, इसे लिखने योग्य सिस्टम पथ में **WptsExtensions.dll** नाम से सहेजें और कंप्यूटर को **रीस्टार्ट** करें (या सेवा को रीस्टार्ट करें या प्रभावित सेवा/प्रोग्राम को फिर से चलाने के लिए जो भी करना हो करें)।

जब सेवा फिर से शुरू होती है, तो **dll को लोड और निष्पादित किया जाना चाहिए** (आप **procmon** ट्रिक का **पुनः उपयोग** कर सकते हैं यह जांचने के लिए कि **पुस्तकालय अपेक्षित रूप से लोड हुआ था**)। 

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमें ट्विटर पर फॉलो करें** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
