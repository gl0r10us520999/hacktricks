# Objetos en memoria

{% hint style="success" %}
Aprende y practica Hacking en AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

## CFRuntimeClass

Los objetos CF\* provienen de CoreFoundation, que proporciona más de 50 clases de objetos como `CFString`, `CFNumber` o `CFAllocatior`.

Todas estas clases son instancias de la clase `CFRuntimeClass`, la cual al ser llamada devuelve un índice a la `__CFRuntimeClassTable`. CFRuntimeClass está definida en [**CFRuntime.h**](https://opensource.apple.com/source/CF/CF-1153.18/CFRuntime.h.auto.html):
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

### Secciones de memoria utilizadas

La mayoría de los datos utilizados por el tiempo de ejecución de ObjectiveC cambiarán durante la ejecución, por lo tanto utiliza algunas secciones del segmento **\_\_DATA** en la memoria:

- **`__objc_msgrefs`** (`message_ref_t`): Referencias de mensajes
- **`__objc_ivar`** (`ivar`): Variables de instancia
- **`__objc_data`** (`...`): Datos mutables
- **`__objc_classrefs`** (`Class`): Referencias de clases
- **`__objc_superrefs`** (`Class`): Referencias de superclases
- **`__objc_protorefs`** (`protocol_t *`): Referencias de protocolos
- **`__objc_selrefs`** (`SEL`): Referencias de selectores
- **`__objc_const`** (`...`): Datos de clase `r/o` y otros datos (con suerte) constantes
- **`__objc_imageinfo`** (`versión, flags`): Utilizado durante la carga de imagen: Versión actualmente `0`; Las banderas especifican soporte preoptimizado de GC, etc.
- **`__objc_protolist`** (`protocol_t *`): Lista de protocolos
- **`__objc_nlcatlist`** (`category_t`): Puntero a Categorías No Perezosas definidas en este binario
- **`__objc_catlist`** (`category_t`): Puntero a Categorías definidas en este binario
- **`__objc_nlclslist`** (`classref_t`): Puntero a clases Objective-C No Perezosas definidas en este binario
- **`__objc_classlist`** (`classref_t`): Punteros a todas las clases Objective-C definidas en este binario

También utiliza algunas secciones en el segmento **`__TEXT`** para almacenar valores constantes si no es posible escribir en esta sección:

- **`__objc_methname`** (Cadena-C): Nombres de métodos
- **`__objc_classname`** (Cadena-C): Nombres de clases
- **`__objc_methtype`** (Cadena-C): Tipos de métodos

### Codificación de tipos

Objective-C utiliza un cierto enmascaramiento para codificar los tipos de selector y variables de tipos simples y complejos:

- Los tipos primitivos usan la primera letra del tipo `i` para `int`, `c` para `char`, `l` para `long`... y usa la letra mayúscula en caso de que sea sin signo (`L` para `unsigned Long`).
- Otros tipos de datos cuyas letras se utilizan o son especiales, usan otras letras o símbolos como `q` para `long long`, `b` para `bitfields`, `B` para `booleans`, `#` para `classes`, `@` para `id`, `*` para `char pointers`, `^` para `punteros` genéricos y `?` para `indefinido`.
- Los arreglos, estructuras y uniones usan `[`, `{` y `(`

#### Ejemplo de Declaración de Método

{% code overflow="wrap" %}
```objectivec
- (NSString *)processString:(id)input withOptions:(char *)options andError:(id)error;
```
{% endcode %}

El selector sería `processString:withOptions:andError:`

#### Codificación de Tipos

* `id` se codifica como `@`
* `char *` se codifica como `*`

La codificación completa de tipos para el método es:
```less
@24@0:8@16*20^@24
```
#### Desglose Detallado

1. **Tipo de Retorno (`NSString *`)**: Codificado como `@` con longitud 24
2. **`self` (instancia del objeto)**: Codificado como `@`, en el desplazamiento 0
3. **`_cmd` (selector)**: Codificado como `:`, en el desplazamiento 8
4. **Primer argumento (`char * input`)**: Codificado como `*`, en el desplazamiento 16
5. **Segundo argumento (`NSDictionary * options`)**: Codificado como `@`, en el desplazamiento 20
6. **Tercer argumento (`NSError ** error`)**: Codificado como `^@`, en el desplazamiento 24

**Con el selector + la codificación puedes reconstruir el método.**

### **Clases**

Las clases en Objective-C son una estructura con propiedades, punteros a métodos... Es posible encontrar la estructura `objc_class` en el [**código fuente**](https://opensource.apple.com/source/objc4/objc4-756.2/runtime/objc-runtime-new.h.auto.html):
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
Esta clase utiliza algunos bits del campo isa para indicar información sobre la clase.

Luego, la estructura tiene un puntero a la estructura `class_ro_t` almacenada en disco que contiene atributos de la clase como su nombre, métodos base, propiedades y variables de instancia.\
Durante el tiempo de ejecución, se utiliza una estructura adicional `class_rw_t` que contiene punteros que pueden ser modificados, como métodos, protocolos, propiedades...
