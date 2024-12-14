# その他のウェブトリック

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}

### Hostヘッダー

バックエンドが**Hostヘッダー**を信頼していくつかのアクションを実行することが何度かあります。例えば、パスワードリセットを送信する**ドメインとしてその値を使用する**ことがあります。したがって、パスワードをリセットするためのリンクを含むメールを受け取ったとき、使用されるドメインはHostヘッダーに入力したものです。その後、他のユーザーのパスワードリセットを要求し、ドメインをあなたが制御するものに変更して、彼らのパスワードリセットコードを盗むことができます。[WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2)。

{% hint style="warning" %}
ユーザーがリセットパスワードリンクをクリックするのを待つ必要がない場合もあることに注意してください。なぜなら、**スパムフィルターや他の中間デバイス/ボットがそれをクリックして分析する可能性があるからです**。
{% endhint %}

### セッションブール値

時々、いくつかの検証を正しく完了すると、バックエンドは**セッションのセキュリティ属性に「True」という値のブール値を追加するだけです**。その後、別のエンドポイントは、そのチェックに成功したかどうかを知ることができます。\
しかし、もしあなたが**チェックに合格し**、セッションがそのセキュリティ属性に「True」値を付与された場合、**同じ属性に依存する他のリソースにアクセスしようとすることができますが、アクセスする権限はないはずです**。[WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a)。

### 登録機能

既存のユーザーとして登録を試みてください。また、同等の文字（ドット、たくさんのスペース、Unicode）を使用してみてください。

### メールの乗っ取り

メールを登録し、確認する前にメールを変更します。次に、新しい確認メールが最初に登録したメールに送信されると、任意のメールを乗っ取ることができます。また、最初のメールを確認するために2番目のメールを有効にできる場合も、任意のアカウントを乗っ取ることができます。

### Atlassianを使用した企業の内部サービスデスクへのアクセス

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACEメソッド

開発者は、プロダクション環境でさまざまなデバッグオプションを無効にするのを忘れることがあります。例えば、HTTP `TRACE`メソッドは診断目的で設計されています。これが有効になっている場合、ウェブサーバーは`TRACE`メソッドを使用したリクエストに対して、受信した正確なリクエストを応答にエコーします。この動作は通常無害ですが、時には内部認証ヘッダーの名前などの情報漏洩につながることがあります。![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}
