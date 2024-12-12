# Wireshark tricks

## Wireshark tricks

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी की **विज्ञापनित करना चाहते हैं HackTricks** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks swag प्राप्त करें**](https://peass.creator-spring.com)
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके अपने हैकिंग ट्रिक्स साझा करें।

</details>

## अपनी Wireshark कौशल क्षमताएँ सुधारें

### ट्यूटोरियल

निम्नलिखित ट्यूटोरियल कुछ शानदार मूलभूत ट्रिक्स सीखने के लिए हैं:

* [https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
* [https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* [https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* [https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)

### विशेषज्ञ सूचना

_**विश्लेषण** --> **विशेषज्ञ सूचना**_ पर क्लिक करने पर आपको पैकेट्स में क्या हो रहा है का **अवलोकन** मिलेगा:

![](<../../../.gitbook/assets/image (570).png>)

**संक्षिप्त पते**

_**सांख्यिकी --> संक्षिप्त पते**_ के तहत आपको कई **सूचनाएं** मिलेंगी जो वायरशार्क द्वारा "**संक्षेपित**" की गई थीं, जैसे पोर्ट/परिवहन से प्रोटोकॉल, MAC से निर्माता आदि। संचार में क्या शामिल है यह जानना दिलचस्प है।

![](<../../../.gitbook/assets/image (571).png>)

**प्रोटोकॉल वर्गीकरण**

_**सांख्यिकी --> प्रोटोकॉल वर्गीकरण**_ के तहत आपको संचार में शामिल **प्रोटोकॉल** मिलेंगे और उनके बारे में डेटा।

![](<../../../.gitbook/assets/image (572).png>)

**बातचीतें**

_**सांख्यिकी --> बातचीतें**_ के तहत आपको संचार में बातचीतों का **सारांश** मिलेगा और उनके बारे में डेटा।

![](<../../../.gitbook/assets/image (573).png>)

**अंत स्थल**

_**सांख्यिकी --> अंत स्थल**_ के तहत आपको संचार में अंत स्थलों का **सारांश** मिलेगा और प्रत्येक के बारे में डेटा।

![](<../../../.gitbook/assets/image (575).png>)

**DNS जानकारी**

_**सांख्यिकी --> DNS**_ के तहत आपको कैप्चर किए गए DNS अनुरोधों के बारे में सांख्यिकी मिलेगी।

![](<../../../.gitbook/assets/image (577).png>)

**I/O ग्राफ**

_**सांख्यिकी --> I/O ग्राफ**_ के तहत आपको एक **संचार का ग्राफ** मिलेगा।

![](<../../../.gitbook/assets/image (574).png>)

### फ़िल्टर

यहाँ आप प्रोटोकॉल के आधार पर वायरशार्क फ़िल्टर पा सकते हैं: [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)\
अन्य दिलचस्प फ़िल्टर:

* `(http.request or ssl.handshake.type == 1) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफिक
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफिक + TCP SYN
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002 or dns) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफिक + TCP SYN + DNS अनुरोध

### खोज

अगर आप सत्रों के पैकेट्स में **सामग्री** की **खोज** करना चाहते हैं तो _CTRL+f_ दबाएं। आप मुख्य सूचना पट्टी में नई लेयर जोड़ सकते हैं (संख्या, समय, स्रोत, आदि) दाएं बटन दबाकर और फिर संपादन स्तंभ दबाकर।

### मुफ्त pcap लैब्स

**मुफ्त चुनौतियों के साथ अभ्यास करें: [https://www.malware-traffic-analysis.net/](https://www.malware-traffic-analysis.net)**

## डोमेन्स की पहचान

आप एक स्तंभ जो होस्ट HTTP हेडर दिखाता है जोड़ सकते हैं:

![](<../../../.gitbook/assets/image (403).png>)

और एक स्तंभ जो एक प्रारंभिक HTTPS कनेक्शन से सर्वर नाम जोड़ता है (**ssl.handshake.type == 1**):

![](<../../../.gitbook/assets/image (408) (1).png>)

## स्थानीय होस्टनेम की पहचान

### DHCP से

वर्तमान Wireshark में `bootp` की बजाय `DHCP` की खोज करनी होगी

![](<../../../.gitbook/assets/image (404).png>)

### NBNS से

![](<../../../.gitbook/assets/image (405).png>)

## TLS की डिक्रिप्शन

### सर्वर निजी कुंजी के साथ https ट्रैफिक की डिक्रिप्शन

_edit>preference>protocol>ssl>_

![](<../../../.gitbook/assets/image (98).png>)

_Edit_ दबाएं और सर्वर और निजी कुंजी के सभी डेटा को जोड़ें (_IP, पोर्ट, प्रोटोकॉल, कुंजी फ़ाइल और पासवर्ड_)

### सममित सत्र कुंजियों के साथ https ट्रैफिक की डिक्रिप्शन

फायरफॉक्स और क्रोम दोनों के पास TLS सत्र कुंजियों को लॉग करने की क्षमता है, जिन्हें Wireshark के साथ ट्रांसपोर्ट करने के लिए उपयोग किया जा सकता है। यह सुरक्षित संचार का विस्तृत विश्लेषण संभव बनाता है। इस डिक्रिप्शन को कैसे किया जाए, इसके बारे में अधिक विवरण एक मार्गदर्शिका में मिल सकता है [Red Flag Security](https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/).

इसे डिटेक्ट करने के लिए वातावरण में `SSLKEYLOGFILE` के लिए खोजें

एक साझा कुंजी की फ़ाइल इस तरह दिखेगी:

![](<../../../.gitbook/assets/image (99).png>)

इसे वायरशार्क में आयात करने के लिए जाएं \_edit > preference > protocol > ssl > और इसे (पूर्व)-मास्टर-सीक्रेट लॉग फ़ाइल नाम में आयात करें:

![](<../../../.gitbook/assets/image (100).png>)

## ADB संचार

जहां APK भेजा गया था, वहां से ADB संचार से APK निकालें:
```python
from scapy.all import *

pcap = rdpcap("final2.pcapng")

def rm_data(data):
splitted = data.split(b"DATA")
if len(splitted) == 1:
return data
else:
return splitted[0]+splitted[1][4:]

all_bytes = b""
for pkt in pcap:
if Raw in pkt:
a = pkt[Raw]
if b"WRTE" == bytes(a)[:4]:
all_bytes += rm_data(bytes(a)[24:])
else:
all_bytes += rm_data(bytes(a))
print(all_bytes)

f = open('all_bytes.data', 'w+b')
f.write(all_bytes)
f.close()
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
