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


## smss.exe

**セッションマネージャ**。\
セッション0は**csrss.exe**と**wininit.exe**（**OS** **サービス**）を開始し、セッション1は**csrss.exe**と**winlogon.exe**（**ユーザー** **セッション**）を開始します。しかし、プロセスツリーには**子プロセスなしのその** **バイナリ**の**プロセスが1つだけ**表示されるはずです。

また、セッション0および1以外のセッションは、RDPセッションが発生していることを意味する可能性があります。


## csrss.exe

**クライアント/サーバー実行サブシステムプロセス**。\
**プロセス**と**スレッド**を管理し、他のプロセスに**Windows** **API**を提供し、**ドライブレター**をマッピングし、**一時ファイル**を作成し、**シャットダウン** **プロセス**を処理します。

セッション0に1つ、セッション1にもう1つ**実行中**です（プロセスツリーに**2つのプロセス**）。新しいセッションごとにもう1つが作成されます。


## winlogon.exe

**Windowsログオンプロセス**。\
ユーザーの**ログオン**/**ログオフ**を担当します。**logonui.exe**を起動してユーザー名とパスワードを要求し、その後**lsass.exe**を呼び出してそれらを検証します。

次に、**`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`**の**Userinit**キーで指定された**userinit.exe**を起動します。

さらに、前のレジストリには**Shellキー**に**explorer.exe**が含まれている必要があり、そうでない場合は**マルウェアの持続性手法**として悪用される可能性があります。


## wininit.exe

**Windows初期化プロセス**。 \
セッション0で**services.exe**、**lsass.exe**、および**lsm.exe**を起動します。プロセスは1つだけである必要があります。


## userinit.exe

**Userinitログオンアプリケーション**。\
**HKCU**の**ntduser.dat**を読み込み、**ユーザー** **環境**を初期化し、**ログオン** **スクリプト**と**GPO**を実行します。

**explorer.exe**を起動します。


## lsm.exe

**ローカルセッションマネージャ**。\
smss.exeと連携してユーザーセッションを操作します：ログオン/ログオフ、シェルの開始、デスクトップのロック/ロック解除など。

W7以降、lsm.exeはサービス（lsm.dll）に変換されました。

W7ではプロセスは1つだけで、その中にDLLを実行するサービスがあります。


## services.exe

**サービスコントロールマネージャ**。\
**自動起動**として構成された**サービス**と**ドライバ**を**読み込みます**。

これは**svchost.exe**、**dllhost.exe**、**taskhost.exe**、**spoolsv.exe**などの親プロセスです。

サービスは`HKLM\SYSTEM\CurrentControlSet\Services`で定義されており、このプロセスはsc.exeによってクエリ可能なサービス情報のDBをメモリ内に保持します。

**一部の** **サービス**は**独自のプロセス**で実行され、他のサービスは**svchost.exeプロセスを共有**します。

プロセスは1つだけである必要があります。


## lsass.exe

**ローカルセキュリティ権限サブシステム**。\
ユーザーの**認証**を担当し、**セキュリティ** **トークン**を作成します。これは`HKLM\System\CurrentControlSet\Control\Lsa`にある認証パッケージを使用します。

**セキュリティ** **イベント** **ログ**に書き込み、プロセスは1つだけである必要があります。

このプロセスはパスワードをダンプするために高度に攻撃されることを考慮してください。


## svchost.exe

**汎用サービスホストプロセス**。\
複数のDLLサービスを1つの共有プロセスでホストします。

通常、**svchost.exe**は`-k`フラグで起動されます。これにより、レジストリ**HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost**にクエリが送信され、-kで言及された引数を持つキーがあり、同じプロセスで起動するサービスが含まれます。

例えば：`-k UnistackSvcGroup`は次のサービスを起動します：`PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

**フラグ`-s`**が引数と共に使用される場合、svchostはこの引数で指定されたサービスのみを**起動するように求められます**。

`svchost.exe`のプロセスは複数存在します。いずれかが**`-k`フラグを使用していない場合**、それは非常に疑わしいです。**services.exeが親でない場合**も非常に疑わしいです。


## taskhost.exe

このプロセスはDLLから実行されるプロセスのホストとして機能します。また、DLLから実行されるサービスを読み込みます。

W8ではこれをtaskhostex.exeと呼び、W10ではtaskhostw.exeと呼びます。


## explorer.exe

これは**ユーザーのデスクトップ**を担当し、ファイル拡張子を介してファイルを起動するプロセスです。

**ログオンユーザーごとに** **1つだけ**のプロセスが生成される必要があります。

これは**userinit.exe**から実行され、終了する必要があるため、このプロセスの**親**は表示されないはずです。


# 悪意のあるプロセスを捕まえる

* 期待されるパスから実行されていますか？（Windowsバイナリは一時場所から実行されません）
* 奇妙なIPと通信していますか？
* デジタル署名を確認してください（Microsoftのアーティファクトは署名されている必要があります）
* 正しく綴られていますか？
* 期待されるSIDの下で実行されていますか？
* 親プロセスは期待されるものでしょうか（あれば）？
* 子プロセスは期待されるものでしょうか？（cmd.exe、wscript.exe、powershell.exeはありませんか？）


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
