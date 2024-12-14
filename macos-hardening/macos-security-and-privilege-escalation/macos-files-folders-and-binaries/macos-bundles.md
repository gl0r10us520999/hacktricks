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

Bundlovi u macOS služe kao kontejneri za razne resurse uključujući aplikacije, biblioteke i druge potrebne datoteke, čineći ih da izgledaju kao jedinstveni objekti u Finder-u, kao što su poznate `*.app` datoteke. Najčešće se susreće `.app` bundle, iako su prisutni i drugi tipovi poput `.framework`, `.systemextension`, i `.kext`.

### Essential Components of a Bundle

Unutar bundla, posebno unutar `<application>.app/Contents/` direktorijuma, smeštene su razne važne resurse:

* **\_CodeSignature**: Ovaj direktorijum čuva detalje o potpisivanju koda koji su ključni za verifikaciju integriteta aplikacije. Možete pregledati informacije o potpisivanju koda koristeći komande kao što su: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Sadrži izvršni binarni fajl aplikacije koji se pokreće prilikom interakcije korisnika.
* **Resources**: Repozitorij za komponente korisničkog interfejsa aplikacije uključujući slike, dokumente i opise interfejsa (nib/xib datoteke).
* **Info.plist**: Deluje kao glavni konfiguracioni fajl aplikacije, ključan za sistem da prepozna i interaguje sa aplikacijom na odgovarajući način.

#### Important Keys in Info.plist

Fajl `Info.plist` je kamen temeljac za konfiguraciju aplikacije, sadrži ključeve kao što su:

* **CFBundleExecutable**: Specifikuje ime glavnog izvršnog fajla smeštenog u `Contents/MacOS` direktorijumu.
* **CFBundleIdentifier**: Pruža globalni identifikator za aplikaciju, koji macOS široko koristi za upravljanje aplikacijama.
* **LSMinimumSystemVersion**: Ukazuje na minimalnu verziju macOS-a potrebnu za pokretanje aplikacije.

### Exploring Bundles

Da biste istražili sadržaj bundla, kao što je `Safari.app`, može se koristiti sledeća komanda: `bash ls -lR /Applications/Safari.app/Contents`

Ova istraživanja otkrivaju direktorijume poput `_CodeSignature`, `MacOS`, `Resources`, i fajlove poput `Info.plist`, svaki služi jedinstvenoj svrsi od obezbeđivanja aplikacije do definisanja njenog korisničkog interfejsa i operativnih parametara.

#### Additional Bundle Directories

Pored uobičajenih direktorijuma, bundlovi mogu takođe uključivati:

* **Frameworks**: Sadrži bundlovane frameworke koje koristi aplikacija. Frameworks su poput dylibs sa dodatnim resursima.
* **PlugIns**: Direktorijum za plug-inove i ekstenzije koje poboljšavaju mogućnosti aplikacije.
* **XPCServices**: Sadrži XPC servise koje aplikacija koristi za komunikaciju van procesa.

Ova struktura osigurava da su svi potrebni komponenti enkapsulirani unutar bundla, olakšavajući modularno i sigurno okruženje aplikacije.

Za detaljnije informacije o `Info.plist` ključevima i njihovim značenjima, Apple-ova dokumentacija za programere pruža opsežne resurse: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
