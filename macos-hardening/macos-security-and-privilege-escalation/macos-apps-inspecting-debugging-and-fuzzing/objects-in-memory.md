# メモリ内のオブジェクト

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}

## CFRuntimeClass

CF\*オブジェクトはCoreFoundationから来ており、`CFString`、`CFNumber`、または`CFAllocator`のような50以上のクラスのオブジェクトを提供します。

これらのすべてのクラスは`CFRuntimeClass`のインスタンスであり、呼び出されると`__CFRuntimeClassTable`へのインデックスを返します。CFRuntimeClassは[**CFRuntime.h**](https://opensource.apple.com/source/CF/CF-1153.18/CFRuntime.h.auto.html)で定義されています：
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

### メモリセクションの使用

ObjectiveCランタイムによって使用されるデータのほとんどは、実行中に変更されるため、メモリ内の**\_\_DATA**セグメントからいくつかのセクションを使用します：

* **`__objc_msgrefs`** (`message_ref_t`): メッセージ参照
* **`__objc_ivar`** (`ivar`): インスタンス変数
* **`__objc_data`** (`...`): 可変データ
* **`__objc_classrefs`** (`Class`): クラス参照
* **`__objc_superrefs`** (`Class`): スーパークラス参照
* **`__objc_protorefs`** (`protocol_t *`): プロトコル参照
* **`__objc_selrefs`** (`SEL`): セレクタ参照
* **`__objc_const`** (`...`): クラスの`r/o`データおよびその他の（おそらく）定数データ
* **`__objc_imageinfo`** (`version, flags`): 画像のロード中に使用されます：現在のバージョンは`0`；フラグは最適化されたGCサポートなどを指定します。
* **`__objc_protolist`** (`protocol_t *`): プロトコルリスト
* **`__objc_nlcatlist`** (`category_t`): このバイナリで定義された非遅延カテゴリへのポインタ
* **`__objc_catlist`** (`category_t`): このバイナリで定義されたカテゴリへのポインタ
* **`__objc_nlclslist`** (`classref_t`): このバイナリで定義された非遅延Objective-Cクラスへのポインタ
* **`__objc_classlist`** (`classref_t`): このバイナリで定義されたすべてのObjective-Cクラスへのポインタ

また、書き込みが不可能な場合に定数値を格納するために、**`__TEXT`**セグメント内のいくつかのセクションも使用します：

* **`__objc_methname`** (C-String): メソッド名
* **`__objc_classname`** (C-String): クラス名
* **`__objc_methtype`** (C-String): メソッドタイプ

### タイプエンコーディング

Objective-Cは、セレクタと変数の型を単純および複雑な型のエンコードにいくつかのマングリングを使用します：

* プリミティブ型は、型の最初の文字を使用します。`i`は`int`、`c`は`char`、`l`は`long`... そして符号なしの場合は大文字を使用します（`L`は`unsigned Long`）。
* 他のデータ型は、使用される文字や特別な文字に対して、`q`は`long long`、`b`は`bitfields`、`B`は`booleans`、`#`は`classes`、`@`は`id`、`*`は`char pointers`、`^`は一般的な`pointers`、`?`は`undefined`を使用します。
* 配列、構造体、共用体は`[`、`{`、`(`を使用します。

#### 例 メソッド宣言

{% code overflow="wrap" %}
```objectivec
- (NSString *)processString:(id)input withOptions:(char *)options andError:(id)error;
```
{% endcode %}

セレクタは `processString:withOptions:andError:` になります。

#### 型エンコーディング

* `id` は `@` としてエンコードされます
* `char *` は `*` としてエンコードされます

メソッドの完全な型エンコーディングは：
```less
@24@0:8@16*20^@24
```
#### 詳細な内訳

1. **戻り値の型 (`NSString *`)**: `@` でエンコードされ、長さは 24
2. **`self` (オブジェクトインスタンス)**: `@` でエンコードされ、オフセット 0
3. **`_cmd` (セレクタ)**: `:` でエンコードされ、オフセット 8
4. **最初の引数 (`char * input`)**: `*` でエンコードされ、オフセット 16
5. **2 番目の引数 (`NSDictionary * options`)**: `@` でエンコードされ、オフセット 20
6. **3 番目の引数 (`NSError ** error`)**: `^@` でエンコードされ、オフセット 24

**セレクタとエンコーディングを使ってメソッドを再構築できます。**

### **クラス**

Objective-C のクラスは、プロパティ、メソッドポインタを持つ構造体です... 構造体 `objc_class` を [**ソースコード**](https://opensource.apple.com/source/objc4/objc4-756.2/runtime/objc-runtime-new.h.auto.html) で見つけることができます:
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
このクラスは、クラスに関する情報を示すためにisaフィールドの一部のビットを使用します。

次に、構造体はディスクに保存された`class_ro_t`構造体へのポインタを持ち、クラスの名前、基本メソッド、プロパティ、およびインスタンス変数などの属性を含みます。\
ランタイム中に、メソッド、プロトコル、プロパティなどのポインタを含む追加の構造体`class_rw_t`が使用され、これらは変更可能です。



{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください!
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}
