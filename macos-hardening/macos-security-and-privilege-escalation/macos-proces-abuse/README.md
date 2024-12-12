# macOS 프로세스 남용

{% hint style="success" %}
AWS 해킹 배우고 실습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우고 실습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop) 확인하기!
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f)에 가입하거나 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 저장소에 PR을 제출하여 해킹 요령을 공유하세요.

</details>
{% endhint %}

## 프로세스 기본 정보

프로세스는 실행 중인 실행 파일의 인스턴스이지만, 프로세스는 코드를 실행하지 않습니다. 이들은 스레드입니다. 따라서 **프로세스는 메모리, 디스크립터, 포트, 권한을 제공하는 실행 중인 스레드의 컨테이너일 뿐**입니다.

전통적으로 프로세스는 다른 프로세스(단 PID 1 제외) 내에서 **`fork`**를 호출하여 현재 프로세스의 정확한 사본을 생성한 다음 **자식 프로세스**는 일반적으로 새 실행 파일을 로드하고 실행하기 위해 **`execve`**를 호출했습니다. 그런 다음, 이 프로세스를 더 빠르게 만들기 위해 **`vfork`**가 도입되었습니다.\
그런 다음 **`posix_spawn`**이 도입되어 **`vfork`**와 **`execve`**를 하나의 호출로 결합하고 다음과 같은 플래그를 허용했습니다:

* `POSIX_SPAWN_RESETIDS`: 유효 ID를 실제 ID로 재설정
* `POSIX_SPAWN_SETPGROUP`: 프로세스 그룹 소속 설정
* `POSUX_SPAWN_SETSIGDEF`: 시그널 기본 동작 설정
* `POSIX_SPAWN_SETSIGMASK`: 시그널 마스크 설정
* `POSIX_SPAWN_SETEXEC`: 동일한 프로세스에서 실행(`execve`와 유사하지만 더 많은 옵션 제공)
* `POSIX_SPAWN_START_SUSPENDED`: 일시 중지된 상태에서 시작
* `_POSIX_SPAWN_DISABLE_ASLR`: ASLR 없이 시작
* `_POSIX_SPAWN_NANO_ALLOCATOR:` libmalloc의 Nano 할당기 사용
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` 데이터 세그먼트에서 `rwx` 허용
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: exec(2) 시 모든 파일 기술을 기본적으로 닫음
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` ASLR 슬라이드의 상위 비트를 무작위화

또한, `posix_spawn`은 생성된 프로세스의 일부 측면을 제어하는 **`posix_spawnattr`** 배열을 지정하고, 디스크립터의 상태를 수정하는 **`posix_spawn_file_actions`**을 허용합니다.

프로세스가 종료되면 부모 프로세스에게 **리턴 코드를 전송**하며(부모가 종료된 경우 새 부모는 PID 1입니다), `SIGCHLD` 신호를 사용합니다. 부모는 `wait4()` 또는 `waitid()`를 호출하여 이 값을 가져와야 하며, 그렇지 않으면 자식은 좀비 상태로 남아 있어 목록에는 표시되지만 리소스를 소비하지 않습니다.

### PID

PID(프로세스 식별자)는 고유한 프로세스를 식별합니다. XNU에서 **PID**는 **64비트**로 단조롭게 증가하며 **절대로 감싸지 않습니다**(남용을 방지하기 위해).

### 프로세스 그룹, 세션 및 연합

**프로세스**는 그룹에 포함되어 처리를 쉽게 할 수 있습니다. 예를 들어 셸 스크립트의 명령은 동일한 프로세스 그룹에 있으므로 kill을 사용하여 **함께 시그널을 보낼 수 있습니다**.\
또한 **세션에서 프로세스를 그룹화**할 수 있습니다. 프로세스가 세션을 시작하면(`setsid(2)`), 자식 프로세스는 자체 세션을 시작하지 않는 한 세션 내에 설정됩니다.

Coalition은 Darwin에서 프로세스를 그룹화하는 또 다른 방법입니다. 연합에 가입하는 프로세스는 풀 리소스에 액세스할 수 있으며, 레저를 공유하거나 Jetsam을 직면할 수 있습니다. 연합에는 리더, XPC 서비스, 익스텐션이 다른 역할이 있습니다.

### 자격 증명 및 페르소나

각 프로세스는 시스템에서의 권한을 식별하는 **자격 증명**을 보유합니다. 각 프로세스에는 하나의 주 `uid`와 하나의 주 `gid`가 있습니다(여러 그룹에 속할 수도 있음).\
실행 파일에 `setuid/setgid` 비트가 있는 경우 사용자 및 그룹 ID를 변경할 수도 있습니다.\
새로운 uid/gid를 설정하는 여러 함수가 있습니다.

시스템 호출 **`persona`**는 **대체** 자격 증명 세트를 제공합니다. 페르소나를 채택하면 해당 uid, gid 및 그룹 멤버십을 **동시에** 가정합니다. [**소스 코드**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h)에서 해당 구조체를 찾을 수 있습니다.
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## 스레드 기본 정보

1. **POSIX 스레드 (pthreads):** macOS는 C/C++를 위한 표준 스레딩 API 일부인 POSIX 스레드 (`pthreads`)를 지원합니다. macOS에서의 pthreads 구현은 `/usr/lib/system/libsystem_pthread.dylib`에 있으며, 이는 공개적으로 이용 가능한 `libpthread` 프로젝트에서 가져온 것입니다. 이 라이브러리는 스레드를 생성하고 관리하는 데 필요한 함수를 제공합니다.
2. **스레드 생성:** `pthread_create()` 함수는 새로운 스레드를 생성하는 데 사용됩니다. 내부적으로 이 함수는 `bsdthread_create()`를 호출하며, 이는 XNU 커널( macOS가 기반으로 하는 커널)에 특화된 하위 수준 시스템 호출입니다. 이 시스템 호출은 `pthread_attr` (속성)에서 파생된 다양한 플래그를 취하여 스케줄링 정책 및 스택 크기를 포함한 스레드 동작을 지정합니다.
* **기본 스택 크기:** 새로운 스레드의 기본 스택 크기는 512 KB이며, 일반 작업에 충분하지만 필요에 따라 스레드 속성을 통해 조정할 수 있습니다.
3. **스레드 초기화:** `__pthread_init()` 함수는 스레드 설정 중에 중요하며, `env[]` 인수를 활용하여 스택의 위치 및 크기에 대한 세부 정보를 포함할 수 있는 환경 변수를 구문 분석합니다.

#### macOS에서의 스레드 종료

1. **스레드 종료:** 일반적으로 `pthread_exit()`를 호출하여 스레드를 종료합니다. 이 함수를 통해 스레드는 정리를 수행하고 필요한 정리를 하여 다른 조인하는 스레드에게 반환 값을 보낼 수 있습니다.
2. **스레드 정리:** `pthread_exit()`를 호출하면 모든 관련 스레드 구조를 제거하는 `pthread_terminate()` 함수가 호출됩니다. 이는 Mach 스레드 포트(Mach는 XNU 커널의 통신 서브시스템)를 할당 해제하고, 스레드와 관련된 커널 수준 구조를 제거하는 시스템 호출인 `bsdthread_terminate`을 호출합니다.

#### 동기화 메커니즘

공유 리소스에 대한 액세스를 관리하고 경쟁 조건을 피하기 위해 macOS는 여러 동기화 기본 요소를 제공합니다. 이러한 요소는 데이터 무결성과 시스템 안정성을 보장하기 위해 멀티 스레딩 환경에서 중요합니다:

1. **뮤텍스:**
* **일반 뮤텍스 (시그니처: 0x4D555458):** 60바이트(뮤텍스 56바이트 및 시그니처 4바이트)의 메모리 풋프린트를 가진 표준 뮤텍스.
* **빠른 뮤텍스 (시그니처: 0x4d55545A):** 일반 뮤텍스와 유사하지만 빠른 작업을 위해 최적화되어 있으며, 크기도 60바이트입니다.
2. **조건 변수:**
* 특정 조건이 발생할 때까지 대기하는 데 사용되며, 44바이트(40바이트와 4바이트 시그니처) 크기입니다.
* **조건 변수 속성 (시그니처: 0x434e4441):** 조건 변수의 구성 속성으로, 12바이트 크기입니다.
3. **한 번만 변수 (시그니처: 0x4f4e4345):**
* 초기화 코드 조각이 한 번만 실행되도록 보장합니다. 크기는 12바이트입니다.
4. **읽기-쓰기 잠금:**
* 한 번에 여러 리더 또는 하나의 라이터를 허용하여 공유 데이터에 효율적으로 액세스할 수 있습니다.
* **읽기-쓰기 잠금 (시그니처: 0x52574c4b):** 196바이트 크기입니다.
* **읽기-쓰기 잠금 속성 (시그니처: 0x52574c41):** 읽기-쓰기 잠금의 속성으로, 20바이트 크기입니다.

{% hint style="success" %}
이러한 객체들의 마지막 4바이트는 오버플로우를 감지하는 데 사용됩니다.
{% endhint %}

### 스레드 지역 변수 (TLV)

**스레드 지역 변수 (TLV)**는 macOS의 실행 파일 형식인 Mach-O 파일의 맥락에서, 멀티 스레드 응용 프로그램의 **각 스레드**에 특정한 변수를 선언하는 데 사용됩니다. 이를 통해 각 스레드가 변수의 별도 인스턴스를 가지므로 충돌을 피하고 뮤텍스와 같은 명시적 동기화 메커니즘 없이 데이터 무결성을 유지할 수 있습니다.

C 및 관련 언어에서는 **`__thread`** 키워드를 사용하여 스레드 지역 변수를 선언할 수 있습니다. 아래는 예시에서 작동 방식입니다:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
이 코드 스니펫은 `tlv_var`를 스레드 로컬 변수로 정의합니다. 이 코드를 실행하는 각 스레드는 자체 `tlv_var`를 가지며, 한 스레드가 `tlv_var`를 변경해도 다른 스레드의 `tlv_var`에는 영향을 주지 않습니다.

Mach-O 이진 파일에서 스레드 로컬 변수와 관련된 데이터는 특정 섹션으로 구성됩니다:

* **`__DATA.__thread_vars`**: 이 섹션에는 스레드 로컬 변수에 대한 메타데이터가 포함되며, 변수의 유형 및 초기화 상태와 같은 정보가 포함됩니다.
* **`__DATA.__thread_bss`**: 이 섹션은 명시적으로 초기화되지 않은 스레드 로컬 변수에 사용됩니다. 이는 0으로 초기화된 데이터를 위해 할당된 메모리의 일부입니다.

또한 Mach-O는 스레드가 종료될 때 스레드 로컬 변수를 관리하기 위한 **`tlv_atexit`**라는 특정 API를 제공합니다. 이 API를 사용하면 스레드가 종료될 때 스레드 로컬 데이터를 정리하는 특별한 함수인 소멸자를 **등록**할 수 있습니다.

### 스레딩 우선순위

스레드 우선순위를 이해하는 것은 운영 체제가 어떤 스레드를 언제 실행할지 결정하는 방식을 살펴보는 것을 포함합니다. 이 결정은 각 스레드에 할당된 우선순위 수준에 의해 영향을 받습니다. macOS 및 Unix와 유사한 시스템에서는 `nice`, `renice`, 품질 서비스(QoS) 클래스와 같은 개념을 사용하여 이를 처리합니다.

#### Nice 및 Renice

1. **Nice:**
* 프로세스의 `nice` 값은 우선순위에 영향을 주는 숫자입니다. 각 프로세스는 -20(가장 높은 우선순위)에서 19(가장 낮은 우선순위)까지의 nice 값 범위를 가집니다. 프로세스가 생성될 때의 기본 nice 값은 일반적으로 0입니다.
* 낮은 nice 값(더 가까운 -20)은 프로세스를 더 "이기적"으로 만들어 다른 nice 값이 더 높은 프로세스에 비해 더 많은 CPU 시간을 제공합니다.
2. **Renice:**
* `renice`는 이미 실행 중인 프로세스의 nice 값을 변경하는 데 사용되는 명령어입니다. 이를 사용하여 프로세스의 우선순위를 동적으로 조정하여 새 nice 값에 따라 CPU 시간 할당을 증가 또는 감소시킬 수 있습니다.
* 예를 들어, 프로세스가 일시적으로 더 많은 CPU 자원이 필요한 경우 `renice`를 사용하여 nice 값을 낮출 수 있습니다.

#### 품질 서비스(QoS) 클래스

QoS 클래스는 **Grand Central Dispatch (GCD)**와 같은 시스템에서 특히 스레드 우선순위를 처리하는 더 현대적인 방법입니다. QoS 클래스를 사용하면 중요도나 긴급성에 따라 작업을 다른 수준으로 분류할 수 있습니다. macOS는 이러한 QoS 클래스를 기반으로 자동으로 스레드 우선순위를 관리합니다:

1. **사용자 상호작용:**
* 이 클래스는 현재 사용자와 상호작용하거나 즉각적인 결과를 제공하여 사용자 경험을 향상시키기 위해 필요한 작업을 위한 것입니다. 이러한 작업은 인터페이스를 반응적으로 유지하기 위해 가장 높은 우선순위가 부여됩니다(예: 애니메이션 또는 이벤트 처리).
2. **사용자 시작:**
* 사용자가 시작하고 즉각적인 결과를 기대하는 작업으로, 문서를 열거나 계산이 필요한 버튼을 클릭하는 등이 해당됩니다. 이러한 작업은 높은 우선순위를 가지지만 사용자 상호작용보다는 아래에 위치합니다.
3. **유틸리티:**
* 이러한 작업은 오랜 시간이 걸리며 일반적으로 진행률 표시기가 표시됩니다(예: 파일 다운로드, 데이터 가져오기). 사용자 시작 작업보다 우선순위가 낮으며 즉시 완료할 필요가 없습니다.
4. **백그라운드:**
* 이 클래스는 사용자에게 표시되지 않고 백그라운드에서 작동하는 작업을 위한 것입니다. 이러한 작업은 인덱싱, 동기화 또는 백업과 같은 작업일 수 있습니다. 이러한 작업은 가장 낮은 우선순위를 가지며 시스템 성능에 미미한 영향을 미칩니다.

QoS 클래스를 사용하면 개발자가 정확한 우선순위 숫자를 관리할 필요 없이 작업의 성격에 집중하고 시스템이 CPU 자원을 최적화합니다.

또한 스케줄링 정책을 지정하는 다양한 **스레드 스케줄링 정책**이 있습니다. 이는 `thread_policy_[set/get]`을 사용하여 수행할 수 있으며, 경쟁 조건 공격에 유용할 수 있습니다.
### Python Injection

만약 환경 변수 **`PYTHONINSPECT`**가 설정되어 있다면, 파이썬 프로세스는 실행이 완료되면 파이썬 CLI로 전환됩니다. 또한 **`PYTHONSTARTUP`**을 사용하여 대화형 세션의 시작 시에 실행할 파이썬 스크립트를 지정할 수도 있습니다.\
그러나 **`PYTHONINSPECT`**가 대화형 세션을 생성할 때 **`PYTHONSTARTUP`** 스크립트는 실행되지 않음을 유의하십시오.

**`PYTHONPATH`** 및 **`PYTHONHOME`**과 같은 다른 환경 변수도 파이썬 명령이 임의의 코드를 실행하는 데 유용할 수 있습니다.

**`pyinstaller`**로 컴파일된 실행 파일은 내장된 파이썬을 사용하더라도 이러한 환경 변수를 사용하지 않습니다.

{% hint style="danger" %}
전반적으로 환경 변수를 남용하여 파이썬이 임의의 코드를 실행하도록 하는 방법을 찾지 못했습니다.\
그러나 대부분의 사람들은 **Hombrew**를 사용하여 파이썬을 설치하며, 이는 기본 관리자 사용자에 대해 **쓰기 가능한 위치**에 파이썬을 설치합니다. 다음과 같이 이를 탈취할 수 있습니다:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
심지어 **루트**도 파이썬을 실행할 때 이 코드를 실행할 것입니다.
{% endhint %}

## 탐지

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield))은 다음과 같은 프로세스 인젝션 작업을 **탐지하고 차단**할 수 있는 오픈 소스 애플리케이션입니다:

* **환경 변수 사용**: 다음 환경 변수의 존재를 모니터링합니다: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** 및 **`ELECTRON_RUN_AS_NODE`**
* **`task_for_pid`** 호출 사용: 한 프로세스가 다른 프로세스의 **작업 포트를 가져오려고 할 때** 코드를 삽입할 수 있게 합니다.
* **Electron 앱 매개변수**: 누군가는 **`--inspect`**, **`--inspect-brk`** 및 **`--remote-debugging-port`** 커맨드 라인 인수를 사용하여 Electron 앱을 디버깅 모드로 시작하고 이에 코드를 삽입할 수 있습니다.
* **심볼릭 링크** 또는 **하드 링크** 사용: 일반적으로 가장 흔한 남용은 **사용자 권한으로 링크를 만들고**, **보다 높은 권한** 위치를 가리키게 하는 것입니다. 하드 링크와 심볼릭 링크 모두에 대한 감지는 매우 간단합니다. 링크를 생성하는 프로세스가 대상 파일보다 **다른 권한 수준**을 가지고 있으면 **경고**를 생성합니다. 심볼릭 링크의 경우, 링크의 대상에 대한 정보가 생성 전에 없기 때문에 차단이 불가능합니다. 이는 Apple의 EndpointSecuriy 프레임워크의 제한입니다.

### 다른 프로세스에 의한 호출

[**이 블로그 포스트**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html)에서는 **`task_name_for_pid`** 함수를 사용하여 다른 **프로세스가 프로세스에 코드를 삽입하는 것**에 대한 정보를 얻고 그 다른 프로세스에 대한 정보를 얻는 방법에 대해 알 수 있습니다.

해당 함수를 호출하려면 해당 프로세스를 실행하는 사용자와 **동일한 uid**이거나 **루트**여야 합니다 (그리고 이는 프로세스에 대한 정보를 반환하며 코드를 삽입하는 방법은 아닙니다).

## 참고 자료

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

{% hint style="success" %}
AWS 해킹 학습 및 실습:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 학습 및 실습: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* **HackTricks** 및 **HackTricks Cloud** 깃허브 저장소에 PR을 제출하여 해킹 트릭을 **공유**하세요.

</details>
{% endhint %}
