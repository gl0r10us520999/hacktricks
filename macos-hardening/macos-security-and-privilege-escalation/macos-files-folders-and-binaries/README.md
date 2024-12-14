# macOS Datoteke, Fascikle, Binarni & Memorija

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podrška HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Raspored hijerarhije datoteka

* **/Applications**: Instalirane aplikacije bi trebale biti ovde. Svi korisnici će moći da im pristupe.
* **/bin**: Binarne datoteke komandne linije
* **/cores**: Ako postoji, koristi se za čuvanje core dump-ova
* **/dev**: Sve se tretira kao datoteka, tako da ovde možete videti hardverske uređaje.
* **/etc**: Konfiguracione datoteke
* **/Library**: Ovdje se može naći mnogo poddirektorijuma i datoteka povezanih sa preferencama, kešom i logovima. Folder Library postoji u root-u i u direktorijumu svakog korisnika.
* **/private**: Nedokumentovano, ali mnoge od pomenutih fascikli su simboličke veze ka privatnom direktorijumu.
* **/sbin**: Osnovne sistemske binarne datoteke (vezane za administraciju)
* **/System**: Datoteka za pokretanje OS X. Ovde bi trebali pronaći uglavnom samo Apple specifične datoteke (ne treće strane).
* **/tmp**: Datoteke se brišu nakon 3 dana (to je soft link ka /private/tmp)
* **/Users**: Kućni direktorijum za korisnike.
* **/usr**: Konfiguracione i sistemske binarne datoteke
* **/var**: Log datoteke
* **/Volumes**: Montirani diskovi će se ovde pojaviti.
* **/.vol**: Pokretanjem `stat a.txt` dobijate nešto poput `16777223 7545753 -rw-r--r-- 1 username wheel ...` gde je prvi broj ID broj volumena gde datoteka postoji, a drugi je inode broj. Možete pristupiti sadržaju ove datoteke kroz /.vol/ sa tom informacijom pokretanjem `cat /.vol/16777223/7545753`

### Fascikle aplikacija

* **Sistemske aplikacije** se nalaze pod `/System/Applications`
* **Instalirane** aplikacije se obično instaliraju u `/Applications` ili u `~/Applications`
* **Podaci aplikacija** mogu se naći u `/Library/Application Support` za aplikacije koje se pokreću kao root i `~/Library/Application Support` za aplikacije koje se pokreću kao korisnik.
* Treće strane **daemoni** koji **moraju raditi kao root** obično se nalaze u `/Library/PrivilegedHelperTools/`
* **Sandboxed** aplikacije su mapirane u folder `~/Library/Containers`. Svaka aplikacija ima folder nazvan prema ID-u paketa aplikacije (`com.apple.Safari`).
* **Kernel** se nalazi u `/System/Library/Kernels/kernel`
* **Apple-ove kernel ekstenzije** se nalaze u `/System/Library/Extensions`
* **Treće strane kernel ekstenzije** se čuvaju u `/Library/Extensions`

### Datoteke sa osetljivim informacijama

MacOS čuva informacije kao što su lozinke na nekoliko mesta:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Ranljivi pkg instalateri

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X Specifične Ekstenzije

* **`.dmg`**: Apple Disk Image datoteke su vrlo česte za instalatere.
* **`.kext`**: Mora pratiti specifičnu strukturu i to je OS X verzija drajvera. (to je paket)
* **`.plist`**: Takođe poznat kao property list, čuva informacije u XML ili binarnom formatu.
* Može biti XML ili binarni. Binarne se mogu čitati sa:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Apple aplikacije koje prate strukturu direktorijuma (to je paket).
* **`.dylib`**: Dinamičke biblioteke (poput Windows DLL datoteka)
* **`.pkg`**: Iste su kao xar (eXtensible Archive format). Komanda instalater može se koristiti za instalaciju sadržaja ovih datoteka.
* **`.DS_Store`**: Ova datoteka se nalazi u svakom direktorijumu, čuva atribute i prilagođavanja direktorijuma.
* **`.Spotlight-V100`**: Ova fascikla se pojavljuje u root direktorijumu svakog volumena na sistemu.
* **`.metadata_never_index`**: Ako se ova datoteka nalazi u root-u volumena, Spotlight neće indeksirati taj volumen.
* **`.noindex`**: Datoteke i fascikle sa ovom ekstenzijom neće biti indeksirane od strane Spotlight-a.
* **`.sdef`**: Datoteke unutar paketa koje specificiraju kako je moguće interagovati sa aplikacijom iz AppleScript-a.

### macOS Paketi

Paket je **direktorijum** koji **izgleda kao objekat u Finder-u** (primer paketa su `*.app` datoteke).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Shared Library Cache (SLC)

Na macOS-u (i iOS-u) sve sistemske deljene biblioteke, poput framework-a i dylib-ova, su **kombinovane u jednu datoteku**, nazvanu **dyld shared cache**. Ovo poboljšava performanse, jer se kod može učitati brže.

Ovo se nalazi u macOS-u u `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` i u starijim verzijama možda ćete moći da pronađete **shared cache** u **`/System/Library/dyld/`**.\
U iOS-u ih možete pronaći u **`/System/Library/Caches/com.apple.dyld/`**.

Slično dyld shared cache-u, kernel i kernel ekstenzije su takođe kompajlirani u kernel cache, koji se učitava prilikom pokretanja.

Da biste izvukli biblioteke iz jedne datoteke dylib shared cache, bilo je moguće koristiti binarni [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) koji možda više ne radi, ali možete koristiti i [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

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
Napomena da čak i ako alat `dyld_shared_cache_util` ne radi, možete proslediti **deljeni dyld binarni fajl Hopper-u** i Hopper će moći da identifikuje sve biblioteke i da vam omogući da **izaberete koju želite da istražujete**:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Neki ekstraktori neće raditi jer su dylibs prelinkovani sa hardkodiranim adresama, pa bi mogli skakati na nepoznate adrese.

{% hint style="success" %}
Takođe je moguće preuzeti Shared Library Cache drugih \*OS uređaja u macos-u koristeći emulator u Xcode-u. Biće preuzeti unutar: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, kao: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Mapiranje SLC

**`dyld`** koristi syscall **`shared_region_check_np`** da zna da li je SLC mapiran (što vraća adresu) i **`shared_region_map_and_slide_np`** da mapira SLC.

Napomena da čak i ako je SLC pomeren pri prvom korišćenju, svi **procesi** koriste **istu kopiju**, što **ukida ASLR** zaštitu ako je napadač mogao da pokrene procese u sistemu. Ovo je zapravo iskorišćeno u prošlosti i ispravljeno sa shared region pager-om.

Branch pools su mali Mach-O dylibs koji kreiraju male prostore između mapiranja slika, čineći nemogućim da se funkcije međusobno preklapaju.

### Override SLCs

Korišćenjem env varijabli:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Ovo će omogućiti učitavanje novog shared library cache-a.
* **`DYLD_SHARED_CACHE_DIR=avoid`** i ručno zameniti biblioteke sa symlinkovima na deljeni cache sa pravim (biće potrebno da ih ekstraktujete).

## Specijalne dozvole za fajlove

### Dozvole za foldere

U **folderu**, **read** omogućava **listanje**, **write** omogućava **brisanje** i **pisanje** fajlova u njemu, a **execute** omogućava **prolazak** kroz direktorijum. Dakle, na primer, korisnik sa **dozvolom za čitanje nad fajlom** unutar direktorijuma gde **nema dozvolu za izvršavanje** **neće moći da pročita** fajl.

### Modifikatori zastavica

Postoje neke zastavice koje se mogu postaviti na fajlovima koje će učiniti da se fajl ponaša drugačije. Možete **proveriti zastavice** fajlova unutar direktorijuma sa `ls -lO /path/directory`

* **`uchg`**: Poznata kao **uchange** zastavica će **sprečiti bilo koju akciju** promene ili brisanja **fajla**. Da biste je postavili, uradite: `chflags uchg file.txt`
* Root korisnik može **ukloniti zastavicu** i izmeniti fajl.
* **`restricted`**: Ova zastavica čini da fajl bude **zaštićen SIP-om** (ne možete dodati ovu zastavicu na fajl).
* **`Sticky bit`**: Ako direktorijum ima sticky bit, **samo** **vlasnik direktorijuma ili root može preimenovati ili obrisati** fajlove. Obično se postavlja na /tmp direktorijum da bi se sprečilo da obični korisnici brišu ili premeste fajlove drugih korisnika.

Sve zastavice se mogu naći u fajlu `sys/stat.h` (pronađite ga koristeći `mdfind stat.h | grep stat.h`) i su:

* `UF_SETTABLE` 0x0000ffff: Mask of owner changeable flags.
* `UF_NODUMP` 0x00000001: Ne dump-uj fajl.
* `UF_IMMUTABLE` 0x00000002: Fajl se ne može menjati.
* `UF_APPEND` 0x00000004: Upisi u fajl mogu samo dodavati.
* `UF_OPAQUE` 0x00000008: Direktorijum je neprozirni u odnosu na uniju.
* `UF_COMPRESSED` 0x00000020: Fajl je komprimovan (neki fajl-sistemi).
* `UF_TRACKED` 0x00000040: Nema obaveštenja za brisanje/preimenovanje za fajlove sa ovim postavkama.
* `UF_DATAVAULT` 0x00000080: Potrebna je dozvola za čitanje i pisanje.
* `UF_HIDDEN` 0x00008000: Način da se naglasi da ovaj predmet ne bi trebao biti prikazan u GUI.
* `SF_SUPPORTED` 0x009f0000: Mask of superuser supported flags.
* `SF_SETTABLE` 0x3fff0000: Mask of superuser changeable flags.
* `SF_SYNTHETIC` 0xc0000000: Mask of system read-only synthetic flags.
* `SF_ARCHIVED` 0x00010000: Fajl je arhiviran.
* `SF_IMMUTABLE` 0x00020000: Fajl se ne može menjati.
* `SF_APPEND` 0x00040000: Upisi u fajl mogu samo dodavati.
* `SF_RESTRICTED` 0x00080000: Potrebna je dozvola za pisanje.
* `SF_NOUNLINK` 0x00100000: Predmet se ne može ukloniti, preimenovati ili montirati.
* `SF_FIRMLINK` 0x00800000: Fajl je firmlink.
* `SF_DATALESS` 0x40000000: Fajl je objekat bez podataka.

### **File ACLs**

File **ACLs** sadrže **ACE** (Access Control Entries) gde se mogu dodeliti **granularne dozvole** različitim korisnicima.

Moguće je dodeliti **direktorijumu** ove dozvole: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
A za **fajl**: `read`, `write`, `append`, `execute`.

Kada fajl sadrži ACLs, naći ćete **"+" kada listate dozvole kao u**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Možete **pročitati ACL-ove** datoteke sa:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Možete pronaći **sve datoteke sa ACL-ovima** sa (ovo je veoma sporo):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Extended Attributes

Proširene atribute imaju ime i bilo koju željenu vrednost, a mogu se videti koristeći `ls -@` i manipulisati koristeći komandu `xattr`. Neki uobičajeni prošireni atributi su:

* `com.apple.resourceFork`: Kompatibilnost sa resursnim fork-ovima. Takođe vidljivo kao `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Mehanizam karantina Gatekeeper-a (III/6)
* `metadata:*`: MacOS: razni metapodaci, kao što su `_backup_excludeItem`, ili `kMD*`
* `com.apple.lastuseddate` (#PS): Datum poslednje upotrebe datoteke
* `com.apple.FinderInfo`: MacOS: Informacije o Finder-u (npr., boje oznaka)
* `com.apple.TextEncoding`: Specifikuje kodiranje teksta ASCII datoteka
* `com.apple.logd.metadata`: Koristi se od strane logd na datotekama u `/var/db/diagnostics`
* `com.apple.genstore.*`: Generacijsko skladištenje (`/.DocumentRevisions-V100` u korenu datotečnog sistema)
* `com.apple.rootless`: MacOS: Koristi se od strane System Integrity Protection za označavanje datoteke (III/10)
* `com.apple.uuidb.boot-uuid`: logd oznake boot epoha sa jedinstvenim UUID
* `com.apple.decmpfs`: MacOS: Transparentna kompresija datoteka (II/7)
* `com.apple.cprotect`: \*OS: Podaci o enkripciji po datoteci (III/11)
* `com.apple.installd.*`: \*OS: Metapodaci koje koristi installd, npr., `installType`, `uniqueInstallID`

### Resource Forks | macOS ADS

Ovo je način da se dobiju **Alternativni podaci stream-ovi u MacOS** mašinama. Možete sačuvati sadržaj unutar proširenog atributa pod nazivom **com.apple.ResourceFork** unutar datoteke tako što ćete ga sačuvati u **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Možete **pronaći sve datoteke koje sadrže ovu proširenu atribut** sa: 

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Proširena atribut `com.apple.decmpfs` označava da je datoteka pohranjena enkriptovana, `ls -l` će prijaviti **veličinu 0** i kompresovani podaci su unutar ovog atributa. Kada god se datoteka pristupi, biće dekriptovana u memoriji.

Ovaj atribut se može videti sa `ls -lO` označen kao kompresovan jer su kompresovane datoteke takođe označene oznakom `UF_COMPRESSED`. Ako se kompresovana datoteka ukloni sa ovom oznakom `chflags nocompressed </path/to/file>`, sistem neće znati da je datoteka bila kompresovana i stoga neće moći da dekompresuje i pristupi podacima (misliće da je zapravo prazna).

Alat afscexpand može se koristiti za prisilno dekompresovanje datoteke.

## **Univerzalne binarne datoteke &** Mach-o Format

Mac OS binarne datoteke obično se kompajliraju kao **univerzalne binarne datoteke**. **Univerzalna binarna datoteka** može **podržavati više arhitektura u istoj datoteci**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS Procesna Memorija

## macOS iskopavanje memorije

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Kategorija Rizika Datoteke Mac OS

Direktorij `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` je mesto gde se čuva informacija o **riziku povezanom sa različitim ekstenzijama datoteka**. Ovaj direktorij kategorizuje datoteke u različite nivoe rizika, utičući na to kako Safari obrađuje ove datoteke prilikom preuzimanja. Kategorije su sledeće:

* **LSRiskCategorySafe**: Datoteke u ovoj kategoriji se smatraju **potpuno sigurnim**. Safari će automatski otvoriti ove datoteke nakon što budu preuzete.
* **LSRiskCategoryNeutral**: Ove datoteke dolaze bez upozorenja i **ne otvaraju se automatski** od strane Safarija.
* **LSRiskCategoryUnsafeExecutable**: Datoteke pod ovom kategorijom **pokreću upozorenje** koje ukazuje da je datoteka aplikacija. Ovo služi kao mera sigurnosti da upozori korisnika.
* **LSRiskCategoryMayContainUnsafeExecutable**: Ova kategorija je za datoteke, kao što su arhive, koje mogu sadržati izvršnu datoteku. Safari će **pokrenuti upozorenje** osim ako ne može da potvrdi da su svi sadržaji sigurni ili neutralni.

## Log datoteke

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Sadrži informacije o preuzetim datotekama, kao što je URL sa kojeg su preuzete.
* **`/var/log/system.log`**: Glavni log OSX sistema. com.apple.syslogd.plist je odgovoran za izvršavanje syslogging-a (možete proveriti da li je on onemogućen tražeći "com.apple.syslogd" u `launchctl list`).
* **`/private/var/log/asl/*.asl`**: Ovo su Apple sistemski logovi koji mogu sadržati zanimljive informacije.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Čuva nedavno pristupane datoteke i aplikacije putem "Finder-a".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Čuva stavke koje se pokreću prilikom pokretanja sistema.
* **`$HOME/Library/Logs/DiskUtility.log`**: Log datoteka za DiskUtility aplikaciju (informacije o diskovima, uključujući USB).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Podaci o bežičnim pristupnim tačkama.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lista deaktiviranih demona.

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
