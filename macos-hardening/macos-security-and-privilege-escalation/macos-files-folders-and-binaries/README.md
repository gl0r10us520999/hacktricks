# macOS Lêers, Gidse, Binaries & Geheue

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Lêer hiërargie uitleg

* **/Applications**: Die geïnstalleerde toepassings behoort hier te wees. Alle gebruikers sal toegang tot hulle hê.
* **/bin**: Opdraglyn binaries
* **/cores**: As dit bestaan, word dit gebruik om kernaflae te stoor
* **/dev**: Alles word as 'n lêer behandel, so jy mag hardeware toestelle hier gestoor sien.
* **/etc**: Konfigurasie lêers
* **/Library**: 'n Baie van subgidse en lêers wat verband hou met voorkeure, kas en logs kan hier gevind word. 'n Biblioteek gids bestaan in die wortel en op elke gebruiker se gids.
* **/private**: Nie gedokumenteer nie, maar baie van die genoemde gidse is simboliese skakels na die private gids.
* **/sbin**: Essensiële stelselbinaries (verbonde aan administrasie)
* **/System**: Lêer om OS X te laat loop. Jy behoort meestal net Apple spesifieke lêers hier te vind (nie derdeparty nie).
* **/tmp**: Lêers word na 3 dae verwyder (dit is 'n sagte skakel na /private/tmp)
* **/Users**: Tuisgids vir gebruikers.
* **/usr**: Konfig en stelselbinaries
* **/var**: Log lêers
* **/Volumes**: Die gemonteerde skywe sal hier verskyn.
* **/.vol**: Deur `stat a.txt` te loop, kry jy iets soos `16777223 7545753 -rw-r--r-- 1 username wheel ...` waar die eerste nommer die id nommer van die volume is waar die lêer bestaan en die tweede een die inode nommer is. Jy kan die inhoud van hierdie lêer deur /.vol/ met daardie inligting verkry deur `cat /.vol/16777223/7545753` te loop.

### Toepassings Gidse

* **Stelsel toepassings** is geleë onder `/System/Applications`
* **Geïnstalleerde** toepassings word gewoonlik in `/Applications` of in `~/Applications` geïnstalleer
* **Toepassing data** kan gevind word in `/Library/Application Support` vir die toepassings wat as wortel loop en `~/Library/Application Support` vir toepassings wat as die gebruiker loop.
* Derdeparty toepassings **daemons** wat **as wortel moet loop** is gewoonlik geleë in `/Library/PrivilegedHelperTools/`
* **Sandboxed** toepassings is in die `~/Library/Containers` gids gemap. Elke toepassing het 'n gids wat volgens die toepassing se bundel ID genoem word (`com.apple.Safari`).
* Die **kernel** is geleë in `/System/Library/Kernels/kernel`
* **Apple se kernel uitbreidings** is geleë in `/System/Library/Extensions`
* **Derdeparty kernel uitbreidings** word gestoor in `/Library/Extensions`

### Lêers met Sensitiewe Inligting

MacOS stoor inligting soos wagwoorde op verskeie plekke:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Kwetsbare pkg installers

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X Spesifieke Uitbreidings

* **`.dmg`**: Apple Disk Image lêers is baie algemeen vir installers.
* **`.kext`**: Dit moet 'n spesifieke struktuur volg en dit is die OS X weergawe van 'n bestuurder. (dit is 'n bundel)
* **`.plist`**: Ook bekend as eiendom lys stoor inligting in XML of binêre formaat.
* Kan XML of binêr wees. Binêre kan gelees word met:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Apple toepassings wat die gidsstruktuur volg (dit is 'n bundel).
* **`.dylib`**: Dinamiese biblioteke (soos Windows DLL lêers)
* **`.pkg`**: Is dieselfde as xar (eXtensible Archive formaat). Die installer opdrag kan gebruik word om die inhoud van hierdie lêers te installeer.
* **`.DS_Store`**: Hierdie lêer is op elke gids, dit stoor die eienskappe en aanpassings van die gids.
* **`.Spotlight-V100`**: Hierdie gids verskyn op die wortelgids van elke volume op die stelsel.
* **`.metadata_never_index`**: As hierdie lêer op die wortel van 'n volume is, sal Spotlight daardie volume nie indekseer nie.
* **`.noindex`**: Lêers en gidse met hierdie uitbreiding sal nie deur Spotlight geïndekseer word nie.
* **`.sdef`**: Lêers binne bundels wat spesifiseer hoe dit moontlik is om met die toepassing van 'n AppleScript te kommunikeer.

### macOS Bundels

'n Bundel is 'n **gids** wat **soos 'n objek in Finder lyk** (n Bundel voorbeeld is `*.app` lêers).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Gedeelde Biblioteek Kas (SLC)

Op macOS (en iOS) is al die stelsel gedeelde biblioteke, soos raamwerke en dylibs, **gecombineer in 'n enkele lêer**, genoem die **dyld gedeelde kas**. Dit verbeter die prestasie, aangesien kode vinniger gelaai kan word.

Dit is geleë in macOS in `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` en in ouer weergawes mag jy die **gedeelde kas** in **`/System/Library/dyld/`** vind.\
In iOS kan jy hulle vind in **`/System/Library/Caches/com.apple.dyld/`**.

Soos die dyld gedeelde kas, is die kernel en die kernel uitbreidings ook saamgecompileer in 'n kernel kas, wat by opstarttyd gelaai word.

Om die biblioteke uit die enkele lêer dylib gedeelde kas te onttrek, was dit moontlik om die binêre [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) te gebruik wat dalk nie vandag werk nie, maar jy kan ook [**dyldextractor**](https://github.com/arandomdev/dyldextractor) gebruik: 

{% code overflow="wrap" %}
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
Let daarop dat selfs al werk die `dyld_shared_cache_util` hulpmiddel nie, kan jy die **gedeelde dyld-binary aan Hopper** oorhandig en Hopper sal in staat wees om al die biblioteke te identifiseer en jou te laat **kies watter een** jy wil ondersoek:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Sommige ekstrakteurs sal nie werk nie, aangesien dylibs vooraf gekoppel is met hard-gecodeerde adresse, wat beteken dat hulle na onbekende adresse kan spring.

{% hint style="success" %}
Dit is ook moontlik om die Gedeelde Biblioteek Kaste van ander \*OS toestelle in macos af te laai deur 'n emulator in Xcode te gebruik. Hulle sal binne afgelaai word: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, soos: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Kaart SLC

**`dyld`** gebruik die syscall **`shared_region_check_np`** om te weet of die SLC gemap is (wat die adres teruggee) en **`shared_region_map_and_slide_np`** om die SLC te map.

Let daarop dat selfs al is die SLC op die eerste gebruik geskuif, gebruik al die **prosesse** die **dieselfde kopie**, wat die **ASLR** beskerming uitgeskakel het as die aanvaller in staat was om prosesse in die stelsel te laat loop. Dit is eintlik in die verlede uitgebuit en reggestel met 'n gedeelde streek pager.

Branch pools is klein Mach-O dylibs wat klein ruimtes tussen beeldmappings skep, wat dit onmoontlik maak om die funksies te interpose.

### Oorskry SLCs

Gebruik die omgewingsveranderlikes:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Dit sal toelaat om 'n nuwe gedeelde biblioteek kas te laai.
* **`DYLD_SHARED_CACHE_DIR=avoid`** en vervang handmatig die biblioteke met symlinks na die gedeelde kas met die werklike een (jy sal dit moet onttrek).

## Spesiale Lêer Toestemmings

### Gids toestemmings

In 'n **gids**, **lees** laat jou toe om dit te **lys**, **skryf** laat jou toe om **te verwyder** en **te skryf** lêers daarop, en **uitvoer** laat jou toe om die gids te **deursoek**. So, byvoorbeeld, 'n gebruiker met **lees toestemming oor 'n lêer** binne 'n gids waar hy **nie uitvoer** toestemming het nie, **sal nie in staat wees om** die lêer te lees nie.

### Vlag modifiers

Daar is 'n paar vlag wat in die lêers gestel kan word wat die lêer anders kan laat optree. Jy kan die **vlag** van die lêers binne 'n gids nagaan met `ls -lO /path/directory`

* **`uchg`**: Bekend as **uchange** vlag sal **enige aksie** wat die **lêer** verander of verwyder, **voorkom**. Om dit in te stel, doen: `chflags uchg file.txt`
* Die wortel gebruiker kan die **vlag verwyder** en die lêer wysig.
* **`restricted`**: Hierdie vlag maak die lêer **beskerm deur SIP** (jy kan nie hierdie vlag aan 'n lêer toevoeg nie).
* **`Sticky bit`**: As 'n gids met 'n sticky bit, **slegs** die **gids eienaar of wortel kan hernoem of verwyder** lêers. Tipies word dit op die /tmp gids gestel om gewone gebruikers te verhoed om ander gebruikers se lêers te verwyder of te skuif.

Al die vlae kan in die lêer `sys/stat.h` gevind word (vind dit met `mdfind stat.h | grep stat.h`) en is:

* `UF_SETTABLE` 0x0000ffff: Masker van eienaar veranderbare vlae.
* `UF_NODUMP` 0x00000001: Moet nie lêer dump nie.
* `UF_IMMUTABLE` 0x00000002: Lêer mag nie verander word nie.
* `UF_APPEND` 0x00000004: Skrywe na lêer mag slegs bygevoeg word.
* `UF_OPAQUE` 0x00000008: Gids is ondoorgrondelik ten opsigte van unie.
* `UF_COMPRESSED` 0x00000020: Lêer is gecomprimeer (sommige lêerstelsels).
* `UF_TRACKED` 0x00000040: Geen kennisgewings vir verwyderings/hernoemings vir lêers met hierdie ingestel nie.
* `UF_DATAVAULT` 0x00000080: Regte benodig vir lees en skryf.
* `UF_HIDDEN` 0x00008000: Wenke dat hierdie item nie in 'n GUI vertoon moet word nie.
* `SF_SUPPORTED` 0x009f0000: Masker van supergebruiker ondersteun vlae.
* `SF_SETTABLE` 0x3fff0000: Masker van supergebruiker veranderbare vlae.
* `SF_SYNTHETIC` 0xc0000000: Masker van stelsels lees-alleen sintetiese vlae.
* `SF_ARCHIVED` 0x00010000: Lêer is geargiveer.
* `SF_IMMUTABLE` 0x00020000: Lêer mag nie verander word nie.
* `SF_APPEND` 0x00040000: Skrywe na lêer mag slegs bygevoeg word.
* `SF_RESTRICTED` 0x00080000: Regte benodig vir skryf.
* `SF_NOUNLINK` 0x00100000: Item mag nie verwyder, hernoem of gemonteer word nie.
* `SF_FIRMLINK` 0x00800000: Lêer is 'n firmlink.
* `SF_DATALESS` 0x40000000: Lêer is 'n dataloos objek.

### **Lêer ACLs**

Lêer **ACLs** bevat **ACE** (Toegang Beheer Inskrywings) waar meer **fynere toestemmings** aan verskillende gebruikers toegeken kan word.

Dit is moontlik om 'n **gids** hierdie toestemmings te gee: `lys`, `soek`, `voeg_lêer_by`, `voeg_subgids_by`, `verwyder_kind`, `verwyder_kind`.\
En aan 'n **lêer**: `lees`, `skryf`, `byvoeg`, `uitvoer`.

Wanneer die lêer ACLs bevat, sal jy **'n "+" vind wanneer jy die toestemmings lys soos in**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
U kan die **ACLs** van die lêer lees met:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
U kan **alle lêers met ACL's** vind met (dit is baie stadig):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Extended Attributes

Uitgebreide eienskappe het 'n naam en enige gewenste waarde, en kan gesien word met `ls -@` en gemanipuleer word met die `xattr` opdrag. Sommige algemene uitgebreide eienskappe is:

* `com.apple.resourceFork`: Hulpbronvork kompatibiliteit. Ook sigbaar as `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Gatekeeper kwarantynmeganisme (III/6)
* `metadata:*`: MacOS: verskeie metadata, soos `_backup_excludeItem`, of `kMD*`
* `com.apple.lastuseddate` (#PS): Laaste lêer gebruik datum
* `com.apple.FinderInfo`: MacOS: Finder inligting (bv., kleur Tags)
* `com.apple.TextEncoding`: Spesifiseer tekskodering van ASCII tekslêers
* `com.apple.logd.metadata`: Gebruik deur logd op lêers in `/var/db/diagnostics`
* `com.apple.genstore.*`: Generasionele berging (`/.DocumentRevisions-V100` in die wortel van die lêerstelsel)
* `com.apple.rootless`: MacOS: Gebruik deur Stelselintegriteitbeskerming om lêer te merk (III/10)
* `com.apple.uuidb.boot-uuid`: logd merkings van opstart epoches met unieke UUID
* `com.apple.decmpfs`: MacOS: Deursigtige lêer kompressie (II/7)
* `com.apple.cprotect`: \*OS: Per-lêer versleuteling data (III/11)
* `com.apple.installd.*`: \*OS: Metadata gebruik deur installd, bv., `installType`, `uniqueInstallID`

### Resource Forks | macOS ADS

Dit is 'n manier om **Alternatiewe Data Strome in MacOS** masjiene te verkry. Jy kan inhoud binne 'n uitgebreide eienskap genaamd **com.apple.ResourceFork** binne 'n lêer stoor deur dit in **file/..namedfork/rsrc** te stoor.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
U kan **alle lêers wat hierdie uitgebreide attribuut bevat** vind met: 

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Die uitgebreide attribuut `com.apple.decmpfs` dui aan dat die lêer versleuteld gestoor word, `ls -l` sal 'n **grootte van 0** rapporteer en die gecomprimeerde data is binne hierdie attribuut. Wanneer die lêer toegang verkry, sal dit in geheue ontsleutel word.

Hierdie attribuut kan gesien word met `ls -lO` wat as gecomprimeerd aangedui word omdat gecomprimeerde lêers ook met die vlag `UF_COMPRESSED` gemerk is. As 'n gecomprimeerde lêer verwyder word met hierdie vlag met `chflags nocompressed </path/to/file>`, sal die stelsel nie weet dat die lêer gecomprimeerd was nie en daarom sal dit nie in staat wees om die data te ontsluit en toegang te verkry nie (dit sal dink dat dit eintlik leeg is).

Die hulpmiddel afscexpand kan gebruik word om 'n lêer te dwing om te ontsluit.

## **Universal binaries &** Mach-o Formaat

Mac OS lêers word gewoonlik gecompileer as **universal binaries**. 'n **universal binary** kan **meerdere argitekture in dieselfde lêer ondersteun**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS Proses Geheue

## macOS geheue dumping

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Risiko Kategorief lêers Mac OS

Die gids `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` is waar inligting oor die **risiko geassosieer met verskillende lêer uitbreidings gestoor word**. Hierdie gids kategoriseer lêers in verskillende risikoniveaus, wat beïnvloed hoe Safari hierdie lêers hanteer wanneer hulle afgelaai word. Die kategorieë is soos volg:

* **LSRiskCategorySafe**: Lêers in hierdie kategorie word beskou as **heeltemal veilig**. Safari sal hierdie lêers outomaties oopmaak nadat hulle afgelaai is.
* **LSRiskCategoryNeutral**: Hierdie lêers kom sonder waarskuwings en word **nie outomaties oopgemaak** deur Safari nie.
* **LSRiskCategoryUnsafeExecutable**: Lêers onder hierdie kategorie **aktiveer 'n waarskuwing** wat aandui dat die lêer 'n toepassing is. Dit dien as 'n sekuriteitsmaatreël om die gebruiker te waarsku.
* **LSRiskCategoryMayContainUnsafeExecutable**: Hierdie kategorie is vir lêers, soos argiewe, wat 'n uitvoerbare lêer mag bevat. Safari sal **'n waarskuwing aktiveer** tensy dit kan verifieer dat alle inhoud veilig of neutraal is.

## Log lêers

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Bevat inligting oor afgelaaide lêers, soos die URL waarvandaan hulle afgelaai is.
* **`/var/log/system.log`**: Hooflog van OSX stelsels. com.apple.syslogd.plist is verantwoordelik vir die uitvoering van syslogging (jy kan kyk of dit gedeaktiveer is deur te soek na "com.apple.syslogd" in `launchctl list`.
* **`/private/var/log/asl/*.asl`**: Dit is die Apple Stelsellogs wat dalk interessante inligting kan bevat.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Stoor onlangs toeganklike lêers en toepassings deur "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Stoor items om te begin by stelselaan.
* **`$HOME/Library/Logs/DiskUtility.log`**: Log lêer vir die DiskUtility App (inligting oor skywe, insluitend USB's)
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Data oor draadlose toegangspunte.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lys van gedeaktiveerde daemons.

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
