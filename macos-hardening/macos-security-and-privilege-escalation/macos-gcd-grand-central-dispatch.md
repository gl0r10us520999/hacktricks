# macOS GCD - Grand Central Dispatch

{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 पर **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
{% endhint %}

## मूल जानकारी

**ग्रैंड सेंट्रल डिस्पैच (GCD),** जिसे **लिबडिस्पैच** (`libdispatch.dyld`) के रूप में भी जाना जाता है, macOS और iOS दोनों में उपलब्ध है। यह एक प्रौद्योगिकी है जिसे Apple ने बनाया है ताकि एप्लिकेशन समर्थन को अनुक्रमिक (मल्टीथ्रेडेड) क्रियान्वयन के लिए अनुकूल बनाए रख सके मल्टीकोर हार्डवेयर पर।

**GCD** आपके एप्लिकेशन को **ब्लॉक ऑब्जेक्ट** के रूप में **कार्यों को सबमिट** करने के लिए **FIFO कतारें** प्रदान करता है और प्रबंधित करता है। डिस्पैच कतारों में सबमिट किए गए ब्लॉक को सिस्टम द्वारा पूरी तरह से प्रबंधित धागों पर **क्रियान्वित** किया जाता है। GCD स्वचालित रूप से धागों को निर्मित करता है जो डिस्पैच कतारों में कार्यों को क्रियान्वित करने के लिए और उपलब्ध कोरों पर चलाने के लिए कार्यान्वित करता है।

{% hint style="success" %}
सारांश में, **पैरलल** में कोड को निष्पादित करने के लिए, प्रक्रियाएँ **GCD को कोड के ब्लॉक भेज सकती हैं**, जो उनके निष्पादन का ध्यान रखेगा। इसलिए, प्रक्रियाएँ नए धागे नहीं बनाती हैं; **GCD अपने धागों के साथ दिए गए कोड को निष्पादित करता है** (जो आवश्यक होने पर बढ़ सकते या कम हो सकते हैं)।
{% endhint %}

यह पैरलल निष्पादन सफलतापूर्वक प्रबंधित करने में बहुत सहायक है, प्रक्रियाएँ बनाती हैं धागे की संख्या को कम और पैरलल निष्पादन को अनुकूलित करने के लिए। यह उन कार्यों के लिए आदर्श है जिन्हें **बड़ी पैरललता** (ब्रूट-फोर्सिंग?) की आवश्यकता होती है या जिन्हें मुख्य धागा ब्लॉक नहीं करना चाहिए: उदाहरण के लिए, iOS पर मुख्य धागा UI इंटरेक्शन को संभालता है, इसलिए ऐसी किसी भी अन्य कार्यक्षमता जो ऐप्लिकेशन को अटका सकती है (खोज, वेब तक पहुंचना, फ़ाइल पढ़ना...) इस तरह से प्रबंधित किया जाता है।

### ब्लॉक्स

एक ब्लॉक एक **स्वयं समाहित कोड का खंड** है (जैसे एक फ़ंक्शन जिसमें तर्क वाले वेरिएबल्स के साथ एक मान लौटाता है) और बाउंड वेरिएबल्स को भी निर्दिष्ट कर सकता है।\
हालांकि, कंपाइलर स्तर पर ब्लॉक्स मौजूद नहीं हैं, वे `os_object`s हैं। इन ऑब्जेक्ट्स में दो संरचनाएँ होती हैं:

* **ब्लॉक लिटरल**:&#x20;
* यह **`isa`** फ़ील्ड से शुरू होता है, जो ब्लॉक की क्लास को दर्शाता है:
* `NSConcreteGlobalBlock` (डेटा से ब्लॉक्स)
* `NSConcreteMallocBlock` (हीप में ब्लॉक्स)
* `NSConcreateStackBlock` (स्टैक में ब्लॉक्स)
* इसमें **`flags`** होते हैं (ब्लॉक डेस्क्रिप्टर में मौजूद फ़ील्ड्स की सूचना देते हैं) और कुछ आरक्षित बाइट्स
* कॉल करने के लिए फ़ंक्शन पॉइंटर
* ब्लॉक डेस्क्रिप्टर के लिए पॉइंटर
* ब्लॉक इम्पोर्टेड वेरिएबल्स (अगर कोई हो)
* **ब्लॉक डेस्क्रिप्टर**: इसका आकार उपस्थित डेटा पर निर्भर करता है (जैसा पिछले फ़्लैग्स में इंदिकिट किया गया है)
* इसमें कुछ आरक्षित बाइट्स होते हैं
* इसका आकार
* आम तौर पर इसमें पैराम्स के लिए कितनी जगह की आवश्यकता है यह जानने के लिए एक ऑब्जेक्टिव-सी स्टाइल हस्ताक्षर के लिए एक पॉइंटर होता है (फ़्लैग `BLOCK_HAS_SIGNATURE`)
* यदि वेरिएबल्स संदर्भित हैं, तो इस ब्लॉक में भी एक कॉपी हेल्पर (मूल्य की कॉपी करना) और डिस्पोज़ हेल्पर (इसे मुक्त करना) के पॉइंटर होंगे।

### कतारें

एक डिस्पैच कतार एक नामित ऑब्जेक्ट है जो कार्यों के लिए ब्लॉक्स की FIFO क्रमबद्धता प्रदान करता है।

कतारों में सेट किए गए ब्लॉक्स को क्रियान्वित करने के लिए, और इनमें 2 मोड हैं: `DISPATCH_QUEUE_SERIAL` और `DISPATCH_QUEUE_CONCURRENT`। बेशक **सीरियल** वाला **रेस कंडीशन** समस्याएँ नहीं होंगी क्योंकि एक ब्लॉक को तब तक क्रियान्वित नहीं किया जाएगा जब तक पिछला नहीं समाप्त हो जाएगा। लेकिन **कतार के दूसरे प्रकार में यह समस्या हो सकती है**।

डिफ़ॉल्ट कतारें:

* `.main-thread`: `dispatch_get_main_queue()` से
* `.libdispatch-manager`: GCD की कतार प्रबंधक
* `.root.libdispatch-manager`: GCD की कतार प्रबंधक
* `.root.maintenance-qos`: सबसे कम प्राथमिकता वाले कार्य
* `.root.maintenance-qos.overcommit`
* `.root.background-qos`: `DISPATCH_QUEUE_PRIORITY_BACKGROUND` के रूप में उपलब्ध है
* `.root.background-qos.overcommit`
* `.root.utility-qos`: `DISPATCH_QUEUE_PRIORITY_NON_INTERACTIVE` के रूप में उपलब्ध है
* `.root.utility-qos.overcommit`
* `.root.default-qos`: `DISPATCH_QUEUE_PRIORITY_DEFAULT` के रूप में उपलब्ध है
* `.root.background-qos.overcommit`
* `.root.user-initiated-qos`: `DISPATCH_QUEUE_PRIORITY_HIGH` के रूप में उपलब्ध है
* `.root.background-qos.overcommit`
* `.root.user-interactive-qos`: सबसे उच्च प्राथमिकता

ध्यान दें कि यह सिस्टम होगा जो **हर समय किस धागा को किस कतार का हैंडल करेगा** (कई धागे एक ही कतार में काम कर सकते हैं या एक ही धागा किसी समय अलग-अलग कतारों में काम कर सकता है)

#### गुण

**`dispatch_queue_create`** के साथ कतार बनाते समय तीसरा तत्व **`dispatch_queue_attr_t`** होता है, जो आम तौर पर या तो `DISPATCH_QUEUE_SERIAL` (जो वास्तव में NULL है) होता है या `DISPATCH_QUEUE_CONCURRENT` जो एक `dispatch_queue_attr_t` स्ट्रक्टर का पॉइंटर होता है जो कतार
```c
struct Block {
void *isa; // NSConcreteStackBlock,...
int flags;
int reserved;
void *invoke;
struct BlockDescriptor *descriptor;
// captured variables go here
};
```
और यह **पैरललिज़्म** का उपयोग करने के लिए एक उदाहरण है **`dispatch_async`** के साथ:
```objectivec
#import <Foundation/Foundation.h>

// Define a block
void (^backgroundTask)(void) = ^{
// Code to be executed in the background
for (int i = 0; i < 10; i++) {
NSLog(@"Background task %d", i);
sleep(1);  // Simulate a long-running task
}
};

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Create a dispatch queue
dispatch_queue_t backgroundQueue = dispatch_queue_create("com.example.backgroundQueue", NULL);

// Submit the block to the queue for asynchronous execution
dispatch_async(backgroundQueue, backgroundTask);

// Continue with other work on the main queue or thread
for (int i = 0; i < 10; i++) {
NSLog(@"Main task %d", i);
sleep(1);  // Simulate a long-running task
}
}
return 0;
}
```
## स्विफ्ट

**`libswiftDispatch`** एक लाइब्रेरी है जो ग्रांड सेंट्रल डिस्पैच (GCD) फ्रेमवर्क के लिए **स्विफ्ट बाइंडिंग** प्रदान करती है जो मूल रूप से C में लिखी गई है।\
**`libswiftDispatch`** लाइब्रेरी C GCD APIs को एक और स्विफ्ट-मित्र इंटरफेस में लपेटती है, जिससे स्विफ्ट डेवलपर्स को GCD के साथ काम करना आसान और समझने में अधिक सुविधा होती है।

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**कोड उदाहरण**:
```swift
import Foundation

// Define a closure (the Swift equivalent of a block)
let backgroundTask: () -> Void = {
for i in 0..<10 {
print("Background task \(i)")
sleep(1)  // Simulate a long-running task
}
}

// Entry point
autoreleasepool {
// Create a dispatch queue
let backgroundQueue = DispatchQueue(label: "com.example.backgroundQueue")

// Submit the closure to the queue for asynchronous execution
backgroundQueue.async(execute: backgroundTask)

// Continue with other work on the main queue
for i in 0..<10 {
print("Main task \(i)")
sleep(1)  // Simulate a long-running task
}
}
```
## फ्रिडा

निम्नलिखित फ्रिडा स्क्रिप्ट का उपयोग कई `डिस्पैच` फ़ंक्शन में **हुक करने** और कतार का नाम, बैकट्रेस और ब्लॉक निकालने के लिए किया जा सकता है: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
```bash
frida -U <prog_name> -l libdispatch.js

dispatch_sync
Calling queue: com.apple.UIKit._UIReusePool.reuseSetAccess
Callback function: 0x19e3a6488 UIKitCore!__26-[_UIReusePool addObject:]_block_invoke
Backtrace:
0x19e3a6460 UIKitCore!-[_UIReusePool addObject:]
0x19e3a5db8 UIKitCore!-[UIGraphicsRenderer _enqueueContextForReuse:]
0x19e3a57fc UIKitCore!+[UIGraphicsRenderer _destroyCGContext:withRenderer:]
[...]
```
## गिडरा

वर्तमान में गिडरा को न तो ObjectiveC **`dispatch_block_t`** संरचना समझता है, न ही **`swift_dispatch_block`** को।

तो यदि आप चाहते हैं कि यह उन्हें समझे, तो आप बस **उन्हें घोषित** कर सकते हैं:

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

फिर, कोड में एक स्थान ढूंढें जहाँ वे **उपयोग** किए जा रहे हैं:

{% hint style="success" %}
"ब्लॉक" के सभी संदर्भों का ध्यान रखें ताकि आप समझ सकें कि संरचना का उपयोग कैसे किया जा रहा है।
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1164).png" alt="" width="563"><figcaption></figcaption></figure>

माउस दायां क्लिक करें -> परिवर्तन चर करें और इस मामले में **`swift_dispatch_block`** का चयन करें:

<figure><img src="../../.gitbook/assets/image (1165).png" alt="" width="563"><figcaption></figcaption></figure>

गिडरा स्वचालित रूप से सब कुछ लिख देगा:

<figure><img src="../../.gitbook/assets/image (1166).png" alt="" width="563"><figcaption></figcaption></figure>

## संदर्भ

* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
