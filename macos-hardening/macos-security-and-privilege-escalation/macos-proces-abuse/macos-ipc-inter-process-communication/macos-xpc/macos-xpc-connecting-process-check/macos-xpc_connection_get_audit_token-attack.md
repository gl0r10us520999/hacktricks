# macOS xpc\_connection\_get\_audit\_token 攻撃

{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}

**詳細情報は元の投稿を確認してください:** [**https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/**](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)。これは要約です：

## Machメッセージの基本情報

Machメッセージが何か知らない場合は、このページを確認してください：

{% content-ref url="../../" %}
[..](../../)
{% endcontent-ref %}

現時点では、([ここからの定義](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing)):\
Machメッセージは_machポート_を介して送信され、これは**単一受信者、複数送信者通信**チャネルで、machカーネルに組み込まれています。**複数のプロセスがメッセージを送信できます**が、任意の時点で**単一のプロセスのみがそれを読み取ることができます**。ファイルディスクリプタやソケットと同様に、machポートはカーネルによって割り当てられ、管理され、プロセスは整数を見て、それを使用してカーネルにどのmachポートを使用したいかを示します。

## XPC接続

XPC接続がどのように確立されるか知らない場合は、次を確認してください：

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

## 脆弱性の概要

知っておくべき興味深い点は、**XPCの抽象化は一対一の接続ですが、**複数の送信者を持つ技術の上に構築されているため、以下のようになります：

* Machポートは単一受信者、**複数送信者**です。
* XPC接続の監査トークンは、**最近受信したメッセージからコピーされた監査トークン**です。
* XPC接続の**監査トークン**を取得することは、多くの**セキュリティチェック**にとって重要です。

前述の状況は有望に聞こえますが、これが問題を引き起こさないシナリオもいくつかあります（[ここから](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing)）：

* 監査トークンは、接続を受け入れるかどうかを決定するための認証チェックにしばしば使用されます。これはサービスポートへのメッセージを使用して行われるため、**接続はまだ確立されていません**。このポートへの追加のメッセージは、追加の接続要求として処理されます。したがって、接続を受け入れる前の**チェックは脆弱ではありません**（これは、`-listener:shouldAcceptNewConnection:`内で監査トークンが安全であることも意味します）。したがって、私たちは**特定のアクションを検証するXPC接続を探しています**。
* XPCイベントハンドラは同期的に処理されます。これは、1つのメッセージのイベントハンドラが次のメッセージのために呼び出される前に完了する必要があることを意味し、同時にディスパッチキュー上でも同様です。したがって、**XPCイベントハンドラ内では監査トークンは他の通常の（非応答！）メッセージによって上書きされることはありません**。

この脆弱性を悪用する可能性のある2つの異なる方法：

1. バリアント1:
* **攻撃**はサービス**A**およびサービス**B**に**接続**します。
* サービス**B**は、ユーザーができないサービスAの**特権機能**を呼び出すことができます。
* サービス**A**は、**`dispatch_async`**内の**イベントハンドラ**の中ではなく、**`xpc_connection_get_audit_token`**を呼び出します。
* したがって、**異なる**メッセージが**監査トークンを上書き**する可能性があります。なぜなら、それはイベントハンドラの外で非同期にディスパッチされているからです。
* 攻撃は**サービスBにサービスAへの送信権を渡します**。
* したがって、svc **B**は実際にサービス**A**に**メッセージを送信**します。
* **攻撃**は**特権アクションを呼び出そうとします**。RC svc **A**はこの**アクションの**認証を**チェック**し、**svc Bが監査トークンを上書きした**（攻撃に特権アクションを呼び出すアクセスを与える）状態になります。
2. バリアント2:
* サービス**B**は、ユーザーができないサービスAの**特権機能**を呼び出すことができます。
* 攻撃は**サービスA**に接続し、**応答を期待するメッセージを送信**します。
* 攻撃は**サービス**Bにその**応答ポート**を渡すメッセージを送信します。
* サービス**B**が**応答**すると、**サービスA**にメッセージを**送信**し、**攻撃**は異なる**メッセージをサービスAに送信**して特権機能に到達しようとし、サービスBからの応答が監査トークンを完璧なタイミングで上書きすることを期待します（競合条件）。

## バリアント1: イベントハンドラの外でxpc\_connection\_get\_audit\_tokenを呼び出す <a href="#variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler" id="variant-1-calling-xpc_connection_get_audit_token-outside-of-an-event-handler"></a>

シナリオ：

* 接続できる2つのmachサービス**`A`**と**`B`**（サンドボックスプロファイルと接続を受け入れる前の認証チェックに基づく）。
* _**A**_は、**`B`**が通過できる特定のアクションの**認証チェック**を持っている必要があります（しかし、私たちのアプリはできません）。
* たとえば、Bがいくつかの**権限**を持っているか、**root**として実行されている場合、Aに特権アクションを実行するように要求できるかもしれません。
* この認証チェックのために、**`A`**は非同期的に監査トークンを取得します。たとえば、**`dispatch_async`**から`xpc_connection_get_audit_token`を呼び出すことによって。

{% hint style="danger" %}
この場合、攻撃者は**競合条件**を引き起こし、**Aにアクションを実行するように要求する**攻撃を何度も行うことができます。その間に**Bが`A`にメッセージを送信**します。RCが**成功**すると、**Bの監査トークン**がメモリにコピーされ、**私たちの攻撃**のリクエストがAによって**処理されている間**に特権アクションに**アクセス**できるようになります。
{% endhint %}

これは、**`A`**が`smd`で、**`B`**が`diagnosticd`で発生しました。関数[`SMJobBless`](https://developer.apple.com/documentation/servicemanagement/1431078-smjobbless?language=objc)は、特権ヘルパーツールを新たにインストールするために使用できます（**root**として）。もし**rootとして実行されているプロセスが**`smd`に接触すると、他のチェックは行われません。

したがって、サービス**B**は**`diagnosticd`**であり、**root**として実行され、プロセスを**監視**するために使用できます。監視が開始されると、**毎秒複数のメッセージを送信**します。

攻撃を実行するには：

1. 標準XPCプロトコルを使用して`smd`という名前のサービスに**接続**を開始します。
2. `diagnosticd`に二次的な**接続**を形成します。通常の手順とは異なり、2つの新しいmachポートを作成して送信するのではなく、クライアントポートの送信権が`smd`接続に関連付けられた**送信権**の複製に置き換えられます。
3. その結果、XPCメッセージは`diagnosticd`にディスパッチできますが、`diagnosticd`からの応答は`smd`に再ルーティングされます。`smd`にとっては、ユーザーと`diagnosticd`の両方からのメッセージが同じ接続から発信されているように見えます。

![攻撃プロセスを示す画像](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/exploit.png)

4. 次のステップは、`diagnosticd`に選択したプロセス（ユーザー自身のプロセスの可能性があります）の監視を開始するように指示することです。同時に、`smd`に対してルーチンの1004メッセージの洪水が送信されます。ここでの意図は、特権のあるツールをインストールすることです。
5. このアクションは、`handle_bless`関数内で競合条件を引き起こします。タイミングが重要です：`xpc_connection_get_pid`関数呼び出しは、ユーザーのプロセスのPIDを返す必要があります（特権ツールはユーザーのアプリバンドル内に存在します）。しかし、`xpc_connection_get_audit_token`関数は、特に`connection_is_authorized`サブルーチン内で、`diagnosticd`に属する監査トークンを参照する必要があります。

## バリアント2: 応答の転送

XPC（プロセス間通信）環境では、イベントハンドラは同時に実行されませんが、応答メッセージの処理には独自の動作があります。具体的には、応答を期待するメッセージを送信するための2つの異なる方法があります：

1. **`xpc_connection_send_message_with_reply`**: ここでは、XPCメッセージが指定されたキューで受信され、処理されます。
2. **`xpc_connection_send_message_with_reply_sync`**: 逆に、この方法では、XPCメッセージが現在のディスパッチキューで受信され、処理されます。

この区別は重要です。なぜなら、**応答パケットがXPCイベントハンドラの実行と同時に解析される可能性があるからです**。特に、`_xpc_connection_set_creds`は監査トークンの部分的な上書きを防ぐためにロックを実装していますが、接続オブジェクト全体に対してこの保護を拡張していません。したがって、パケットの解析とそのイベントハンドラの実行の間の間隔で監査トークンが置き換えられる脆弱性が生じます。

この脆弱性を悪用するには、次のセットアップが必要です：

* **`A`**および**`B`**と呼ばれる2つのmachサービスで、どちらも接続を確立できます。
* サービス**`A`**は、**`B`**のみが実行できる特定のアクションのための認証チェックを含む必要があります（ユーザーのアプリケーションはできません）。
* サービス**`A`**は、応答を期待するメッセージを送信する必要があります。
* ユーザーは、**`B`**に応答するメッセージを送信できます。

悪用プロセスは次のステップを含みます：

1. サービス**`A`**が応答を期待するメッセージを送信するのを待ちます。
2. **`A`**に直接応答するのではなく、応答ポートをハイジャックしてサービス**`B`**にメッセージを送信します。
3. 次に、禁止されたアクションに関するメッセージをディスパッチし、**`B`**からの応答と同時に処理されることを期待します。

以下は、説明された攻撃シナリオの視覚的表現です：

!\[https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/variant2.png]\(../../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png)

<figure><img src="../../../../../../.gitbook/assets/image (33).png" alt="https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/variant2.png" width="563"><figcaption></figcaption></figure>

## 発見の問題

* **インスタンスの特定の困難**: `xpc_connection_get_audit_token`の使用例を静的および動的に検索するのは困難でした。
* **方法論**: Fridaを使用して`xpc_connection_get_audit_token`関数をフックし、イベントハンドラから発信されない呼び出しをフィルタリングしました。しかし、この方法はフックされたプロセスに限定され、アクティブな使用が必要でした。
* **分析ツール**: IDA/Ghidraのようなツールを使用して到達可能なmachサービスを調査しましたが、プロセスは時間がかかり、dyld共有キャッシュに関与する呼び出しによって複雑化しました。
* **スクリプトの制限**: `dispatch_async`ブロックからの`xpc_connection_get_audit_token`への呼び出しを分析するためのスクリプトを試みましたが、ブロックの解析とdyld共有キャッシュとの相互作用の複雑さによって妨げられました。

## 修正 <a href="#the-fix" id="the-fix"></a>

* **報告された問題**: `smd`内で見つかった一般的および特定の問題を詳細に記載した報告がAppleに提出されました。
* **Appleの対応**: Appleは、`smd`内の問題に対処し、`xpc_connection_get_audit_token`を`xpc_dictionary_get_audit_token`に置き換えました。
* **修正の性質**: `xpc_dictionary_get_audit_token`関数は、受信したXPCメッセージに関連付けられたmachメッセージから直接監査トークンを取得するため、安全と見なされています。ただし、`xpc_connection_get_audit_token`と同様に、公開APIの一部ではありません。
* **より広範な修正の不在**: Appleが接続の保存された監査トークンと一致しないメッセージを破棄するなど、より包括的な修正を実装しなかった理由は不明です。特定のシナリオ（例：`setuid`の使用）での正当な監査トークンの変更の可能性が要因かもしれません。
* **現在の状況**: この問題はiOS 17およびmacOS 14に残っており、それを特定し理解しようとする人々にとって課題となっています。

{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
