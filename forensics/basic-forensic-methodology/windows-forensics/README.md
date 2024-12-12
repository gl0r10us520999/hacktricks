# Windows Artifakati

## Windows Artifakati

{% hint style="success" %}
Naučite i vežbajte AWS hakovanje: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte GCP hakovanje: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Generički Windows Artifakti

### Windows 10 Obaveštenja

Na putanji `\Users\<korisničko_ime>\AppData\Local\Microsoft\Windows\Notifications` možete pronaći bazu podataka `appdb.dat` (pre Windows godišnjice) ili `wpndatabase.db` (nakon Windows godišnjice).

Unutar ove SQLite baze podataka, možete pronaći tabelu `Notification` sa svim obaveštenjima (u XML formatu) koje mogu sadržati zanimljive podatke.

### Vremenska Linija

Vremenska linija je Windows karakteristika koja pruža **hronološku istoriju** posećenih web stranica, uređenih dokumenata i pokrenutih aplikacija.

Baza podataka se nalazi na putanji `\Users\<korisničko_ime>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db`. Ovu bazu podataka možete otvoriti sa alatom za SQLite ili sa alatom [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) **koji generiše 2 datoteke koje se mogu otvoriti sa alatom** [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md).

### ADS (Alternate Data Streams)

Preuzete datoteke mogu sadržati **ADS Zone.Identifier** koji ukazuje **kako** je datoteka **preuzeta** sa intraneta, interneta, itd. Neki softveri (poput pretraživača) obično dodaju **još** **informacija** poput **URL-a** sa kog je datoteka preuzeta.

## **Rezervne Kopije Datoteka**

### Kanta za Reciklažu

U Vista/Win7/Win8/Win10 **Kanta za Reciklažu** se može naći u fascikli **`$Recycle.bin`** u korenu drajva (`C:\$Recycle.bin`).\
Kada se datoteka obriše u ovoj fascikli, kreiraju se 2 specifične datoteke:

* `$I{id}`: Informacije o datoteci (datum kada je obrisana}
* `$R{id}`: Sadržaj datoteke

![](<../../../.gitbook/assets/image (486).png>)

Imajući ove datoteke, možete koristiti alat [**Rifiuti**](https://github.com/abelcheung/rifiuti2) da dobijete originalnu adresu obrisanih datoteka i datum kada su obrisane (koristite `rifiuti-vista.exe` za Vista – Win10).
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### Kopije senki zapisa

Senka kopije je tehnologija koja je uključena u Microsoft Windows i može kreirati **rezervne kopije** ili snimke fajlova ili volumena računara, čak i kada su u upotrebi.

Ove rezervne kopije obično se nalaze u `\System Volume Information` od korena fajl sistema, a ime je sastavljeno od **UID-ova** prikazanih na sledećoj slici:

![](<../../../.gitbook/assets/image (520).png>)

Montiranjem forenzičke slike sa **ArsenalImageMounter**-om, alat [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) može se koristiti za inspekciju senke kopije i čak **izvlačenje fajlova** iz rezervnih kopija senki.

![](<../../../.gitbook/assets/image (521).png>)

Unos u registar `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` sadrži fajlove i ključeve **koji se ne smeju rezervisati**:

![](<../../../.gitbook/assets/image (522).png>)

Registar `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` takođe sadrži informacije o konfiguraciji `Volume Shadow Copies`.

### Office automatski sačuvani fajlovi

Office automatski sačuvani fajlovi se mogu pronaći u: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## Stavke ljuske

Stavka ljuske je stavka koja sadrži informacije o tome kako pristupiti drugom fajlu.

### Nedavni dokumenti (LNK)

Windows **automatski** **kreira** ove **prečice** kada korisnik **otvori, koristi ili kreira fajl** u:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Office: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

Kada se kreira folder, takođe se kreira veza ka folderu, roditeljskom folderu i pradedinom folderu.

Ove automatski kreirane link datoteke **sadrže informacije o poreklu** kao da je to **fajl** **ili** **folder**, **MAC** **vremena** tog fajla, **informacije o volumenu** gde je fajl smešten i **folder ciljnog fajla**. Ove informacije mogu biti korisne za oporavak tih fajlova u slučaju da su uklonjeni.

Takođe, **datum kreiranja linka** datoteke je prvo **vreme** kada je originalni fajl **prvi** **put** **korišćen**, a **datum** **izmene** link datoteke je **poslednje** **vreme** kada je originalni fajl korišćen.

Za inspekciju ovih fajlova možete koristiti [**LinkParser**](http://4discovery.com/our-tools/).

U ovom alatu ćete pronaći **2 seta** vremenskih oznaka:

* **Prvi set:**
1. FileModifiedDate
2. FileAccessDate
3. FileCreationDate
* **Drugi set:**
1. LinkModifiedDate
2. LinkAccessDate
3. LinkCreationDate.

Prvi set vremenskih oznaka odnosi se na **vremenske oznake samog fajla**. Drugi set se odnosi na **vremenske oznake povezanog fajla**.

Možete dobiti iste informacije pokretanjem Windows CLI alata: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
### Jumplists

Ovo su nedavne datoteke koje su označene po aplikaciji. To je lista **nedavnih datoteka koje je koristila aplikacija** do kojih možete pristupiti u svakoj aplikaciji. Mogu se kreirati **automatski ili po želji**.

**Jumplists** koje se automatski kreiraju čuvaju se u `C:\Users\{korisničko_ime}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\`. Jumplisti se nazivaju prema formatu `{id}.autmaticDestinations-ms` gde je početni ID ID aplikacije.

Prilagođeni jumplisti čuvaju se u `C:\Users\{korisničko_ime}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` i obično ih kreira aplikacija jer se nešto **važno** dogodilo s datotekom (možda je označena kao omiljena).

**Vreme kreiranja** bilo kog jumplista pokazuje **prvi put kada je datoteka pristupljena** i **vreme poslednje izmene**.

Jumpliste možete pregledati koristeći [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md).

![](<../../../.gitbook/assets/image (474).png>)

(Napomena da su vremenske oznake koje pruža JumplistExplorer povezane sa samom datotekom jumplista)

### Shellbags

[**Pratite ovaj link da saznate šta su shellbags.**](interesting-windows-registry-keys.md#shellbags)

## Upotreba Windows USB uređaja

Moguće je identifikovati da je USB uređaj korišćen zahvaljujući kreiranju:

* Windows Recent fascikle
* Microsoft Office Recent fascikle
* Jumplisti

Imajte na umu da neke LNK datoteke umesto da pokazuju na originalnu putanju, pokazuju na fasciklu WPDNSE:

![](<../../../.gitbook/assets/image (476).png>)

Datoteke u fascikli WPDNSE su kopija originalnih datoteka, pa neće preživeti restart računara, a GUID se uzima iz shellbaga.

### Informacije iz registra

[Proverite ovu stranicu da saznate](interesting-windows-registry-keys.md#usb-information) koje registarske ključeve sadrže zanimljive informacije o povezanim USB uređajima.

### setupapi

Proverite datoteku `C:\Windows\inf\setupapi.dev.log` da biste dobili vremenske oznake kada je USB veza uspostavljena (tražite `Section start`).

![](<../../../.gitbook/assets/image (477) (2) (2) (2) (2) (2) (2) (2) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (14).png>)

### USB Detective

[**USBDetective**](https://usbdetective.com) se može koristiti za dobijanje informacija o USB uređajima koji su bili povezani sa slikom.

![](<../../../.gitbook/assets/image (483).png>)

### Čišćenje Plug and Play

Zakazani zadatak poznat kao 'Plug and Play Cleanup' je primarno dizajniran za uklanjanje zastarelih verzija drajvera. Za razliku od navedene svrhe zadržavanja najnovije verzije drajver paketa, online izvori sugerišu da takođe cilja drajvere koji su neaktivni 30 dana. Stoga, drajveri za prenosive uređaje koji nisu povezani u poslednjih 30 dana mogu biti podložni brisanju.

Zadatak se nalazi na sledećoj putanji:
`C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup`.

Prikazan je snimak ekrana sadržaja zadatka:
![](https://2.bp.blogspot.com/-wqYubtuR_W8/W19bV5S9XyI/AAAAAAAANhU/OHsBDEvjqmg9ayzdNwJ4y2DKZnhCdwSMgCLcBGAs/s1600/xml.png)

**Ključne komponente i podešavanja zadatka:**
- **pnpclean.dll**: Ova DLL je odgovorna za stvarni proces čišćenja.
- **UseUnifiedSchedulingEngine**: Postavljeno na `TRUE`, što ukazuje na korišćenje generičkog motora za zakazivanje zadataka.
- **MaintenanceSettings**:
- **Period ('P1M')**: Usmerava Task Scheduler da pokrene zadatak čišćenja mesečno tokom redovnog automatskog održavanja.
- **Deadline ('P2M')**: Nalaže Task Scheduleru da, ako zadatak ne uspe dva uzastopna meseca, izvrši zadatak tokom hitnog automatskog održavanja.

Ova konfiguracija obezbeđuje redovno održavanje i čišćenje drajvera, sa odredbama za ponovni pokušaj zadatka u slučaju uzastopnih neuspeha.

**Za više informacija pogledajte:** [**https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html**](https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html)

## Emailovi

Emailovi sadrže **2 interesantna dela: Zaglavlja i sadržaj** emaila. U **zaglavljima** možete pronaći informacije kao što su:

* **Ko** je poslao emailove (adresa e-pošte, IP, mail serveri koji su preusmerili email)
* **Kada** je email poslat

Takođe, unutar zaglavlja `References` i `In-Reply-To` možete pronaći ID poruka:

![](<../../../.gitbook/assets/image (484).png>)

### Windows Mail aplikacija

Ova aplikacija čuva emailove u HTML ili tekstualnom formatu. Emailove možete pronaći unutar podfoldera unutar `\Users\<korisničko_ime>\AppData\Local\Comms\Unistore\data\3\`. Emailovi se čuvaju sa ekstenzijom `.dat`.

**Metapodaci** emailova i **kontakti** mogu se pronaći unutar **EDB baze podataka**: `\Users\<korisničko_ime>\AppData\Local\Comms\UnistoreDB\store.vol`

**Promenite ekstenziju** datoteke sa `.vol` na `.edb` i možete koristiti alatku [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) da je otvorite. Unutar tabele `Message` možete videti emailove.

### Microsoft Outlook

Kada se koriste Exchange serveri ili Outlook klijenti, postojaće neka MAPI zaglavlja:

* `Mapi-Client-Submit-Time`: Vreme sistema kada je email poslat
* `Mapi-Conversation-Index`: Broj dečijih poruka niti i vremenska oznaka svake poruke niti
* `Mapi-Entry-ID`: Identifikator poruke.
* `Mappi-Message-Flags` i `Pr_last_Verb-Executed`: Informacije o MAPI klijentu (poruka pročitana? nepročitana? odgovorena? preusmerena? van kancelarije?)

U Microsoft Outlook klijentu, sve poslate/primljene poruke, podaci o kontaktima i podaci o kalendaru čuvaju se u PST datoteci u:

* `%USERPROFILE%\Local Settings\Application Data\Microsoft\Outlook` (WinXP)
* `%USERPROFILE%\AppData\Local\Microsoft\Outlook`

Putanja registra `HKEY_CURRENT_USER\Software\Microsoft\WindowsNT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook` ukazuje na datoteku koja se koristi.

Možete otvoriti PST datoteku koristeći alatku [**Kernel PST Viewer**](https://www.nucleustechnologies.com/es/visor-de-pst.html).

![](<../../../.gitbook/assets/image (485).png>)
### Microsoft Outlook OST Files

**OST fajl** se generiše od strane Microsoft Outlook-a kada je konfigurisan sa **IMAP** ili **Exchange** serverom, čuvajući slične informacije kao PST fajl. Ovaj fajl je sinhronizovan sa serverom, zadržavajući podatke za **poslednjih 12 meseci** do **maksimalne veličine od 50GB**, i nalazi se u istom direktorijumu kao PST fajl. Za pregled OST fajla, može se koristiti [**Kernel OST pregledač**](https://www.nucleustechnologies.com/ost-viewer.html).

### Dobijanje Priloga

Izgubljeni prilozi mogu biti povratni sa:

- Za **IE10**: `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook`
- Za **IE11 i novije**: `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook`

### Thunderbird MBOX Fajlovi

**Thunderbird** koristi **MBOX fajlove** za čuvanje podataka, smeštenih u `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles`.

### Sličice Slika

- **Windows XP i 8-8.1**: Pristupanje folderu sa sličicama generiše `thumbs.db` fajl koji čuva preglede slika, čak i nakon brisanja.
- **Windows 7/10**: `thumbs.db` se kreira prilikom pristupa preko mreže putem UNC putanje.
- **Windows Vista i noviji**: Pregledi sličica su centralizovani u `%userprofile%\AppData\Local\Microsoft\Windows\Explorer` sa fajlovima nazvanim **thumbcache\_xxx.db**. [**Thumbsviewer**](https://thumbsviewer.github.io) i [**ThumbCache Viewer**](https://thumbcacheviewer.github.io) su alati za pregledanje ovih fajlova.

### Informacije o Windows Registru

Windows Registry, koji čuva obimne podatke o sistemu i korisničkoj aktivnosti, nalazi se u fajlovima:

- `%windir%\System32\Config` za različite `HKEY_LOCAL_MACHINE` podključeve.
- `%UserProfile%{User}\NTUSER.DAT` za `HKEY_CURRENT_USER`.
- Windows Vista i novije verzije prave rezervne kopije `HKEY_LOCAL_MACHINE` registarskih fajlova u `%Windir%\System32\Config\RegBack\`.
- Dodatno, informacije o izvršenju programa se čuvaju u `%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT` od Windows Vista i Windows 2008 Server nadalje.

### Alati

Neki alati su korisni za analizu registarskih fajlova:

* **Registry Editor**: Instaliran je u Windows-u. To je grafički interfejs za navigaciju kroz Windows registar trenutne sesije.
* [**Registry Explorer**](https://ericzimmerman.github.io/#!index.md): Omogućava učitavanje registarskog fajla i navigaciju kroz njih pomoću grafičkog interfejsa. Takođe sadrži obeleživače koji ističu ključeve sa zanimljivim informacijama.
* [**RegRipper**](https://github.com/keydet89/RegRipper3.0): Ima grafički interfejs koji omogućava navigaciju kroz učitani registar i takođe sadrži dodatke koji ističu zanimljive informacije unutar učitanog registra.
* [**Windows Registry Recovery**](https://www.mitec.cz/wrr.html): Još jedna aplikacija sa grafičkim interfejsom sposobna da izvuče važne informacije iz učitanog registra.

### Obnavljanje Obrisanih Elemenata

Kada se ključ obriše, označava se kao takav, ali dok prostor koji zauzima nije potreban, neće biti uklonjen. Stoga, korišćenjem alata poput **Registry Explorer** moguće je povratiti ove obrisane ključeve.

### Vreme Poslednje Izmena

Svaki Ključ-Vrednost sadrži **vremensku oznaku** koja pokazuje kada je poslednji put bio izmenjen.

### SAM

Fajl/hive **SAM** sadrži heševe **korisnika, grupa i lozinki korisnika** sistema.

U `SAM\Domains\Account\Users` možete dobiti korisničko ime, RID, poslednju prijavu, poslednju neuspelu prijavu, brojač prijava, politiku lozinke i kada je nalog kreiran. Da biste dobili **heševe**, takođe **trebate** fajl/hive **SYSTEM**.

### Zanimljivi unosi u Windows Registru

{% content-ref url="interesting-windows-registry-keys.md" %}
[interesting-windows-registry-keys.md](interesting-windows-registry-keys.md)
{% endcontent-ref %}

## Izvršeni Programi

### Osnovni Windows Procesi

U [ovom postu](https://jonahacks.medium.com/investigating-common-windows-processes-18dee5f97c1d) možete saznati o zajedničkim Windows procesima za otkrivanje sumnjivih ponašanja.

### Nedavni Windows Aplikacije

Unutar registra `NTUSER.DAT` na putanji `Software\Microsoft\Current Version\Search\RecentApps` možete pronaći podključeve sa informacijama o **izvršenoj aplikaciji**, **poslednjem vremenu** kada je izvršena, i **broju puta** koliko je pokrenuta.

### BAM (Background Activity Moderator)

Možete otvoriti fajl `SYSTEM` sa registarskim editorom i unutar putanje `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}` možete pronaći informacije o **aplikacijama izvršenim od strane svakog korisnika** (obratite pažnju na `{SID}` u putanji) i u **koje vreme** su izvršene (vreme je unutar Vrednosti podataka registra).

### Windows Prefetch

Prefetching je tehnika koja omogućava računaru da tiho **preuzme neophodne resurse potrebne za prikaz sadržaja** kojem korisnik **može pristupiti u bliskoj budućnosti** kako bi resursi mogli biti pristupljeni brže.

Windows prefetch se sastoji od kreiranja **keševa izvršenih programa** kako bi ih mogao učitati brže. Ovi keševi se kreiraju kao `.pf` fajlovi unutar putanje: `C:\Windows\Prefetch`. Postoji ograničenje od 128 fajlova u XP/VISTA/WIN7 i 1024 fajlova u Win8/Win10.

Ime fajla se kreira kao `{ime_programa}-{heš}.pf` (heš je zasnovan na putanji i argumentima izvršne datoteke). U W10 ovi fajlovi su kompresovani. Imajte na umu da samo prisustvo fajla ukazuje da je **program izvršen** u nekom trenutku.

Fajl `C:\Windows\Prefetch\Layout.ini` sadrži **imena foldera fajlova koji su predviđeni**. Ovaj fajl sadrži **informacije o broju izvršenja**, **datumima** izvršenja i **fajlovima** **otvorenim** od strane programa.

Za pregled ovih fajlova možete koristiti alat [**PEcmd.exe**](https://github.com/EricZimmerman/PECmd):
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
![](<../../../.gitbook/assets/image (487).png>)

### Superprefetch

**Superprefetch** ima isti cilj kao prefetch, **ubrzava učitavanje programa** predviđajući šta će se učitati sledeće. Međutim, ne zamenjuje prefetch servis.\
Ovaj servis će generisati bazu podataka u `C:\Windows\Prefetch\Ag*.db`.

U ovim bazama podataka možete pronaći **ime programa**, **broj izvršavanja**, **otvorene datoteke**, **pristupane zapremine**, **potpunu putanju**, **vremenske okvire** i **vremenske oznake**.

Ove informacije možete pristupiti koristeći alat [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/).

### SRUM

**System Resource Usage Monitor** (SRUM) **prati** **resurse** **konzumirane** **od procesa**. Pojavio se u W8 i podatke čuva u ESE bazi podataka smeštenoj u `C:\Windows\System32\sru\SRUDB.dat`.

Daje sledeće informacije:

* AppID i Putanja
* Korisnik koji je izvršio proces
* Poslati bajtovi
* Primljeni bajtovi
* Mrežni interfejs
* Trajanje veze
* Trajanje procesa

Ove informacije se ažuriraju svakih 60 minuta.

Možete dobiti podatke iz ovog fajla koristeći alat [**srum\_dump**](https://github.com/MarkBaggett/srum-dump).
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**AppCompatCache**, poznat i kao **ShimCache**, čini deo **Baze podataka o kompatibilnosti aplikacija** koju je razvio **Microsoft** kako bi rešio probleme sa kompatibilnošću aplikacija. Ovaj sistemski komponent beleži različite delove metapodataka datoteka, uključujući:

- Puni put datoteke
- Veličina datoteke
- Vreme poslednje izmene pod **$Standard\_Information** (SI)
- Vreme poslednjeg ažuriranja ShimCache-a
- Zastava izvršenja procesa

Ovi podaci se čuvaju u registru na određenim lokacijama zavisno od verzije operativnog sistema:

- Za XP, podaci se čuvaju pod `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache` sa kapacitetom od 96 unosa.
- Za Server 2003, kao i za Windows verzije 2008, 2012, 2016, 7, 8 i 10, putanja skladištenja je `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache`, sa kapacitetom od 512 i 1024 unosa, redom.

Za analizu sačuvanih informacija, preporučuje se korišćenje alata [**AppCompatCacheParser**](https://github.com/EricZimmerman/AppCompatCacheParser).

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

Datoteka **Amcache.hve** je suštinski registarski hive koji beleži detalje o aplikacijama koje su izvršene na sistemu. Obično se nalazi na lokaciji `C:\Windows\AppCompat\Programas\Amcache.hve`.

Ova datoteka je značajna jer čuva zapise nedavno izvršenih procesa, uključujući putanje do izvršnih datoteka i njihove SHA1 heš vrednosti. Ove informacije su neprocenjive za praćenje aktivnosti aplikacija na sistemu.

Za ekstrakciju i analizu podataka iz **Amcache.hve**, može se koristiti alat [**AmcacheParser**](https://github.com/EricZimmerman/AmcacheParser). Sledeća komanda je primer kako koristiti AmcacheParser za analizu sadržaja datoteke **Amcache.hve** i izlaz rezultata u CSV formatu:
```bash
AmcacheParser.exe -f C:\Users\genericUser\Desktop\Amcache.hve --csv C:\Users\genericUser\Desktop\outputFolder
```
Među generisanim CSV fajlovima, `Amcache_Unassociated file entries` je posebno značajan zbog obilja informacija koje pruža o nepovezanim unosima fajlova.

Najinteresantniji CSV fajl koji je generisan je `Amcache_Unassociated file entries`.

### RecentFileCache

Ovaj artefakt može biti pronađen samo u W7 u `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` i sadrži informacije o nedavnom izvršavanju nekih binarnih fajlova.

Možete koristiti alat [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) za parsiranje fajla.

### Planirani zadaci

Možete ih izdvojiti iz `C:\Windows\Tasks` ili `C:\Windows\System32\Tasks` i čitati ih kao XML fajlove.

### Servisi

Možete ih pronaći u registru pod `SYSTEM\ControlSet001\Services`. Možete videti šta će biti izvršeno i kada.

### **Windows prodavnica**

Instalirane aplikacije se mogu pronaći u `\ProgramData\Microsoft\Windows\AppRepository\`\
Ova repozitorijum ima **log** sa **svakom instaliranom aplikacijom** u sistemu unutar baze podataka **`StateRepository-Machine.srd`**.

Unutar tabele Aplikacija ove baze podataka, moguće je pronaći kolone: "ID aplikacije", "Broj paketa" i "Prikazano ime". Ove kolone sadrže informacije o preinstaliranim i instaliranim aplikacijama i može se pronaći da li su neke aplikacije deinstalirane jer bi ID-jevi instaliranih aplikacija trebalo da budu uzastopni.

Takođe je moguće **pronaći instaliranu aplikaciju** unutar putanje registra: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
I **deinstalirane** **aplikacije** u: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Windows događaji

Informacije koje se pojavljuju unutar Windows događaja su:

* Šta se desilo
* Vremenska oznaka (UTC + 0)
* Uključeni korisnici
* Uključeni hostovi (ime računara, IP adresa)
* Resursi korišćeni (fajlovi, folderi, štampači, servisi)

Logovi se nalaze u `C:\Windows\System32\config` pre Windows Vista i u `C:\Windows\System32\winevt\Logs` posle Windows Vista. Pre Windows Vista, logovi događaja su bili u binarnom formatu, a posle toga su u **XML formatu** i koriste **.evtx** ekstenziju.

Lokacija fajlova događaja se može pronaći u registru SYSTEM pod **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

Mogu se vizualizovati pomoću Windows pregledača događaja (**`eventvwr.msc`**) ili drugim alatima poput [**Event Log Explorer**](https://eventlogxp.com) **ili** [**Evtx Explorer/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)**.**

## Razumevanje Windows sigurnosnog beleženja događaja

Pristupni događaji se beleže u sigurnosnom konfiguracionom fajlu koji se nalazi na lokaciji `C:\Windows\System32\winevt\Security.evtx`. Veličina ovog fajla je podesiva, i kada se dostigne kapacitet, stariji događaji se prepisuju. Zabeleženi događaji uključuju prijave i odjave korisnika, korisničke radnje, promene sigurnosnih postavki, kao i pristup fajlovima, folderima i deljenim resursima.

### Ključni ID-jevi događaja za autentifikaciju korisnika:

- **EventID 4624**: Ukazuje na uspešnu autentifikaciju korisnika.
- **EventID 4625**: Označava neuspešnu autentifikaciju.
- **EventIDs 4634/4647**: Predstavljaju događaje odjave korisnika.
- **EventID 4672**: Označava prijavu sa administratorskim privilegijama.

#### Pod-tipovi unutar EventID 4634/4647:

- **Interaktivno (2)**: Direktna korisnička prijava.
- **Mrežno (3)**: Pristup deljenim fasciklama.
- **Batch (4)**: Izvršavanje batch procesa.
- **Servis (5)**: Pokretanje servisa.
- **Proxy (6)**: Proksi autentifikacija.
- **Otključavanje (7)**: Ekran otključan lozinkom.
- **Mrežni čisti tekst (8)**: Prenos lozinke u čistom tekstu, često od strane IIS-a.
- **Nove akreditacije (9)**: Korišćenje različitih akreditacija za pristup.
- **Udaljeno interaktivno (10)**: Udaljena radna površina ili prijava na terminalne usluge.
- **Keš interaktivno (11)**: Prijava sa keširanim akreditacijama bez kontakta sa kontrolorom domena.
- **Keš udaljeno interaktivno (12)**: Udaljena prijava sa keširanim akreditacijama.
- **Keš otključavanje (13)**: Otključavanje sa keširanim akreditacijama.

#### Status i pod-status kodovi za EventID 4625:

- **0xC0000064**: Korisničko ime ne postoji - Može ukazivati na napad enumeracije korisničkih imena.
- **0xC000006A**: Ispravno korisničko ime ali pogrešna lozinka - Moguć pokušaj pogađanja lozinke ili napad grubom silom.
- **0xC0000234**: Korisnički nalog zaključan - Može pratiti napad grubom silom koji rezultira višestrukim neuspešnim prijavama.
- **0xC0000072**: Nalog onemogućen - Neovlašćeni pokušaji pristupa onemogućenim nalozima.
- **0xC000006F**: Prijavljivanje van dozvoljenog vremena - Ukazuje na pokušaje pristupa van postavljenih vremena prijave, mogući znak neovlašćenog pristupa.
- **0xC0000070**: Kršenje ograničenja radne stanice - Može biti pokušaj prijave sa neovlašćene lokacije.
- **0xC0000193**: Istečen nalog - Pokušaji pristupa sa isteklim korisničkim nalozima.
- **0xC0000071**: Istačena lozinka - Pokušaji prijave sa zastarelim lozinkama.
- **0xC0000133**: Problemi sa sinhronizacijom vremena - Velike razlike u vremenu između klijenta i servera mogu ukazivati na sofisticiranije napade poput "pass-the-ticket".
- **0xC0000224**: Obavezna promena lozinke - Česte obavezne promene mogu ukazivati na pokušaj destabilizacije sigurnosti naloga.
- **0xC0000225**: Ukazuje na problem u sistemu umesto sigurnosnog problema.
- **0xC000015b**: Odbijen tip prijave - Pokušaj pristupa sa neovlašćenim tipom prijave, kao što je korisnik koji pokušava da izvrši prijavu servisa.

#### EventID 4616:
- **Promena vremena**: Modifikacija sistemskog vremena, može zamagliti vremensku liniju događaja.

#### EventID 6005 i 6006:
- **Pokretanje i gašenje sistema**: EventID 6005 označava pokretanje sistema, dok EventID 6006 označava gašenje sistema.

#### EventID 1102:
- **Brisanje logova**: Sigurnosni logovi se brišu, što često ukazuje na prikrivanje nezakonitih aktivnosti.

#### EventID-ovi za praćenje USB uređaja:
- **20001 / 20003 / 10000**: Prvo povezivanje USB uređaja.
- **10100**: Ažuriranje drajvera USB uređaja.
- **EventID 112**: Vreme umetanja USB uređaja.

Za praktične primere simuliranja ovih tipova prijava i prilika za izvlačenje akreditacija, pogledajte [detaljni vodič Altered Security-a](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them).

Detalji događaja, uključujući statusne i pod-statusne kodove, pružaju dodatne uvide u uzroke događaja, posebno značajne u Event ID 4625.

### Obnavljanje Windows događaja

Da biste povećali šanse za obnavljanje obrisanih Windows događaja, preporučljivo je isključiti sumnjivi računar direktnim isključivanjem. **Bulk_extractor**, alat za obnavljanje koji specifično navodi ekstenziju `.evtx`, preporučuje se za pokušaj obnavljanja takvih događaja.

### Identifikacija uobičajenih napada putem Windows događaja

Za sveobuhvatan vodič o korišćenju Windows Event ID-jeva u identifikaciji uobičajenih sajber napada, posetite [Red Team Recipe](https://redteamrecipe.com/event-codes/).

#### Napadi grubom silom

Identifikovani sa više zapisa EventID 4625, praćeni EventID 4624 ako napad uspe.

#### Promena vremena

Zabeleženo sa EventID 4616, promene u sistemu vremena mogu otežati forenzičku analizu.

#### Praćenje USB uređaja

Korisni System EventID-ovi za praćenje USB uređaja uključuju 20001/20003/10000 za početno korišćenje, 10100 za ažuriranje drajvera i EventID 112 od DeviceSetupManager za vremenske oznake umetanja.
#### Događaji napajanja sistema

EventID 6005 označava pokretanje sistema, dok EventID 6006 označava gašenje.

#### Brisanje logova

Bezbednosni EventID 1102 signalizira brisanje logova, što je ključni događaj za forenzičku analizu.

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}
