# WmicExec

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を通じてゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい** 場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)、当社の独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) コレクションを発見する
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) または [**telegramグループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) をフォローする。
* **ハッキングトリックを共有するために PR を提出して** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) の github リポジトリに。

</details>

## 動作の説明

ユーザー名とパスワードまたはハッシュがわかっているホストでプロセスを開くことができます。WMI を使用してコマンドを実行することで、Wmiexec によって半インタラクティブなシェル体験が提供されます。

**dcomexec.py:** 異なる DCOM エンドポイントを利用し、このスクリプトは wmiexec.py に似た半インタラクティブなシェルを提供します。具体的には、ShellBrowserWindow DCOM オブジェクトを活用しています。現在、MMC20、Application、Shell Windows、Shell Browser Window オブジェクトをサポートしています。（出典: [Hacking Articles](https://www.hackingarticles.in/beginners-guide-to-impacket-tool-kit-part-1/)）

## WMI の基礎

### 名前空間

ディレクトリスタイルの階層構造で構成されており、WMI のトップレベルコンテナは \root で、その下に名前空間として組織された追加のディレクトリがあります。
名前空間をリストするコマンド:
```bash
# Retrieval of Root namespaces
gwmi -namespace "root" -Class "__Namespace" | Select Name

# Enumeration of all namespaces (administrator privileges may be required)
Get-WmiObject -Class "__Namespace" -Namespace "Root" -List -Recurse 2> $null | select __Namespace | sort __Namespace

# Listing of namespaces within "root\cimv2"
Get-WmiObject -Class "__Namespace" -Namespace "root\cimv2" -List -Recurse 2> $null | select __Namespace | sort __Namespace
```
特定の名前空間内のクラスは、次のコマンドを使用してリストアップできます:
```bash
gwmwi -List -Recurse # Defaults to "root\cimv2" if no namespace specified
gwmi -Namespace "root/microsoft" -List -Recurse
```
### **クラス**

WMIクラス名（例：win32\_process）とそれが存在する名前空間を知ることは、WMI操作にとって重要です。
`win32`で始まるクラスをリストするコマンド：
```bash
Get-WmiObject -Recurse -List -class win32* | more # Defaults to "root\cimv2"
gwmi -Namespace "root/microsoft" -List -Recurse -Class "MSFT_MpComput*"
```
クラスの呼び出し:
```bash
# Defaults to "root/cimv2" when namespace isn't specified
Get-WmiObject -Class win32_share
Get-WmiObject -Namespace "root/microsoft/windows/defender" -Class MSFT_MpComputerStatus
```
### メソッド

メソッドは、WMIクラスの1つ以上の実行可能な関数です。
```bash
# Class loading, method listing, and execution
$c = [wmiclass]"win32_share"
$c.methods
# To create a share: $c.Create("c:\share\path","name",0,$null,"My Description")
```

```bash
# Method listing and invocation
Invoke-WmiMethod -Class win32_share -Name Create -ArgumentList @($null, "Description", $null, "Name", $null, "c:\share\path",0)
```
## WMI列挙

### WMIサービスの状態

WMIサービスが稼働しているかどうかを確認するコマンド：
```bash
# WMI service status check
Get-Service Winmgmt

# Via CMD
net start | findstr "Instrumentation"
```
### システムおよびプロセス情報

WMIを介してシステムおよびプロセス情報を収集する：
```bash
Get-WmiObject -ClassName win32_operatingsystem | select * | more
Get-WmiObject win32_process | Select Name, Processid
```
攻撃者にとって、WMIはシステムやドメインに関する機密データを列挙するための強力なツールです。
```bash
wmic computerystem list full /format:list
wmic process list /format:list
wmic ntdomain list /format:list
wmic useraccount list /format:list
wmic group list /format:list
wmic sysaccount list /format:list
```
### **手動リモートWMIクエリ**

特定の情報（例：ローカル管理者またはログオンユーザー）を取得するために、慎重なコマンド構築を行うことで、リモートWMIのクエリが可能です。

リモートマシン上でのローカル管理者のステルス識別やログオンユーザーの特定は、特定のWMIクエリを使用して達成できます。`wmic`は、複数のノードでコマンドを同時に実行するために、テキストファイルから読み取ることもサポートしています。

Empireエージェントを展開するなど、WMIを介してプロセスをリモートで実行するには、以下のコマンド構造が使用され、正常な実行は「0」という戻り値で示されます。
```bash
wmic /node:hostname /user:user path win32_process call create "empire launcher string here"
```
このプロセスは、WMIのリモート実行およびシステム列挙の機能を示し、システム管理およびペネトレーションテストの両方での有用性を強調しています。


## 参考文献
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-3-wmi-and-winrm/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## 自動ツール

* [**SharpLateral**](https://github.com/mertdas/SharpLateral):

{% code overflow="wrap" %}
```bash
SharpLateral redwmi HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe
```
{% endcode %}

<details>

<summary><strong>ゼロからヒーローまでのAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦で**@carlospolopm**をフォローする
* **ハッキングトリックを共有するために、[HackTricks](https://github.com/carlospolop/hacktricks)と[HackTricks Cloud](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出する**
