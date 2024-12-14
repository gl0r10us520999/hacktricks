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

Mac OS binarni fajlovi obično se kompajliraju kao **univerzalni binarni fajlovi**. **Univerzalni binarni fajl** može **podržavati više arhitektura u istom fajlu**.

Ovi binarni fajlovi prate **Mach-O strukturu** koja se u osnovi sastoji od:

* Header
* Load Commands
* Data

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (470).png>)

## Fat Header

Pretražujte fajl sa: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC ili FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* broj struktura koje slede */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* specifikator cpu (int) */
cpu_subtype_t	cpusubtype;	/* specifikator mašine (int) */
uint32_t	offset;		/* pomeraj fajla do ovog objektnog fajla */
uint32_t	size;		/* veličina ovog objektnog fajla */
uint32_t	align;		/* poravnanje kao stepen od 2 */
};
</code></pre>

Header ima **magic** bajtove praćene **brojem** **arhitektura** koje fajl **sadrži** (`nfat_arch`) i svaka arhitektura će imati `fat_arch` strukturu.

Proverite to sa:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O univerzalni binarni fajl sa 2 arhitekture: [x86_64:Mach-O 64-bit izvršni fajl x86_64] [arm64e:Mach-O 64-bit izvršni fajl arm64e]
/bin/ls (za arhitekturu x86_64):	Mach-O 64-bit izvršni fajl x86_64
/bin/ls (za arhitekturu arm64e):	Mach-O 64-bit izvršni fajl arm64e

% otool -f -v /bin/ls
Fat headers
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>arhitektura x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capabilities 0x0
<strong>    offset 16384
</strong><strong>    size 72896
</strong>    align 2^14 (16384)
<strong>arhitektura arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capabilities PTR_AUTH_VERSION USERSPACE 0
<strong>    offset 98304
</strong><strong>    size 88816
</strong>    align 2^14 (16384)
</code></pre>

ili koristeći [Mach-O View](https://sourceforge.net/projects/machoview/) alat:

<figure><img src="../../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

Kao što možda mislite, obično univerzalni binarni fajl kompajliran za 2 arhitekture **udvostručuje veličinu** jednog kompajliranog za samo 1 arh.

## **Mach-O Header**

Header sadrži osnovne informacije o fajlu, kao što su magic bajtovi koji ga identifikuju kao Mach-O fajl i informacije o ciljnjoj arhitekturi. Možete ga pronaći u: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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

Postoje različite vrste datoteka, možete ih pronaći definisane u [**izvoru koda, na primer ovde**](https://opensource.apple.com/source/xnu/xnu-2050.18.24/EXTERNAL\_HEADERS/mach-o/loader.h). Najvažnije su:

* `MH_OBJECT`: Relokabilna objektna datoteka (intermedijarni proizvodi kompajlacije, još nisu izvršne).
* `MH_EXECUTE`: Izvršne datoteke.
* `MH_FVMLIB`: Fiksna VM biblioteka.
* `MH_CORE`: Ispravke koda
* `MH_PRELOAD`: Preučitana izvršna datoteka (više nije podržana u XNU)
* `MH_DYLIB`: Dinamičke biblioteke
* `MH_DYLINKER`: Dinamički linker
* `MH_BUNDLE`: "Plugin datoteke". Generisane korišćenjem -bundle u gcc i eksplicitno učitane od strane `NSBundle` ili `dlopen`.
* `MH_DYSM`: Prateća `.dSym` datoteka (datoteka sa simbolima za debagovanje).
* `MH_KEXT_BUNDLE`: Ekstenzije jezgra.
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

Izvorni kod takođe definiše nekoliko zastavica korisnih za učitavanje biblioteka:

* `MH_NOUNDEFS`: Nema neodređenih referenci (potpuno povezano)
* `MH_DYLDLINK`: Dyld povezivanje
* `MH_PREBOUND`: Dinamičke reference su unapred povezane.
* `MH_SPLIT_SEGS`: Datoteka deli r/o i r/w segmente.
* `MH_WEAK_DEFINES`: Binarni fajl ima slabe definisane simbole
* `MH_BINDS_TO_WEAK`: Binarni fajl koristi slabe simbole
* `MH_ALLOW_STACK_EXECUTION`: Omogućava izvršavanje steka
* `MH_NO_REEXPORTED_DYLIBS`: Biblioteka nema LC\_REEXPORT komande
* `MH_PIE`: Nezavisni izvršni fajl
* `MH_HAS_TLV_DESCRIPTORS`: Postoji sekcija sa lokalnim promenljivama niti
* `MH_NO_HEAP_EXECUTION`: Nema izvršavanja za heap/data stranice
* `MH_HAS_OBJC`: Binarni fajl ima oBject-C sekcije
* `MH_SIM_SUPPORT`: Podrška za simulator
* `MH_DYLIB_IN_CACHE`: Koristi se za dylibs/frameworks u kešu deljenih biblioteka.

## **Mach-O Load commands**

**Raspored datoteke u memoriji** je ovde specificiran, detaljno opisuje **lokaciju tabele simbola**, kontekst glavne niti na početku izvršavanja i potrebne **deljene biblioteke**. Uputstva su data dinamičkom učitaču **(dyld)** o procesu učitavanja binarnog fajla u memoriju.

Koristi **load\_command** strukturu, definisanu u pomenutom **`loader.h`**:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
There are about **50 different types of load commands** that the system handles differently. The most common ones are: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, and `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
U suštini, ova vrsta Load Command definiše **kako učitati \_\_TEXT** (izvršni kod) **i \_\_DATA** (podaci za proces) **segmente** prema **offsetima navedenim u Data sekciji** kada se izvršava binarni fajl.
{% endhint %}

Ove komande **definišu segmente** koji su **mapirani** u **virtuelni memorijski prostor** procesa kada se izvršava.

Postoje **različite vrste** segmenata, kao što je **\_\_TEXT** segment, koji sadrži izvršni kod programa, i **\_\_DATA** segment, koji sadrži podatke koje koristi proces. Ovi **segmenti se nalaze u data sekciji** Mach-O fajla.

**Svaki segment** može biti dalje **podeljen** na više **sekcija**. **Struktura load command** sadrži **informacije** o **ovim sekcijama** unutar odgovarajućeg segmenta.

U zaglavlju prvo nalazite **zaglavlje segmenta**:

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

Ako **dodate** **offset sekcije** (0x37DC) + **offset** gde **arhitektura počinje**, u ovom slučaju `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

Takođe je moguće dobiti **informacije o headerima** iz **komandne linije** sa:
```bash
otool -lv /bin/ls
```
```markdown
Uobičajeni segmenti koje učitava ovaj cmd:

* **`__PAGEZERO`:** Upravlja kernelom da **mapira** **adresu nula** tako da **ne može biti čitana, pisana ili izvršena**. Varijable maxprot i minprot u strukturi su postavljene na nulu da označe da nema **prava za čitanje-pisanje-izvršavanje na ovoj stranici**.
* Ova alokacija je važna za **ublažavanje ranjivosti dereferenciranja NULL pokazivača**. To je zato što XNU primenjuje strogu nulu stranice koja osigurava da je prva stranica (samo prva) memorije nedostupna (osim u i386). Binarni fajl može ispuniti ove zahteve kreiranjem male \_\_PAGEZERO (koristeći `-pagezero_size`) da pokrije prvih 4k i da ostatak 32bitne memorije bude dostupan u korisničkom i kernel modu.
* **`__TEXT`**: Sadrži **izvršni** **kod** sa **pravima za čitanje** i **izvršavanje** (bez prava za pisanje). Uobičajeni delovi ovog segmenta:
* `__text`: Kompajlirani binarni kod
* `__const`: Konstantni podaci (samo za čitanje)
* `__[c/u/os_log]string`: C, Unicode ili os log string konstante
* `__stubs` i `__stubs_helper`: Uključeni tokom procesa učitavanja dinamičke biblioteke
* `__unwind_info`: Podaci o vraćanju steka.
* Imajte na umu da je sav ovaj sadržaj potpisan, ali takođe označen kao izvršan (stvarajući više opcija za eksploataciju delova koji ne moraju nužno imati ovo pravo, poput sekcija posvećenih stringovima).
* **`__DATA`**: Sadrži podatke koji su **čitljivi** i **pisljivi** (bez izvršnih)**.**
* `__got:` Globalna tabela ofseta
* `__nl_symbol_ptr`: Nepasivan (vezan pri učitavanju) pokazivač simbola
* `__la_symbol_ptr`: Pasivan (vezan pri korišćenju) pokazivač simbola
* `__const`: Trebalo bi da budu podaci samo za čitanje (ne baš)
* `__cfstring`: CoreFoundation stringovi
* `__data`: Globalne promenljive (koje su inicijalizovane)
* `__bss`: Staticke promenljive (koje nisu inicijalizovane)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, itd): Informacije koje koristi Objective-C runtime
* **`__DATA_CONST`**: \_\_DATA.\_\_const nije garantovano da bude konstantan (prava za pisanje), niti su drugi pokazivači i GOT. Ova sekcija čini `__const`, neke inicijalizatore i GOT tabelu (jednom kada se reše) **samo za čitanje** koristeći `mprotect`.
* **`__LINKEDIT`**: Sadrži informacije za linker (dyld) kao što su, simbol, string i unosi tabele relokacije. To je generički kontejner za sadržaje koji nisu ni u `__TEXT` ni u `__DATA`, a njegov sadržaj je opisan u drugim komandama učitavanja.
* dyld informacije: Rebase, Nepasivni/pasivni/slabi binding opkodi i informacije o izvozu
* Funkcije počinju: Tabela start adresa funkcija
* Podaci u kodu: Podaci u ostrvima u \_\_text
* Tabela simbola: Simboli u binarnom
* Tabela indirektnih simbola: Pokazivači/stub simboli
* Tabela stringova
* Potpis koda
* **`__OBJC`**: Sadrži informacije koje koristi Objective-C runtime. Iako se ove informacije mogu naći i u \_\_DATA segmentu, unutar raznih \_\_objc\_\* sekcija.
* **`__RESTRICT`**: Segment bez sadržaja sa jednom sekcijom nazvanom **`__restrict`** (takođe prazna) koja osigurava da kada se izvršava binarni fajl, ignoriše DYLD promenljive okruženja.

Kao što je bilo moguće videti u kodu, **segmenti takođe podržavaju zastavice** (iako se ne koriste mnogo):

* `SG_HIGHVM`: Samo core (nije korišćeno)
* `SG_FVMLIB`: Nije korišćeno
* `SG_NORELOC`: Segment nema relokaciju
* `SG_PROTECTED_VERSION_1`: Enkripcija. Koristi se, na primer, od strane Findera za enkripciju teksta `__TEXT` segmenta.

### **`LC_UNIXTHREAD/LC_MAIN`**

**`LC_MAIN`** sadrži ulaznu tačku u **atributu entryoff.** Tokom učitavanja, **dyld** jednostavno **dodaje** ovu vrednost na (u memoriji) **bazu binarnog fajla**, a zatim **skače** na ovu instrukciju da započne izvršavanje koda binarnog fajla.

**`LC_UNIXTHREAD`** sadrži vrednosti koje registri moraju imati prilikom pokretanja glavne niti. Ovo je već zastarelo, ali **`dyld`** ga i dalje koristi. Moguće je videti vrednosti registara postavljenih ovim: 
```
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

Sadrži informacije o **digitalnom potpisu Macho-O datoteke**. Sadrži samo **offset** koji **pokazuje** na **blob potpisa**. Ovo je obično na samom kraju datoteke.\
Međutim, možete pronaći neke informacije o ovoj sekciji u [**ovom blog postu**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) i ovom [**gistu**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4).

### **`LC_ENCRYPTION_INFO[_64]`**

Podrška za enkripciju binarnih datoteka. Međutim, naravno, ako napadač uspe da kompromituje proces, moći će da isprazni memoriju neenkriptovano.

### **`LC_LOAD_DYLINKER`**

Sadrži **putanju do izvršne datoteke dinamičkog linkera** koja mapira deljene biblioteke u adresni prostor procesa. **Vrednost je uvek postavljena na `/usr/lib/dyld`**. Važno je napomenuti da se u macOS-u mapiranje dylib-a dešava u **korisničkom režimu**, a ne u režimu jezgra.

### **`LC_IDENT`**

Zastarjela, ali kada je konfigurisana da generiše dump-ove na paniku, kreira se Mach-O core dump i verzija jezgra se postavlja u `LC_IDENT` komandi.

### **`LC_UUID`**

Nasumični UUID. Koristan je za bilo šta direktno, ali XNU ga kešira sa ostatkom informacija o procesu. Može se koristiti u izveštajima o padu.

### **`LC_DYLD_ENVIRONMENT`**

Omogućava da se označe promenljive okruženja za dyld pre nego što se proces izvrši. Ovo može biti veoma opasno jer može omogućiti izvršavanje proizvoljnog koda unutar procesa, tako da se ova komanda učitavanja koristi samo u dyld verzijama sa `#define SUPPORT_LC_DYLD_ENVIRONMENT` i dodatno ograničava obradu samo na promenljive oblika `DYLD_..._PATH` koje specificiraju putanje učitavanja.

### **`LC_LOAD_DYLIB`**

Ova komanda učitavanja opisuje **dinamičku** **biblioteku** zavisnost koja **naredjuje** **učitaču** (dyld) da **učita i poveže navedenu biblioteku**. Postoji `LC_LOAD_DYLIB` komanda učitavanja **za svaku biblioteku** koja je potrebna Mach-O binarnoj datoteci.

* Ova komanda učitavanja je struktura tipa **`dylib_command`** (koja sadrži strukturu dylib, opisujući stvarnu zavisnu dinamičku biblioteku):
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

Takođe možete dobiti ove informacije iz CLI-a sa:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Neke potencijalne biblioteke povezane sa malverom su:

* **DiskArbitration**: Praćenje USB drajvova
* **AVFoundation:** Snimanje zvuka i videa
* **CoreWLAN**: Wifi skeniranja.

{% hint style="info" %}
Mach-O binarni fajl može sadržati jednog ili **više** **konstruktora**, koji će biti **izvršeni** **pre** adrese navedene u **LC\_MAIN**.\
Offseti svih konstruktora se čuvaju u **\_\_mod\_init\_func** sekciji **\_\_DATA\_CONST** segmenta.
{% endhint %}

## **Mach-O Podaci**

U srži fajla se nalazi region podataka, koji se sastoji od nekoliko segmenata kako je definisano u regionu komandi za učitavanje. **Različite sekcije podataka mogu biti smeštene unutar svakog segmenta**, pri čemu svaka sekcija **sadrži kod ili podatke** specifične za tip.

{% hint style="success" %}
Podaci su u suštini deo koji sadrži sve **informacije** koje se učitavaju komandom za učitavanje **LC\_SEGMENTS\_64**
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

Ovo uključuje:

* **Tabela funkcija:** Koja sadrži informacije o funkcijama programa.
* **Tabela simbola**: Koja sadrži informacije o spoljnim funkcijama korišćenim od strane binarnog fajla
* Takođe može sadržati interne funkcije, imena promenljivih i još mnogo toga.

Da biste to proverili, možete koristiti [**Mach-O View**](https://sourceforge.net/projects/machoview/) alat:

<figure><img src="../../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

Ili iz CLI:
```bash
size -m /bin/ls
```
## Objetive-C Zajedničke Sekcije

U `__TEXT` segmentu (r-x):

* `__objc_classname`: Imena klasa (stringovi)
* `__objc_methname`: Imena metoda (stringovi)
* `__objc_methtype`: Tipovi metoda (stringovi)

U `__DATA` segmentu (rw-):

* `__objc_classlist`: Pokazivači na sve Objetive-C klase
* `__objc_nlclslist`: Pokazivači na Non-Lazy Objective-C klase
* `__objc_catlist`: Pokazivač na Kategorije
* `__objc_nlcatlist`: Pokazivač na Non-Lazy Kategorije
* `__objc_protolist`: Lista protokola
* `__objc_const`: Konstantni podaci
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
