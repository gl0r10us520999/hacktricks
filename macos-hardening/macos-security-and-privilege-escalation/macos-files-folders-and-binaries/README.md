# macOS 파일, 폴더, 바이너리 및 메모리

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **해킹 트릭을 공유하려면 [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 리포지토리에 PR을 제출하세요.**

</details>
{% endhint %}

## 파일 계층 구조

* **/Applications**: 설치된 앱이 여기에 있어야 합니다. 모든 사용자가 접근할 수 있습니다.
* **/bin**: 명령줄 바이너리
* **/cores**: 존재하는 경우, 코어 덤프를 저장하는 데 사용됩니다.
* **/dev**: 모든 것이 파일로 취급되므로 하드웨어 장치가 여기에 저장될 수 있습니다.
* **/etc**: 구성 파일
* **/Library**: 기본 설정, 캐시 및 로그와 관련된 많은 하위 디렉토리와 파일이 여기에 있습니다. 루트와 각 사용자 디렉토리에 Library 폴더가 존재합니다.
* **/private**: 문서화되지 않았지만 언급된 많은 폴더는 개인 디렉토리에 대한 심볼릭 링크입니다.
* **/sbin**: 필수 시스템 바이너리(관리와 관련됨)
* **/System**: OS X을 실행하기 위한 파일입니다. 여기에는 주로 Apple 특정 파일만 있어야 합니다(서드파티 아님).
* **/tmp**: 파일은 3일 후에 삭제됩니다(이는 /private/tmp에 대한 소프트 링크입니다).
* **/Users**: 사용자의 홈 디렉토리입니다.
* **/usr**: 구성 및 시스템 바이너리
* **/var**: 로그 파일
* **/Volumes**: 마운트된 드라이브가 여기에 나타납니다.
* **/.vol**: `stat a.txt`를 실행하면 `16777223 7545753 -rw-r--r-- 1 username wheel ...`와 같은 결과를 얻습니다. 여기서 첫 번째 숫자는 파일이 존재하는 볼륨의 ID 번호이고 두 번째는 inode 번호입니다. 이 정보를 사용하여 `cat /.vol/16777223/7545753`를 실행하여 이 파일의 내용을 접근할 수 있습니다.

### 애플리케이션 폴더

* **시스템 애플리케이션**은 `/System/Applications`에 위치합니다.
* **설치된** 애플리케이션은 일반적으로 `/Applications` 또는 `~/Applications`에 설치됩니다.
* **애플리케이션 데이터**는 루트로 실행되는 애플리케이션의 경우 `/Library/Application Support`에, 사용자로 실행되는 애플리케이션의 경우 `~/Library/Application Support`에 있습니다.
* 서드파티 애플리케이션 **데몬**은 **루트로 실행해야 하는** 경우 일반적으로 `/Library/PrivilegedHelperTools/`에 위치합니다.
* **샌드박스** 앱은 `~/Library/Containers` 폴더에 매핑됩니다. 각 앱은 애플리케이션의 번들 ID(`com.apple.Safari`)에 따라 이름이 지정된 폴더를 가집니다.
* **커널**은 `/System/Library/Kernels/kernel`에 위치합니다.
* **Apple의 커널 확장**은 `/System/Library/Extensions`에 위치합니다.
* **서드파티 커널 확장**은 `/Library/Extensions`에 저장됩니다.

### 민감한 정보가 포함된 파일

MacOS는 비밀번호와 같은 정보를 여러 장소에 저장합니다:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### 취약한 pkg 설치 프로그램

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X 특정 확장

* **`.dmg`**: Apple 디스크 이미지 파일은 설치 프로그램에 매우 자주 사용됩니다.
* **`.kext`**: 특정 구조를 따라야 하며 OS X 버전의 드라이버입니다. (번들입니다)
* **`.plist`**: 속성 목록으로 알려져 있으며 XML 또는 이진 형식으로 정보를 저장합니다.
* XML 또는 이진일 수 있습니다. 이진 파일은 다음과 같이 읽을 수 있습니다:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: 디렉토리 구조를 따르는 Apple 애플리케이션입니다(번들입니다).
* **`.dylib`**: 동적 라이브러리(Windows DLL 파일과 유사)
* **`.pkg`**: xar(확장 가능한 아카이브 형식)와 동일합니다. 설치 명령을 사용하여 이러한 파일의 내용을 설치할 수 있습니다.
* **`.DS_Store`**: 이 파일은 각 디렉토리에 있으며 디렉토리의 속성과 사용자 정의를 저장합니다.
* **`.Spotlight-V100`**: 이 폴더는 시스템의 모든 볼륨의 루트 디렉토리에 나타납니다.
* **`.metadata_never_index`**: 이 파일이 볼륨의 루트에 있으면 Spotlight는 해당 볼륨을 인덱싱하지 않습니다.
* **`.noindex`**: 이 확장을 가진 파일과 폴더는 Spotlight에 의해 인덱싱되지 않습니다.
* **`.sdef`**: 번들 내의 파일로, AppleScript에서 애플리케이션과 상호작용하는 방법을 지정합니다.

### macOS 번들

번들은 **Finder에서 객체처럼 보이는** **디렉토리**입니다(번들의 예는 `*.app` 파일입니다).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld 공유 라이브러리 캐시(SLC)

macOS(및 iOS)에서 모든 시스템 공유 라이브러리, 프레임워크 및 dylibs는 **단일 파일**로 **결합되어** 있으며, 이를 **dyld 공유 캐시**라고 합니다. 이는 코드가 더 빠르게 로드될 수 있으므로 성능이 향상됩니다.

macOS에서는 `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/`에 위치하며, 이전 버전에서는 **`/System/Library/dyld/`**에서 **공유 캐시**를 찾을 수 있습니다.\
iOS에서는 **`/System/Library/Caches/com.apple.dyld/`**에서 찾을 수 있습니다.

dyld 공유 캐시와 유사하게, 커널 및 커널 확장도 커널 캐시에 컴파일되어 부팅 시 로드됩니다.

단일 파일 dylib 공유 캐시에서 라이브러리를 추출하기 위해 [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) 바이너리를 사용할 수 있었으나 현재는 작동하지 않을 수 있으며, [**dyldextractor**](https://github.com/arandomdev/dyldextractor)를 사용할 수도 있습니다:

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
`dyld_shared_cache_util` 도구가 작동하지 않더라도 **공유 dyld 바이너리를 Hopper에 전달하면** Hopper가 모든 라이브러리를 식별하고 **조사할 라이브러리를 선택할 수 있게** 해줍니다:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

일부 추출기는 dylibs가 하드 코딩된 주소로 미리 링크되어 있기 때문에 작동하지 않을 수 있으며, 이로 인해 알 수 없는 주소로 점프할 수 있습니다.

{% hint style="success" %}
Xcode의 에뮬레이터를 사용하여 macOS에서 다른 \*OS 장치의 공유 라이브러리 캐시를 다운로드하는 것도 가능합니다. 이들은 다음 경로에 다운로드됩니다: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, 예: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### SLC 매핑

**`dyld`**는 SLC가 매핑되었는지 확인하기 위해 시스템 호출 **`shared_region_check_np`**를 사용하고 (주소를 반환함) **`shared_region_map_and_slide_np`**를 사용하여 SLC를 매핑합니다.

SLC가 첫 번째 사용 시 슬라이드되더라도 모든 **프로세스**는 **같은 복사본**을 사용하므로, 공격자가 시스템에서 프로세스를 실행할 수 있다면 **ASLR** 보호가 제거됩니다. 이는 과거에 실제로 악용되었으며 공유 영역 페이저로 수정되었습니다.

브랜치 풀은 이미지 매핑 사이에 작은 공간을 만들어 함수의 개입을 불가능하게 하는 작은 Mach-O dylibs입니다.

### SLC 재정의

환경 변수를 사용하여:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> 이는 새로운 공유 라이브러리 캐시를 로드할 수 있게 해줍니다.
* **`DYLD_SHARED_CACHE_DIR=avoid`** 및 실제 라이브러리로 공유 캐시의 심볼릭 링크로 라이브러리를 수동으로 교체합니다 (추출해야 함).

## 특별 파일 권한

### 폴더 권한

**폴더**에서 **읽기**는 **목록을 나열할 수 있게** 하고, **쓰기**는 **파일을 삭제하고 작성할 수 있게** 하며, **실행**은 **디렉토리를 탐색할 수 있게** 합니다. 예를 들어, **실행 권한이 없는 디렉토리** 내의 **파일에 대한 읽기 권한이 있는 사용자**는 **파일을 읽을 수 없습니다**.

### 플래그 수정자

파일에 설정할 수 있는 몇 가지 플래그가 있으며, 이는 파일이 다르게 동작하게 만듭니다. `ls -lO /path/directory`로 디렉토리 내 파일의 **플래그를 확인할 수 있습니다**.

* **`uchg`**: **uchange** 플래그로 알려져 있으며 **파일**의 변경 또는 삭제를 **방지합니다**. 설정하려면: `chflags uchg file.txt`
* 루트 사용자는 **플래그를 제거하고 파일을 수정할 수 있습니다**.
* **`restricted`**: 이 플래그는 파일이 **SIP에 의해 보호되도록** 합니다 (이 플래그를 파일에 추가할 수 없습니다).
* **`Sticky bit`**: 스티키 비트가 있는 디렉토리에서는 **오직** **디렉토리 소유자 또는 루트만 파일을 이름 변경하거나 삭제할 수 있습니다**. 일반적으로 이는 /tmp 디렉토리에 설정되어 일반 사용자가 다른 사용자의 파일을 삭제하거나 이동하지 못하도록 합니다.

모든 플래그는 파일 `sys/stat.h`에서 찾을 수 있으며 ( `mdfind stat.h | grep stat.h`를 사용하여 찾을 수 있음) 다음과 같습니다:

* `UF_SETTABLE` 0x0000ffff: 소유자 변경 가능 플래그의 마스크.
* `UF_NODUMP` 0x00000001: 파일 덤프를 하지 않음.
* `UF_IMMUTABLE` 0x00000002: 파일을 변경할 수 없음.
* `UF_APPEND` 0x00000004: 파일에 대한 쓰기는 오직 추가만 가능.
* `UF_OPAQUE` 0x00000008: 디렉토리는 유니온에 대해 불투명함.
* `UF_COMPRESSED` 0x00000020: 파일이 압축됨 (일부 파일 시스템).
* `UF_TRACKED` 0x00000040: 이 설정이 있는 파일에 대한 삭제/이름 변경에 대한 알림 없음.
* `UF_DATAVAULT` 0x00000080: 읽기 및 쓰기에 대한 권한 필요.
* `UF_HIDDEN` 0x00008000: 이 항목이 GUI에 표시되지 않아야 함을 나타냄.
* `SF_SUPPORTED` 0x009f0000: 슈퍼유저 지원 플래그의 마스크.
* `SF_SETTABLE` 0x3fff0000: 슈퍼유저 변경 가능 플래그의 마스크.
* `SF_SYNTHETIC` 0xc0000000: 시스템 읽기 전용 합성 플래그의 마스크.
* `SF_ARCHIVED` 0x00010000: 파일이 아카이브됨.
* `SF_IMMUTABLE` 0x00020000: 파일을 변경할 수 없음.
* `SF_APPEND` 0x00040000: 파일에 대한 쓰기는 오직 추가만 가능.
* `SF_RESTRICTED` 0x00080000: 쓰기에 대한 권한 필요.
* `SF_NOUNLINK` 0x00100000: 항목을 제거, 이름 변경 또는 마운트할 수 없음.
* `SF_FIRMLINK` 0x00800000: 파일이 firmlink임.
* `SF_DATALESS` 0x40000000: 파일이 데이터 없는 객체임.

### **파일 ACLs**

파일 **ACLs**는 **ACE** (Access Control Entries)를 포함하여 서로 다른 사용자에게 더 **세분화된 권한**을 부여할 수 있습니다.

**디렉토리**에 다음 권한을 부여할 수 있습니다: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
그리고 **파일**에 대해서는: `read`, `write`, `append`, `execute`.

파일에 ACL이 포함되어 있으면 권한을 나열할 때 **"+"를 찾을 수 있습니다**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
파일의 **ACL을 읽으려면** 다음을 사용하세요:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
모든 **ACL이 있는 파일**을 찾으려면 (이것은 매우 느립니다):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### 확장 속성

확장 속성은 이름과 원하는 값을 가지며, `ls -@`를 사용하여 볼 수 있고 `xattr` 명령어를 사용하여 조작할 수 있습니다. 일반적인 확장 속성은 다음과 같습니다:

* `com.apple.resourceFork`: 리소스 포크 호환성. `filename/..namedfork/rsrc`로도 볼 수 있음
* `com.apple.quarantine`: MacOS: Gatekeeper 격리 메커니즘 (III/6)
* `metadata:*`: MacOS: 다양한 메타데이터, 예: `_backup_excludeItem`, 또는 `kMD*`
* `com.apple.lastuseddate` (#PS): 마지막 파일 사용 날짜
* `com.apple.FinderInfo`: MacOS: Finder 정보 (예: 색상 태그)
* `com.apple.TextEncoding`: ASCII 텍스트 파일의 텍스트 인코딩을 지정
* `com.apple.logd.metadata`: `/var/db/diagnostics`의 파일에서 logd에 의해 사용됨
* `com.apple.genstore.*`: 세대 저장소 (`/.DocumentRevisions-V100` 파일 시스템의 루트에 위치)
* `com.apple.rootless`: MacOS: 시스템 무결성 보호에 의해 파일에 레이블을 붙이는 데 사용됨 (III/10)
* `com.apple.uuidb.boot-uuid`: 고유 UUID로 부팅 에포크의 logd 마킹
* `com.apple.decmpfs`: MacOS: 투명 파일 압축 (II/7)
* `com.apple.cprotect`: \*OS: 파일별 암호화 데이터 (III/11)
* `com.apple.installd.*`: \*OS: installd에 의해 사용되는 메타데이터, 예: `installType`, `uniqueInstallID`

### 리소스 포크 | macOS ADS

이는 **MacOS에서 대체 데이터 스트림을 얻는 방법**입니다. **file/..namedfork/rsrc**에 있는 **com.apple.ResourceFork**라는 확장 속성 안에 내용을 저장할 수 있습니다.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
당신은 **이 확장 속성을 포함하는 모든 파일을 찾을 수 있습니다**: 

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

확장 속성 `com.apple.decmpfs`는 파일이 암호화되어 저장됨을 나타냅니다. `ls -l`은 **크기가 0**인 것으로 보고하며, 압축된 데이터는 이 속성 안에 있습니다. 파일에 접근할 때마다 메모리에서 복호화됩니다.

이 속성은 `ls -lO`로 확인할 수 있으며, 압축된 파일은 `UF_COMPRESSED` 플래그로 태그가 붙습니다. 압축된 파일이 `chflags nocompressed </path/to/file>`로 제거되면, 시스템은 파일이 압축되었다는 것을 알지 못하므로 데이터를 복원하고 접근할 수 없습니다(실제로 비어 있다고 생각할 것입니다).

도구 afscexpand를 사용하여 파일을 강제로 압축 해제할 수 있습니다.

## **유니버설 바이너리 및** Mach-o 형식

Mac OS 바이너리는 일반적으로 **유니버설 바이너리**로 컴파일됩니다. **유니버설 바이너리**는 **같은 파일에서 여러 아키텍처를 지원할 수 있습니다**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS 프로세스 메모리

## macOS 메모리 덤프

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## 리스크 카테고리 파일 Mac OS

디렉토리 `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System`는 **다양한 파일 확장자와 관련된 리스크에 대한 정보가 저장되는 곳**입니다. 이 디렉토리는 파일을 다양한 리스크 수준으로 분류하여 Safari가 다운로드 시 이러한 파일을 처리하는 방식에 영향을 미칩니다. 카테고리는 다음과 같습니다:

* **LSRiskCategorySafe**: 이 카테고리의 파일은 **완전히 안전한** 것으로 간주됩니다. Safari는 다운로드 후 자동으로 이러한 파일을 엽니다.
* **LSRiskCategoryNeutral**: 이 파일은 경고가 없으며 Safari에 의해 **자동으로 열리지 않습니다**.
* **LSRiskCategoryUnsafeExecutable**: 이 카테고리의 파일은 **경고를 발생시킵니다**, 이는 파일이 애플리케이션임을 나타냅니다. 이는 사용자를 경고하기 위한 보안 조치입니다.
* **LSRiskCategoryMayContainUnsafeExecutable**: 이 카테고리는 실행 파일을 포함할 수 있는 아카이브와 같은 파일을 위한 것입니다. Safari는 모든 내용이 안전하거나 중립적임을 확인할 수 없는 경우 **경고를 발생시킵니다**.

## 로그 파일

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: 다운로드된 파일에 대한 정보, 예를 들어 다운로드된 URL을 포함합니다.
* **`/var/log/system.log`**: OSX 시스템의 주요 로그입니다. com.apple.syslogd.plist는 syslogging의 실행을 담당합니다(비활성화된 경우 `launchctl list`에서 "com.apple.syslogd"를 찾아 확인할 수 있습니다).
* **`/private/var/log/asl/*.asl`**: 흥미로운 정보를 포함할 수 있는 Apple 시스템 로그입니다.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "Finder"를 통해 최근에 접근한 파일과 애플리케이션을 저장합니다.
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: 시스템 시작 시 실행할 항목을 저장합니다.
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility 앱의 로그 파일(드라이브에 대한 정보, USB 포함).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: 무선 액세스 포인트에 대한 데이터입니다.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: 비활성화된 데몬 목록입니다.

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
