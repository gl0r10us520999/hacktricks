# macOS Fajlovi, Folderi, Binarni fajlovi i Memorija

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Struktura hijerarhije fajlova

* **/Applications**: Instalirane aplikacije treba da budu ovde. Svi korisnici će imati pristup njima.
* **/bin**: Binarni fajlovi komandne linije
* **/cores**: Ako postoji, koristi se za čuvanje core dump-ova
* **/dev**: Sve se tretira kao fajl, pa možete videti hardverske uređaje ovde.
* **/etc**: Konfiguracioni fajlovi
* **/Library**: Mnogo poddirektorijuma i fajlova vezanih za postavke, keš i logove se mogu naći ovde. Postoji Library folder u root-u i u svakom korisničkom direktorijumu.
* **/private**: Nedokumentovano, ali mnogi pomenuti folderi su simboličke veze ka privatnom direktorijumu.
* **/sbin**: Bitni sistemski binarni fajlovi (vezani za administraciju)
* **/System**: Fajl za pokretanje OS X-a. Trebalo bi da ovde uglavnom pronađete samo Apple specifične fajlove (ne treće strane).
* **/tmp**: Fajlovi se brišu nakon 3 dana (to je soft link ka /private/tmp)
* **/Users**: Matični direktorijum za korisnike.
* **/usr**: Konfiguracioni i sistemski binarni fajlovi
* **/var**: Log fajlovi
* **/Volumes**: Montirani drajvovi će se pojaviti ovde.
* **/.vol**: Pokretanjem `stat a.txt` dobijate nešto poput `16777223 7545753 -rw-r--r-- 1 username wheel ...` gde je prvi broj ID broj volumena gde fajl postoji, a drugi je broj inode-a. Možete pristupiti sadržaju ovog fajla kroz /.vol/ sa tim informacijama pokretanjem `cat /.vol/16777223/7545753`

### Folderi Aplikacija

* **Sistemski aplikacije** se nalaze pod `/System/Applications`
* **Instalirane** aplikacije obično se instaliraju u `/Applications` ili u `~/Applications`
* **Podaci aplikacije** se mogu naći u `/Library/Application Support` za aplikacije koje se izvršavaju kao root i `~/Library/Application Support` za aplikacije koje se izvršavaju kao korisnik.
* Demoni **trećih strana** aplikacije koji **mora da se izvršavaju kao root** obično se nalaze u `/Library/PrivilegedHelperTools/`
* **Sandbox** aplikacije su mapirane u folder `~/Library/Containers`. Svaka aplikacija ima folder nazvan prema ID-u paketa aplikacije (`com.apple.Safari`).
* **Kernel** se nalazi u `/System/Library/Kernels/kernel`
* **Apple-ove kernel ekstenzije** se nalaze u `/System/Library/Extensions`
* **Ekstenzije kernela trećih strana** se čuvaju u `/Library/Extensions`

### Fajlovi sa Osetljivim Informacijama

macOS čuva informacije poput lozinki na nekoliko mesta:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Ranjivi pkg instalateri

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X Specifične Ekstenzije

* **`.dmg`**: Fajlovi Apple Disk Image su veoma česti za instalatere.
* **`.kext`**: Mora pratiti specifičnu strukturu i to je OS X verzija drajvera. (to je paket)
* **`.plist`**: Takođe poznat kao property list čuva informacije u XML ili binarnom formatu.
* Može biti XML ili binarni. Binarni se mogu čitati sa:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Apple aplikacije koje prate strukturu direktorijuma (To je paket).
* **`.dylib`**: Dinamičke biblioteke (kao Windows DLL fajlovi)
* **`.pkg`**: Isti su kao xar (eXtensible Archive format). Komanda installer se može koristiti za instaliranje sadržaja ovih fajlova.
* **`.DS_Store`**: Ovaj fajl se nalazi u svakom direktorijumu, čuva atribute i prilagođavanja direktorijuma.
* **`.Spotlight-V100`**: Ovaj folder se pojavljuje na korenskom direktorijumu svakog volumena na sistemu.
* **`.metadata_never_index`**: Ako se ovaj fajl nalazi na korenu volumena, Spotlight neće indeksirati taj volumen.
* **`.noindex`**: Fajlovi i folderi sa ovom ekstenzijom neće biti indeksirani od strane Spotlight-a.
* **`.sdef`**: Fajlovi unutar paketa koji specificiraju kako je moguće interagovati sa aplikacijom putem AppleScript-a.

### macOS Paketi

Paket je **direktorijum** koji **izgleda kao objekat u Finder-u** (primer paketa su fajlovi `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Deljeni Keš Biblioteka (SLC)

Na macOS-u (i iOS-u) sve sistemske deljene biblioteke, poput okvira i dylib-ova, **kombinovane su u jedan fajl**, nazvan **dyld deljeni keš**. Ovo poboljšava performanse, jer se kod može učitati brže.

Ovo se nalazi na macOS-u u `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` i u starijim verzijama možda možete pronaći **deljeni keš** u **`/System/Library/dyld/`**.\
Na iOS-u ih možete pronaći u **`/System/Library/Caches/com.apple.dyld/`**.

Slično dyld deljenom kešu, kernel i kernel ekstenzije takođe su kompilovane u keš kernela, koji se učitava prilikom pokretanja.

Da biste izvukli biblioteke iz jednog fajla dylib deljenog keša, bilo je moguće koristiti binarni [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) koji možda ne radi više, ali možete koristiti i [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

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
Imajte na umu da čak i ako alat `dyld_shared_cache_util` ne radi, možete proslediti **deljeni dyld binarni fajl Hopper-u** i Hopper će moći da identifikuje sve biblioteke i omogući vam da **izaberete koju** želite da istražite:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Neki ekstraktori neće raditi jer su dylibs prelinkovani sa hardkodiranim adresama i zbog toga mogu skakati na nepoznate adrese.

{% hint style="success" %}
Takođe je moguće preuzeti Deljeni bibliotečki keš drugih \*OS uređaja na macOS-u koristeći emulator u Xcode-u. Biće preuzeti unutar: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, kao:`$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Mapiranje SLC

**`dyld`** koristi sistemski poziv **`shared_region_check_np`** da bi znao da li je SLC mapiran (što vraća adresu) i **`shared_region_map_and_slide_np`** da mapira SLC.

Imajte na umu da čak i ako je SLC kliznuo pri prvom korišćenju, svi **procesi** koriste **isti primerak**, što **eliminiše ASLR** zaštitu ako je napadač bio u mogućnosti da pokrene procese u sistemu. Ovo je zapravo iskorišćeno u prošlosti i popravljeno sa deljenim region pagerom.

Grane bazena su male Mach-O dylibs koje stvaraju male prostore između mapiranja slika čineći nemogućim interponovanje funkcija.

### Poništavanje SLC-a

Korišćenjem okruženjskih promenljivih:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Ovo će omogućiti učitavanje novog deljenog bibliotečkog keša
* **`DYLD_SHARED_CACHE_DIR=avoid`** i ručno zamenite biblioteke simboličkim linkovima ka deljenom kešu sa pravim (morate ih izdvojiti)

## Posebne Dozvole za Fajlove

### Dozvole za foldere

U **folderu**, **čitanje** omogućava da ga **listate**, **pisanje** omogućava da ga **obrišete** i **pišete** fajlove na njemu, a **izvršavanje** omogućava da **pretražujete** direktorijum. Na primer, korisnik sa **dozvolom za čitanje nad fajlom** unutar direktorijuma gde **nema dozvolu za izvršavanje** **neće moći da pročita** fajl.

### Modifikatori zastavica

Postoje neke zastavice koje se mogu postaviti u fajlovima koje će fajl ponašati drugačije. Možete **proveriti zastavice** fajlova unutar direktorijuma sa `ls -lO /path/directory`

* **`uchg`**: Poznat kao **uchange** flag će **sprečiti bilo koju akciju** menjanja ili brisanja **fajla**. Da biste ga postavili uradite: `chflags uchg file.txt`
* Korisnik root može **ukloniti zastavicu** i izmeniti fajl
* **`restricted`**: Ova zastavica čini da fajl bude **zaštićen SIP-om** (ne možete dodati ovu zastavicu fajlu).
* **`Sticky bit`**: Ako je direktorijum sa sticky bitom, **samo** vlasnik direktorijuma ili root mogu preimenovati ili obrisati fajlove. Obično je postavljeno na /tmp direktorijum da spreči obične korisnike da brišu ili premeštaju fajlove drugih korisnika.

Sve zastavice se mogu naći u fajlu `sys/stat.h` (pronađite ga koristeći `mdfind stat.h | grep stat.h`) i to su:

* `UF_SETTABLE` 0x0000ffff: Mask of owner changeable flags.
* `UF_NODUMP` 0x00000001: Do not dump file.
* `UF_IMMUTABLE` 0x00000002: File may not be changed.
* `UF_APPEND` 0x00000004: Writes to file may only append.
* `UF_OPAQUE` 0x00000008: Directory is opaque wrt. union.
* `UF_COMPRESSED` 0x00000020: File is compressed (some file-systems).
* `UF_TRACKED` 0x00000040: No notifications for deletes/renames for files with this set.
* `UF_DATAVAULT` 0x00000080: Entitlement required for reading and writing.
* `UF_HIDDEN` 0x00008000: Hint that this item should not be displayed in a GUI.
* `SF_SUPPORTED` 0x009f0000: Mask of superuser supported flags.
* `SF_SETTABLE` 0x3fff0000: Mask of superuser changeable flags.
* `SF_SYNTHETIC` 0xc0000000: Mask of system read-only synthetic flags.
* `SF_ARCHIVED` 0x00010000: File is archived.
* `SF_IMMUTABLE` 0x00020000: File may not be changed.
* `SF_APPEND` 0x00040000: Writes to file may only append.
* `SF_RESTRICTED` 0x00080000: Entitlement required for writing.
* `SF_NOUNLINK` 0x00100000: Item may not be removed, renamed or mounted on.
* `SF_FIRMLINK` 0x00800000: File is a firmlink.
* `SF_DATALESS` 0x40000000: File is dataless object.

### **Fajl ACLs**

Fajl **ACLs** sadrže **ACE** (Access Control Entries) gde se mogu dodeliti više **granularnih dozvola** različitim korisnicima.

Moguće je dodeliti **direktorijumu** ove dozvole: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
I fajlu: `read`, `write`, `append`, `execute`.

Kada fajl sadrži ACLs, videćete **"+" kada nabrajate dozvole kao u**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Možete **pročitati ACL-ove** datoteke pomoću:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Možete pronaći **sve datoteke sa ACL-ovima** sa (ovo je veeery sporo):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Prošireni atributi

Prošireni atributi imaju ime i željenu vrednost, mogu se videti korišćenjem `ls -@` i manipulisati korišćenjem `xattr` komande. Neki od čestih proširenih atributa su:

* `com.apple.resourceFork`: Kompatibilnost resursnih viljuški. Takođe vidljivo kao `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Mehanizam karantina Gatekeeper-a (III/6)
* `metadata:*`: MacOS: razni metapodaci, kao što su `_backup_excludeItem`, ili `kMD*`
* `com.apple.lastuseddate` (#PS): Datum poslednje upotrebe fajla
* `com.apple.FinderInfo`: MacOS: Informacije Findera (npr. boja oznaka)
* `com.apple.TextEncoding`: Određuje tekstualno kodiranje ASCII tekstualnih fajlova
* `com.apple.logd.metadata`: Korišćeno od strane logd na fajlovima u `/var/db/diagnostics`
* `com.apple.genstore.*`: Generacijsko skladište (`/.DocumentRevisions-V100` u korenu fajl sistema)
* `com.apple.rootless`: MacOS: Korišćeno od strane Sistema zaštite integriteta za obeležavanje fajla (III/10)
* `com.apple.uuidb.boot-uuid`: Obeležavanje boot epoha sa jedinstvenim UUID-om od strane logd-a
* `com.apple.decmpfs`: MacOS: Transparentna kompresija fajlova (II/7)
* `com.apple.cprotect`: \*OS: Podaci o šifrovanju po fajlu (III/11)
* `com.apple.installd.*`: \*OS: Metapodaci korišćeni od strane installd-a, npr., `installType`, `uniqueInstallID`

### Resursne viljuške | macOS ADS

Ovo je način da se dobiju **Alternativni podaci u toku u MacOS** mašinama. Možete sačuvati sadržaj unutar proširenog atributa nazvanog **com.apple.ResourceFork** unutar fajla tako što ga sačuvate u **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Možete **pronaći sve datoteke koje sadrže ovaj prošireni atribut** sa: 

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Prošireni atribut `com.apple.decmpfs` ukazuje da je datoteka pohranjena šifrovano, `ls -l` će prijaviti **veličinu 0** i komprimirani podaci su unutar ovog atributa. Svaki put kada se datoteka pristupi, biće dešifrovana u memoriji.

Ovaj atribut može se videti sa `ls -lO` označen kao komprimiran jer su komprimirane datoteke takođe označene zastavicom `UF_COMPRESSED`. Ako se komprimirana datoteka ukloni ova zastavica sa `chflags nocompressed </putanja/do/datoteke>`, sistem neće znati da je datoteka bila komprimirana i stoga neće moći da dekomprimuje i pristupi podacima (misliće da je zapravo prazna).

Alat afscexpand može se koristiti da prisili dekompresiju datoteke.

## **Univerzalne binarne datoteke &** Mach-o Format

Binarne datoteke Mac OS obično su kompajlirane kao **univerzalne binarne datoteke**. **Univerzalna binarna datoteka** može **podržavati više arhitektura u istoj datoteci**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS Proces memorije

## Dumpovanje memorije macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Kategorija rizika datoteka Mac OS

Direktorijum `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` je mesto gde se čuva informacija o **riziku povezanom sa različitim ekstenzijama datoteka**. Ovaj direktorijum kategorizuje datoteke u različite nivoe rizika, utičući na to kako Safari obrađuje ove datoteke prilikom preuzimanja. Kategorije su sledeće:

* **LSRiskCategorySafe**: Datoteke u ovoj kategoriji se smatraju **potpuno sigurnim**. Safari će automatski otvoriti ove datoteke nakon što budu preuzete.
* **LSRiskCategoryNeutral**: Ove datoteke ne dolaze sa upozorenjima i **ne otvaraju se automatski** u Safariju.
* **LSRiskCategoryUnsafeExecutable**: Datoteke u ovoj kategoriji **pokreću upozorenje** koje ukazuje da je datoteka aplikacija. Ovo služi kao sigurnosna mera da upozori korisnika.
* **LSRiskCategoryMayContainUnsafeExecutable**: Ova kategorija je za datoteke, poput arhiva, koje mogu sadržati izvršnu datoteku. Safari će **pokrenuti upozorenje** osim ako može da potvrdi da su svi sadržaji sigurni ili neutralni.

## Log datoteke

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Sadrži informacije o preuzetim datotekama, poput URL adrese sa koje su preuzete.
* **`/var/log/system.log`**: Glavni log OSX sistema. com.apple.syslogd.plist je odgovoran za izvršavanje sistemskog logovanja (možete proveriti da li je onemogućen traženjem "com.apple.syslogd" u `launchctl list`.
* **`/private/var/log/asl/*.asl`**: Ovo su Apple System Logs koji mogu sadržati zanimljive informacije.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Čuva nedavno pristupljene datoteke i aplikacije putem "Finder"-a.
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Čuva stavke koje se pokreću prilikom pokretanja sistema.
* **`$HOME/Library/Logs/DiskUtility.log`**: Log datoteka za DiskUtility aplikaciju (informacije o drajvovima, uključujući USB-ove).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Podaci o bežičnim pristupnim tačkama.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lista deaktiviranih demona.

{% hint style="success" %}
Naučite & vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite & vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili **telegram grupi** ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
