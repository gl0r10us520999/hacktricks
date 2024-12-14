# Bellekteki Nesneler

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## CFRuntimeClass

CF\* nesneleri, `CFString`, `CFNumber` veya `CFAllocator` gibi 50'den fazla nesne sınıfı sağlayan CoreFoundation'dan gelir.

Tüm bu sınıflar, çağrıldığında `__CFRuntimeClassTable`'a bir indeks döndüren `CFRuntimeClass` sınıfının örnekleridir. CFRuntimeClass, [**CFRuntime.h**](https://opensource.apple.com/source/CF/CF-1153.18/CFRuntime.h.auto.html) dosyasında tanımlanmıştır:
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

### Bellek bölümleri kullanıldı

ObjectiveC çalışma zamanı tarafından kullanılan verilerin çoğu yürütme sırasında değişecektir, bu nedenle bellekteki **\_\_DATA** segmentinden bazı bölümleri kullanır:

* **`__objc_msgrefs`** (`message_ref_t`): Mesaj referansları
* **`__objc_ivar`** (`ivar`): Örnek değişkenleri
* **`__objc_data`** (`...`): Değişken veri
* **`__objc_classrefs`** (`Class`): Sınıf referansları
* **`__objc_superrefs`** (`Class`): Üst sınıf referansları
* **`__objc_protorefs`** (`protocol_t *`): Protokol referansları
* **`__objc_selrefs`** (`SEL`): Seçici referansları
* **`__objc_const`** (`...`): Sınıf `r/o` verisi ve diğer (umarım) sabit veriler
* **`__objc_imageinfo`** (`version, flags`): Görüntü yükleme sırasında kullanılır: Mevcut sürüm `0`; Bayraklar önceden optimize edilmiş GC desteğini belirtir, vb.
* **`__objc_protolist`** (`protocol_t *`): Protokol listesi
* **`__objc_nlcatlist`** (`category_t`): Bu ikili dosyada tanımlanan Tembel Olmayan Kategorilere işaretçi
* **`__objc_catlist`** (`category_t`): Bu ikili dosyada tanımlanan Kategorilere işaretçi
* **`__objc_nlclslist`** (`classref_t`): Bu ikili dosyada tanımlanan Tembel Olmayan Objective-C sınıflarına işaretçi
* **`__objc_classlist`** (`classref_t`): Bu ikili dosyada tanımlanan tüm Objective-C sınıflarına işaretçiler

Ayrıca, bu bölümde yazmanın mümkün olmadığı sabit değerleri depolamak için **`__TEXT`** segmentinde birkaç bölüm kullanır:

* **`__objc_methname`** (C-String): Metot adları
* **`__objc_classname`** (C-String): Sınıf adları
* **`__objc_methtype`** (C-String): Metot türleri

### Tür Kodlaması

Objective-C, basit ve karmaşık türlerin seçici ve değişken türlerini kodlamak için bazı karıştırmalar kullanır:

* Temel türler, türün ilk harfini kullanır `i` için `int`, `c` için `char`, `l` için `long`... ve işaretsizse büyük harf kullanır (`L` için `unsigned Long`).
* Harfleri kullanılan veya özel olan diğer veri türleri, `long long` için `q`, `bitfields` için `b`, `booleans` için `B`, `classes` için `#`, `id` için `@`, `char pointers` için `*`, `generic pointers` için `^` ve `undefined` için `?` gibi diğer harfler veya semboller kullanır.
* Diziler, yapılar ve birleşimler `[`, `{` ve `(` kullanır.

#### Örnek Metot Bildirimi

{% code overflow="wrap" %}
```objectivec
- (NSString *)processString:(id)input withOptions:(char *)options andError:(id)error;
```
{% endcode %}

Seçici `processString:withOptions:andError:` olacaktır.

#### Tür Kodlaması

* `id` `@` olarak kodlanır
* `char *` `*` olarak kodlanır

Yöntem için tam tür kodlaması:
```less
@24@0:8@16*20^@24
```
#### Detaylı Analiz

1. **Dönüş Tipi (`NSString *`)**: `@` ile kodlanmış, uzunluk 24
2. **`self` (nesne örneği)**: `@` ile kodlanmış, ofset 0
3. **`_cmd` (seçici)**: `:` ile kodlanmış, ofset 8
4. **İlk argüman (`char * input`)**: `*` ile kodlanmış, ofset 16
5. **İkinci argüman (`NSDictionary * options`)**: `@` ile kodlanmış, ofset 20
6. **Üçüncü argüman (`NSError ** error`)**: `^@` ile kodlanmış, ofset 24

**Seçici ile birlikte kodlama, metodu yeniden oluşturmanıza olanak tanır.**

### **Sınıflar**

Objective-C'deki sınıflar, özellikler, yöntem işaretçileri ile bir yapıdadır... `objc_class` yapısını [**kaynak kodda**](https://opensource.apple.com/source/objc4/objc4-756.2/runtime/objc-runtime-new.h.auto.html) bulmak mümkündür:
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
Bu sınıf, sınıf hakkında bazı bilgileri belirtmek için isa alanının bazı bitlerini kullanır.

Daha sonra, yapı, sınıfın adı, temel yöntemleri, özellikleri ve örnek değişkenleri gibi sınıfın niteliklerini içeren disk üzerinde saklanan `class_ro_t` yapısına bir işaretçi içerir.\
Çalışma zamanında, yöntemler, protokoller, özellikler gibi değiştirilebilen işaretçileri içeren ek bir yapı `class_rw_t` kullanılır...
