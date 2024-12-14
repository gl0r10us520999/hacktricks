# macOS Launch/Environment Constraints & Trust Cache

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

## 基本情報

macOSの起動制約は、**プロセスがどのように、誰が、どこから開始できるかを規制することによって**セキュリティを強化するために導入されました。macOS Venturaで開始され、**各システムバイナリを異なる制約カテゴリに分類するフレームワーク**を提供します。これらは、システムバイナリとそのハッシュを含むリストである**信頼キャッシュ**内に定義されています。これらの制約は、システム内のすべての実行可能バイナリに拡張され、**特定のバイナリを起動するための要件を定義する一連の**ルール**を含みます。ルールには、バイナリが満たすべき自己制約、親プロセスが満たすべき親制約、関連する他のエンティティが遵守すべき責任制約が含まれます​。

このメカニズムは、macOS Sonoma以降、**環境制約**を通じてサードパーティアプリにも拡張され、開発者が**環境制約のためのキーと値のセットを指定することによってアプリを保護できるようにします。**

**起動環境およびライブラリ制約**は、**`launchd`プロパティリストファイル**に保存するか、コード署名で使用する**別のプロパティリスト**ファイルに保存する制約辞書で定義します。

制約には4つのタイプがあります：

* **自己制約**：**実行中の**バイナリに適用される制約。
* **親プロセス**：**プロセスの親に**適用される制約（例えば、**`launchd`**がXPサービスを実行している場合）。
* **責任制約**：XPC通信で**サービスを呼び出すプロセスに**適用される制約。
* **ライブラリロード制約**：ロードできるコードを選択的に記述するためにライブラリロード制約を使用します。

したがって、プロセスが別のプロセスを起動しようとするとき — `execve(_:_:_:)`または`posix_spawn(_:_:_:_:_:_:)`を呼び出すことによって — オペレーティングシステムは、**実行可能**ファイルが**自身の自己制約を満たしているかどうかを確認します**。また、**親**プロセスの実行可能ファイルが**実行可能ファイの親制約を満たしているかどうか**、および**責任**プロセスの実行可能ファイルが**実行可能ファイルの責任プロセス制約を満たしているかどうか**も確認します。これらの起動制約のいずれかが満たされない場合、オペレーティングシステムはプログラムを実行しません。

ライブラリをロードする際に**ライブラリ制約の一部が真でない場合**、プロセスは**ライブラリをロードしません**。

## LCカテゴリ

LCは、**事実**と**論理演算**（and、or..）で構成され、事実を組み合わせます。

[**LCが使用できる事実は文書化されています**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints)。例えば：

* is-init-proc：実行可能ファイルがオペレーティングシステムの初期化プロセス（`launchd`）である必要があるかどうかを示すブール値。
* is-sip-protected：実行可能ファイルがシステム整合性保護（SIP）によって保護されているファイルである必要があるかどうかを示すブール値。
* `on-authorized-authapfs-volume:` オペレーティングシステムが認可された、認証されたAPFSボリュームから実行可能ファイルをロードしたかどうかを示すブール値。
* `on-authorized-authapfs-volume`：オペレーティングシステムが認可された、認証されたAPFSボリュームから実行可能ファイルをロードしたかどうかを示すブール値。
* Cryptexesボリューム
* `on-system-volume:` オペレーティングシステムが現在起動しているシステムボリュームから実行可能ファイルをロードしたかどうかを示すブール値。
* /System内...
* ...

Appleのバイナリが署名されると、それは**信頼キャッシュ内のLCカテゴリに割り当てられます**。

* **iOS 16 LCカテゴリ**は[**ここで逆引きされ、文書化されています**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056)。
* 現在の**LCカテゴリ（macOS 14 - Sonoma）**は逆引きされ、その[**説明はここにあります**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53)。

例えば、カテゴリ1は：
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: システムまたはCryptexesボリューム内である必要があります。
* `launch-type == 1`: システムサービスである必要があります（LaunchDaemons内のplist）。
* `validation-category == 1`: オペレーティングシステムの実行可能ファイル。
* `is-init-proc`: Launchd

### LCカテゴリの逆アセンブル

詳細については[**こちらにあります**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints)が、基本的には、これらは**AMFI (AppleMobileFileIntegrity)**で定義されているため、**KEXT**を取得するにはカーネル開発キットをダウンロードする必要があります。**`kConstraintCategory`**で始まるシンボルが**興味深い**ものです。これらを抽出すると、DER (ASN.1) エンコードされたストリームが得られ、[ASN.1 Decoder](https://holtstrom.com/michael/tools/asn1decoder.php)またはpython-asn1ライブラリとその`dump.py`スクリプト、[andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master)を使用してデコードする必要があります。これにより、より理解しやすい文字列が得られます。

## 環境制約

これらは**サードパーティアプリケーション**で設定されたLaunch Constraintsです。開発者は、アプリケーション内で使用する**事実**と**論理演算子**を選択して、自身へのアクセスを制限できます。

アプリケーションの環境制約を列挙することが可能です：
```bash
codesign -d -vvvv app.app
```
## 信頼キャッシュ

**macOS** にはいくつかの信頼キャッシュがあります：

* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
* **`/System/Library/Security/OSLaunchPolicyData`**

iOS では **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`** にあるようです。

{% hint style="warning" %}
Apple Silicon デバイス上の macOS では、Apple 署名のバイナリが信頼キャッシュにない場合、AMFI はそれを読み込むことを拒否します。
{% endhint %}

### 信頼キャッシュの列挙

前述の信頼キャッシュファイルは **IMG4** および **IM4P** 形式であり、IM4P は IMG4 形式のペイロードセクションです。

データベースのペイロードを抽出するには、[**pyimg4**](https://github.com/m1stadev/PyIMG4) を使用できます：

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
{% endcode %}

（別のオプションは、ツール [**img4tool**](https://github.com/tihmstar/img4tool) を使用することで、リリースが古くてもM1で実行でき、適切な場所にインストールすればx86\_64でも実行できます）。

今、ツール [**trustcache**](https://github.com/CRKatri/trustcache) を使用して、情報を読みやすい形式で取得できます：
```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```
信頼キャッシュは以下の構造に従いますので、**LCカテゴリは4番目の列です**
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
Then, you could use a script such as [**this one**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) to extract data.

From that data you can check the Apps with a **launch constraints value of `0`** , which are the ones that aren't constrained ([**check here**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) for what each value is).

## 攻撃の緩和策

Launch Constrainsは、**プロセスが予期しない条件で実行されないことを確認することによって、いくつかの古い攻撃を緩和しました：** たとえば、予期しない場所からの実行や、予期しない親プロセスによって呼び出されること（launchdのみが起動するべき場合）です。

さらに、Launch Constraintsは**ダウングレード攻撃も緩和します。**

しかし、彼らは**一般的なXPC**の悪用、**Electron**コードの注入、または**ライブラリ検証なしのdylib注入**を緩和しません（ライブラリをロードできるチームIDが知られていない限り）。

### XPCデーモン保護

Sonomaリリースでは、デーモンXPCサービスの**責任構成**が注目されるポイントです。XPCサービスは自分自身に対して責任を持ち、接続クライアントが責任を持つのではありません。これはフィードバックレポートFB13206884に文書化されています。この設定は欠陥があるように見えるかもしれませんが、特定のXPCサービスとの相互作用を許可します：

- **XPCサービスの起動**：バグと見なされる場合、この設定は攻撃者のコードを通じてXPCサービスを起動することを許可しません。
- **アクティブサービスへの接続**：XPCサービスがすでに実行中である場合（元のアプリケーションによって起動された可能性があります）、接続するための障壁はありません。

XPCサービスに制約を実装することは、**潜在的な攻撃のウィンドウを狭めることによって有益かもしれませんが、**主な懸念には対処していません。XPCサービスのセキュリティを確保するためには、**接続クライアントを効果的に検証することが根本的に必要です。** これがサービスのセキュリティを強化する唯一の方法です。また、言及された責任構成は現在機能していることに注意する価値がありますが、意図された設計とは一致しないかもしれません。

### Electron保護

アプリケーションが**LaunchServiceによって開かれる必要がある**場合（親の制約内）。これは、**`open`**を使用することで達成できます（環境変数を設定できます）または**Launch Services API**を使用することで達成できます（環境変数を指定できます）。

## 参考文献

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

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
