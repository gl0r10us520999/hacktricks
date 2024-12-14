# macOS Bundles

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

Bundles katika macOS hutumikia kama vyombo vya rasilimali mbalimbali ikiwa ni pamoja na programu, maktaba, na faili nyingine muhimu, na kuonekana kama vitu vya pekee katika Finder, kama vile faili maarufu za `*.app`. Bundle inayokutana mara nyingi ni bundle ya `.app`, ingawa aina nyingine kama `.framework`, `.systemextension`, na `.kext` pia ni za kawaida.

### Essential Components of a Bundle

Ndani ya bundle, hasa ndani ya saraka ya `<application>.app/Contents/`, rasilimali muhimu mbalimbali zimehifadhiwa:

* **\_CodeSignature**: Saraka hii inahifadhi maelezo ya saini ya msimbo muhimu kwa kuthibitisha uhalali wa programu. Unaweza kuchunguza taarifa za saini ya msimbo kwa kutumia amri kama: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Inashikilia binary inayoweza kutekelezwa ya programu inayotumika wakati wa mwingiliano wa mtumiaji.
* **Resources**: Hifadhi ya vipengele vya interface ya mtumiaji wa programu ikiwa ni pamoja na picha, hati, na maelezo ya interface (faili za nib/xib).
* **Info.plist**: Inafanya kazi kama faili kuu ya usanidi wa programu, muhimu kwa mfumo kutambua na kuingiliana na programu ipasavyo.

#### Important Keys in Info.plist

Faili ya `Info.plist` ni msingi wa usanidi wa programu, ikiwa na funguo kama:

* **CFBundleExecutable**: Inaelezea jina la faili kuu inayoweza kutekelezwa iliyoko katika saraka ya `Contents/MacOS`.
* **CFBundleIdentifier**: Inatoa kitambulisho cha kimataifa kwa programu, kinachotumika sana na macOS kwa usimamizi wa programu.
* **LSMinimumSystemVersion**: Inaonyesha toleo la chini la macOS linalohitajika kwa programu kutekelezwa.

### Exploring Bundles

Ili kuchunguza yaliyomo kwenye bundle, kama `Safari.app`, amri ifuatayo inaweza kutumika: `bash ls -lR /Applications/Safari.app/Contents`

Uchunguzi huu unaonyesha saraka kama `_CodeSignature`, `MacOS`, `Resources`, na faili kama `Info.plist`, kila moja ikihudumu kusudi la kipekee kutoka kwa kulinda programu hadi kufafanua interface yake ya mtumiaji na vigezo vya uendeshaji.

#### Additional Bundle Directories

Zaidi ya saraka za kawaida, bundles zinaweza pia kujumuisha:

* **Frameworks**: Inashikilia frameworks zilizojumuishwa zinazotumiwa na programu. Frameworks ni kama dylibs zenye rasilimali za ziada.
* **PlugIns**: Saraka ya plug-ins na nyongeza zinazoongeza uwezo wa programu.
* **XPCServices**: Inashikilia huduma za XPC zinazotumiwa na programu kwa mawasiliano yasiyo ya mchakato.

Muundo huu unahakikisha kuwa vipengele vyote muhimu vimefungwa ndani ya bundle, kurahisisha mazingira ya programu ya moduli na salama.

Kwa maelezo zaidi kuhusu funguo za `Info.plist` na maana zao, hati za waendelezaji wa Apple zinatoa rasilimali nyingi: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
