# 機密マウント

{% hint style="success" %}
AWSハッキングの学習と実践：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングトリックを共有するために** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc`および`/sys`の適切な名前空間分離なしでの公開は、攻撃面の拡大や情報漏洩など、重大なセキュリティリスクをもたらします。これらのディレクトリには機密ファイルが含まれており、誤って構成されたり、認証されていないユーザーによってアクセスされたりすると、コンテナの脱出、ホストの変更、またはさらなる攻撃を助ける情報の提供につながる可能性があります。たとえば、`-v /proc:/host/proc`を誤ってマウントすると、パスベースの性質によりAppArmor保護がバイパスされ、`/host/proc`が保護されなくなります。

**各潜在的な脆弱性の詳細は** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**で見つけることができます。**

## procfsの脆弱性

### `/proc/sys`

このディレクトリは通常、`sysctl(2)`を介してカーネル変数を変更するためのアクセスを許可し、いくつかの懸念のあるサブディレクトリを含んでいます。

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html)で説明されています。
* 最初の128バイトを引数として使用してコアファイル生成時に実行するプログラムを定義することができます。ファイルがパイプ `|` で始まる場合、コードの実行につながる可能性があります。
*   **テストと悪用例**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # 書き込みアクセスをテスト
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # カスタムハンドラを設定
sleep 5 && ./crash & # ハンドラをトリガー
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)で詳細に説明されています。
* カーネルモジュールローダーへのパスを含み、カーネルモジュールの読み込み時に呼び出されます。
*   **アクセスの確認例**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobeへのアクセスを確認
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)で言及されています。
* OOM条件が発生したときにカーネルがパニックするかOOMキラーを呼び出すかを制御するグローバルフラグです。

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)によると、ファイルシステムに関するオプションと情報が含まれています。
* 書き込みアクセスを有効にすると、ホストに対するさまざまなサービス拒否攻撃が可能になります。

#### **`/proc/sys/fs/binfmt_misc`**

* マジックナンバーに基づいて非ネイティブバイナリ形式のインタプリタを登録することができます。
* `/proc/sys/fs/binfmt_misc/register` が書き込み可能である場合、特権昇格やルートシェルアクセスにつながる可能性があります。
* 関連する悪用と説明:
* [binfmt\_miscを使用した貧弱なルートキット](https://github.com/toffan/binfmt\_misc)
* 詳細なチュートリアル: [ビデオリンク](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### `/proc`内のその他のファイル

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC` が有効になっている場合、カーネル構成を公開する可能性があります。
* 実行中のカーネルの脆弱性を特定するために攻撃者に役立ちます。

#### **`/proc/sysrq-trigger`**

* Sysrqコマンドを呼び出すことを許可し、即座のシステム再起動やその他の重要なアクションを引き起こす可能性があります。
*   **ホストの再起動例**:

```bash
echo b > /proc/sysrq-trigger # ホストを再起動
```

#### **`/proc/kmsg`**

* カーネルリングバッファメッセージを公開します。
* カーネルの悪用、アドレスリーク、機密システム情報の提供に役立ちます。

#### **`/proc/kallsyms`**

* カーネルがエクスポートしたシンボルとそのアドレスをリストします。
* 特にKASLRを克服するためにカーネル悪用の開発に不可欠です。
* `kptr_restrict` が `1` または `2` に設定されていると、アドレス情報が制限されます。
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)で詳細が説明されています。

#### **`/proc/[pid]/mem`**

* カーネルメモリデバイス `/dev/mem` とのインターフェースです。
* 歴史的に特権昇格攻撃の脆弱性がありました。
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)で詳細が説明されています。

#### **`/proc/kcore`**

* ELFコア形式でシステムの物理メモリを表します。
* 読み取りによりホストシステムや他のコンテナのメモリ内容が漏洩する可能性があります。
* ファイルサイズが大きいと読み取りの問題やソフトウェアのクラッシュが発生する可能性があります。
* 2019年の [Dumping /proc/kcore](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) での詳細な使用法。

#### **`/proc/kmem`**

* カーネル仮想メモリを表す `/dev/kmem` の代替インターフェースです。
* 読み取りと書き込みが可能であり、したがってカーネルメモリの直接的な変更が可能です。

#### **`/proc/mem`**

* 物理メモリを表す `/dev/mem` の代替インターフェースです。
* 読み取りと書き込みが可能であり、すべてのメモリの変更には仮想アドレスから物理アドレスを解決する必要があります。

#### **`/proc/sched_debug`**

* PID名前空間の保護をバイパスしてプロセススケジューリング情報を返します。
* プロセス名、ID、およびcgroup識別子を公開します。

#### **`/proc/[pid]/mountinfo`**

* プロセスのマウント名前空間内のマウントポイントに関する情報を提供します。
* コンテナの `rootfs` またはイメージの場所を公開します。

### `/sys`の脆弱性

#### **`/sys/kernel/uevent_helper`**

* カーネルデバイス `uevents` を処理するために使用されます。
* `/sys/kernel/uevent_helper` に書き込むと、`uevent` トリガー時に任意のスクリプトが実行される可能性があります。
*   **悪用の例**: %%%bash

#### ペイロードを作成

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### コンテナのOverlayFSマウントからホストパスを見つける

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### uevent\_helperを悪意のあるヘルパーに設定

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### ueventをトリガー

echo change > /sys/class/mem/null/uevent

#### 出力を読む

cat /output %%%
#### **`/sys/class/thermal`**

* 温度設定を制御し、DoS攻撃や物理的損傷を引き起こす可能性があります。

#### **`/sys/kernel/vmcoreinfo`**

* カーネルアドレスを漏洩し、KASLRを危険にさらす可能性があります。

#### **`/sys/kernel/security`**

* `securityfs` インターフェースを収容し、AppArmorのようなLinuxセキュリティモジュールの構成を可能にします。
* アクセスすることで、コンテナがMACシステムを無効にする可能性があります。

#### **`/sys/firmware/efi/vars` および `/sys/firmware/efi/efivars`**

* NVRAM内のEFI変数とやり取りするためのインターフェースを公開します。
* 誤構成や悪用により、ブリック化したノートパソコンや起動不能なホストマシンにつながる可能性があります。

#### **`/sys/kernel/debug`**

* `debugfs` はカーネルに対する「ルールのない」デバッグインターフェースを提供します。
* 制限のない性質から、セキュリティ問題の歴史があります。

### 参考文献

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
AWSハッキングの学習と実践:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や [**telegramグループ**](https://t.me/peass)に**参加**または**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}
