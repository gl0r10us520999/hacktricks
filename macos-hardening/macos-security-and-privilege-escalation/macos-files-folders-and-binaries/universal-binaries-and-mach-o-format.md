# macOS यूनिवर्सल बाइनरीज और Mach-O प्रारूप

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## मूल जानकारी

Mac OS बाइनरीज आम तौर पर **यूनिवर्सल बाइनरीज** के रूप में कॉम्पाइल की जाती हैं। एक **यूनिवर्सल बाइनरी** में **एक ही फ़ाइल में कई विश्वकर्माओं का समर्थन** कर सकती है।

ये बाइनरीज **Mach-O संरचना** का पालन करती हैं जो मुख्य रूप से निम्नलिखित से बना होता है:

* हेडर
* लोड कमांड्स
* डेटा

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (470).png>)

## फैट हेडर

फ़ाइल को खोजें: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* number of structs that follow */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* cpu specifier (int) */
cpu_subtype_t	cpusubtype;	/* machine specifier (int) */
uint32_t	offset;		/* file offset to this object file */
uint32_t	size;		/* size of this object file */
uint32_t	align;		/* alignment as a power of 2 */
};
</code></pre>

हेडर में **मैजिक** बाइट्स होते हैं जिनके बाद फ़ाइल में मौजूद **विश्वकर्माओं** की संख्या (`nfat_arch`) होती है और प्रत्येक विश्वकर्म के पास `fat_arch` संरचना होती है।

इसे इस प्रकार देखें:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/bin/ls (for architecture x86_64):	Mach-O 64-bit executable x86_64
/bin/ls (for architecture arm64e):	Mach-O 64-bit executable arm64e

% otool -f -v /bin/ls
Fat headers
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>architecture x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capabilities 0x0
<strong>    offset 16384
</strong><strong>    size 72896
</strong>    align 2^14 (16384)
<strong>architecture arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capabilities PTR_AUTH_VERSION USERSPACE 0
<strong>    offset 98304
</strong><strong>    size 88816
</strong>    align 2^14 (16384)
</code></pre>

या [Mach-O View](https://sourceforge.net/projects/machoview/) उपकरण का उपयोग करके:

<figure><img src="../../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

जैसा कि आप सोच रहे होंगे, आम तौर पर 2 विश्वकर्मों के लिए कॉम्पाइल किया गया यूनिवर्सल बाइनरी एक विश्वकर्म के लिए कॉम्पाइल किए जाने वाले बाइनरी के **आकार को दोगुना** कर देता है।

## **Mach-O हेडर**

हेडर में फ़ाइल के बारे में मूल जानकारी होती है, जैसे जादू बाइट्स जिससे इसे एक Mach-O फ़ाइल के रूप में पहचाना जा सकता है और लक्षित विश्वकर्म के बारे में जानकारी। आप इसे यहाँ पा सकते हैं: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
```c
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */
struct mach_header {
uint32_t	magic;		/* mach magic number identifier */
cpu_type_t	cputype;	/* cpu specifier (e.g. I386) */
cpu_subtype_t	cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file (usage and alignment for the file) */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
};

#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
struct mach_header_64 {
uint32_t	magic;		/* mach magic number identifier */
int32_t		cputype;	/* cpu specifier */
int32_t		cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
uint32_t	reserved;	/* reserved */
};
```
### Mach-O फ़ाइल प्रकार

विभिन्न फ़ाइल प्रकार हैं, आप उन्हें [**उदाहरण के लिए यहाँ पर**](https://opensource.apple.com/source/xnu/xnu-2050.18.24/EXTERNAL_HEADERS/mach-o/loader.h) परिभाषित पा सकते हैं। सबसे महत्वपूर्ण वाले इस प्रकार हैं:

* `MH_OBJECT`: स्थानांतरणीय ऑब्जेक्ट फ़ाइल (कॉम्पाइलेशन के बीच के उत्पाद, अभी तक क्रियाशील नहीं)।
* `MH_EXECUTE`: क्रियाशील फ़ाइलें।
* `MH_FVMLIB`: स्थिर VM पुस्तकालय फ़ाइल।
* `MH_CORE`: कोड डंप्स
* `MH_PRELOAD`: पूर्व-लोड क्रियाशील फ़ाइल (अब XNU में समर्थित नहीं)
* `MH_DYLIB`: डायनामिक पुस्तकालयें
* `MH_DYLINKER`: डायनामिक लिंकर
* `MH_BUNDLE`: "प्लगइन फ़ाइलें"। -bundle का उपयोग करके जनरेट किया जाता है और `NSBundle` या `dlopen` द्वारा स्पष्ट रूप से लोड किया जाता है।
* `MH_DYSM`: सहायक `.dSym` फ़ाइल (डीबगिंग के लिए प्रतीक वाली फ़ाइल)।
* `MH_KEXT_BUNDLE`: कर्नेल एक्सटेंशन्स।
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
या [Mach-O View](https://sourceforge.net/projects/machoview/) का उपयोग करें:

<figure><img src="../../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

## **Mach-O ध्वज**

स्रोत कोड भी कई ध्वजों को परिभाषित करता है जो पुस्तकालयों को लोड करने के लिए उपयोगी हैं:

* `MH_NOUNDEFS`: कोई अपरिभाषित संदर्भ नहीं (पूरी तरह से लिंक किया गया)
* `MH_DYLDLINK`: Dyld लिंकिंग
* `MH_PREBOUND`: डायनामिक संदर्भ पूर्वबंधित।
* `MH_SPLIT_SEGS`: फ़ाइल आर / ओ और आर / डब्ल्यू सेगमेंट को विभाजित करती है।
* `MH_WEAK_DEFINES`: बाइनरी में कमजोर परिभाषित प्रतीक हैं
* `MH_BINDS_TO_WEAK`: बाइनरी कमजोर प्रतीक का उपयोग करती है
* `MH_ALLOW_STACK_EXECUTION`: स्टैक को क्रियाशील बनाएं
* `MH_NO_REEXPORTED_DYLIBS`: पुस्तकालय LC\_REEXPORT कमांड नहीं
* `MH_PIE`: स्थान स्वतंत्र कार्यक्षम
* `MH_HAS_TLV_DESCRIPTORS`: धागे से धागे स्थानीय चर के साथ एक खंड है
* `MH_NO_HEAP_EXECUTION`: हीप / डेटा पृष्ठों के लिए कोई क्रियान्वयन नहीं
* `MH_HAS_OBJC`: बाइनरी में ओब्जेक्ट-सी खंड हैं
* `MH_SIM_SUPPORT`: सिम्युलेटर समर्थन
* `MH_DYLIB_IN_CACHE`: साझा पुस्तकालय कैश में डायलिब्स / फ़्रेमवर्क पर उपयोग किया जाता है।

## **Mach-O लोड कमांड्स**

**मेमोरी में फ़ाइल का खाका** यहाँ निर्दिष्ट किया गया है, मुख्य धागे की स्थान, क्रियान्वयन प्रारंभ पर मुख्य धागे का संदर्भ, और आवश्यक **साझा पुस्तकालय**। निर्देश दिए गए हैं डायनामिक लोडर **(dyld)** को मेमोरी में बाइनरी के लोडिंग प्रक्रिया पर।

उपयोग करता है **load\_command** संरचना, उल्लिखित **`loader.h`** में परिभाषित:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
व्यवस्था को विभिन्न प्रकार के लोड कमांड का सामना करना है। सबसे सामान्य लोड कमांड हैं: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, और `LC_CODE_SIGNATURE`।

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
मुख्य रूप से, यह प्रकार का लोड कमांड **कैसे \_\_TEXT** (कार्यात्मक कोड) **और \_\_DATA** (प्रक्रिया के लिए डेटा) **सेगमेंट** को **डेटा खंड में दिखाए गए ऑफसेट के अनुसार** बाइनरी को निष्पादित किया जाता है।
{% endhint %}

ये कमांड **सेगमेंट को परिभाषित** करते हैं जो एक प्रक्रिया के **वर्चुअल मेमोरी स्पेस** में **मैप** किए जाते हैं जब यह निष्पादित होता है।

इनमें **विभिन्न प्रकार** के सेगमेंट होते हैं, जैसे **\_\_TEXT** सेगमेंट, जो किसी प्रोग्राम के कार्यात्मक कोड को धारित करता है, और **\_\_DATA** सेगमेंट, जो प्रक्रिया द्वारा उपयोग किए जाने वाले डेटा को समेतता है। ये **सेगमेंट डेटा सेक्शन** में स्थित होते हैं मैक-ओ फ़ाइल में।

**प्रत्येक सेगमेंट** को और अधिक सेक्शनों में **विभाजित** किया जा सकता है। **लोड कमांड संरचना** में **जानकारी** होती है **इन सेगमेंट के भीतर के सेक्शनों** के बारे में।

हेडर में पहले आपको **सेगमेंट हेडर** मिलता है:

<pre class="language-c"><code class="lang-c">struct segment_command_64 { /* for 64-bit architectures */
uint32_t	cmd;		/* LC_SEGMENT_64 */
uint32_t	cmdsize;	/* includes sizeof section_64 structs */
char		segname[16];	/* segment name */
uint64_t	vmaddr;		/* memory address of this segment */
uint64_t	vmsize;		/* memory size of this segment */
uint64_t	fileoff;	/* file offset of this segment */
uint64_t	filesize;	/* amount to map from the file */
int32_t		maxprot;	/* maximum VM protection */
int32_t		initprot;	/* initial VM protection */
<strong>	uint32_t	nsects;		/* number of sections in segment */
</strong>	uint32_t	flags;		/* flags */
};
</code></pre>

सेगमेंट हेडर का उदाहरण:

<figure><img src="../../../.gitbook/assets/image (1126).png" alt=""><figcaption></figcaption></figure>

यह हेडर **उन सेक्शनों की संख्या को परिभाषित करता है जिनके हेडर इसके बाद दिखाई देते हैं**:
```c
struct section_64 { /* for 64-bit architectures */
char		sectname[16];	/* name of this section */
char		segname[16];	/* segment this section goes in */
uint64_t	addr;		/* memory address of this section */
uint64_t	size;		/* size in bytes of this section */
uint32_t	offset;		/* file offset of this section */
uint32_t	align;		/* section alignment (power of 2) */
uint32_t	reloff;		/* file offset of relocation entries */
uint32_t	nreloc;		/* number of relocation entries */
uint32_t	flags;		/* flags (section type and attributes)*/
uint32_t	reserved1;	/* reserved (for offset or index) */
uint32_t	reserved2;	/* reserved (for count or sizeof) */
uint32_t	reserved3;	/* reserved */
};
```
उदाहरण का **खंड शीर्षक**:

<figure><img src="../../../.gitbook/assets/image (1108).png" alt=""><figcaption></figcaption></figure>

यदि आप **खंड ऑफसेट** (0x37DC) + **ऑफसेट** जहां **आर्क शुरू होता है**, इस मामले में `0x18000` जोड़ें --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

यह भी संभव है कि **हेडर्स जानकारी** को **कमांड लाइन** से प्राप्त किया जा सके:
```bash
otool -lv /bin/ls
```
इस cmd द्वारा लोड किए जाने वाले सामान्य सेगमेंट:

* **`__PAGEZERO`:** यह कर्नेल को **पता देता है** कि **एड्रेस जीरो** को **पढ़ा नहीं जा सकता, लिखा नहीं जा सकता, या नहीं चलाया जा सकता**। संरचना में maxprot और minprot चर मान शून्य पर सेट किए जाते हैं जिससे इस पृष्ठ पर **कोई पढ़ने-लिखने-चलाने का अधिकार नहीं हो**।
* यह आवंटन NULL प्वाइंटर डिफरेंस वल्नरेबिलिटी को **कम करने** के लिए महत्वपूर्ण है। यह इसलिए है क्योंकि XNU एक हार्ड पेज जीरो को प्रयोग करता है जो सुनिश्चित करता है कि मेमोरी का पहला पेज (केवल पहला) अप्राप्य है (केवल i386 में)। एक बाइनरी इस आवश्यकता को पूरा कर सकता है एक छोटा \_\_PAGEZERO (का उपयोग करके `-pagezero_size`) बनाकर पहले 4k को कवर करना और बाकी 32बिट मेमोरी को उपयोगकर्ता और कर्नेल मोड में दोनों के लिए पहुंचने योग्य बनाने के लिए।
* **`__TEXT`**: **पाठ्यक्रियात्मक** **कोड** को **पढ़ने** और **चलाने** की अनुमति है (कोई लेखन नहीं)**।** इस सेगमेंट के सामान्य खंड:
* `__text`: कंपाइल किया गया बाइनरी कोड
* `__const`: स्थिर डेटा (केवल पढ़ने के लिए)
* `__[c/u/os_log]string`: C, यूनिकोड या os लॉग स्ट्रिंग स्थिर
* `__stubs` और `__stubs_helper`: डायनामिक लाइब्रेरी लोडिंग प्रक्रिया के दौरान संलग्न
* `__unwind_info`: स्टैक अनवाइंड डेटा।
* ध्यान दें कि यह सभी सामग्री साइन की गई है लेकिन इसे भी चलाया जा सकता है (जो इस विशेषाधिकार की आवश्यकता नहीं है, जैसे स्ट्रिंग समर्पित खंडों के शोषण के विकल्पों के लिए और अधिक विकसित करने के लिए)।
* **`__DATA`**: डेटा को शामिल करता है जो **पढ़ने योग्य** और **लिखने योग्य** है (कोई चलाया नहीं)**।**
* `__got:` ग्लोबल ऑफसेट टेबल
* `__nl_symbol_ptr`: नॉन लेजी (लोड पर बाइंड) सिम्बल पॉइंटर
* `__la_symbol_ptr`: लेजी (उपयोग पर बाइंड) सिम्बल पॉइंटर
* `__const`: केवल पढ़ने योग्य डेटा (वास्तव में नहीं)
* `__cfstring`: कोर फाउंडेशन स्ट्रिंग्स
* `__data`: ग्लोबल वेरिएबल्स (जिन्होंने प्रारंभ किया गया है)
* `__bss`: स्टेटिक वेरिएबल्स (जिन्होंने प्रारंभ नहीं किया गया है)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, आदि): ऑब्जेक्टिव-सी रनटाइम द्वारा उपयोग किया जाने वाली जानकारी
* **`__DATA_CONST`**: \_\_DATA.\_\_const को स्थिर माना नहीं जाता है (लेखन अनुमतियाँ), न ही अन्य पॉइंटर्स और गॉट। इस खंड ने `mprotect` का उपयोग करके `__const`, कुछ प्रारंभकर्ताओं और गॉट टेबल (एक बार सुलझाया गया) को **केवल पढ़ने योग्य** बनाया है।
* **`__LINKEDIT`**: लिंकर (डायल्ड) के लिए जानकारी शामिल करता है जैसे, सिम्बल, स्ट्रिंग, और स्थानांतरण तालिका प्रविष्टियाँ। यह `__TEXT` या `__DATA` में नहीं है और इसकी सामग्री को अन्य लोड कमांड्स में वर्णित किया गया है।
* डायल्ड जानकारी: Rebase, Non-lazy/lazy/weak बाइंडिंग ऑपकोड और निर्यात जानकारी
* फ़ंक्शन शुरू होते हैं: फ़ंक्शन के प्रारंभ पतों की शुरुआत की तालिका
* कोड में डेटा: \_\_text में डेटा आइलैंड्स
* सिम्बल टेबल: बाइनरी में सिम्बल्स
* इंडायरेक्ट सिम्बल टेबल: पॉइंटर/स्टब सिम्बल्स
* स्ट्रिंग टेबल
* कोड हस्ताक्षर
* **`__OBJC`**: ऑब्जेक्टिव-सी रनटाइम द्वारा उपयोग की जाने वाली जानकारी शामिल करता है। हालांकि यह जानकारी \_\_DATA सेगमेंट में भी पाई जा सकती है, विभिन्न \_\_objc\_\* खंडों में।
* **`__RESTRICT`**: एक सेगमेंट जिसमें सामग्री नहीं है जिसे **`__restrict`** कहा जाता है (भी खाली) जो सुनिश्चित करता है कि बाइनरी को चलाते समय, यह DYLD पर्यावरणीय चरों को नजरअंदाज करेगा।

जैसा कि कोड में देखा जा सका, **सेगमेंट भी झंडे समर्थन करते हैं** (हालांकि वे बहुत अधिक उपयोग नहीं होते):

* `SG_HIGHVM`: केवल कोर (उपयोग नहीं होता)
* `SG_FVMLIB`: उपयोग नहीं होता
* `SG_NORELOC`: सेगमेंट में कोई स्थानांतरण नहीं है
* `SG_PROTECTED_VERSION_1`: एन्क्रिप्शन। उदाहरण के लिए फाइंडर द्वारा पाठ को एन्क्रिप्ट करने के लिए `__TEXT` सेगमेंट का उपयोग किया जाता है।

### **`LC_UNIXTHREAD/LC_MAIN`**

**`LC_MAIN`** में **entryoff विशेषता में प्रवेश बिंदु** होता है। लोड समय पर, **dyld** बस के (मेमोरी में) **बाइनरी के आधार** में इस मान को **जोड़ता** है, फिर इस निर्देशिका पर जाकर बाइनरी के कोड का निष्पादन शुरू करने के लिए **उछलता** है।

**`LC_UNIXTHREAD`** में रजिस्टर के मान शामिल होते हैं जब मुख्य धागा शुरू होता है। यह पहले से ही प्रायोगिक हो गया था लेकिन **`dyld`** अब भी इसका उपयोग करता है। इसके साथ इस द्वारा सेट किए गए रजिस्टरों के मान देखना संभव है:
```bash
otool -l /usr/lib/dyld
[...]
Load command 13
cmd LC_UNIXTHREAD
cmdsize 288
flavor ARM_THREAD_STATE64
count ARM_THREAD_STATE64_COUNT
x0  0x0000000000000000 x1  0x0000000000000000 x2  0x0000000000000000
x3  0x0000000000000000 x4  0x0000000000000000 x5  0x0000000000000000
x6  0x0000000000000000 x7  0x0000000000000000 x8  0x0000000000000000
x9  0x0000000000000000 x10 0x0000000000000000 x11 0x0000000000000000
x12 0x0000000000000000 x13 0x0000000000000000 x14 0x0000000000000000
x15 0x0000000000000000 x16 0x0000000000000000 x17 0x0000000000000000
x18 0x0000000000000000 x19 0x0000000000000000 x20 0x0000000000000000
x21 0x0000000000000000 x22 0x0000000000000000 x23 0x0000000000000000
x24 0x0000000000000000 x25 0x0000000000000000 x26 0x0000000000000000
x27 0x0000000000000000 x28 0x0000000000000000  fp 0x0000000000000000
lr 0x0000000000000000 sp  0x0000000000000000  pc 0x0000000000004b70
cpsr 0x00000000

[...]
```
### **`LC_CODE_SIGNATURE`**

में **Macho-O फ़ाइल के कोड हस्ताक्षर** के बारे में जानकारी होती है। यह केवल एक **ओफ़्सेट** शामिल करता है जो **हस्ताक्षर ब्लॉब** की **ओर पहुंचता है**। यह आम तौर पर फ़ाइल के अंत में होता है।\
हालांकि, आप इस खंड के बारे में कुछ जानकारी [**इस ब्लॉग पोस्ट**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) और इस [**गिस्ट**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4) में पा सकते हैं।

### **`LC_ENCRYPTION_INFO[_64]`**

बाइनरी एन्क्रिप्शन का समर्थन। हालांकि, यदि कोई हमलावर प्रक्रिया को कंप्रमाइज करने में सफल होता है, तो वह यादृच्छिक रूप से मेमोरी को अनएन्क्रिप्ट कर सकेगा।

### **`LC_LOAD_DYLINKER`**

**डायनामिक लिंकर एक्जीक्यूटेबल का पथ** शामिल होता है जो साझा लाइब्रेरी को प्रक्रिया पता स्थान में मैप करता है। **मान हमेशा `/usr/lib/dyld` पर सेट किया जाता है**। यह महत्वपूर्ण है कि macOS में dylib मैपिंग **उपयोगकर्ता मोड** में होती है, कर्नेल मोड में नहीं।

### **`LC_IDENT`**

पुराना है लेकिन जब पैनिक पर डंप जेनरेट करने के लिए कॉन्फ़िगर किया जाता है, तो एक Mach-O कोर डंप बनाया जाता है और कर्नेल संस्करण `LC_IDENT` कमांड में सेट किया जाता है।

### **`LC_UUID`**

रैंडम UUID। यह किसी काम के लिए सीधे उपयोगी नहीं है लेकिन XNU इसे प्रक्रिया जानकारी के बाकी साथ कैश करता है। यह क्रैश रिपोर्ट में उपयोग किया जा सकता है।

### **`LC_DYLD_ENVIRONMENT`**

प्रक्रिया को निष्पादित होने से पहले dyld को वातावरण चरों को सूचित करने की अनुमति देता है। यह बहुत ही खतरनाक हो सकता है क्योंकि यह प्रक्रिया के अंदर विचारहीन कोड निष्पादित करने की अनुमति दे सकता है, इसलिए यह लोड कमांड केवल `#define SUPPORT_LC_DYLD_ENVIRONMENT` के साथ डायल्ड बिल्ड में उपयोग किया जाता है और आगे संविदान केवल `DYLD_..._PATH` रूप के चरों को निर्दिष्ट करने के लिए प्रसंस्करण को प्रतिबंधित करता है।

### **`LC_LOAD_DYLIB`**

यह लोड कमांड एक **डायनामिक लाइब्रेरी** आवश्यकता का वर्णन करता है जो **लोडर** (dyld) को **उस लाइब्रेरी को लोड और लिंक करने के लिए निर्देशित** करता है। Mach-O बाइनरी के प्रत्येक लाइब्रेरी के लिए एक `LC_LOAD_DYLIB` लोड कमांड होता है।

* यह लोड कमांड एक प्रकार की संरचना है **`dylib_command`** (जिसमें वास्तविक निर्भर डायनामिक लाइब्रेरी का वर्णन करने वाला स्ट्रक्ट dylib होता है):
```objectivec
struct dylib_command {
uint32_t        cmd;            /* LC_LOAD_{,WEAK_}DYLIB */
uint32_t        cmdsize;        /* includes pathname string */
struct dylib    dylib;          /* the library identification */
};

struct dylib {
union lc_str  name;                 /* library's path name */
uint32_t timestamp;                 /* library's build time stamp */
uint32_t current_version;           /* library's current version number */
uint32_t compatibility_version;     /* library's compatibility vers number*/
};
```
![](<../../../.gitbook/assets/image (486).png>)

आप इस जानकारी को क्लाइंट लाइन से भी प्राप्त कर सकते हैं:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
कुछ संभावित मैलवेयर संबंधित लाइब्रेरी हैं:

* **DiskArbitration**: USB ड्राइव की मॉनिटरिंग
* **AVFoundation:** ऑडियो और वीडियो कैप्चर
* **CoreWLAN**: वाईफाई स्कैन।

{% hint style="info" %}
एक Mach-O बाइनरी में एक या **एक से अधिक निर्माताओं** को शामिल किया जा सकता है, जो **LC\_MAIN** में निर्दिष्ट पते से **पहले** **क्रियान्वित** होंगे।\
किसी भी निर्माताओं के ऑफसेट **\_\_DATA\_CONST** सेगमेंट के **\_\_mod\_init\_func** खंड में रखे जाते हैं।
{% endhint %}

## **Mach-O डेटा**

फ़ाइल के मूल में डेटा क्षेत्र है, जो लोड-कमांड्स क्षेत्र में परिभाषित कई सेगमेंटों से बना होता है। **प्रत्येक सेगमेंट में कई डेटा खंड हो सकते हैं**, हर खंड **कोड या डेटा** विशेष एक प्रकार के के लिए होता है।

{% hint style="success" %}
डेटा मुख्य रूप से उस सभी **जानकारी** को समेत करता है जो लोड कमांड्स **LC\_SEGMENTS\_64** द्वारा लोड की जाती है।
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

इसमें शामिल हैं:

* **कार्य सारणी:** जो कार्य के बारे में जानकारी रखती है।
* **प्रतीक सारणी**: जो बाइनरी द्वारा उपयोग की गई बाह्य कार्य के बारे में जानकारी रखती है
* यह आंतरिक कार्य, चर नामों को भी शामिल कर सकता है और अधिक।

इसे जांचने के लिए आप [**Mach-O View**](https://sourceforge.net/projects/machoview/) टूल का उपयोग कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

या CLI से:
```bash
size -m /bin/ls
```
## Objective-C सामान्य खंड

`__TEXT` सेगमेंट (r-x) में:

* `__objc_classname`: कक्षा के नाम (स्ट्रिंग्स)
* `__objc_methname`: मेथड के नाम (स्ट्रिंग्स)
* `__objc_methtype`: मेथड के प्रकार (स्ट्रिंग्स)

`__DATA` सेगमेंट (rw-) में:

* `__objc_classlist`: सभी Objective-C कक्षाओं के पॉइंटर
* `__objc_nlclslist`: Non-Lazy Objective-C कक्षाओं के पॉइंटर
* `__objc_catlist`: श्रेणियों के लिए पॉइंटर
* `__objc_nlcatlist`: Non-Lazy श्रेणियों के लिए पॉइंटर
* `__objc_protolist`: प्रोटोकॉल सूची
* `__objc_const`: स्थिर डेटा
* `__objc_imageinfo`, `__objc_selrefs`, `objc__protorefs`...

## Swift

* `_swift_typeref`, `_swift3_capture`, `_swift3_assocty`, `_swift3_types, _swift3_proto`, `_swift3_fieldmd`, `_swift3_builtin`, `_swift3_reflstr`
