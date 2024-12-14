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

## Temel Bilgiler

Mac OS ikili dosyaları genellikle **evrensel ikili dosyalar** olarak derlenir. Bir **evrensel ikili dosya**, **aynı dosyada birden fazla mimariyi destekleyebilir**.

Bu ikili dosyalar, temelde aşağıdaki **Mach-O yapısını** takip eder:

* Başlık
* Yükleme Komutları
* Veri

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (470).png>)

## Fat Başlık

Dosyayı aramak için: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC veya FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* takip eden yapıların sayısı */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* cpu belirleyici (int) */
cpu_subtype_t	cpusubtype;	/* makine belirleyici (int) */
uint32_t	offset;		/* bu nesne dosyasına dosya ofseti */
uint32_t	size;		/* bu nesne dosyasının boyutu */
uint32_t	align;		/* 2'nin kuvveti olarak hizalama */
};
</code></pre>

Başlık, **magic** baytları ile başlar ve dosyanın **içerdiği** **mimari** sayısını (`nfat_arch`) takip eder ve her mimari bir `fat_arch` yapısına sahip olacaktır.

Bunu kontrol etmek için:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O evrensel ikili dosya, 2 mimari ile: [x86_64:Mach-O 64-bit yürütülebilir x86_64] [arm64e:Mach-O 64-bit yürütülebilir arm64e]
/bin/ls (mimari x86_64 için):	Mach-O 64-bit yürütülebilir x86_64
/bin/ls (mimari arm64e için):	Mach-O 64-bit yürütülebilir arm64e

% otool -f -v /bin/ls
Fat başlıklar
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>mimari x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capabilities 0x0
<strong>    offset 16384
</strong><strong>    size 72896
</strong>    align 2^14 (16384)
<strong>mimari arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capabilities PTR_AUTH_VERSION USERSPACE 0
<strong>    offset 98304
</strong><strong>    size 88816
</strong>    align 2^14 (16384)
</code></pre>

veya [Mach-O View](https://sourceforge.net/projects/machoview/) aracını kullanarak:

<figure><img src="../../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

Düşündüğünüz gibi, genellikle 2 mimari için derlenmiş bir evrensel ikili dosya, yalnızca 1 mimari için derlenmiş olanın **boyutunu iki katına çıkarır**.

## **Mach-O Başlığı**

Başlık, dosyanın Mach-O dosyası olarak tanımlanmasını sağlayan magic baytları ve hedef mimari hakkında bilgi gibi temel bilgileri içerir. Bunu şu komutla bulabilirsiniz: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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
### Mach-O Dosya Türleri

Farklı dosya türleri vardır, bunları [**örneğin burada**](https://opensource.apple.com/source/xnu/xnu-2050.18.24/EXTERNAL\_HEADERS/mach-o/loader.h) tanımlanmış bulabilirsiniz. En önemli olanları şunlardır:

* `MH_OBJECT`: Yeniden yerleştirilebilir nesne dosyası (derlemenin ara ürünleri, henüz çalıştırılabilir değil).
* `MH_EXECUTE`: Çalıştırılabilir dosyalar.
* `MH_FVMLIB`: Sabit VM kütüphane dosyası.
* `MH_CORE`: Kod Dökümleri
* `MH_PRELOAD`: Önceden yüklenmiş çalıştırılabilir dosya (XNU'da artık desteklenmiyor)
* `MH_DYLIB`: Dinamik Kütüphaneler
* `MH_DYLINKER`: Dinamik Bağlayıcı
* `MH_BUNDLE`: "Eklenti dosyaları". gcc'de -bundle kullanılarak oluşturulur ve `NSBundle` veya `dlopen` ile açıkça yüklenir.
* `MH_DYSM`: Eşlik eden `.dSym` dosyası (hata ayıklama için semboller içeren dosya).
* `MH_KEXT_BUNDLE`: Çekirdek Uzantıları.
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
Or using [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

## **Mach-O Bayrakları**

Kaynak kodu, kütüphaneleri yüklemek için yararlı olan birkaç bayrak tanımlar:

* `MH_NOUNDEFS`: Tanımsız referans yok (tamamen bağlantılı)
* `MH_DYLDLINK`: Dyld bağlantısı
* `MH_PREBOUND`: Dinamik referanslar önceden bağlanmış.
* `MH_SPLIT_SEGS`: Dosya r/o ve r/w segmentlerini böler.
* `MH_WEAK_DEFINES`: İkili zayıf tanımlı sembollere sahiptir
* `MH_BINDS_TO_WEAK`: İkili zayıf semboller kullanır
* `MH_ALLOW_STACK_EXECUTION`: Yığını çalıştırılabilir hale getir
* `MH_NO_REEXPORTED_DYLIBS`: Kütüphane LC\_REEXPORT komutları yok
* `MH_PIE`: Konum Bağımsız Yürütülebilir
* `MH_HAS_TLV_DESCRIPTORS`: Thread yerel değişkenlerle bir bölüm var
* `MH_NO_HEAP_EXECUTION`: Yığın/veri sayfaları için yürütme yok
* `MH_HAS_OBJC`: İkili oBject-C bölümleri var
* `MH_SIM_SUPPORT`: Simülatör desteği
* `MH_DYLIB_IN_CACHE`: Paylaşılan kütüphane önbelleğinde kullanılan dylibs/frameworks.

## **Mach-O Yükleme komutları**

**Dosyanın bellek düzeni** burada belirtilmiştir, **sembol tablosunun konumu**, yürütme başlangıcındaki ana iş parçacığının bağlamı ve gerekli **paylaşılan kütüphaneler** hakkında ayrıntılar sunar. Dinamik yükleyiciye **(dyld)** ikilinin bellek yükleme süreci hakkında talimatlar verilir.

**load\_command** yapısını kullanır, belirtilen **`loader.h`** dosyasında tanımlanmıştır:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
There are about **50 different types of load commands** that the system handles differently. The most common ones are: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, and `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
Temelde, bu tür bir Load Command, ikili dosya çalıştırıldığında **__TEXT** (çalıştırılabilir kod) **ve __DATA** (işlem için veri) **segmentlerini** **Data bölümünde belirtilen ofsetlere** göre **nasıl yükleyeceğini** tanımlar.
{% endhint %}

Bu komutlar, bir işlem çalıştırıldığında **sanallaştırılmış bellek alanına** **haritalanan** **segmentleri** **tanımlar**.

**__TEXT** segmenti gibi, bir programın çalıştırılabilir kodunu tutan ve **__DATA** segmenti gibi, işlem tarafından kullanılan verileri içeren **farklı türde** segmentler vardır. Bu **segmentler, Mach-O dosyasının veri bölümünde** yer alır.

**Her segment**, daha fazla **bölümlere** **ayrılabilir**. **Load command yapısı**, ilgili segment içindeki **bu bölümler** hakkında **bilgi** içerir.

Başlıkta önce **segment başlığı** bulunur:

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
Örnek **bölüm başlığı**:

<figure><img src="../../../.gitbook/assets/image (1108).png" alt=""><figcaption></figcaption></figure>

Eğer **bölüm ofsetini** (0x37DC) + **mimari başlangıç ofsetini** eklerseniz, bu durumda `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

Ayrıca **başlık bilgilerini** **komut satırından** almak da mümkündür:
```bash
otool -lv /bin/ls
```
Common segments loaded by this cmd:

* **`__PAGEZERO`:** Çekirdekten **adres sıfırını** **haritalandırmasını** ister, böylece **okunamaz, yazılamaz veya çalıştırılamaz**. Yapının maxprot ve minprot değişkenleri, bu sayfada **okuma-yazma-çalıştırma hakları olmadığını** belirtmek için sıfıra ayarlanır.
* Bu tahsis, **NULL işaretçi dereferans hatalarını azaltmak** için önemlidir. Bunun nedeni, XNU'nun ilk sayfanın (sadece ilk) bellekte erişilemez olmasını sağlayan katı bir sayfa sıfırını zorlamasıdır (i386 dışında). Bir ikili, ilk 4k'yi kapsamak için küçük bir \_\_PAGEZERO oluşturarak ve geri kalan 32bit belleği hem kullanıcı hem de çekirdek modunda erişilebilir hale getirerek bu gereksinimleri karşılayabilir.
* **`__TEXT`**: **okunabilir** ve **çalıştırılabilir** (yazılabilir değil) **kod** içerir. Bu segmentin yaygın bölümleri:
* `__text`: Derlenmiş ikili kod
* `__const`: Sabit veri (sadece okunabilir)
* `__[c/u/os_log]string`: C, Unicode veya os log sabit dizeleri
* `__stubs` ve `__stubs_helper`: Dinamik kütüphane yükleme sürecinde yer alır
* `__unwind_info`: Yığın açılma verisi.
* Tüm bu içeriğin imzalandığını ancak aynı zamanda çalıştırılabilir olarak işaretlendiğini unutmayın (bu ayrıcalığı gerektirmeyen bölümlerin istismar seçeneklerini artırır, örneğin dizeye özel bölümler).
* **`__DATA`**: **okunabilir** ve **yazılabilir** (çalıştırılabilir değil) veri içerir.
* `__got:` Küresel Ofset Tablosu
* `__nl_symbol_ptr`: Tembel olmayan (yükleme sırasında bağlanan) sembol işaretçisi
* `__la_symbol_ptr`: Tembel (kullanımda bağlanan) sembol işaretçisi
* `__const`: Sadece okunabilir veri olmalıdır (gerçekte değil)
* `__cfstring`: CoreFoundation dizeleri
* `__data`: Küresel değişkenler (başlatılmış olanlar)
* `__bss`: Statik değişkenler (başlatılmamış olanlar)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, vb.): Objective-C çalışma zamanı tarafından kullanılan bilgiler
* **`__DATA_CONST`**: \_\_DATA.\_\_const sabit olacağı garanti edilmez (yazma izinleri), diğer işaretçiler ve GOT da öyle. Bu bölüm, `__const`, bazı başlatıcılar ve GOT tablosunu (bir kez çözüldüğünde) **sadece okunabilir** hale getirir.
* **`__LINKEDIT`**: Bağlayıcı (dyld) için sembol, dize ve yer değiştirme tablosu girişleri gibi bilgileri içerir. `__TEXT` veya `__DATA` içinde olmayan içerikler için genel bir konteynerdir ve içeriği diğer yükleme komutlarında tanımlanır.
* dyld bilgisi: Yeniden temel alma, Tembel/tembel/zaaf bağlama opcode'ları ve dışa aktarma bilgisi
* Fonksiyon başlangıçları: Fonksiyonların başlangıç adresleri tablosu
* Kod İçindeki Veri: \_\_text içindeki veri adaları
* Sembol Tablosu: İkili içindeki semboller
* Dolaylı Sembol Tablosu: İşaretçi/stub sembolleri
* Dize Tablosu
* Kod İmzası
* **`__OBJC`**: Objective-C çalışma zamanı tarafından kullanılan bilgileri içerir. Bu bilgilere \_\_DATA segmentinde, \_\_objc\_\* bölümlerinde de rastlanabilir.
* **`__RESTRICT`**: İçeriği olmayan ve **`__restrict`** (aynı zamanda boş) adında tek bir bölümü olan bir segmenttir; bu, ikili çalıştırıldığında DYLD çevresel değişkenlerini göz ardı edeceğini garanti eder.

Kodda görüldüğü gibi, **segmentler ayrıca bayrakları destekler** (her ne kadar çok kullanılmasa da):

* `SG_HIGHVM`: Sadece çekirdek (kullanılmıyor)
* `SG_FVMLIB`: Kullanılmıyor
* `SG_NORELOC`: Segmentin yer değiştirmesi yok
* `SG_PROTECTED_VERSION_1`: Şifreleme. Örneğin, Finder tarafından `__TEXT` segmentini şifrelemek için kullanılır.

### **`LC_UNIXTHREAD/LC_MAIN`**

**`LC_MAIN`** **entryoff** niteliğinde giriş noktasını içerir. Yükleme sırasında, **dyld** bu değeri (bellekteki) **ikili tabanına** **ekler**, ardından **bu talimata atlar** ve ikilinin kodunun yürütülmesine başlar.

**`LC_UNIXTHREAD`** ana iş parçacığını başlatırken kayıtların sahip olması gereken değerleri içerir. Bu zaten kullanımdan kaldırılmıştır ama **`dyld`** hala bunu kullanır. Bu kayıtların ayarlandığı değerleri görmek mümkündür:
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

Macho-O dosyasının **kod imzası** hakkında bilgi içerir. Sadece **imza blob'una** **işaret eden** bir **offset** içerir. Bu genellikle dosyanın en sonunda bulunur.\
Ancak, bu bölüm hakkında bazı bilgileri [**bu blog yazısında**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) ve bu [**gists**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4) bulabilirsiniz.

### **`LC_ENCRYPTION_INFO[_64]`**

İkili şifreleme desteği. Ancak, elbette, bir saldırgan süreci tehlikeye atmayı başarırsa, şifrelenmemiş belleği dökebilecektir.

### **`LC_LOAD_DYLINKER`**

Paylaşılan kütüphaneleri süreç adres alanına haritalayan **dinamik bağlayıcı yürütülebilir dosyasının yolunu** içerir. **Değer her zaman `/usr/lib/dyld` olarak ayarlanır**. macOS'ta, dylib haritalamanın **kullanıcı modunda** gerçekleştiğini belirtmek önemlidir, çekirdek modunda değil.

### **`LC_IDENT`**

Eski ama panik durumunda dökümler oluşturacak şekilde yapılandırıldığında, bir Mach-O çekirdek dökümü oluşturulur ve çekirdek sürümü `LC_IDENT` komutunda ayarlanır.

### **`LC_UUID`**

Rastgele UUID. Doğrudan herhangi bir şey için faydalıdır ama XNU bunu süreç bilgileriyle birlikte önbelleğe alır. Çökme raporlarında kullanılabilir.

### **`LC_DYLD_ENVIRONMENT`**

Süreç çalıştırılmadan önce dyld'ye ortam değişkenlerini belirtmeye olanak tanır. Bu, süreç içinde rastgele kod çalıştırılmasına izin verebileceğinden oldukça tehlikeli olabilir, bu nedenle bu yükleme komutu yalnızca `#define SUPPORT_LC_DYLD_ENVIRONMENT` ile derlenmiş dyld'de kullanılır ve işleme yalnızca `DYLD_..._PATH` biçimindeki değişkenlerle sınırlıdır.

### **`LC_LOAD_DYLIB`**

Bu yükleme komutu, **yükleyiciye** (dyld) **söz konusu kütüphaneyi yüklemesi ve bağlaması** için talimat veren bir **dinamik** **kütüphane** bağımlılığını tanımlar. Mach-O ikilisinin gerektirdiği **her kütüphane için bir `LC_LOAD_DYLIB` yükleme komutu vardır**.

* Bu yükleme komutu, gerçek bağımlı dinamik kütüphaneyi tanımlayan bir struct dylib içeren **`dylib_command`** türünde bir yapıdır:
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

Bu bilgiyi cli ile de alabilirsiniz:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Bazı potansiyel kötü amaçlı yazılım ile ilgili kütüphaneler şunlardır:

* **DiskArbitration**: USB sürücüleri izleme
* **AVFoundation:** Ses ve video yakalama
* **CoreWLAN**: Wifi taramaları.

{% hint style="info" %}
Bir Mach-O ikili dosyası bir veya **daha fazla** **constructor** içerebilir, bu **LC\_MAIN**'de belirtilen adresten **önce** **çalıştırılacaktır**.\
Herhangi bir constructor'ın ofsetleri **\_\_mod\_init\_func** bölümünde **\_\_DATA\_CONST** segmentinde tutulur.
{% endhint %}

## **Mach-O Verisi**

Dosyanın merkezinde, yükleme komutları bölgesinde tanımlanan birkaç segmentten oluşan veri bölgesi yer alır. **Her segment içinde çeşitli veri bölümleri barındırabilir**, her bölüm **bir türle ilgili kod veya veriyi** tutar.

{% hint style="success" %}
Veri, esasen yükleme komutları **LC\_SEGMENTS\_64** tarafından yüklenen tüm **bilgileri** içeren kısımdır.
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

Bu şunları içerir:

* **Fonksiyon tablosu:** Program fonksiyonları hakkında bilgi tutar.
* **Sembol tablosu**: İkili dosya tarafından kullanılan dış fonksiyonlar hakkında bilgi içerir.
* Ayrıca iç fonksiyonlar, değişken adları ve daha fazlasını da içerebilir.

Bunu kontrol etmek için [**Mach-O View**](https://sourceforge.net/projects/machoview/) aracını kullanabilirsiniz:

<figure><img src="../../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

Ya da cli'dan:
```bash
size -m /bin/ls
```
## Objetive-C Ortak Bölümler

`__TEXT` segmentinde (r-x):

* `__objc_classname`: Sınıf adları (stringler)
* `__objc_methname`: Metot adları (stringler)
* `__objc_methtype`: Metot türleri (stringler)

`__DATA` segmentinde (rw-):

* `__objc_classlist`: Tüm Objective-C sınıflarına işaretçiler
* `__objc_nlclslist`: Tembel Olmayan Objective-C sınıflarına işaretçiler
* `__objc_catlist`: Kategorilere işaretçi
* `__objc_nlcatlist`: Tembel Olmayan Kategorilere işaretçi
* `__objc_protolist`: Protokol listesi
* `__objc_const`: Sabit veri
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
