# macOS MIG - Mach Interface Generator

{% hint style="success" %}
एडवांस्ड वेब सर्विसेज हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
जीसीपी हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 पर **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## मूल जानकारी

MIG को **Mach IPC** कोड निर्माण की प्रक्रिया को सरल बनाने के लिए बनाया गया था। यह मौजूदा परिभाषा के साथ सर्वर और क्लाइंट के बीच संवाद के लिए आवश्यक कोड **उत्पन्न** करता है। यद्यपि उत्पन्न कोड बेहद बेहद भद्दा हो, एक डेवलपर को उसे आयात करने की आवश्यकता होगी और उसका कोड पहले की तुलना में बहुत सरल होगा।

परिभाषा इंटरफेस परिभाषा भाषा (IDL) में निर्दिष्ट की जाती है जिसमें `.defs` एक्सटेंशन का उपयोग किया जाता है।

इन परिभाषाओं में 5 खंड होते हैं:

* **सबसिस्टम घोषणा**: सबसिस्टम शब्द का उपयोग **नाम** और **आईडी** की घोषणा के लिए किया जाता है। इसे **`KernelServer`** के रूप में चिह्नित करना भी संभव है अगर सर्वर कर्नल में चलाना चाहिए।
* **समावेश और आयात**: MIG C-प्रीप्रोसेसर का उपयोग करता है, इसलिए यह आयात का उपयोग कर सकता है। इसके अतिरिक्त, उपयोगकर्ता या सर्वर उत्पन्न कोड के लिए `uimport` और `simport` का उपयोग किया जा सकता है।
* **प्रकार की घोषणाएँ**: डेटा प्रकारों को परिभाषित करना संभव है हालांकि सामान्यत: यह `mach_types.defs` और `std_types.defs` को आयात करेगा। कस्टम वालों के लिए कुछ सिंटेक्स का उपयोग किया जा सकता है:
* \[i`n/out]tran`: एक फ़ंक्शन जिसे आउटगोइंग संदेश से अनुवाद करने की आवश्यकता है
* `c[user/server]type`: दूसरे सी टाइप को मैप करना।
* `destructor`: इस फ़ंक्शन को कॉल करें जब प्रकार रिलीज़ होता है।
* **ऑपरेशन**: ये आरपीसी विधियों की परिभाषाएँ हैं। यहाँ 5 विभिन्न प्रकार हैं:
* `routine`: जवाब की उम्मीद है
* `simpleroutine`: जवाब की उम्मीद नहीं है
* `procedure`: जवाब की उम्मीद है
* `simpleprocedure`: जवाब की उम्मीद नहीं है
* `function`: जवाब की उम्मीद है

### उदाहरण

एक परिभाषा फ़ाइल बनाएं, इस मामले में एक बहुत ही सरल फ़ंक्शन के साथ:

{% code title="myipc.defs" %}
```cpp
subsystem myipc 500; // Arbitrary name and id

userprefix USERPREF;        // Prefix for created functions in the client
serverprefix SERVERPREF;    // Prefix for created functions in the server

#include <mach/mach_types.defs>
#include <mach/std_types.defs>

simpleroutine Subtract(
server_port :  mach_port_t;
n1          :  uint32_t;
n2          :  uint32_t);
```
{% endcode %}

नोट करें कि पहला **आर्ग्यूमेंट बाइंड करने के लिए पोर्ट है** और MIG **स्वचालित रूप से उत्तर पोर्ट का हैंडल करेगा** (जब तक क्लाइंट कोड में `mig_get_reply_port()` को कॉल नहीं किया जाता है)। इसके अतिरिक्त, **ऑपरेशन्स की आईडी** क्रमांकांकित होगी जो सबसिस्टम आईडी द्वारा निर्दिष्ट करने के साथ प्रारंभ होगी (तो अगर कोई ऑपरेशन पुराना हो जाता है तो उसे हटा दिया जाता है और `skip` का उपयोग इसकी आईडी का उपयोग करने के लिए किया जाता है)।

अब MIG का उपयोग करें ताकि सर्वर और क्लाइंट कोड जेनरेट किया जा सके जो एक-दूसरे के भीतर संवाद करने के लिए सक्षम हों और Subtract फ़ंक्शन को कॉल करने के लिए:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
कई नए फ़ाइलें मौजूदा निर्देशिका में बनाई जाएंगी।

{% hint style="success" %}
आप अपने सिस्टम में एक अधिक जटिल उदाहरण ढूंढ सकते हैं: `mdfind mach_port.defs`\
और आप इसे कंपाइल कर सकते हैं उसी फ़ोल्डर से जहां फ़ाइल है: `mig -DLIBSYSCALL_INTERFACE mach_ports.defs`
{% endhint %}

फ़ाइलों **`myipcServer.c`** और **`myipcServer.h`** में आप **`SERVERPREFmyipc_subsystem`** संरचना की घोषणा और परिभाषा पा सकते हैं, जो मूल रूप से प्राप्त संदेश आईडी पर कॉल करने के लिए फ़ंक्शन को परिभाषित करता है (हमने 500 की शुरुआती संख्या दी है): 

{% tabs %}
{% tab title="myipcServer.c" %}
```c
/* Description of this subsystem, for use in direct RPC */
const struct SERVERPREFmyipc_subsystem SERVERPREFmyipc_subsystem = {
myipc_server_routine,
500, // start ID
501, // end ID
(mach_msg_size_t)sizeof(union __ReplyUnion__SERVERPREFmyipc_subsystem),
(vm_address_t)0,
{
{ (mig_impl_routine_t) 0,
// Function to call
(mig_stub_routine_t) _XSubtract, 3, 0, (routine_arg_descriptor_t)0, (mach_msg_size_t)sizeof(__Reply__Subtract_t)},
}
};
```
{% endtab %}

{% tab title="myipcServer.h" %}सर्वर द्वारा प्रदान की गई फ़ंक्शन का उपयोग करके एक नया ट्रांजेक्शन शुरू करें।{% endtab %}
```c
/* Description of this subsystem, for use in direct RPC */
extern const struct SERVERPREFmyipc_subsystem {
mig_server_routine_t	server;	/* Server routine */
mach_msg_id_t	start;	/* Min routine number */
mach_msg_id_t	end;	/* Max routine number + 1 */
unsigned int	maxsize;	/* Max msg size */
vm_address_t	reserved;	/* Reserved */
struct routine_descriptor	/* Array of routine descriptors */
routine[1];
} SERVERPREFmyipc_subsystem;
```
{% endtab %}
{% endtabs %}

पिछले संरचना के आधार पर **`myipc_server_routine`** फ़ंक्शन **संदेश आईडी** प्राप्त करेगा और सही फ़ंक्शन को कॉल करने के लिए लौटेगा:
```c
mig_external mig_routine_t myipc_server_routine
(mach_msg_header_t *InHeadP)
{
int msgh_id;

msgh_id = InHeadP->msgh_id - 500;

if ((msgh_id > 0) || (msgh_id < 0))
return 0;

return SERVERPREFmyipc_subsystem.routine[msgh_id].stub_routine;
}
```
इस उदाहरण में हमने परिभाषणों में केवल 1 फ़ंक्शन को परिभाषित किया है, लेकिन अगर हम अधिक फ़ंक्शनों को परिभाषित करते, तो वे **`SERVERPREFmyipc_subsystem`** के एरे के अंदर होते और पहला फ़ंक्शन **500** आईडी को सौंपा जाता, दूसरा **501** आईडी को...

अगर फ़ंक्शन से एक **जवाब** भेजने की उम्मीद थी तो फ़ंक्शन `mig_internal kern_return_t __MIG_check__Reply__<नाम>` भी मौजूद होता।

वास्तव में इस संरचना को पहचानना संभव है **`myipcServer.h`** में संरचना **`subsystem_to_name_map_myipc`** में (**अन्य फ़ाइलों में **`subsystem_to_name_map_***`**):
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
अंत में, सर्वर को काम करने के लिए एक और महत्वपूर्ण फ़ंक्शन **`myipc_server`** होगा, जो वास्तव में प्राप्त आईडी से संबंधित फ़ंक्शन को **कॉल** करेगा:

<pre class="language-c"><code class="lang-c">mig_external boolean_t myipc_server
(mach_msg_header_t *InHeadP, mach_msg_header_t *OutHeadP)
{
/*
* typedef struct {
* 	mach_msg_header_t Head;
* 	NDR_record_t NDR;
* 	kern_return_t RetCode;
* } mig_reply_error_t;
*/

mig_routine_t routine;

OutHeadP->msgh_bits = MACH_MSGH_BITS(MACH_MSGH_BITS_REPLY(InHeadP->msgh_bits), 0);
OutHeadP->msgh_remote_port = InHeadP->msgh_reply_port;
/* Minimal size: routine() will update it if different */
OutHeadP->msgh_size = (mach_msg_size_t)sizeof(mig_reply_error_t);
OutHeadP->msgh_local_port = MACH_PORT_NULL;
OutHeadP->msgh_id = InHeadP->msgh_id + 100;
OutHeadP->msgh_reserved = 0;

if ((InHeadP->msgh_id > 500) || (InHeadP->msgh_id &#x3C; 500) ||
<strong>	    ((routine = SERVERPREFmyipc_subsystem.routine[InHeadP->msgh_id - 500].stub_routine) == 0)) {
</strong>		((mig_reply_error_t *)OutHeadP)->NDR = NDR_record;
((mig_reply_error_t *)OutHeadP)->RetCode = MIG_BAD_ID;
return FALSE;
}
<strong>	(*routine) (InHeadP, OutHeadP);
</strong>	return TRUE;
}
</code></pre>

पिछले हाइलाइट किए गए लाइनों की जांच करें जो आईडी द्वारा कॉल करने के लिए फ़ंक्शन तक पहुंचती हैं।

निम्नलिखित कोड एक सरल **सर्वर** और **क्लाइंट** बनाने के लिए है जहां क्लाइंट सर्वर से फ़ंक्शन सब्ट्रैक्ट कॉल कर सकता है:

{% tabs %}
{% tab title="myipc_server.c" %}
```c
// gcc myipc_server.c myipcServer.c -o myipc_server

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcServer.h"

kern_return_t SERVERPREFSubtract(mach_port_t server_port, uint32_t n1, uint32_t n2)
{
printf("Received: %d - %d = %d\n", n1, n2, n1 - n2);
return KERN_SUCCESS;
}

int main() {

mach_port_t port;
kern_return_t kr;

// Register the mach service
kr = bootstrap_check_in(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_check_in() failed with code 0x%x\n", kr);
return 1;
}

// myipc_server is the function that handles incoming messages (check previous exlpanation)
mach_msg_server(myipc_server, sizeof(union __RequestUnion__SERVERPREFmyipc_subsystem), port, MACH_MSG_TIMEOUT_NONE);
}
```
{% endtab %}

{% tab title="myipc_client.c" %}यहाँ हम एक उदाहरण देख रहे हैं जो एक माइक्रोकर्नल एप्लिकेशन के साथ एक आईपीसी क्लाइंट को कैसे जोड़ता है। यह उदाहरण एक मैक एप्लिकेशन के लिए है जो एक आईपीसी सर्वर के साथ कम्यूनिकेट करता ह। यहाँ हम एक मैक एप्लिकेशन को एक आईपीसी सर्वर के साथ कम्यूनिकेट करने के लिए एक आईपीसी क्लाइंट को कैसे जोड़ सकते हैं इसका उदाहरण देखेंगे। %}
```c
// gcc myipc_client.c myipcUser.c -o myipc_client

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcUser.h"

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("Port right name %d\n", port);
USERPREFSubtract(port, 40, 2);
}
```
{% endtab %}
{% endtabs %}

### NDR\_record

NDR\_record को `libsystem_kernel.dylib` द्वारा निर्यात किया गया है, और यह एक संरचना है जो MIG को **डेटा को परिवर्तित करने की अनुमति देती है ताकि यह उस सिस्टम के प्रति निर्विकार हो जिसमें इसका उपयोग किया जा रहा है** क्योंकि MIG का उपयोग विभिन्न सिस्टमों के बीच किया जाने के लिए सोचा गया था (और केवल एक ही मशीन में नहीं).

यह दिलचस्प है क्योंकि यदि `_NDR_record` एक बाइनरी में एक निर्भरता के रूप में पाया जाता है (`jtool2 -S <binary> | grep NDR` या `nm`), तो इसका मतलब है कि बाइनरी एक MIG client या Server है।

इसके अतिरिक्त **MIG सर्वर** के पास `__DATA.__const` में डिस्पैच टेबल होता है (या macOS कर्नेल में `__CONST.__constdata` में और अन्य \*OS कर्नेल में `__DATA_CONST.__const` में होता है)। यह **`jtool2`** के साथ डंप किया जा सकता है।

और **MIG clients** `__mach_msg` के साथ भेजने के लिए `__NDR_record` का उपयोग करेंगे।

## बाइनरी विश्लेषण

### jtool

जैसे कि अब बहुत सारी बाइनरी MIG का उपयोग मच पोर्ट्स को उजागर करने के लिए करती हैं, इसे पहचानना और प्रत्येक संदेश आईडी के साथ MIG द्वारा कार्यों को पहचानना कितना दिलचस्प है, यह जानना उत्तम है।

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) एक Mach-O बाइनरी से MIG जानकारी को पार्स कर सकता है जिसमें संदेश आईडी और कार्य को पहचानने के लिए फ़ंक्शन को निर्दिष्ट किया जा सकता है:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
इसके अतिरिक्त, MIG functions केवल उस वास्तविक फ़ंक्शन के wrappers होते हैं जो कॉल किया जाता है, जिसका मतलब है कि उसके disassembly प्राप्त करने और BL के लिए grepping करने से आप वास्तविक फ़ंक्शन को खोजने में सक्षम हो सकते हैं:
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### असेम्बली

पहले ही उल्लेख किया गया था कि **प्राप्त संदेश आईडी के आधार पर सही फ़ंक्शन को बुलाने की जिम्मेदारी लेने वाला फ़ंक्शन** `myipc_server` था। हालांकि, आम तौर पर बाइनरी के प्रतीक (कोई फ़ंक्शन नाम नहीं) नहीं होते, इसलिए यह देकना दिलचस्प होता है कि **डीकंपाइल कैसे दिखता है** क्योंकि यह हमेशा बहुत समान होगा (इस फ़ंक्शन का कोड फ़ंक्शनों से अलग होता है):

{% tabs %}
{% tab title="मेरा आईपीसी सर्वर डीकंपाइल 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// सही फ़ंक्शन पॉइंटर्स खोजने के लिए प्रारंभिक निर्देश
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// इस फ़ंक्शन को पहचानने में मदद करने वाला sign_extend_64 कॉल
// यह rax में उस पॉइंटर को स्टोर करता है जिसे कॉल किया जाना चाहिए
// पता करें कि पता 0x100004040 (फ़ंक्शन पतों का एरे) का उपयोग किया गया है
// 0x1f4 = 500 (प्रारंभिक आईडी)
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// अगर - अन्यथा, अगर गलत होता है, जबकि अन्यथा सही फ़ंक्शन को बुलाता है और सही होता है
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// 2 तर्कों के साथ सही फ़ंक्शन को बुलाने वाला गणनात्मक पता
<strong>                    (var_20)(var_10, var_18);
</strong>                    var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
rax = var_4;
return rax;
}
</code></pre>
{% endtab %}

{% tab title="मेरा आईपीसी सर्वर डीकंपाइल 2" %}
यह एक अलग Hopper मुफ्त संस्करण में डीकंपाइल किया गया समान फ़ंक्शन है:

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// सही फ़ंक्शन पॉइंटर्स खोजने के लिए प्रारंभिक निर्देश
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f | 0x0;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 > 0x0) {
if (CPU_FLAGS &#x26; G) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 &#x3C; 0x0) {
if (CPU_FLAGS &#x26; L) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
// 0x1f4 = 500 (प्रारंभिक आईडी)
<strong>                    r8 = r8 - 0x1f4;
</strong>                    asm { smaddl     x8, w8, w9, x10 };
r8 = *(r8 + 0x8);
var_20 = r8;
r8 = r8 - 0x0;
if (r8 != 0x0) {
if (CPU_FLAGS &#x26; NE) {
r8 = 0x1;
}
}
// पिछले संस्करण की तरह समान अगर नहीं तो अगर गलती वापस आती है
// पता करें कि पता 0x100004040 (फ़ंक्शन पतों का एरे) का उपयोग किया गया है
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// फ़ंक्शन होना चाहिए वहाँ जहाँ गणना किया गया पता है
<strong>                            (var_20)(var_10, var_18);
</strong>                            var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
r0 = var_4;
return r0;
}

</code></pre>
{% endtab %}
{% endtabs %}

वास्तव में अगर आप **`0x100004000`** फ़ंक्शन पर जाते हैं तो आपको **`routine_descriptor`** संरचनाओं का एरे मिलेगा। संरचना का पहला तत्व वहाँ **फ़ंक्शन** का कार्यान्वयन किया गया है, और **संरचना 0x28 बाइट लेती है**, इसलिए प्रत्येक 0x28 बाइट (बाइट 0 से प्रारंभ होकर) आप 8 बाइट प्राप्त कर सकते हैं और वह **फ़ंक्शन का पता** होगा जो बुलाया जाएगा:

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

यह डेटा [**इस Hopper स्क्रिप्ट का उपयोग करके**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py) निकाला जा सकता है।
### डीबग

MIG द्वारा उत्पन्न कोड भी `kernel_debug` को कॉल करता है ताकि प्रवेश और निकासी पर ऑपरेशन के बारे में लॉग उत्पन्न किया जा सके। इन्हें **`trace`** या **`kdv`** का उपयोग करके जांचना संभव है: `kdv all | grep MIG`

## संदर्भ

* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
