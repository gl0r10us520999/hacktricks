# macOS XPC कनेक्टिंग प्रोसेस चेक

{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) चेक करें!
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* हैकिंग ट्रिक्स साझा करें और [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

## XPC कनेक्टिंग प्रोसेस चेक

जब एक XPC सेवा से कनेक्शन स्थापित किया जाता है, तो सर्वर यह जांचेगा कि क्या कनेक्शन की अनुमति है। ये सामान्यतः की जाने वाली जांचें हैं:

1. जांचें कि क्या कनेक्टिंग **प्रोसेस एक Apple-साइन किए गए** प्रमाणपत्र के साथ साइन किया गया है (जो केवल Apple द्वारा दिया जाता है)।
* यदि यह **सत्यापित नहीं है**, तो एक हमलावर एक **नकली प्रमाणपत्र** बना सकता है जो किसी अन्य जांच से मेल खाता है।
2. जांचें कि क्या कनेक्टिंग प्रोसेस **संस्थान के प्रमाणपत्र** के साथ साइन किया गया है (टीम आईडी सत्यापन)।
* यदि यह **सत्यापित नहीं है**, तो Apple से **कोई भी डेवलपर प्रमाणपत्र** साइनिंग के लिए उपयोग किया जा सकता है, और सेवा से कनेक्ट किया जा सकता है।
3. जांचें कि क्या कनेक्टिंग प्रोसेस **एक उचित बंडल आईडी** रखता है।
* यदि यह **सत्यापित नहीं है**, तो **उसी संगठन द्वारा साइन किए गए** किसी भी टूल का उपयोग XPC सेवा के साथ इंटरैक्ट करने के लिए किया जा सकता है।
4. (4 या 5) जांचें कि क्या कनेक्टिंग प्रोसेस में **एक उचित सॉफ़्टवेयर संस्करण संख्या** है।
* यदि यह **सत्यापित नहीं है**, तो एक पुराना, असुरक्षित क्लाइंट, जो प्रोसेस इंजेक्शन के प्रति संवेदनशील है, XPC सेवा से कनेक्ट करने के लिए उपयोग किया जा सकता है, भले ही अन्य जांचें लागू हों।
5. (4 या 5) जांचें कि क्या कनेक्टिंग प्रोसेस में खतरनाक अधिकारों के बिना **एक हार्डनड रनटाइम** है (जैसे कि वे जो मनमाने लाइब्रेरी लोड करने या DYLD env vars का उपयोग करने की अनुमति देते हैं)।
* यदि यह **सत्यापित नहीं है**, तो क्लाइंट **कोड इंजेक्शन के प्रति संवेदनशील** हो सकता है।
6. जांचें कि क्या कनेक्टिंग प्रोसेस में एक **अधिकार** है जो इसे सेवा से कनेक्ट करने की अनुमति देता है। यह Apple बाइनरी के लिए लागू है।
7. **सत्यापन** कनेक्टिंग **क्लाइंट के ऑडिट टोकन** पर **आधारित** होना चाहिए **इसके प्रोसेस आईडी** (**PID**) के बजाय क्योंकि पूर्व **PID पुन: उपयोग हमलों** को रोकता है।
* डेवलपर्स **कभी-कभी ऑडिट टोकन** API कॉल का उपयोग करते हैं क्योंकि यह **निजी** है, इसलिए Apple इसे **किसी भी समय बदल** सकता है। इसके अतिरिक्त, निजी API का उपयोग Mac App Store ऐप्स में अनुमति नहीं है।
* यदि विधि **`processIdentifier`** का उपयोग किया जाता है, तो यह संवेदनशील हो सकता है।
* **`xpc_dictionary_get_audit_token`** का उपयोग **`xpc_connection_get_audit_token`** के बजाय किया जाना चाहिए, क्योंकि बाद वाला भी [कुछ स्थितियों में संवेदनशील हो सकता है](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)।

### संचार हमले

PID पुन: उपयोग हमले के बारे में अधिक जानकारी के लिए चेक करें:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

अधिक जानकारी के लिए **`xpc_connection_get_audit_token`** हमले की जांच करें:

{% content-ref url="macos-xpc_connection_get_audit_token-attack.md" %}
[macos-xpc\_connection\_get\_audit\_token-attack.md](macos-xpc\_connection\_get\_audit\_token-attack.md)
{% endcontent-ref %}

### Trustcache - डाउनग्रेड हमलों की रोकथाम

Trustcache एक रक्षात्मक विधि है जो Apple Silicon मशीनों में पेश की गई है जो Apple बाइनरी के CDHSAH का एक डेटाबेस संग्रहीत करती है ताकि केवल अनुमत गैर-संशोधित बाइनरी को निष्पादित किया जा सके। जो डाउनग्रेड संस्करणों के निष्पादन को रोकता है।

### कोड उदाहरण

सर्वर इस **सत्यापन** को **`shouldAcceptNewConnection`** नामक एक फ़ंक्शन में लागू करेगा।

{% code overflow="wrap" %}
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
{% endcode %}

ऑब्जेक्ट NSXPCConnection में एक **निजी** प्रॉपर्टी **`auditToken`** है (जो उपयोग की जानी चाहिए लेकिन बदल सकती है) और एक **सार्वजनिक** प्रॉपर्टी **`processIdentifier`** है (जो उपयोग नहीं की जानी चाहिए)।

जोड़ने वाली प्रक्रिया को कुछ इस तरह से सत्यापित किया जा सकता है:

{% code overflow="wrap" %}
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements with the PID (vulnerable)
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);

// Check the requirements wuing the auditToken (secure)
SecTaskRef taskRef = SecTaskCreateWithAuditToken(NULL, ((ExtendedNSXPCConnection*)newConnection).auditToken);
SecTaskValidateForRequirement(taskRef, (__bridge CFStringRef)(requirementString))
```
{% endcode %}

यदि एक डेवलपर क्लाइंट के संस्करण की जांच नहीं करना चाहता है, तो वह कम से कम यह जांच सकता है कि क्लाइंट प्रक्रिया इंजेक्शन के लिए संवेदनशील नहीं है:

{% code overflow="wrap" %}
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page.
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
return Yes; // Accept connection
}
```
{% endcode %}

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PRs सबमिट करें।

</details>
{% endhint %}
