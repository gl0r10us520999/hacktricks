# Objects in memory

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

## CFRuntimeClass

CF\* ऑब्जेक्ट CoreFoundation से आते हैं, जो `CFString`, `CFNumber` या `CFAllocator` जैसे 50 से अधिक ऑब्जेक्ट क्लास प्रदान करता है।

इन सभी क्लासों के उदाहरण `CFRuntimeClass` क्लास के होते हैं, जिसे कॉल करने पर यह `__CFRuntimeClassTable` का एक इंडेक्स लौटाता है। CFRuntimeClass को [**CFRuntime.h**](https://opensource.apple.com/source/CF/CF-1153.18/CFRuntime.h.auto.html) में परिभाषित किया गया है:
```objectivec
// Some comments were added to the original code

enum { // Version field constants
_kCFRuntimeScannedObject =     (1UL << 0),
_kCFRuntimeResourcefulObject = (1UL << 2),  // tells CFRuntime to make use of the reclaim field
_kCFRuntimeCustomRefCount =    (1UL << 3),  // tells CFRuntime to make use of the refcount field
_kCFRuntimeRequiresAlignment = (1UL << 4),  // tells CFRuntime to make use of the requiredAlignment field
};

typedef struct __CFRuntimeClass {
CFIndex version;  // This is made a bitwise OR with the relevant previous flags

const char *className; // must be a pure ASCII string, nul-terminated
void (*init)(CFTypeRef cf);  // Initializer function
CFTypeRef (*copy)(CFAllocatorRef allocator, CFTypeRef cf); // Copy function, taking CFAllocatorRef and CFTypeRef to copy
void (*finalize)(CFTypeRef cf); // Finalizer function
Boolean (*equal)(CFTypeRef cf1, CFTypeRef cf2); // Function to be called by CFEqual()
CFHashCode (*hash)(CFTypeRef cf); // Function to be called by CFHash()
CFStringRef (*copyFormattingDesc)(CFTypeRef cf, CFDictionaryRef formatOptions); // Provides a CFStringRef with a textual description of the object// return str with retain
CFStringRef (*copyDebugDesc)(CFTypeRef cf);	// CFStringRed with textual description of the object for CFCopyDescription

#define CF_RECLAIM_AVAILABLE 1
void (*reclaim)(CFTypeRef cf); // Or in _kCFRuntimeResourcefulObject in the .version to indicate this field should be used
// It not null, it's called when the last reference to the object is released

#define CF_REFCOUNT_AVAILABLE 1
// If not null, the following is called when incrementing or decrementing reference count
uint32_t (*refcount)(intptr_t op, CFTypeRef cf); // Or in _kCFRuntimeCustomRefCount in the .version to indicate this field should be used
// this field must be non-NULL when _kCFRuntimeCustomRefCount is in the .version field
// - if the callback is passed 1 in 'op' it should increment the 'cf's reference count and return 0
// - if the callback is passed 0 in 'op' it should return the 'cf's reference count, up to 32 bits
// - if the callback is passed -1 in 'op' it should decrement the 'cf's reference count; if it is now zero, 'cf' should be cleaned up and deallocated (the finalize callback above will NOT be called unless the process is running under GC, and CF does not deallocate the memory for you; if running under GC, finalize should do the object tear-down and free the object memory); then return 0
// remember to use saturation arithmetic logic and stop incrementing and decrementing when the ref count hits UINT32_MAX, or you will have a security bug
// remember that reference count incrementing/decrementing must be done thread-safely/atomically
// objects should be created/initialized with a custom ref-count of 1 by the class creation functions
// do not attempt to use any bits within the CFRuntimeBase for your reference count; store that in some additional field in your CF object

#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wmissing-field-initializers"
#define CF_REQUIRED_ALIGNMENT_AVAILABLE 1
// If not 0, allocation of object must be on this boundary
uintptr_t requiredAlignment; // Or in _kCFRuntimeRequiresAlignment in the .version field to indicate this field should be used; the allocator to _CFRuntimeCreateInstance() will be ignored in this case; if this is less than the minimum alignment the system supports, you'll get higher alignment; if this is not an alignment the system supports (e.g., most systems will only support powers of two, or if it is too high), the result (consequences) will be up to CF or the system to decide

} CFRuntimeClass;
```
## Objective-C

### Memory sections used

Most of the data used by ObjectiveC runtime will change during the execution, therefore it uses some sections from the **\_\_DATA** segment in memory:

* **`__objc_msgrefs`** (`message_ref_t`): संदेश संदर्भ
* **`__objc_ivar`** (`ivar`): उदाहरण चर
* **`__objc_data`** (`...`): परिवर्तनीय डेटा
* **`__objc_classrefs`** (`Class`): वर्ग संदर्भ
* **`__objc_superrefs`** (`Class`): सुपरक्लास संदर्भ
* **`__objc_protorefs`** (`protocol_t *`): प्रोटोकॉल संदर्भ
* **`__objc_selrefs`** (`SEL`): चयनकर्ता संदर्भ
* **`__objc_const`** (`...`): वर्ग `r/o` डेटा और अन्य (उम्मीद है) स्थिर डेटा
* **`__objc_imageinfo`** (`version, flags`): छवि लोड के दौरान उपयोग किया जाता है: वर्तमान संस्करण `0`; ध्वज पूर्व-ऑप्टिमाइज़्ड GC समर्थन आदि निर्दिष्ट करते हैं।
* **`__objc_protolist`** (`protocol_t *`): प्रोटोकॉल सूची
* **`__objc_nlcatlist`** (`category_t`): इस बाइनरी में परिभाषित नॉन-लेज़ी श्रेणियों के लिए पॉइंटर
* **`__objc_catlist`** (`category_t`): इस बाइनरी में परिभाषित श्रेणियों के लिए पॉइंटर
* **`__objc_nlclslist`** (`classref_t`): इस बाइनरी में परिभाषित नॉन-लेज़ी ऑब्जेक्टिव-सी वर्गों के लिए पॉइंटर
* **`__objc_classlist`** (`classref_t`): इस बाइनरी में परिभाषित सभी ऑब्जेक्टिव-सी वर्गों के लिए पॉइंटर

It also uses a few sections in the **`__TEXT`** segment to store constan values of it's not possible to write in this section:

* **`__objc_methname`** (C-String): विधि नाम
* **`__objc_classname`** (C-String): वर्ग नाम
* **`__objc_methtype`** (C-String): विधि प्रकार

### Type Encoding

Objective-c uses some mangling to encode selector and variable types of simple and complex types:

* Primitive types use their first letter of the type `i` for `int`, `c` for `char`, `l` for `long`... and uses the capital letter in case it's unsigned (`L` for `unsigned Long`).
* Other data types whose letters are used or are special, use other letters or symbols like `q` for `long long`, `b` for `bitfields`, `B` for `booleans`, `#` for `classes`, `@` for `id`, `*` for `char pointers` , `^` for generic `pointers` and `?` for `undefined`.
* Arrays, structures and unions use `[`, `{` and `(`

#### Example Method Declaration

{% code overflow="wrap" %}
```objectivec
- (NSString *)processString:(id)input withOptions:(char *)options andError:(id)error;
```
{% endcode %}

चुनावकर्ता होगा `processString:withOptions:andError:`

#### प्रकार एन्कोडिंग

* `id` को `@` के रूप में एन्कोड किया गया है
* `char *` को `*` के रूप में एन्कोड किया गया है

विधि के लिए पूर्ण प्रकार एन्कोडिंग है:
```less
@24@0:8@16*20^@24
```
#### विस्तृत विश्लेषण

1. **रिटर्न प्रकार (`NSString *`)**: `@` के रूप में एन्कोडेड, लंबाई 24
2. **`self` (ऑब्जेक्ट उदाहरण)**: `@` के रूप में एन्कोडेड, ऑफसेट 0 पर
3. **`_cmd` (सेलेक्टर)**: `:` के रूप में एन्कोडेड, ऑफसेट 8 पर
4. **पहला तर्क (`char * input`)**: `*` के रूप में एन्कोडेड, ऑफसेट 16 पर
5. **दूसरा तर्क (`NSDictionary * options`)**: `@` के रूप में एन्कोडेड, ऑफसेट 20 पर
6. **तीसरा तर्क (`NSError ** error`)**: `^@` के रूप में एन्कोडेड, ऑफसेट 24 पर

**सेलेक्टर + एन्कोडिंग के साथ आप विधि को पुनर्निर्माण कर सकते हैं।**

### **क्लासेस**

Objective-C में क्लास एक संरचना है जिसमें प्रॉपर्टीज, मेथड पॉइंटर्स... शामिल हैं। आप [**स्रोत कोड**](https://opensource.apple.com/source/objc4/objc4-756.2/runtime/objc-runtime-new.h.auto.html) में संरचना `objc_class` को ढूंढना संभव है:
```objectivec
struct objc_class : objc_object {
// Class ISA;
Class superclass;
cache_t cache;             // formerly cache pointer and vtable
class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

class_rw_t *data() {
return bits.data();
}
void setData(class_rw_t *newData) {
bits.setData(newData);
}

void setInfo(uint32_t set) {
assert(isFuture()  ||  isRealized());
data()->setFlags(set);
}
[...]
```
यह क्लास isa फ़ील्ड के कुछ बिट्स का उपयोग क्लास के बारे में कुछ जानकारी इंगित करने के लिए करती है।

फिर, स्ट्रक्चर में डिस्क पर संग्रहीत `class_ro_t` स्ट्रक्चर का एक पॉइंटर होता है, जिसमें क्लास के गुण जैसे इसका नाम, बेस मेथड्स, प्रॉपर्टीज और इंस्टेंस वेरिएबल्स होते हैं।\
रनटाइम के दौरान, एक अतिरिक्त स्ट्रक्चर `class_rw_t` का उपयोग किया जाता है जिसमें पॉइंटर्स होते हैं जिन्हें बदला जा सकता है जैसे मेथड्स, प्रोटोकॉल्स, प्रॉपर्टीज...
