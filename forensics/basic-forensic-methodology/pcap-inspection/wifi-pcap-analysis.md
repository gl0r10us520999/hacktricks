{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}


# BSSID जाँचें

जब आप वायरशार्क का उपयोग करके एक कैप्चर प्राप्त करते हैं जिसका मुख्य ट्रैफिक वाईफ़ाई है, तो आप _Wireless --> WLAN Traffic_ के साथ कैप्चर के सभी एसएसआईडी की जांच शुरू कर सकते हैं:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## ब्रूट फ़ोर्स

उस स्क्रीन के एक कॉलम में एक इंजेक्शन मिला है या नहीं, यह दिखाता है कि **क्या पीकैप में कोई प्रमाणीकरण मिला है**। यदि ऐसा है तो आप `aircrack-ng` का उपयोग करके इसे ब्रूट फ़ोर्स कर सकते हैं:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
# डेटा बीकन / साइड चैनल में

यदि आप संदेह कर रहे हैं कि **एक वाईफ़ाई नेटवर्क के बीकन्स में डेटा लीक हो रहा है**, तो आप नेटवर्क के बीकन्स की जांच कर सकते हैं उस नेटवर्क के बीकन्स को चेक करें जैसे निम्नलिखित फ़िल्टर का उपयोग करके: `wlan contains <NAMEofNETWORK>`, या `wlan.ssid == "NAMEofNETWORK"` शंकुलित पैकेट्स के भीतर संदेहास्पद स्ट्रिंग के लिए खोजें।

# एक वाईफ़ाई नेटवर्क में अज्ञात MAC पते खोजें

निम्नलिखित लिंक उपयोगी होगा **एक वाईफ़ाई नेटवर्क के भीतर डेटा भेजने वाली मशीनें** खोजने के लिए:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

यदि आप **MAC पते पहले से जानते हैं तो आप उन्हें आउटपुट से हटा सकते हैं** इस तरह की जाँच जोड़कर: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

एक बार जब आपने नेटवर्क के भीतर **अज्ञात MAC पते** को पहचान लिया है जो भेजने के लिए ट्रैफ़िक का उपयोग कर रहे हैं, तो आप इसका उपयोग कर सकते हैं **फ़िल्टर** जैसे निम्नलिखित: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)` इसका ध्यान दें कि ftp/http/ssh/telnet फ़िल्टर उपयोगी होते हैं अगर आपने ट्रैफ़िक को डिक्रिप्ट कर लिया है।

# ट्रैफ़िक डिक्रिप्ट करें

संपादित करें --> वरीयताएँ --> प्रोटोकॉल --> IEEE 802.11--> संपादित

![](<../../../.gitbook/assets/image (426).png>)
