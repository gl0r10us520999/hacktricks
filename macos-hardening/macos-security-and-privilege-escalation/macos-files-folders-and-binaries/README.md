# macOS फ़ाइलें, फ़ोल्डर, बाइनरी और मेमोरी

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
{% endhint %}

## फ़ाइल व्यवस्था लेआउट

* **/Applications**: स्थापित ऐप्स यहाँ होने चाहिए। सभी उपयोगकर्ता उन्हें एक्सेस कर सकेंगे।
* **/bin**: कमांड लाइन बाइनरी
* **/cores**: अगर मौजूद है, तो यह कोर डंप स्टोर करने के लिए उपयोग किया जाता है।
* **/dev**: सब कुछ एक फ़ाइल के रूप में देखा जाता है, इसलिए आपको हार्डवेयर डिवाइस यहाँ स्टोर किए गए देख सकते हैं।
* **/etc**: कॉन्फ़िगरेशन फ़ाइलें
* **/Library**: इसमें पसंद, कैश और लॉग्स से संबंधित बहुत सारे उपनिर्देशिकाएँ और फ़ाइलें मिल सकती हैं। एक लाइब्रेरी फ़ोल्डर मूल और प्रत्येक उपयोगकर्ता के निर्देशिका में मौजूद है।
* **/private**: अनसंदर्भित है लेकिन उल्लिखित फ़ोल्डरों की बहुत सारी उपनिर्देशिकाएँ निजी निर्देशिका के लिए प्रतीकात्मक लिंक हैं।
* **/sbin**: महत्वपूर्ण सिस्टम बाइनरी (प्रशासन से संबंधित)
* **/System**: OS X को चलाने के लिए फ़ाइल। आपको यहाँ अधिकांशतः केवल Apple विशेष फ़ाइलें मिलेंगी (तीसरे पक्ष की नहीं)।
* **/tmp**: फ़ाइलें 3 दिनों के बाद हटा दी जाती हैं (यह /private/tmp के लिए एक सॉफ़्ट लिंक है)
* **/Users**: उपयोगकर्ताओं के लिए होम निर्देशिका।
* **/usr**: कॉन्फ़िग और सिस्टम बाइनरी
* **/var**: लॉग फ़ाइलें
* **/Volumes**: माउंट किए गए ड्राइव्स यहाँ दिखाई देंगे।
* **/.vol**: `stat a.txt` चलाने पर आपको `16777223 7545753 -rw-r--r-- 1 username wheel ...` जैसी कुछ प्राप्त होगी जहाँ पहला नंबर वह आईडी नंबर है जहाँ फ़ाइल मौजूद है और दूसरा इनोड नंबर है। आप इस फ़ाइल की सामग्री तक पहुँच सकते हैं /.vol/ के माध्यम से उस जानकारी के साथ `cat /.vol/16777223/7545753` चलाकर।

### एप्लिकेशन फ़ोल्डर

* **सिस्टम एप्लिकेशनें** `/System/Applications` के तहत स्थित हैं
* **स्थापित** एप्लिकेशनें आम तौर पर `/Applications` या `~/Applications` में स्थापित होती हैं
* **एप्लिकेशन डेटा** `/Library/Application Support` में पाया जा सकता है जो रूट के रूप में चल रही एप्लिकेशनों के लिए है और `~/Library/Application Support` उपयोगकर्ता के रूप में चलने वाली एप्लिकेशनों के लिए।
* **संयुक्त तृतीय-पक्ष एप्लिकेशनें** जो **रूट के रूप में चलने की आवश्यकता हैं** वे आम तौर पर `/Library/PrivilegedHelperTools/` में स्थित होती हैं
* **सैंडबॉक्स** एप्स को `~/Library/Containers` फ़ोल्डर में मैप किया जाता है। प्रत्येक एप्लिकेशन के पास एक फ़ोल्डर होता है जिसे एप्लिकेशन के बंडल ID (`com.apple.Safari`) के अनुसार नामित किया जाता है।
* **कर्नेल** `/System/Library/Kernels/kernel` में स्थित है
* **Apple के कर्नेल एक्सटेंशन्स** `/System/Library/Extensions` में स्थित हैं
* **तृतीय-पक्ष कर्नेल एक्सटेंशन्स** `/Library/Extensions` में स्टोर किए जाते हैं

### संवेदनशील जानकारी वाली फ़ाइलें

MacOS जैसी जगहों पर पासवर्ड जैसी जानकारी स्टोर करता है:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### वंशावली pkg इंस्टॉलर्स

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X विशेष एक्सटेंशन्स

* **`.dmg`**: Apple डिस्क इमेज फ़ाइलें इंस्टॉलर्स के लिए बहुत आम हैं।
* **`.kext`**: यह एक विशिष्ट संरचना का पालन करना चाहिए और यह ड्राइवर का OS X संस्करण है। (यह एक बंडल है)
* **`.plist`**: XML या बाइनरी स्वरूप में जानकारी स्टोर करता है।
* XML या बाइनरी हो सकता है। बाइनरी वाले को इस प्रकार से पढ़ा जा सकता है:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: एप्पल एप्लिकेशन जो निर्देशिका संरचना का पालन करते हैं (यह एक बंडल है)।
* **`.dylib`**: डायनामिक लाइब्रेरीज (जैसे Windows DLL फ़ाइलें)
* **`.pkg`**: xar (eXtensible Archive format) के समान हैं। इन फ़ाइलों के सामग्री को स्थापित करने के लिए इंस्टॉलर कमांड का उपयोग किया जा सकता है।
* **`.DS_Store`**: इस फ़ोल्डर पर प्रत्येक डायरेक्टरी में, यह डायरेक्टरी की विशेषताएँ और कस्टमाइजेशन स्टोर करता है।
* **`.Spotlight-V100`**: यह फ़ोल्डर प्रत्येक वॉल्यूम के रूट डायरेक्टरी पर दिखाई देता है।
* **`.metadata_never_index`**: अगर यह फ़ाइल किसी वॉल्यूम के रूट पर है तो Spotlight उस वॉल्यूम को इंडेक्स नहीं करेगा।
* **`.noindex`**: इस एक्सटेंशन वाली फ़ाइलें और फ़ोल्डर Spotlight द्वारा इंडेक्स नहीं की जाएंगी।
* **`.sdef`**: बंडल्स के भीतर फ़ाइलें निर्दिष्ट करती हैं कि किस प्रकार से एप्लिकेशन के साथ AppleScript से बातचीत की जा सकती है।

### macOS बंडल्स

एक बंडल एक **निर्देशिका** है जो **फाइंडर में एक वस्तु की तरह दिखता है** (एक बंडल उदाहरण हैं `*.app` फ़ाइलें)।

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld साझा लाइब्रेरी कैश (SLC)

macOS (और iOS) में सभी सिस्टम साझा लाइब्रेरीज, जैसे फ़्रेमवर्क और dylibs, **एक एकल फ़ाइल में संयुक्त किए गए हैं**, जिसे **dyld साझा कैश** कहा जाता है। इसस
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
ध्यान दें कि यदि `dyld_shared_cache_util` टूल काम नहीं करता है, तो आप **साझा dyld बाइनरी को Hopper में पास** कर सकते हैं और Hopper सभी पुस्तकालयों की पहचान करने में सक्षम होगा और आपको **जिसे आप जांचना चाहते हैं उसे चुनने** देगा:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

कुछ एक्सट्रैक्टर्स काम नहीं करेंगे क्योंकि dylibs पूर्व-लिंक होते हैं जिसके कारण वे अज्ञात पतों पर जा सकते हैं।

{% hint style="success" %}
एक इम्युलेटर का उपयोग करके Xcode में \*OS डिवाइसेस की साझा लाइब्रेरी कैश को डाउनलोड करना भी संभव है। वे इस प्रकार डाउनलोड किए जाएंगे: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, जैसे:`$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### SLC को मैप करना

**`dyld`** यह जानने के लिए सिस्कॉल **`shared_region_check_np`** का उपयोग करता है कि क्या SLC को मैप किया गया है (जो पता देता है) और **`shared_region_map_and_slide_np`** का उपयोग करता है SLC को मैप करने के लिए।

ध्यान दें कि यदि SLC को पहली बार उपयोग किया जाता है, तो सभी **प्रक्रियाएं** **एक ही कॉपी** का उपयोग करती हैं, जो यदि हमलावार प्रक्रियाएं सिस्टम में चलाने में सक्षम होता है तो **ASLR** सुरक्षा को **हटा देता है**। यह वास्तव में पिछले में उत्पीड़ित किया गया था और साझा क्षेत्र पेजर के साथ ठीक किया गया था।

ब्रांच पूल छोटे Mach-O dylibs हैं जो छवि मैपिंग के बीच छोटे स्थान बनाते हैं जिससे फ़ंक्शनों को अंतर्दृष्टि नहीं कर सकते।

### SLC को ओवरराइड करें

इस्तेमाल करके एनवायरनमेंट वेरिएबल्स:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> यह एक नया साझा लाइब्रेरी कैश लोड करने की अनुमति देगा
* **`DYLD_SHARED_CACHE_DIR=avoid`** और लाइब्रेरी को वास्तविक वालों के साथ साझा कैश के साथ सिमलिंक्स के साथ मैन्युअल रूप से पुन: स्थानांतरित करें (आपको उन्हें निकालने की आवश्यकता होगी)

## विशेष फ़ाइल अनुमतियाँ

### फ़ोल्डर अनुमतियाँ

एक **फ़ोल्डर** में, **पढ़ने** से उसे **सूचीबद्ध करने** की अनुमति होती है, **लिखने** से उसे हटाने और उस पर फ़ाइलें **लिखने** की अनुमति होती है, और **क्रियान्वयन** उसे निरंतर करने की अनुमति देता है। इसलिए, उदाहरण के लिए, एक उपयोगकर्ता को जिसके पास **फ़ाइल पर पढ़ने की अनुमति** है उस डायरेक्टरी के अंदर जहां उसे **क्रियान्वयन की अनुमति नहीं है** **फ़ाइल पढ़ने** की अनुमति नहीं होगी।

### फ़्लैग संशोधक

कुछ फ़्लैग हैं जो फ़ाइल में सेट किए जा सकते हैं जो फ़ाइल का व्यवहार अलग बना देंगे। आप फ़ाइलों के फ़्लैग्स की जाँच कर सकते हैं डायरेक्टरी के अंदर `ls -lO /path/directory` के साथ

* **`uchg`**: **uchange** फ़्लैग के रूप में जाना जाता है जो किसी भी क्रिया को **बदलने या हटाने से रोकेगा**। इसे सेट करने के लिए: `chflags uchg file.txt`
* रूट उपयोगकर्ता फ़्लैग को **हटा सकता है** और फ़ाइल को संशोधित कर सकता है
* **`restricted`**: यह फ़्लैग फ़ाइल को **SIP द्वारा संरक्षित** बना देगा (आप इस फ़्लैग को फ़ाइल में नहीं जोड़ सकते)।
* **`Sticky bit`**: यदि एक डायरेक्टरी में स्टिकी बिट है, **केवल** डायरेक्टरी के मालिक या रूट **फ़ाइलों को नाम बदल सकते हैं या हटा सकते हैं**। सामान्यत: यह /tmp डायरेक्टरी पर सेट किया जाता है ताकि साधारण उपयोगकर्ता अन्य उपयोगकर्ताओं की फ़ाइलों को हटाने या हटाने से रोक सकें।

सभी फ़्लैग्स को फ़ाइल `sys/stat.h` में पाया जा सकता है (इसे `mdfind stat.h | grep stat.h` का उपयोग करके खोजें) और वे हैं:

* `UF_SETTABLE` 0x0000ffff: मालिक परिवर्तनीय फ़्लैग्स का मास्क।
* `UF_NODUMP` 0x00000001: फ़ाइल को डंप न करें।
* `UF_IMMUTABLE` 0x00000002: फ़ाइल को बदला नहीं जा सकता।
* `UF_APPEND` 0x00000004: फ़ाइल में लिखने केवल जोड़ सकते हैं।
* `UF_OPAQUE` 0x00000008: डायरेक्टरी यूनियन के संबंध में अपारदर्शी है।
* `UF_COMPRESSED` 0x00000020: फ़ाइल संकुचित है (कुछ फ़ाइल-सिस्टम्स)।
* `UF_TRACKED` 0x00000040: इस सेट किए गए फ़ाइलों के लिए हटाने/नाम बदलने के लिए कोई सूचनाएँ नहीं।
* `UF_DATAVAULT` 0x00000080: पढ़ने और लिखने के लिए अधिकार आवश्यक है।
* `UF_HIDDEN` 0x00008000: इस आइटम को GUI में प्रदर्शित न करने की संकेतना।
* `SF_SUPPORTED` 0x009f0000: सुपरयूजर समर्थित फ़्लैग्स का मास्क।
* `SF_SETTABLE` 0x3fff0000: सुपरयूजर परिवर्तनीय फ़्लैग्स का मास्क।
* `SF_SYNTHETIC` 0xc0000000: सिस्टम केवल पढ़ने योग्य सिंथेटिक फ़्लैग्स का मास्क।
* `SF_ARCHIVED` 0x00010000: फ़ाइल संग्रहीत है।
* `SF_IMMUTABLE` 0x00020000: फ़ाइल को बदला नहीं जा सकता।
* `SF_APPEND` 0x00040000: फ़ाइल में लिखने केवल जोड़ सकते हैं।
* `SF_RESTRICTED` 0x00080000: लिखने के लिए अधिकार आवश्यक है।
* `SF_NOUNLINK` 0x00100000: आइटम को हटाया, नाम बदला या माउंट किया नहीं जा सकता।
* `SF_FIRMLINK` 0x00800000: फ़ाइल एक फर्मलिंक है।
* `SF_DATALESS` 0x40000000: फ़ाइल डेटालेस ऑब्जेक्ट है।

### **फ़ाइल ACLs**

फ़ाइल **ACLs** में **ACE** (एक्सेस कंट्रोल एंट्रीज) होते हैं जहां विभिन्न उपयोगकर्ताओं को अधिक **सूक्ष्म अनुमतियाँ** दी जा सकती हैं।

एक **डायरेक्टरी** को ये अनुमतियाँ प्रदान की जा सकती हैं: `सूची`, `खोज`, `फ़ाइल जोड़ें`, `उप-डायरेक्टरी जोड़ें`, `बच्चे को हटाएं`, `बच्चे को हटाएं`।\
और एक **फ़ाइल** को: `पढ़ें`, `लिखें`, `जोड़ें`, `क्रियान्वयन`।

जब फ़ाइल में ACLs होती हैं तो आपको **परमिशन्स को सूचीबद्ध करते समय "+" मिलेगा जैसे कि**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
आप फ़ाइल की ACLs को निम्नलिखित के साथ पढ़ सकते हैं:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
आप **सभी फ़ाइलें एसीएल के साथ** निम्नलिखित के साथ खोज सकते हैं (यह बहुत ही धीमा है):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### विस्तृत गुण

विस्तृत गुणों के पास एक नाम और इच्छित मान होता है, और `ls -@` का उपयोग करके देखा जा सकता है और `xattr` कमांड का उपयोग करके परिवर्तित किया जा सकता है। कुछ सामान्य विस्तृत गुण हैं:

* `com.apple.resourceFork`: संसाधन फोर्क संगतता। फ़ाइल/..namedfork/rsrc के रूप में भी दिखाई देता है
* `com.apple.quarantine`: MacOS: गेटकीपर क्वारंटाइन तंत्र (III/6)
* `metadata:*`: MacOS: विभिन्न मेटाडेटा, जैसे `_backup_excludeItem`, या `kMD*`
* `com.apple.lastuseddate` (#PS): अंतिम फ़ाइल उपयोग तिथि
* `com.apple.FinderInfo`: MacOS: फाइंडर सूचना (जैसे, रंग टैग)
* `com.apple.TextEncoding`: ASCII टेक्स्ट फ़ाइलों की पाठ कोडिंग को निर्दिष्ट करता है
* `com.apple.logd.metadata`: `/var/db/diagnostics` में फ़ाइलों पर logd द्वारा उपयोग किया जाता है
* `com.apple.genstore.*`: पीढ़ीवादी स्टोरेज (`/.DocumentRevisions-V100` फ़ाइल सिस्टम के रूट में)
* `com.apple.rootless`: MacOS: सिस्टम अखंडता संरक्षण द्वारा फ़ाइल को लेबल करने के लिए उपयोग किया जाता है (III/10)
* `com.apple.uuidb.boot-uuid`: बूट युयुआईडी के साथ बूट युगों के logd चिह्नित करने के लिए उपयोग किया जाता है
* `com.apple.decmpfs`: MacOS: पारदर्शी फ़ाइल संपीड़न (II/7)
* `com.apple.cprotect`: \*OS: प्रति-फ़ाइल एन्क्रिप्शन डेटा (III/11)
* `com.apple.installd.*`: \*OS: installd द्वारा उपयोग किए जाने वाले मेटाडेटा, जैसे, `installType`, `uniqueInstallID`

### संसाधन फोर्क्स | macOS ADS

यह **मैकओएस मशीनों में विभिन्न डेटा स्ट्रीम्स प्राप्त करने का एक तरीका** है। आप **com.apple.ResourceFork** नामक एक विस्तृत गुण के अंदर सामग्री सहेज सकते हैं जिसे **फ़ाइल/..namedfork/rsrc** में सहेजकर।
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
आप **इस विस्तारित विशेषता को धारित सभी फ़ाइलें खोज सकते हैं** इसके साथ:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

विस्तारित विशेषता `com.apple.decmpfs` इसका संकेत देती है कि फ़ाइल एन्क्रिप्टेड रूप से स्टोर की गई है, `ls -l` **आकार 0** की रिपोर्ट करेगा और संपीड़ित डेटा इस विशेषता में है। जब भी फ़ाइल तक पहुँचा जाता है, तो यह मेमोरी में डिक्रिप्ट हो जाएगी।

इस विशेषता को `ls -lO` के साथ देखा जा सकता है, जिसे संपीड़ित बताया जाता है क्योंकि संपीड़ित फ़ाइलें भी झंझट यूएफ_कंप्रेस्ड के ध्वज के साथ टैग की जाती हैं। यदि किसी संपीड़ित फ़ाइल को `chflags nocompressed </path/to/file>` के साथ हटाया जाता है, तो सिस्टम को पता नहीं चलेगा कि फ़ाइल संपीड़ित थी और इसलिए यह डिकंप्रेस और डेटा तक पहुँचने में सक्षम नहीं होगा (यह सोचेगा कि यह वास्तव में खाली है)।

उपकरण afscexpand का उपयोग एक डाइल को डिकंप्रेस करने के लिए किया जा सकता है।

## **सामान्य बाइनरी और** Mach-o प्रारूप

Mac OS बाइनरी आम तौर पर **सामान्य बाइनरी** के रूप में कॉम्पाइल किए जाते हैं। एक **सामान्य बाइनरी** में **एक ही फ़ाइल में कई वार्चिटेक्चर का समर्थन कर सकता है**।

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS प्रक्रिया मेमोरी

## macOS मेमोरी डंपिंग

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## जोखिम श्रेणी फ़ाइलें Mac OS

डायरेक्टरी `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` यहाँ जानकारी होती है कि **विभिन्न फ़ाइल एक्सटेंशन के साथ जुड़े जोखिम के बारे में**। यह डायरेक्टरी फ़ाइलों को विभिन्न जोखिम स्तरों में वर्गीकृत करती है, जो सफारी को डाउनलोड करने के बाद इन फ़ाइलों को कैसे हैंडल करना है इस पर प्रभाव डालती है। श्रेणियाँ निम्नलिखित हैं:

* **LSRiskCategorySafe**: इस श्रेणी में फ़ाइलें **पूरी तरह सुरक्षित** मानी जाती हैं। सफारी इन फ़ाइलों को डाउनलोड होने के बाद स्वचालित रूप से खोलेगा।
* **LSRiskCategoryNeutral**: इन फ़ाइलों के साथ कोई चेतावनी नहीं होती है और सफारी द्वारा स्वचालित रूप से नहीं खोली जाती है।
* **LSRiskCategoryUnsafeExecutable**: इस श्रेणी की फ़ाइलें एक चेतावनी ट्रिगर करती हैं जिसमें बताया जाता है कि फ़ाइल एक एप्लिकेशन है। यह उपयोगकर्ता को सूचित करने के लिए एक सुरक्षा उपाय के रूप में काम करता है।
* **LSRiskCategoryMayContainUnsafeExecutable**: यह श्रेणी फ़ाइलों के लिए है, जैसे कि आर्काइव, जो एक एक्सीक्यूटेबल को समाहित कर सकते हैं। सफारी **एक चेतावनी ट्रिगर करेगा** जब तक यह सत्यापित नहीं कर सकता कि सभी सामग्री सुरक्षित या न्यूट्रल है।

## लॉग फ़ाइलें

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: डाउनलोड की गई फ़ाइलों के बारे में जानकारी रखती है, जैसे कि वहाँ से डाउनलोड की गई URL।
* **`/var/log/system.log`**: OSX सिस्टम का मुख्य लॉग। com.apple.syslogd.plist सिस्टमलॉगिंग का निष्पादन जिम्मेदार है (आप "com.apple.syslogd" को `launchctl list` में खोजकर देख सकते हैं कि क्या यह अक्षम है)।
* **`/private/var/log/asl/*.asl`**: ये एप्पल सिस्टम लॉग हैं जिनमें दिलचस्प जानकारी हो सकती है।
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "फाइंडर" के माध्यम से हाल ही में एक्सेस की गई फ़ाइलें और एप्लिकेशन स्टोर करता है।
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: सिस्टम स्टार्टअप पर लॉन्च करने के लिए आइटम स्टोर करता है।
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility ऐप के लिए लॉग फ़ाइल (ड्राइव्स के बारे में जानकारी, इसमें USB भी शामिल हैं)।
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: वायरलेस एक्सेस पॉइंट्स के बारे में डेटा।
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: निष्क्रिय किए गए डेमन्स की सूची।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks) गिटहब रेपो में पीआर जमा करके।

</details>
{% endhint %}
