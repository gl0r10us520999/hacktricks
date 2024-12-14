# Објекти у меморији

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

CF\* објекти долазе из CoreFoundation, који пружа више од 50 класа објеката као што су `CFString`, `CFNumber` или `CFAllocator`.

Све ове класе су инстанце класе `CFRuntimeClass`, која када се позове враћа индекс у `__CFRuntimeClassTable`. CFRuntimeClass је дефинисан у [**CFRuntime.h**](https://opensource.apple.com/source/CF/CF-1153.18/CFRuntime.h.auto.html):
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

Većina podataka koje koristi ObjectiveC runtime će se menjati tokom izvršavanja, stoga koristi neke sekcije iz **\_\_DATA** segmenta u memoriji:

* **`__objc_msgrefs`** (`message_ref_t`): Reference poruka
* **`__objc_ivar`** (`ivar`): Instancne promenljive
* **`__objc_data`** (`...`): Promenljivi podaci
* **`__objc_classrefs`** (`Class`): Reference klasa
* **`__objc_superrefs`** (`Class`): Reference superklasa
* **`__objc_protorefs`** (`protocol_t *`): Reference protokola
* **`__objc_selrefs`** (`SEL`): Reference selektora
* **`__objc_const`** (`...`): Klasa `r/o` podaci i drugi (nadamo se) konstantni podaci
* **`__objc_imageinfo`** (`version, flags`): Koristi se tokom učitavanja slike: Verzija trenutno `0`; Zastavice specificiraju unapred optimizovanu GC podršku, itd.
* **`__objc_protolist`** (`protocol_t *`): Lista protokola
* **`__objc_nlcatlist`** (`category_t`): Pokazivač na Non-Lazy Kategorije definisane u ovoj binarnoj datoteci
* **`__objc_catlist`** (`category_t`): Pokazivač na Kategorije definisane u ovoj binarnoj datoteci
* **`__objc_nlclslist`** (`classref_t`): Pokazivač na Non-Lazy Objective-C klase definisane u ovoj binarnoj datoteci
* **`__objc_classlist`** (`classref_t`): Pokazivači na sve Objective-C klase definisane u ovoj binarnoj datoteci

Takođe koristi nekoliko sekcija u **`__TEXT`** segmentu za čuvanje konstantnih vrednosti ako nije moguće pisati u ovu sekciju:

* **`__objc_methname`** (C-String): Imena metoda
* **`__objc_classname`** (C-String): Imena klasa
* **`__objc_methtype`** (C-String): Tipovi metoda

### Type Encoding

Objective-C koristi određeno mangle-ovanje za kodiranje selektora i tipova promenljivih jednostavnih i složenih tipova:

* Primitivni tipovi koriste prvo slovo tipa `i` za `int`, `c` za `char`, `l` za `long`... i koristi veliko slovo u slučaju da je bez znakova (`L` za `unsigned Long`).
* Drugi tipovi podataka čija su slova korišćena ili su posebna, koriste druga slova ili simbole kao što su `q` za `long long`, `b` za `bitfields`, `B` za `booleans`, `#` za `classes`, `@` za `id`, `*` za `char pointers`, `^` za generičke `pointers` i `?` za `undefined`.
* Nizovi, strukture i unije koriste `[`, `{` i `(`

#### Example Method Declaration

{% code overflow="wrap" %}
```objectivec
- (NSString *)processString:(id)input withOptions:(char *)options andError:(id)error;
```
{% endcode %}

Selektor bi bio `processString:withOptions:andError:`

#### Kodiranje tipa

* `id` se kodira kao `@`
* `char *` se kodira kao `*`

Puno kodiranje tipa za metodu je:
```less
@24@0:8@16*20^@24
```
#### Detailed Breakdown

1. **Return Type (`NSString *`)**: Kodiran kao `@` sa dužinom 24
2. **`self` (instanca objekta)**: Kodiran kao `@`, na offsetu 0
3. **`_cmd` (selektor)**: Kodiran kao `:`, na offsetu 8
4. **Prvi argument (`char * input`)**: Kodiran kao `*`, na offsetu 16
5. **Drugi argument (`NSDictionary * options`)**: Kodiran kao `@`, na offsetu 20
6. **Treći argument (`NSError ** error`)**: Kodiran kao `^@`, na offsetu 24

**Sa selektorom + kodiranjem možete rekonstruisati metodu.**

### **Classes**

Klase u Objective-C su strukture sa svojstvima, pokazivačima na metode... Moguće je pronaći strukturu `objc_class` u [**izvornom kodu**](https://opensource.apple.com/source/objc4/objc4-756.2/runtime/objc-runtime-new.h.auto.html):
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
Ova klasa koristi neke bitove polja isa da bi ukazala na određene informacije o klasi.

Zatim, struktura ima pokazivač na strukturu `class_ro_t` koja je smeštena na disku i sadrži atribute klase kao što su njeno ime, osnovne metode, svojstva i instance varijable.\
Tokom izvršavanja, dodatna struktura `class_rw_t` se koristi i sadrži pokazivače koji se mogu menjati, kao što su metode, protokoli, svojstva...

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
