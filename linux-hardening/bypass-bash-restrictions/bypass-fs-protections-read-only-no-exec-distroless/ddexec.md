# DDexec / EverythingExec

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

## Context

Linuxでは、プログラムを実行するには、それがファイルとして存在し、ファイルシステム階層を通じて何らかの方法でアクセス可能でなければなりません（これは`execve()`の動作方法です）。このファイルはディスク上またはRAM（tmpfs、memfd）に存在する可能性がありますが、ファイルパスが必要です。これにより、Linuxシステム上で実行されるものを制御することが非常に簡単になり、脅威や攻撃者のツールを検出したり、彼らが何かを実行しようとするのを防ぐことが容易になります（例：特権のないユーザーが実行可能ファイルをどこにでも配置することを許可しない）。

しかし、この技術はすべてを変えるために存在します。実行したいプロセスを開始できない場合... **既存のプロセスをハイジャックします**。

この技術により、**読み取り専用、noexec、ファイル名のホワイトリスト、ハッシュのホワイトリストなどの一般的な保護技術をバイパスすることができます...**

## Dependencies

最終的なスクリプトは、以下のツールに依存しており、攻撃しているシステムでアクセス可能である必要があります（デフォルトでは、これらはどこにでも存在します）：
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## テクニック

プロセスのメモリを任意に変更できる場合、そのプロセスを乗っ取ることができます。これは、既存のプロセスをハイジャックし、別のプログラムに置き換えるために使用できます。これを達成するには、`ptrace()`システムコールを使用するか（これはシステムでシステムコールを実行する能力が必要です）、またはより興味深い方法として`/proc/$pid/mem`に書き込むことができます。

ファイル`/proc/$pid/mem`は、プロセスのアドレス空間全体の1対1のマッピングです（_例：_ x86-64では`0x0000000000000000`から`0x7ffffffffffff000`まで）。これは、オフセット`x`でこのファイルから読み書きすることは、仮想アドレス`x`の内容を読み取ったり変更したりすることと同じであることを意味します。

さて、私たちは直面するべき4つの基本的な問題があります：

* 一般的に、ファイルの所有者とrootのみがそれを変更できます。
* ASLR。
* プログラムのアドレス空間にマッピングされていないアドレスに読み書きしようとすると、I/Oエラーが発生します。

これらの問題には、完璧ではないものの良い解決策があります：

* ほとんどのシェルインタプリタは、子プロセスによって継承されるファイルディスクリプタの作成を許可します。書き込み権限を持つシェルの`mem`ファイルを指すfdを作成できます... そのfdを使用する子プロセスはシェルのメモリを変更できるようになります。
* ASLRは問題ではありません。シェルの`maps`ファイルやprocfsの他のファイルをチェックすることで、プロセスのアドレス空間に関する情報を得ることができます。
* したがって、ファイルに対して`lseek()`を行う必要があります。シェルからは、悪名高い`dd`を使用しない限り、これを行うことはできません。

### 詳細に

手順は比較的簡単で、理解するために特別な専門知識は必要ありません：

* 実行したいバイナリとローダーを解析して、必要なマッピングを特定します。次に、カーネルが`execve()`を呼び出すたびに行うのと同様の手順を実行する「シェル」コードを作成します：
* そのマッピングを作成します。
* バイナリをそれに読み込みます。
* 権限を設定します。
* 最後に、プログラムの引数でスタックを初期化し、ローダーに必要な補助ベクタを配置します。
* ローダーにジャンプし、残りの処理（プログラムに必要なライブラリの読み込み）を任せます。
* 実行中のシステムコールから戻るアドレスを`syscall`ファイルから取得します。
* その場所を上書きします。これは実行可能であり、私たちのシェルコードで上書きします（`mem`を通じて書き込み不可のページを変更できます）。
* 実行したいプログラムをプロセスのstdinに渡します（その「シェル」コードによって`read()`されます）。
* この時点で、ローダーがプログラムに必要なライブラリを読み込み、それにジャンプするのはローダーの役割です。

**ツールは** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec) **をチェックしてください**

## EverythingExec

`dd`の代替手段はいくつかあり、そのうちの1つである`tail`は、現在`mem`ファイルを`lseek()`するために使用されるデフォルトのプログラムです（これは`dd`を使用する唯一の目的でした）。これらの代替手段は：
```bash
tail
hexdump
cmp
xxd
```
`SEEKER` 変数を設定することで、使用するシーカーを変更できます。_例:_
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
もしスクリプトに実装されていない別の有効なシーカーを見つけた場合は、`SEEKER_ARGS` 変数を設定して使用することができます:
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Block this, EDRs.

## 参考文献
* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
