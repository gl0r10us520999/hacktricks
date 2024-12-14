# macOS Universal binaries & Mach-O Format

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

## Basic Information

Mac OS binaries kwa kawaida zinakusanywa kama **universal binaries**. **Universal binary** inaweza **kuunga mkono usanifu mwingi katika faili moja**.

Binaries hizi zinafuata **Mach-O structure** ambayo kimsingi inajumuisha:

* Header
* Load Commands
* Data

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (470).png>)

## Fat Header

Tafuta faili kwa: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC au FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* idadi ya structs zinazofuata */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* mwelekeo wa cpu (int) */
cpu_subtype_t	cpusubtype;	/* mwelekeo wa mashine (int) */
uint32_t	offset;		/* ofisi ya faili kwa faili hii ya kitu */
uint32_t	size;		/* ukubwa wa faili hii ya kitu */
uint32_t	align;		/* usawa kama nguvu ya 2 */
};
</code></pre>

Header ina **magic** bytes ikifuatiwa na **idadi** ya **archs** faili **inayo** (`nfat_arch`) na kila arch itakuwa na `fat_arch` struct.

Angalia kwa:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/bin/ls (kwa usanifu x86_64):	Mach-O 64-bit executable x86_64
/bin/ls (kwa usanifu arm64e):	Mach-O 64-bit executable arm64e

% otool -f -v /bin/ls
Fat headers
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>usanifu x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capabilities 0x0
<strong>    offset 16384
</strong><strong>    size 72896
</strong>    align 2^14 (16384)
<strong>usanifu arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capabilities PTR_AUTH_VERSION USERSPACE 0
<strong>    offset 98304
</strong><strong>    size 88816
</strong>    align 2^14 (16384)
</code></pre>

au kwa kutumia zana ya [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

Kama unavyofikiria kawaida universal binary iliyokusanywa kwa usanifu 2 **inaongeza ukubwa** wa moja iliyokusanywa kwa usanifu 1 tu.

## **Mach-O Header**

Header ina taarifa za msingi kuhusu faili, kama vile magic bytes kutambua kama faili la Mach-O na taarifa kuhusu usanifu wa lengo. Unaweza kuipata katika: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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
### Mach-O File Types

Kuna aina tofauti za faili, unaweza kuziona zikifafanuliwa katika [**kanuni ya chanzo kwa mfano hapa**](https://opensource.apple.com/source/xnu/xnu-2050.18.24/EXTERNAL\_HEADERS/mach-o/loader.h). Zile muhimu zaidi ni:

* `MH_OBJECT`: Faili ya kitu inayoweza kuhamishwa (bidhaa za kati za uundaji, bado si executable).
* `MH_EXECUTE`: Faili zinazoweza kutekelezwa.
* `MH_FVMLIB`: Faili ya maktaba ya VM iliyowekwa.
* `MH_CORE`: Mifumo ya Kanuni
* `MH_PRELOAD`: Faili ya executable iliyopakiwa mapema (haipati tena msaada katika XNU)
* `MH_DYLIB`: Maktaba za Kijadi
* `MH_DYLINKER`: Mshikamano wa Kijadi
* `MH_BUNDLE`: "Faili za Plugin". Zilizozalishwa kwa kutumia -bundle katika gcc na kupakiwa wazi na `NSBundle` au `dlopen`.
* `MH_DYSM`: Faili ya mshirika `.dSym` (faili yenye alama za kutatua matatizo).
* `MH_KEXT_BUNDLE`: Nyongeza za Kernel.
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
Or using [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

## **Mach-O Flags**

Msimbo wa chanzo pia unafafanua bendera kadhaa zinazofaa kwa ajili ya kupakia maktaba:

* `MH_NOUNDEFS`: Hakuna marejeleo yasiyofafanuliwa (imeunganishwa kikamilifu)
* `MH_DYLDLINK`: Kuunganisha Dyld
* `MH_PREBOUND`: Marejeleo ya dinamik yanayofungwa mapema.
* `MH_SPLIT_SEGS`: Faili inagawanya sehemu r/o na r/w.
* `MH_WEAK_DEFINES`: Binary ina alama zisizofafanuliwa dhaifu
* `MH_BINDS_TO_WEAK`: Binary inatumia alama dhaifu
* `MH_ALLOW_STACK_EXECUTION`: Fanya stack iweze kutekelezwa
* `MH_NO_REEXPORTED_DYLIBS`: Maktaba haina amri za LC\_REEXPORT
* `MH_PIE`: Mtendaji Huru wa Nafasi
* `MH_HAS_TLV_DESCRIPTORS`: Kuna sehemu yenye mabadiliko ya nyuzi za ndani
* `MH_NO_HEAP_EXECUTION`: Hakuna utekelezaji kwa kurasa za heap/data
* `MH_HAS_OBJC`: Binary ina sehemu za oBject-C
* `MH_SIM_SUPPORT`: Msaada wa simulator
* `MH_DYLIB_IN_CACHE`: Inatumika kwenye dylibs/frameworks katika cache ya maktaba ya pamoja.

## **Mach-O Load commands**

Muundo wa **faili katika kumbukumbu** umefafanuliwa hapa, ukielezea **mahali pa jedwali la alama**, muktadha wa nyuzi kuu wakati wa kuanza utekelezaji, na **maktaba za pamoja** zinazohitajika. Maagizo yanatolewa kwa mzigo wa dinamik **(dyld)** kuhusu mchakato wa kupakia binary katika kumbukumbu.

Inatumia muundo wa **load\_command**, uliofafanuliwa katika **`loader.h`**:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
There are about **50 different types of load commands** that the system handles differently. The most common ones are: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, and `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
Kimsingi, aina hii ya Load Command inaelezea **jinsi ya kupakia \_\_TEXT** (kanuni inayotekelezeka) **na \_\_DATA** (data kwa ajili ya mchakato) **sehemu** kulingana na **offsets zilizotajwa katika sehemu ya Data** wakati binary inatekelezwa.
{% endhint %}

These commands **define segments** that are **mapped** into the **virtual memory space** of a process when it is executed.

There are **different types** of segments, such as the **\_\_TEXT** segment, which holds the executable code of a program, and the **\_\_DATA** segment, which contains data used by the process. These **segments are located in the data section** of the Mach-O file.

**Each segment** can be further **divided** into multiple **sections**. The **load command structure** contains **information** about **these sections** within the respective segment.

In the header first you find the **segment header**:

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

Example of segment header:

<figure><img src="../../../.gitbook/assets/image (1126).png" alt=""><figcaption></figcaption></figure>

This header defines the **number of sections whose headers appear after** it:
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
Example of **section header**:

<figure><img src="../../../.gitbook/assets/image (1108).png" alt=""><figcaption></figcaption></figure>

Ikiwa un **ongeza** **section offset** (0x37DC) + **offset** ambapo **arch inaanza**, katika kesi hii `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

Pia inawezekana kupata **headers information** kutoka kwa **command line** na:
```bash
otool -lv /bin/ls
```
Common segments loaded by this cmd:

* **`__PAGEZERO`:** Instructs the kernel to **map** the **address zero** so it **cannot be read from, written to, or executed**. The maxprot and minprot variables in the structure are set to zero to indicate there are **no read-write-execute rights on this page**.
* This allocation is important to **mitigate NULL pointer dereference vulnerabilities**. This is because XNU enforces a hard page zero that ensures the first page (only the first) of memory is inaccessible (except in i386). A binary could fulfill this requirement by crafting a small \_\_PAGEZERO (using the `-pagezero_size`) to cover the first 4k and having the rest of 32bit memory accessible in both user and kernel mode.
* **`__TEXT`**: Contains **executable** **code** with **read** and **execute** permissions (no writable)**.** Common sections of this segment:
* `__text`: Compiled binary code
* `__const`: Data ya kudumu (read only)
* `__[c/u/os_log]string`: C, Unicode au os logs string constants
* `__stubs` and `__stubs_helper`: Involved during the dynamic library loading process
* `__unwind_info`: Takwimu za stack unwind.
* Note that all this content is signed but also marked as executable (creating more options for exploitation of sections that doesn't necessarily need this privilege, like string dedicated sections).
* **`__DATA`**: Inahusisha data ambayo ni **readable** na **writable** (no executable)**.**
* `__got:` Global Offset Table
* `__nl_symbol_ptr`: Non lazy (bind at load) symbol pointer
* `__la_symbol_ptr`: Lazy (bind on use) symbol pointer
* `__const`: Inapaswa kuwa data ya kusoma tu (sio kweli)
* `__cfstring`: CoreFoundation strings
* `__data`: Global variables (ambazo zimeanzishwa)
* `__bss`: Static variables (ambazo hazijaanzishwa)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, nk): Taarifa inayotumiwa na Objective-C runtime
* **`__DATA_CONST`**: \_\_DATA.\_\_const haikuhakikishiwa kuwa ya kudumu (write permissions), wala viashiria vingine na GOT. Sehemu hii inafanya `__const`, baadhi ya waanzilishi na meza ya GOT (mara tu inapohakikishwa) **read only** kwa kutumia `mprotect`.
* **`__LINKEDIT`**: Inahusisha taarifa kwa linker (dyld) kama, alama, string, na entries za relocation table. Ni chombo cha jumla kwa maudhui ambayo hayapo katika `__TEXT` au `__DATA` na maudhui yake yanaelezewa katika amri nyingine za upakiaji.
* taarifa za dyld: Rebase, Non-lazy/lazy/weak binding opcodes na taarifa za usafirishaji
* Functions starts: Meza ya anwani za kuanzia za kazi
* Data In Code: Data islands katika \_\_text
* SYmbol Table: Alama katika binary
* Indirect Symbol Table: Pointer/stub symbols
* String Table
* Code Signature
* **`__OBJC`**: Inahusisha taarifa inayotumiwa na Objective-C runtime. Ingawa taarifa hii inaweza pia kupatikana katika sehemu ya \_\_DATA, ndani ya sehemu mbalimbali za \_\_objc\_\*.
* **`__RESTRICT`**: Sehemu isiyo na maudhui yenye sehemu moja inayoitwa **`__restrict`** (pia tupu) ambayo inahakikisha kwamba wakati wa kuendesha binary, itapuuzilia mbali mabadiliko ya mazingira ya DYLD.

Kama ilivyokuwa inawezekana kuona katika msimbo, **sehemu pia zinaunga mkono bendera** (ingawa hazitumiki sana):

* `SG_HIGHVM`: Core tu (haijatumiwa)
* `SG_FVMLIB`: Haijatumiwa
* `SG_NORELOC`: Sehemu haina relocation
* `SG_PROTECTED_VERSION_1`: Ulinzi. Inatumika kwa mfano na Finder kupeleka maandiko ya sehemu ya `__TEXT`.

### **`LC_UNIXTHREAD/LC_MAIN`**

**`LC_MAIN`** ina kiingilio katika **entryoff attribute.** Wakati wa upakiaji, **dyld** kwa urahisi **ongeza** thamani hii kwa (katika kumbukumbu) **msingi wa binary**, kisha **jump** kwa hii amri kuanza utekelezaji wa msimbo wa binary.

**`LC_UNIXTHREAD`** ina thamani ambazo register lazima iwe nazo wakati wa kuanzisha thread kuu. Hii tayari ilikuwa imeondolewa lakini **`dyld`** bado inaitumia. Inawezekana kuona thamani za register zilizowekwa na hii kwa:
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

Inashikilia taarifa kuhusu **sahihi ya msimbo wa faili ya Macho-O**. Inashikilia tu **offset** inayopointia **signature blob**. Hii kwa kawaida iko mwishoni mwa faili.\
Hata hivyo, unaweza kupata taarifa fulani kuhusu sehemu hii katika [**hiki kipande cha blog**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) na hii [**gists**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4).

### **`LC_ENCRYPTION_INFO[_64]`**

Msaada wa usimbuaji wa binary. Hata hivyo, bila shaka, ikiwa mshambuliaji atafanikiwa kuathiri mchakato, ataweza kutupa kumbukumbu bila usimbuaji.

### **`LC_LOAD_DYLINKER`**

Inashikilia **njia ya executable ya dynamic linker** inayopanga maktaba za pamoja katika nafasi ya anwani ya mchakato. **Thamani kila wakati imewekwa kwenye `/usr/lib/dyld`**. Ni muhimu kutambua kwamba katika macOS, uhamasishaji wa dylib unafanyika katika **mode ya mtumiaji**, si katika mode ya kernel.

### **`LC_IDENT`**

Imepitwa na wakati lakini wakati imewekwa ili kuunda dumps wakati wa panic, dump ya Mach-O core inaundwa na toleo la kernel linawekwa katika amri ya `LC_IDENT`.

### **`LC_UUID`**

UUID ya nasibu. Ni muhimu kwa chochote moja kwa moja lakini XNU inahifadhi pamoja na taarifa nyingine za mchakato. Inaweza kutumika katika ripoti za ajali.

### **`LC_DYLD_ENVIRONMENT`**

Inaruhusu kuashiria mabadiliko ya mazingira kwa dyld kabla ya mchakato kutekelezwa. Hii inaweza kuwa hatari sana kwani inaweza kuruhusu kutekeleza msimbo usio na mipaka ndani ya mchakato hivyo amri hii ya kupakia inatumika tu katika ujenzi wa dyld na `#define SUPPORT_LC_DYLD_ENVIRONMENT` na inazidisha mchakato tu kwa mabadiliko ya aina `DYLD_..._PATH` yanayoelezea njia za kupakia.

### **`LC_LOAD_DYLIB`**

Amri hii ya kupakia inaelezea utegemezi wa **maktaba** **dynamiki** ambayo **inaelekeza** **loader** (dyld) **kupakia na kuunganisha maktaba hiyo**. Kuna amri ya kupakia `LC_LOAD_DYLIB` **kwa kila maktaba** ambayo binary ya Mach-O inahitaji.

* Amri hii ya kupakia ni muundo wa aina **`dylib_command`** (ambayo ina muundo wa dylib, unaelezea maktaba halisi ya dinamik):
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

Unaweza pia kupata habari hii kutoka kwenye cli kwa:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Some potential malware related libraries are:

* **DiskArbitration**: Monitoring USB drives
* **AVFoundation:** Capture audio and video
* **CoreWLAN**: Wifi scans.

{% hint style="info" %}
A Mach-O binary can contain one or **more** **constructors**, that will be **executed** **before** the address specified in **LC\_MAIN**.\
The offsets of any constructors are held in the **\_\_mod\_init\_func** section of the **\_\_DATA\_CONST** segment.
{% endhint %}

## **Mach-O Data**

At the core of the file lies the data region, which is composed of several segments as defined in the load-commands region. **A variety of data sections can be housed within each segment**, with each section **holding code or data** specific to a type.

{% hint style="success" %}
The data is basically the part containing all the **information** that is loaded by the load commands **LC\_SEGMENTS\_64**
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

This includes:

* **Function table:** Which holds information about the program functions.
* **Symbol table**: Which contains information about the external function used by the binary
* It could also contain internal function, variable names as well and more.

To check it you could use the [**Mach-O View**](https://sourceforge.net/projects/machoview/) tool:

<figure><img src="../../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

Or from the cli:
```bash
size -m /bin/ls
```
## Objetive-C Sehemu za Kawaida

Katika sehemu ya `__TEXT` (r-x):

* `__objc_classname`: Majina ya darasa (nyuzi)
* `__objc_methname`: Majina ya mbinu (nyuzi)
* `__objc_methtype`: Aina za mbinu (nyuzi)

Katika sehemu ya `__DATA` (rw-):

* `__objc_classlist`: Viashiria kwa madarasa yote ya Objetive-C
* `__objc_nlclslist`: Viashiria kwa madarasa ya Objective-C yasiyo na uvivu
* `__objc_catlist`: Kiashiria kwa Kategoria
* `__objc_nlcatlist`: Kiashiria kwa Kategoria zisizo na uvivu
* `__objc_protolist`: Orodha ya protokali
* `__objc_const`: Takwimu za kudumu
* `__objc_imageinfo`, `__objc_selrefs`, `objc__protorefs`...

## Swift

* `_swift_typeref`, `_swift3_capture`, `_swift3_assocty`, `_swift3_types, _swift3_proto`, `_swift3_fieldmd`, `_swift3_builtin`, `_swift3_reflstr`

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
