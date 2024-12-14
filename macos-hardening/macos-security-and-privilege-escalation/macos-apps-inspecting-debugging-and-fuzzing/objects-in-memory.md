# Obiekty w pamięci

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}

## CFRuntimeClass

Obiekty CF\* pochodzą z CoreFoundation, które oferuje ponad 50 klas obiektów, takich jak `CFString`, `CFNumber` czy `CFAllocator`.

Wszystkie te klasy są instancjami klasy `CFRuntimeClass`, która po wywołaniu zwraca indeks do `__CFRuntimeClassTable`. CFRuntimeClass jest zdefiniowana w [**CFRuntime.h**](https://opensource.apple.com/source/CF/CF-1153.18/CFRuntime.h.auto.html):
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

### Sekcje pamięci używane

Większość danych używanych przez runtime ObjectiveC zmienia się podczas wykonywania, dlatego wykorzystuje niektóre sekcje z segmentu **\_\_DATA** w pamięci:

* **`__objc_msgrefs`** (`message_ref_t`): Referencje wiadomości
* **`__objc_ivar`** (`ivar`): Zmienne instancji
* **`__objc_data`** (`...`): Dane mutowalne
* **`__objc_classrefs`** (`Class`): Referencje klas
* **`__objc_superrefs`** (`Class`): Referencje klas nadrzędnych
* **`__objc_protorefs`** (`protocol_t *`): Referencje protokołów
* **`__objc_selrefs`** (`SEL`): Referencje selektorów
* **`__objc_const`** (`...`): Dane klas `r/o` i inne (mam nadzieję) stałe dane
* **`__objc_imageinfo`** (`version, flags`): Używane podczas ładowania obrazu: Wersja obecnie `0`; Flagi określają wsparcie dla preoptymalizowanego GC itp.
* **`__objc_protolist`** (`protocol_t *`): Lista protokołów
* **`__objc_nlcatlist`** (`category_t`): Wskaźnik do kategorii Non-Lazy zdefiniowanych w tym binarnym pliku
* **`__objc_catlist`** (`category_t`): Wskaźnik do kategorii zdefiniowanych w tym binarnym pliku
* **`__objc_nlclslist`** (`classref_t`): Wskaźnik do klas Objective-C Non-Lazy zdefiniowanych w tym binarnym pliku
* **`__objc_classlist`** (`classref_t`): Wskaźniki do wszystkich klas Objective-C zdefiniowanych w tym binarnym pliku

Wykorzystuje również kilka sekcji w segmencie **`__TEXT`** do przechowywania stałych wartości, jeśli nie jest możliwe zapisanie w tej sekcji:

* **`__objc_methname`** (C-String): Nazwy metod
* **`__objc_classname`** (C-String): Nazwy klas
* **`__objc_methtype`** (C-String): Typy metod

### Kodowanie typów

Objective-C używa pewnego mangle'owania do kodowania selektorów i typów zmiennych prostych i złożonych:

* Typy prymitywne używają pierwszej litery typu `i` dla `int`, `c` dla `char`, `l` dla `long`... i używają wielkiej litery w przypadku, gdy jest to typ unsigned (`L` dla `unsigned Long`).
* Inne typy danych, których litery są używane lub są specjalne, używają innych liter lub symboli, takich jak `q` dla `long long`, `b` dla `bitfields`, `B` dla `booleans`, `#` dla `classes`, `@` dla `id`, `*` dla `wskaźników char`, `^` dla ogólnych `wskaźników` i `?` dla `niezdefiniowanych`.
* Tablice, struktury i unie używają `[`, `{` i `(`

#### Przykład deklaracji metody

{% code overflow="wrap" %}
```objectivec
- (NSString *)processString:(id)input withOptions:(char *)options andError:(id)error;
```
{% endcode %}

Selektor to `processString:withOptions:andError:`

#### Kodowanie Typu

* `id` jest kodowane jako `@`
* `char *` jest kodowane jako `*`

Pełne kodowanie typu dla metody to:
```less
@24@0:8@16*20^@24
```
#### Szczegółowe Rozbicie

1. **Typ zwracany (`NSString *`)**: Zakodowany jako `@` o długości 24
2. **`self` (instancja obiektu)**: Zakodowany jako `@`, w przesunięciu 0
3. **`_cmd` (selektor)**: Zakodowany jako `:`, w przesunięciu 8
4. **Pierwszy argument (`char * input`)**: Zakodowany jako `*`, w przesunięciu 16
5. **Drugi argument (`NSDictionary * options`)**: Zakodowany jako `@`, w przesunięciu 20
6. **Trzeci argument (`NSError ** error`)**: Zakodowany jako `^@`, w przesunięciu 24

**Z selektorem + kodowaniem możesz odtworzyć metodę.**

### **Klasy**

Klasy w Objective-C to struktura z właściwościami, wskaźnikami do metod... Możliwe jest znalezienie struktury `objc_class` w [**kodzie źródłowym**](https://opensource.apple.com/source/objc4/objc4-756.2/runtime/objc-runtime-new.h.auto.html):
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
Ta klasa używa kilku bitów pola isa, aby wskazać pewne informacje o klasie.

Następnie struktura ma wskaźnik do struktury `class_ro_t` przechowywanej na dysku, która zawiera atrybuty klasy, takie jak jej nazwa, metody bazowe, właściwości i zmienne instancji.\
Podczas działania programu używana jest dodatkowa struktura `class_rw_t`, która zawiera wskaźniki, które można zmieniać, takie jak metody, protokoły, właściwości...

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
