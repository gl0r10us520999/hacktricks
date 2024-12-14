# macOS Bundles

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Basic Information

Bundels in macOS dien as houers vir 'n verskeidenheid hulpbronne, insluitend toepassings, biblioteke en ander nodige lêers, wat hulle laat voorkom as enkele voorwerpe in Finder, soos die bekende `*.app` lêers. Die mees algemene bundel wat teëgekom word, is die `.app` bundel, hoewel ander tipes soos `.framework`, `.systemextension`, en `.kext` ook algemeen voorkom.

### Essential Components of a Bundle

Binne 'n bundel, veral binne die `<application>.app/Contents/` gids, is 'n verskeidenheid belangrike hulpbronne gehuisves:

* **\_CodeSignature**: Hierdie gids stoor kode-handtekening besonderhede wat noodsaaklik is om die integriteit van die toepassing te verifieer. Jy kan die kode-handtekening inligting ondersoek met opdragte soos: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Bevat die uitvoerbare binêre van die toepassing wat loop wanneer die gebruiker interaksie het.
* **Resources**: 'n Repository vir die toepassing se gebruikerskoppelvlak komponente, insluitend beelde, dokumente, en koppelvlak beskrywings (nib/xib lêers).
* **Info.plist**: Dien as die toepassing se hoof konfigurasielêer, noodsaaklik vir die stelsel om die toepassing korrek te herken en mee te werk.

#### Important Keys in Info.plist

Die `Info.plist` lêer is 'n hoeksteen vir toepassing konfigurasie, wat sleutels soos bevat:

* **CFBundleExecutable**: Gee die naam van die hoof uitvoerbare lêer wat in die `Contents/MacOS` gids geleë is.
* **CFBundleIdentifier**: Verskaf 'n globale identifiseerder vir die toepassing, wat wyd gebruik word deur macOS vir toepassing bestuur.
* **LSMinimumSystemVersion**: Gee die minimum weergawe van macOS aan wat benodig word vir die toepassing om te loop.

### Exploring Bundles

Om die inhoud van 'n bundel, soos `Safari.app`, te verken, kan die volgende opdrag gebruik word: `bash ls -lR /Applications/Safari.app/Contents`

Hierdie verkenning onthul gidse soos `_CodeSignature`, `MacOS`, `Resources`, en lêers soos `Info.plist`, elk wat 'n unieke doel dien van die beveiliging van die toepassing tot die definisie van sy gebruikerskoppelvlak en operasionele parameters.

#### Additional Bundle Directories

Benewens die algemene gidse, kan bundels ook insluit:

* **Frameworks**: Bevat gebundelde raamwerke wat deur die toepassing gebruik word. Raamwerke is soos dylibs met ekstra hulpbronne.
* **PlugIns**: 'n Gids vir inproppe en uitbreidings wat die toepassing se vermoëns verbeter.
* **XPCServices**: Hou XPC dienste wat deur die toepassing gebruik word vir buite-proses kommunikasie.

Hierdie struktuur verseker dat alle nodige komponente binne die bundel ingesluit is, wat 'n modulaire en veilige toepassing omgewing fasiliteer.

Vir meer gedetailleerde inligting oor `Info.plist` sleutels en hul betekenisse, bied die Apple ontwikkelaar dokumentasie uitgebreide hulpbronne: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
