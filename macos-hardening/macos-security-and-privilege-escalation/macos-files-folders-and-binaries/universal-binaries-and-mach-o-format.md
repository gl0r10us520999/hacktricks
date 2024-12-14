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

Mac OS 바이너리는 일반적으로 **유니버설 바이너리**로 컴파일됩니다. **유니버설 바이너리**는 **같은 파일에서 여러 아키텍처를 지원할 수 있습니다**.

이 바이너리는 기본적으로 **Mach-O 구조**를 따르며, 이는 다음으로 구성됩니다:

* 헤더
* 로드 명령
* 데이터

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (470).png>)

## Fat Header

다음 명령어로 파일을 검색합니다: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC 또는 FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* 뒤따르는 구조체의 수 */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* CPU 지정자 (int) */
cpu_subtype_t	cpusubtype;	/* 머신 지정자 (int) */
uint32_t	offset;		/* 이 객체 파일에 대한 파일 오프셋 */
uint32_t	size;		/* 이 객체 파일의 크기 */
uint32_t	align;		/* 2의 거듭제곱으로 정렬 */
};
</code></pre>

헤더에는 **매직** 바이트가 있으며, 그 뒤에 파일이 **포함하는** **아키텍처의 수**(`nfat_arch`)가 있고, 각 아키텍처는 `fat_arch` 구조체를 가집니다.

다음 명령어로 확인합니다:

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

또는 [Mach-O View](https://sourceforge.net/projects/machoview/) 도구를 사용하여:

<figure><img src="../../../.gitbook/assets/image (1094).png" alt=""><figcaption></figcaption></figure>

당신이 생각할 수 있듯이, 일반적으로 2개의 아키텍처를 위해 컴파일된 유니버설 바이너리는 1개의 아키텍처를 위해 컴파일된 것의 **크기를 두 배로 늘립니다**.

## **Mach-O Header**

헤더는 파일에 대한 기본 정보를 포함하며, Mach-O 파일로 식별하기 위한 매직 바이트와 대상 아키텍처에 대한 정보를 포함합니다. 다음 명령어로 찾을 수 있습니다: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
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
### Mach-O 파일 유형

다양한 파일 유형이 있으며, [**소스 코드에서 예를 확인할 수 있습니다**](https://opensource.apple.com/source/xnu/xnu-2050.18.24/EXTERNAL\_HEADERS/mach-o/loader.h). 가장 중요한 것들은 다음과 같습니다:

* `MH_OBJECT`: 재배치 가능한 오브젝트 파일 (컴파일의 중간 산출물, 아직 실행 파일이 아님).
* `MH_EXECUTE`: 실행 파일.
* `MH_FVMLIB`: 고정 VM 라이브러리 파일.
* `MH_CORE`: 코드 덤프
* `MH_PRELOAD`: 미리 로드된 실행 파일 (XNU에서 더 이상 지원되지 않음)
* `MH_DYLIB`: 동적 라이브러리
* `MH_DYLINKER`: 동적 링커
* `MH_BUNDLE`: "플러그인 파일". gcc에서 -bundle을 사용하여 생성되며 `NSBundle` 또는 `dlopen`에 의해 명시적으로 로드됨.
* `MH_DYSM`: 동반 `.dSym` 파일 (디버깅을 위한 기호가 포함된 파일).
* `MH_KEXT_BUNDLE`: 커널 확장.
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
Or using [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

## **Mach-O 플래그**

소스 코드는 라이브러리 로드에 유용한 여러 플래그를 정의합니다:

* `MH_NOUNDEFS`: 정의되지 않은 참조 없음 (완전히 링크됨)
* `MH_DYLDLINK`: Dyld 링크
* `MH_PREBOUND`: 동적 참조가 미리 바인딩됨.
* `MH_SPLIT_SEGS`: 파일이 r/o 및 r/w 세그먼트를 분할합니다.
* `MH_WEAK_DEFINES`: 바이너리에 약한 정의 기호가 있음
* `MH_BINDS_TO_WEAK`: 바이너리가 약한 기호를 사용함
* `MH_ALLOW_STACK_EXECUTION`: 스택을 실행 가능하게 만듭니다
* `MH_NO_REEXPORTED_DYLIBS`: 라이브러리에 LC\_REEXPORT 명령 없음
* `MH_PIE`: 위치 독립 실행 파일
* `MH_HAS_TLV_DESCRIPTORS`: 스레드 로컬 변수가 있는 섹션이 있음
* `MH_NO_HEAP_EXECUTION`: 힙/데이터 페이지에 대한 실행 없음
* `MH_HAS_OBJC`: 바이너리에 oBject-C 섹션이 있음
* `MH_SIM_SUPPORT`: 시뮬레이터 지원
* `MH_DYLIB_IN_CACHE`: 공유 라이브러리 캐시의 dylibs/frameworks에서 사용됨.

## **Mach-O 로드 명령**

**메모리에서의 파일 레이아웃**은 여기에서 지정되며, **기호 테이블의 위치**, 실행 시작 시 메인 스레드의 컨텍스트, 그리고 필요한 **공유 라이브러리**에 대한 세부 정보가 포함됩니다. 동적 로더 **(dyld)**에 바이너리의 메모리 로드 프로세스에 대한 지침이 제공됩니다.

이는 언급된 **`loader.h`**에서 정의된 **load\_command** 구조를 사용합니다:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
There are about **50 different types of load commands** that the system handles differently. The most common ones are: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB`, and `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
기본적으로, 이 유형의 Load Command는 이진 파일이 실행될 때 **데이터 섹션에 표시된 오프셋**에 따라 **\_\_TEXT** (실행 코드) **및 \_\_DATA** (프로세스에 대한 데이터) **세그먼트를 로드하는 방법**을 정의합니다.
{% endhint %}

이 명령은 이진 파일이 실행될 때 프로세스의 **가상 메모리 공간**에 **매핑**되는 **세그먼트**를 **정의**합니다.

**\_\_TEXT** 세그먼트와 같이 프로그램의 실행 코드를 포함하는 **다양한 유형**의 세그먼트가 있으며, **\_\_DATA** 세그먼트는 프로세스에서 사용하는 데이터를 포함합니다. 이러한 **세그먼트는 Mach-O 파일의 데이터 섹션에 위치**합니다.

**각 세그먼트**는 여러 **섹션**으로 추가 **구분**될 수 있습니다. **로드 명령 구조**는 해당 세그먼트 내의 **이 섹션들에 대한 정보**를 포함합니다.

헤더에서는 먼저 **세그먼트 헤더**를 찾습니다:

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

**섹션 오프셋**(0x37DC) + **아키텍처 시작 오프셋**을 추가하면, 이 경우 `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (701).png" alt=""><figcaption></figcaption></figure>

**명령줄**에서 **헤더 정보**를 얻는 것도 가능합니다:
```bash
otool -lv /bin/ls
```
Common segments loaded by this cmd:

* **`__PAGEZERO`:** 커널에 **주소 0**을 **매핑**하라고 지시하여 **읽거나, 쓸 수 없고, 실행할 수 없도록** 합니다. 구조체의 maxprot 및 minprot 변수는 이 페이지에 **읽기-쓰기-실행 권한이 없음**을 나타내기 위해 0으로 설정됩니다.
* 이 할당은 **NULL 포인터 역참조 취약점**을 **완화**하는 데 중요합니다. 이는 XNU가 첫 번째 페이지(오직 첫 번째) 메모리가 접근할 수 없도록 보장하는 하드 페이지 제로를 시행하기 때문입니다(단, i386 제외). 바이너리는 첫 4k를 커버하기 위해 작은 \_\_PAGEZERO를 제작하고 나머지 32비트 메모리를 사용자 및 커널 모드에서 접근 가능하게 하여 이 요구 사항을 충족할 수 있습니다.
* **`__TEXT`**: **읽기** 및 **실행** 권한이 있는 **실행 가능한** **코드**를 포함합니다(쓰기 불가)**.** 이 세그먼트의 일반적인 섹션:
* `__text`: 컴파일된 바이너리 코드
* `__const`: 상수 데이터(읽기 전용)
* `__[c/u/os_log]string`: C, 유니코드 또는 os 로그 문자열 상수
* `__stubs` 및 `__stubs_helper`: 동적 라이브러리 로딩 과정에서 관련됨
* `__unwind_info`: 스택 언와인드 데이터.
* 이 모든 콘텐츠는 서명되지만 실행 가능으로 표시되어 있습니다(이 권한이 반드시 필요하지 않은 섹션, 예를 들어 문자열 전용 섹션의 악용 가능성을 높임).
* **`__DATA`**: **읽기 가능**하고 **쓰기 가능**한 데이터를 포함합니다(실행 불가)**.**
* `__got:` 전역 오프셋 테이블
* `__nl_symbol_ptr`: 비게으른(로드 시 바인딩) 심볼 포인터
* `__la_symbol_ptr`: 게으른(사용 시 바인딩) 심볼 포인터
* `__const`: 읽기 전용 데이터여야 함(실제로는 아님)
* `__cfstring`: CoreFoundation 문자열
* `__data`: 초기화된 전역 변수
* `__bss`: 초기화되지 않은 정적 변수
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist 등): Objective-C 런타임에서 사용하는 정보
* **`__DATA_CONST`**: \_\_DATA.\_\_const는 상수(쓰기 권한이 있음)로 보장되지 않으며, 다른 포인터와 GOT도 마찬가지입니다. 이 섹션은 `mprotect`를 사용하여 `__const`, 일부 초기화기 및 GOT 테이블(해결된 후)을 **읽기 전용**으로 만듭니다.
* **`__LINKEDIT`**: 심볼, 문자열 및 재배치 테이블 항목과 같은 링커(dyld)에 대한 정보를 포함합니다. `__TEXT` 또는 `__DATA`에 없는 콘텐츠를 위한 일반적인 컨테이너이며, 그 내용은 다른 로드 명령에서 설명됩니다.
* dyld 정보: 재배치, 비게으른/게으른/약한 바인딩 opcode 및 내보내기 정보
* 함수 시작: 함수의 시작 주소 테이블
* 코드 내 데이터: \_\_text의 데이터 섬
* 심볼 테이블: 바이너리의 심볼
* 간접 심볼 테이블: 포인터/스텁 심볼
* 문자열 테이블
* 코드 서명
* **`__OBJC`**: Objective-C 런타임에서 사용하는 정보를 포함합니다. 이 정보는 \_\_DATA 세그먼트의 다양한 \_\_objc\_\* 섹션에서도 발견될 수 있습니다.
* **`__RESTRICT`**: 내용이 없는 세그먼트로, **`__restrict`**라는 단일 섹션(비어 있음)을 포함하여 바이너리를 실행할 때 DYLD 환경 변수를 무시하도록 보장합니다.

코드에서 볼 수 있듯이, **세그먼트는 플래그도 지원합니다**(비록 많이 사용되지는 않지만):

* `SG_HIGHVM`: 코어 전용(사용되지 않음)
* `SG_FVMLIB`: 사용되지 않음
* `SG_NORELOC`: 세그먼트에 재배치가 없음
* `SG_PROTECTED_VERSION_1`: 암호화. 예를 들어 Finder가 `__TEXT` 세그먼트를 암호화하는 데 사용됩니다.

### **`LC_UNIXTHREAD/LC_MAIN`**

**`LC_MAIN`**은 **entryoff 속성**에 있는 진입점을 포함합니다. 로드 시, **dyld**는 이 값을 (메모리 내) **바이너리의 기본 주소**에 **더한 후**, 이 명령어로 **점프**하여 바이너리 코드의 실행을 시작합니다.

**`LC_UNIXTHREAD`**는 메인 스레드를 시작할 때 레지스터가 가져야 할 값을 포함합니다. 이는 이미 사용 중단되었지만 **`dyld`**는 여전히 사용합니다. 이를 통해 설정된 레지스터의 값을 볼 수 있습니다:
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

Mach-O 파일의 **코드 서명**에 대한 정보를 포함합니다. 이는 **서명 블롭**을 가리키는 **오프셋**만 포함합니다. 일반적으로 파일의 맨 끝에 위치합니다.\
그러나 이 섹션에 대한 일부 정보는 [**이 블로그 게시물**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/)과 이 [**gists**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4)에서 찾을 수 있습니다.

### **`LC_ENCRYPTION_INFO[_64]`**

바이너리 암호화에 대한 지원. 그러나 물론 공격자가 프로세스를 손상시키면 메모리를 암호화되지 않은 상태로 덤프할 수 있습니다.

### **`LC_LOAD_DYLINKER`**

프로세스 주소 공간에 공유 라이브러리를 매핑하는 **동적 링커 실행 파일**의 **경로**를 포함합니다. **값은 항상 `/usr/lib/dyld`로 설정됩니다**. macOS에서는 dylib 매핑이 **커널 모드**가 아닌 **사용자 모드**에서 발생한다는 점에 유의하는 것이 중요합니다.

### **`LC_IDENT`**

구식이지만 패닉 시 덤프를 생성하도록 구성되면 Mach-O 코어 덤프가 생성되고 커널 버전이 `LC_IDENT` 명령에 설정됩니다.

### **`LC_UUID`**

무작위 UUID. XNU가 나머지 프로세스 정보와 함께 캐시하므로 직접적으로 유용하지는 않지만, 크래시 보고서에서 사용할 수 있습니다.

### **`LC_DYLD_ENVIRONMENT`**

프로세스가 실행되기 전에 dyld에 환경 변수를 지정할 수 있게 해줍니다. 이는 프로세스 내에서 임의의 코드를 실행할 수 있게 할 수 있으므로, 이 로드 명령은 `#define SUPPORT_LC_DYLD_ENVIRONMENT`로 빌드된 dyld에서만 사용되며, 로드 경로를 지정하는 `DYLD_..._PATH` 형식의 변수로만 처리를 추가로 제한합니다.

### **`LC_LOAD_DYLIB`**

이 로드 명령은 **동적** **라이브러리** 의존성을 설명하며, **로더**(dyld)에게 **해당 라이브러리를 로드하고 링크하라고 지시합니다**. Mach-O 바이너리가 요구하는 **각 라이브러리**에 대해 `LC_LOAD_DYLIB` 로드 명령이 있습니다.

* 이 로드 명령은 **`dylib_command`** 유형의 구조체입니다(실제 의존 동적 라이브러리를 설명하는 struct dylib을 포함합니다):
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

이 정보를 CLI를 통해서도 얻을 수 있습니다:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Some potential malware related libraries are:

* **DiskArbitration**: USB 드라이브 모니터링
* **AVFoundation:** 오디오 및 비디오 캡처
* **CoreWLAN**: Wifi 스캔.

{% hint style="info" %}
Mach-O 바이너리는 **LC\_MAIN**에 지정된 주소 **이전**에 **실행**될 **하나** 이상의 **생성자**를 포함할 수 있습니다.\
모든 생성자의 오프셋은 **\_\_mod\_init\_func** 섹션의 **\_\_DATA\_CONST** 세그먼트에 저장됩니다.
{% endhint %}

## **Mach-O 데이터**

파일의 핵심은 데이터 영역으로, 로드 명령 영역에 정의된 여러 세그먼트로 구성됩니다. **각 세그먼트 내에는 다양한 데이터 섹션이 포함될 수 있으며**, 각 섹션은 특정 유형에 대한 **코드 또는 데이터**를 보유합니다.

{% hint style="success" %}
데이터는 기본적으로 로드 명령 **LC\_SEGMENTS\_64**에 의해 로드되는 모든 **정보**를 포함하는 부분입니다.
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

이에는 다음이 포함됩니다:

* **함수 테이블:** 프로그램 함수에 대한 정보를 보유합니다.
* **심볼 테이블**: 바이너리에서 사용되는 외부 함수에 대한 정보를 포함합니다.
* 내부 함수, 변수 이름 등도 포함될 수 있습니다.

확인하려면 [**Mach-O View**](https://sourceforge.net/projects/machoview/) 도구를 사용할 수 있습니다:

<figure><img src="../../../.gitbook/assets/image (1120).png" alt=""><figcaption></figcaption></figure>

또는 CLI에서:
```bash
size -m /bin/ls
```
## Objetive-C 일반 섹션

In `__TEXT` segment (r-x):

* `__objc_classname`: 클래스 이름 (문자열)
* `__objc_methname`: 메서드 이름 (문자열)
* `__objc_methtype`: 메서드 유형 (문자열)

In `__DATA` segment (rw-):

* `__objc_classlist`: 모든 Objective-C 클래스에 대한 포인터
* `__objc_nlclslist`: 비지연 Objective-C 클래스에 대한 포인터
* `__objc_catlist`: 카테고리에 대한 포인터
* `__objc_nlcatlist`: 비지연 카테고리에 대한 포인터
* `__objc_protolist`: 프로토콜 목록
* `__objc_const`: 상수 데이터
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
