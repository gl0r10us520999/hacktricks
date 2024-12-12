# Універсальні бінарні файли та формат Mach-O

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа хакінгу HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа хакінгу HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Основна інформація

Бінарні файли Mac OS зазвичай компілюються як **універсальні бінарні файли**. **Універсальний бінарний файл** може **підтримувати кілька архітектур у одному файлі**.

Ці бінарні файли відповідають **структурі Mach-O**, яка в основному складається з:

* Заголовка
* Команд завантаження
* Даних

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (470).png>)

## Fat Header

Шукайте файл за допомогою: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

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

Заголовок містить **магічні** байти, за якими слідує **кількість** **архітектур**, які містить файл (`nfat_arch`), і кожна архітектура матиме структуру `fat_arch`.

Перевірте це за допомогою:

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

або використовуючи інструмент [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

Як ви, можливо, мислите, зазвичай універсальний бінарний файл, скомпільований для 2 архітектур, **подвоює розмір** того, що скомпільовано лише для 1 архітектури.

## **Заголовок Mach-O**

Заголовок містить базову інформацію про файл, таку як магічні байти для ідентифікації його як файлу Mach-O та інформацію про цільову архітектуру. Ви можете знайти його за допомогою: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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
### Типи файлів Mach-O

Існують різні типи файлів, ви можете знайти їх визначені в [**вихідному коді, наприклад, тут**](https://opensource.apple.com/source/xnu/xnu-2050.18.24/EXTERNAL\_HEADERS/mach-o/loader.h). Найважливіші з них:

* `MH_OBJECT`: Файл об'єкта, який можна переміщати (проміжні продукти компіляції, ще не виконувані файли).
* `MH_EXECUTE`: Виконувані файли.
* `MH_FVMLIB`: Файл фіксованої бібліотеки VM.
* `MH_CORE`: Відкладені коди
* `MH_PRELOAD`: Передзавантажений виконуваний файл (більше не підтримується в XNU)
* `MH_DYLIB`: Динамічні бібліотеки
* `MH_DYLINKER`: Динамічний лінкер
* `MH_BUNDLE`: "Файли плагінів". Створені за допомогою -bundle в gcc та явно завантажені за допомогою `NSBundle` або `dlopen`.
* `MH_DYSM`: Супутній файл `.dSym` (файл з символами для налагодження).
* `MH_KEXT_BUNDLE`: Розширення ядра.
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
Або використовуючи [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

## **Прапорці Mach-O**

Вихідний код також визначає кілька корисних прапорців для завантаження бібліотек:

* `MH_NOUNDEFS`: Немає невизначених посилань (повністю зв'язаний)
* `MH_DYLDLINK`: Посилання Dyld
* `MH_PREBOUND`: Динамічні посилання попередньо зв'язані.
* `MH_SPLIT_SEGS`: Файл розділяє сегменти r/o та r/w.
* `MH_WEAK_DEFINES`: У бінарному файлі є слабкі визначені символи
* `MH_BINDS_TO_WEAK`: Бінарний файл використовує слабкі символи
* `MH_ALLOW_STACK_EXECUTION`: Зробити стек виконавчим
* `MH_NO_REEXPORTED_DYLIBS`: Бібліотека не містить команд LC\_REEXPORT
* `MH_PIE`: Виконавчий файл з незалежними від положення адреси
* `MH_HAS_TLV_DESCRIPTORS`: Є розділ зі змінними локального потоку
* `MH_NO_HEAP_EXECUTION`: Відсутнє виконання для сторінок купи/даних
* `MH_HAS_OBJC`: У бінарному файлі є розділи oBject-C
* `MH_SIM_SUPPORT`: Підтримка симулятора
* `MH_DYLIB_IN_CACHE`: Використовується на dylibs/frameworks у спільному кеші бібліотек.

## **Команди завантаження Mach-O**

Тут вказано **розташування файлу в пам'яті**, деталізуючи **розташування таблиці символів**, контекст основного потоку при початку виконання та необхідні **спільні бібліотеки**. Надаються інструкції для динамічного завантажувача **(dyld)** щодо процесу завантаження бінарного файлу в пам'ять.

Використовує структуру **load\_command**, визначену в зазначеному **`loader.h`**:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
Існує близько **50 різних типів команд завантаження**, які система обробляє по-різному. Найпоширеніші з них: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB` та `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
Фактично, цей тип команди завантаження визначає, **як завантажувати сегменти \_\_TEXT** (виконавчий код) **та \_\_DATA** (дані для процесу) **згідно з вказаними зміщеннями в розділі Дані** при виконанні бінарного файлу.
{% endhint %}

Ці команди **визначають сегменти**, які **відображаються** в **віртуальному просторі пам'яті** процесу під час його виконання.

Існують **різні типи** сегментів, такі як сегмент **\_\_TEXT**, який містить виконавчий код програми, та сегмент **\_\_DATA**, який містить дані, використовувані процесом. Ці **сегменти розташовані в розділі даних** файлу Mach-O.

**Кожен сегмент** може бути поділений на кілька **секцій**. Структура **команди завантаження** містить **інформацію** про **ці секції** у відповідному сегменті.

У заголовку спочатку ви знаходите **заголовок сегмента**:

<pre class="language-c"><code class="lang-c">struct segment_command_64 { /* для 64-бітних архітектур */
uint32_t	cmd;		/* LC_SEGMENT_64 */
uint32_t	cmdsize;	/* включає розмір структур section_64 */
char		segname[16];	/* назва сегмента */
uint64_t	vmaddr;		/* адреса пам'яті цього сегмента */
uint64_t	vmsize;		/* розмір пам'яті цього сегмента */
uint64_t	fileoff;	/* зміщення файлу цього сегмента */
uint64_t	filesize;	/* кількість для відображення з файлу */
int32_t		maxprot;	/* максимальний захист VM */
int32_t		initprot;	/* початковий захист VM */
<strong>	uint32_t	nsects;		/* кількість секцій у сегменті */
</strong>	uint32_t	flags;		/* прапорці */
};
</code></pre>

Приклад заголовка сегмента:

<figure><img src="../../../.gitbook/assets/image (1126).png" alt=""><figcaption></figcaption></figure>

Цей заголовок визначає **кількість секцій, заголовки яких з'являються після** нього:
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
Приклад **заголовка розділу**:

<figure><img src="../../../.gitbook/assets/image (1108).png" alt=""><figcaption></figcaption></figure>

Якщо ви **додаєте** **зміщення розділу** (0x37DC) + **зміщення**, де **починається архітектура**, у цьому випадку `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

Також можна отримати **інформацію про заголовки** з **командного рядка** за допомогою:
```bash
otool -lv /bin/ls
```
Спільні сегменти, завантажені цією командою:

* **`__PAGEZERO`:** Це вказує ядру **відобразити** **адресу нуль**, щоб її **не можна було читати, записувати або виконувати**. Змінні maxprot та minprot у структурі встановлені на нуль, щоб показати, що на цій сторінці **немає прав на читання-запис-виконання**.
* Це виділення важливе для **пом'якшення вразливостей нульового вказівника**. Це тому, що XNU застосовує жорстку сторінку нуль, яка забезпечує, що перша сторінка (тільки перша) пам'яті недоступна (крім в i386). Бінарний файл може відповісти цим вимогам, створивши невеликий \_\_PAGEZERO (використовуючи `-pagezero_size`) для охоплення перших 4 кб та зробивши решту пам'яті 32-біт доступною як у режимі користувача, так і в режимі ядра.
* **`__TEXT`**: Містить **виконуваний** **код** з дозволами на **читання** та **виконання** (без можливості запису)**.** Загальні розділи цього сегмента:
* `__text`: Скомпільований бінарний код
* `__const`: Константні дані (тільки для читання)
* `__[c/u/os_log]string`: Константи рядків C, Unicode або os logs
* `__stubs` та `__stubs_helper`: Використовуються під час процесу завантаження динамічних бібліотек
* `__unwind_info`: Дані розгортання стеку.
* Зверніть увагу, що весь цей вміст підписаний, але також позначений як виконуваний (створюючи більше варіантів для експлуатації розділів, які не обов'язково потребують цієї привілеї, наприклад, розділів, присвячених рядкам).
* **`__DATA`**: Містить дані, які можна **читати** та **писати** (без можливості виконання)**.**
* `__got:` Глобальна таблиця зміщень
* `__nl_symbol_ptr`: Вказівник на символи Non lazy (bind at load)
* `__la_symbol_ptr`: Вказівник на символи Lazy (bind on use)
* `__const`: Повинні бути дані тільки для читання (насправді ні)
* `__cfstring`: Строки CoreFoundation
* `__data`: Глобальні змінні (які були ініціалізовані)
* `__bss`: Статичні змінні (які не були ініціалізовані)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, тощо): Інформація, використовувана середовищем виконання Objective-C
* **`__DATA_CONST`**: \_\_DATA.\_\_const не гарантується, що воно є постійним (права на запис), так само як і інші вказівники та таблиця GOT. Цей розділ робить `__const`, деякі ініціалізатори та таблицю GOT (після вирішення) **тільки для читання** за допомогою `mprotect`.
* **`__LINKEDIT`**: Містить інформацію для лінкера (dyld), таку як, символи, рядки та записи таблиці перенесення. Це загальний контейнер для вмісту, який не знаходиться ані в `__TEXT`, ані в `__DATA`, а його вміст описаний в інших командах завантаження.
* Інформація dyld: Rebase, опкоди для не-лінивого/лінивого/слабкого зв'язування та інформація про експорт
* Початок функцій: Таблиця початкових адрес функцій
* Дані у коді: Дані на островах в \_\_text
* Таблиця символів: Символи в бінарному файлі
* Таблиця непрямих символів: Вказівники/заглушки символів
* Таблиця рядків
* Підпис коду
* **`__OBJC`**: Містить інформацію, використовувану середовищем виконання Objective-C. Хоча цю інформацію також можна знайти в сегменті \_\_DATA, в різних розділах \_\_objc\_\*.
* **`__RESTRICT`**: Сегмент без вмісту з одним розділом під назвою **`__restrict`** (також порожній), який забезпечує, що під час виконання бінарного файлу будуть ігноруватися змінні середовища DYLD.

Як можна було побачити в коді, **сегменти також підтримують прапорці** (хоча вони не використовуються дуже часто):

* `SG_HIGHVM`: Тільки для ядра (не використовується)
* `SG_FVMLIB`: Не використовується
* `SG_NORELOC`: Сегмент не має перерозташування
* `SG_PROTECTED_VERSION_1`: Шифрування. Використовується, наприклад, Finder для шифрування тексту в сегменті `__TEXT`.

### **`LC_UNIXTHREAD/LC_MAIN`**

**`LC_MAIN`** містить точку входу в атрибуті **entryoff.** Під час завантаження **dyld** просто **додає** це значення до (в пам'яті) **бази бінарного файлу**, а потім **переходить** до цієї інструкції для початку виконання коду бінарного файлу.

**`LC_UNIXTHREAD`** містить значення, які повинні мати регістри при запуску головного потоку. Це вже застаріло, але **`dyld`** все ще використовує його. Можна побачити значення регістрів, встановлені цим, за допомогою:
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

Містить інформацію про **підпис коду файлу Mach-O**. Він містить лише **зсув**, який **вказує** на **блоб підпису**. Зазвичай це знаходиться в самому кінці файлу.\
Однак ви можете знайти деяку інформацію про цей розділ у [**цьому дописі у блозі**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) та у цьому [**фрагменті**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4).

### **`LC_ENCRYPTION_INFO[_64]`**

Підтримка шифрування бінарного коду. Однак, звісно, якщо зловмисник здатний скомпрометувати процес, він зможе витягнути пам'ять в розшифрованому вигляді.

### **`LC_LOAD_DYLINKER`**

Містить **шлях до виконуваного файлу динамічного зв'язувача**, який відображає спільні бібліотеки в адресний простір процесу. **Значення завжди встановлено на `/usr/lib/dyld`**. Важливо зауважити, що в macOS відображення dylib відбувається в **режимі користувача**, а не в режимі ядра.

### **`LC_IDENT`**

Застарілий, але якщо налаштовано на генерацію дампів при паніці, створюється дамп ядра Mach-O, і версія ядра встановлюється в команді `LC_IDENT`.

### **`LC_UUID`**

Випадковий UUID. Він корисний для чого завгодно, але XNU кешує його разом з іншою інформацією про процес. Він може бути використаний у звітах про аварії.

### **`LC_DYLD_ENVIRONMENT`**

Дозволяє вказати змінні середовища для dyld перед виконанням процесу. Це може бути дуже небезпечно, оскільки це може дозволити виконати довільний код всередині процесу, тому ця команда завантаження використовується лише в dyld, побудованому з `#define SUPPORT_LC_DYLD_ENVIRONMENT` та додатково обмежує обробку лише змінних у формі `DYLD_..._PATH`, що вказують шляхи завантаження.

### **`LC_LOAD_DYLIB`**

Ця команда завантаження описує **залежність динамічної бібліотеки**, яка **вказує** **завантажувачу** (dyld) **завантажити та зв'язати цю бібліотеку**. Існує команда завантаження `LC_LOAD_DYLIB` **для кожної бібліотеки**, яку потребує бінарний файл Mach-O.

* Ця команда завантаження є структурою типу **`dylib_command`** (яка містить структуру dylib, що описує фактичну залежну динамічну бібліотеку):
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

Ви також можете отримати цю інформацію з командного рядка за допомогою:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Деякі потенційно пов'язані з шкідливим ПЗ бібліотеки:

* **DiskArbitration**: Моніторинг USB-накопичувачів
* **AVFoundation:** Захоплення аудіо та відео
* **CoreWLAN**: Сканування Wifi.

{% hint style="info" %}
Mach-O бінарний файл може містити один або **більше конструкторів**, які будуть **виконані перед** адресою, вказаною в **LC\_MAIN**.\
Зміщення будь-яких конструкторів зберігаються в розділі **\_\_mod\_init\_func** сегмента **\_\_DATA\_CONST**.
{% endhint %}

## **Дані Mach-O**

У основі файлу знаходиться область даних, яка складається з кількох сегментів, які визначені в області команд завантаження. **В кожному сегменті можуть міститися різні секції даних**, причому кожна секція **містить код або дані**, що специфічні для типу.

{% hint style="success" %}
Дані - це в основному частина, що містить всю **інформацію**, яка завантажується командами завантаження **LC\_SEGMENTS\_64**
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

Це включає:

* **Таблиця функцій:** Яка містить інформацію про функції програми.
* **Таблиця символів**: Яка містить інформацію про зовнішню функцію, використану бінарним файлом
* Також може містити внутрішні функції, назви змінних та інше.

Для перевірки цього можна використовувати інструмент [**Mach-O View**](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

Або з командного рядка:
```bash
size -m /bin/ls
```
## Загальні розділи Objective-C

У сегменті `__TEXT` (r-x):

* `__objc_classname`: Імена класів (рядки)
* `__objc_methname`: Назви методів (рядки)
* `__objc_methtype`: Типи методів (рядки)

У сегменті `__DATA` (rw-):

* `__objc_classlist`: Вказівники на всі класи Objective-C
* `__objc_nlclslist`: Вказівники на неліниві класи Objective-C
* `__objc_catlist`: Вказівник на категорії
* `__objc_nlcatlist`: Вказівник на неліниві категорії
* `__objc_protolist`: Список протоколів
* `__objc_const`: Константні дані
* `__objc_imageinfo`, `__objc_selrefs`, `objc__protorefs`...

## Swift

* `_swift_typeref`, `_swift3_capture`, `_swift3_assocty`, `_swift3_types, _swift3_proto`, `_swift3_fieldmd`, `_swift3_builtin`, `_swift3_reflstr`
