# Analiza firmware-a

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## **Uvod**

Firmware je osnovni softver koji omogućava uređajima pravilno funkcionisanje upravljajući i olakšavajući komunikaciju između hardverskih komponenti i softvera sa kojim korisnici interaguju. Čuva se u trajnoj memoriji, obezbeđujući uređaju pristup vitalnim instrukcijama od trenutka kada se uključi, što dovodi do pokretanja operativnog sistema. Ispitivanje i potencijalno modifikovanje firmware-a ključan je korak u identifikaciji bezbednosnih ranjivosti.

## **Prikupljanje informacija**

**Prikupljanje informacija** je ključni početni korak u razumevanju sastava uređaja i tehnologija koje koristi. Ovaj proces uključuje prikupljanje podataka o:

* Arhitekturi CPU-a i operativnom sistemu koji pokreće
* Specifičnostima bootloader-a
* Rasporedu hardvera i tehničkim listovima
* Metrikama koda i lokacijama izvornog koda
* Spoljnim bibliotekama i tipovima licenci
* Istorijatu ažuriranja i regulatornim sertifikatima
* Arhitektonskim i dijagramima toka
* Bezbednosnim procenama i identifikovanim ranjivostima

Za ovu svrhu, alati za **otvorenu obaveštajnu službu (OSINT)** su neprocenjivi, kao i analiza dostupnih komponenti otvorenog koda putem ručnih i automatskih pregleda. Alati poput [Coverity Scan](https://scan.coverity.com) i [Semmle’s LGTM](https://lgtm.com/#explore) nude besplatnu statičku analizu koja se može iskoristiti za pronalaženje potencijalnih problema.

## **Dobijanje firmware-a**

Dobijanje firmware-a može se pristupiti na različite načine, svaki sa svojim nivoom složenosti:

* **Direktno** od izvora (programeri, proizvođači)
* **Izgradnja** prema pruženim instrukcijama
* **Preuzimanje** sa zvaničnih sajtova podrške
* Korišćenje **Google dork** upita za pronalaženje hostovanih firmware fajlova
* Direktno pristupanje **cloud skladištu**, sa alatima poput [S3Scanner](https://github.com/sa7mon/S3Scanner)
* Presretanje **ažuriranja** putem tehnika čovek-u-sredini
* **Izdvajanje** sa uređaja putem veza poput **UART**, **JTAG**, ili **PICit**
* **Špijuniranje** za zahtevima za ažuriranje unutar komunikacije uređaja
* Identifikovanje i korišćenje **hardkodiranih tačaka za ažuriranje**
* **Izdvajanje** iz bootloader-a ili mreže
* **Uklanjanje i čitanje** čipa za skladištenje, kada sve drugo zakaže, korišćenjem odgovarajućih hardverskih alata

## Analiza firmware-a

Sada kada **imate firmware**, treba da izvučete informacije o njemu kako biste znali kako da ga obradite. Različiti alati koje možete koristiti za to:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```
Ako ne pronađete mnogo sa tim alatima, proverite **entropiju** slike pomoću `binwalk -E <bin>`, ako je entropija niska, tada verovatno nije šifrovana. Ako je entropija visoka, verovatno je šifrovana (ili kompresovana na neki način).

Osim toga, možete koristiti ove alate za izdvajanje **datoteka ugrađenih u firmware**:

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

Ili [**binvis.io**](https://binvis.io/#/) ([kod](https://code.google.com/archive/p/binvis/)) za inspekciju datoteke.

### Dobijanje fajl sistema

Pomoću prethodno komentarisanih alata poput `binwalk -ev <bin>` trebalo bi da ste mogli da **izvučete fajl sistem**.\
Binwalk obično izvlači unutar **foldera nazvanog po tipu fajl sistema**, koji obično može biti jedan od sledećih: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs.

#### Ručno izdvajanje fajl sistema

Ponekad, binwalk **neće imati magični bajt fajl sistema u svojim potpisima**. U tim slučajevima, koristite binwalk da **pronađete offset fajl sistema i izdvojite kompresovani fajl sistem** iz binarnog fajla i **ručno izdvojite** fajl sistem prema njegovom tipu koristeći korake ispod.
```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```
Izvršite sledeću **dd komandu** za izdvajanje Squashfs fajl sistema.
```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```
Alternativno, mogla bi se pokrenuti i sledeća komanda.

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* Za squashfs (korišćeno u primeru iznad)

`$ unsquashfs dir.squashfs`

Datoteke će biti u direktorijumu "`squashfs-root`" nakon toga.

* CPIO arhivske datoteke

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* Za jffs2 fajl sisteme

`$ jefferson rootfsfile.jffs2`

* Za ubifs fajl sisteme sa NAND flešom

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`

## Analiza Firmware-a

Kada se firmware dobije, bitno je razložiti ga radi razumevanja njegove strukture i potencijalnih ranjivosti. Ovaj proces uključuje korišćenje različitih alata za analizu i izvlačenje vrednih podataka iz firmware slike.

### Alati za Početnu Analizu

Skup komandi je obezbeđen za početnu inspekciju binarnog fajla (nazvanog `<bin>`). Ove komande pomažu u identifikovanju tipova fajlova, izvlačenju stringova, analizi binarnih podataka i razumevanju detalja particija i fajl sistema:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #prints offsets in hexadecimal
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head #useful for finding signatures in the header
fdisk -lu <bin> #lists partitions and filesystems, if there are multiple
```
Za procenu statusa šifrovanja slike, **entropija** se proverava pomoću `binwalk -E <bin>`. Niska entropija ukazuje na nedostatak šifrovanja, dok visoka entropija ukazuje na moguće šifrovanje ili kompresiju.

Za izdvajanje **ugrađenih datoteka**, preporučuju se alati i resursi poput dokumentacije **file-data-carving-recovery-tools** i **binvis.io** za inspekciju datoteka.

### Izdvajanje fajl sistema

Korišćenjem `binwalk -ev <bin>`, obično se može izdvojiti fajl sistem, često u direktorijumu nazvanom po tipu fajl sistema (npr. squashfs, ubifs). Međutim, kada **binwalk** ne uspe da prepozna tip fajl sistema zbog nedostajućih magičnih bajtova, potrebno je ručno izdvajanje. To uključuje korišćenje `binwalk` za lociranje ofseta fajl sistema, a zatim korišćenje `dd` komande za izdvajanje fajl sistema:
```bash
$ binwalk DIR850L_REVB.bin

$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs
```
### Analiza fajl sistema

Sa izvučenim fajl sistemom, počinje potraga za bezbednosnim propustima. Pažnja se posvećuje nesigurnim mrežnim demonima, hardkodiranim pristupnim podacima, API endpointima, funkcionalnostima ažuriranja servera, nekompajliranom kodu, startap skriptama i kompajliranim binarnim fajlovima za offline analizu.

**Ključne lokacije** i **stavke** koje treba pregledati uključuju:

- **etc/shadow** i **etc/passwd** za korisničke podatke
- SSL sertifikate i ključeve u **etc/ssl**
- Konfiguracione i skript fajlove za potencijalne ranjivosti
- Ugrađene binarne fajlove za dalju analizu
- Uobičajene IoT veb servere i binarne fajlove

Nekoliko alata pomaže u otkrivanju osetljivih informacija i ranjivosti unutar fajl sistema:

- [**LinPEAS**](https://github.com/carlospolop/PEASS-ng) i [**Firmwalker**](https://github.com/craigz28/firmwalker) za pretragu osetljivih informacija
- [**Alat za analizu i poređenje firmware-a (FACT)**](https://github.com/fkie-cad/FACT\_core) za sveobuhvatnu analizu firmware-a
- [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer), [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep), [**ByteSweep-go**](https://gitlab.com/bytesweep/bytesweep-go) i [**EMBA**](https://github.com/e-m-b-a/emba) za statičku i dinamičku analizu

### Provere bezbednosti kompajliranih binarnih fajlova

Izvorni kod i kompajlirani binarni fajlovi pronađeni u fajl sistemu moraju biti pažljivo pregledani zbog ranjivosti. Alati poput **checksec.sh** za Unix binarne fajlove i **PESecurity** za Windows binarne fajlove pomažu u identifikaciji nezaštićenih binarnih fajlova koji bi mogli biti iskorišćeni.

## Emulacija firmware-a za dinamičku analizu

Proces emulacije firmware-a omogućava **dinamičku analizu** ili rada uređaja ili pojedinačnog programa. Ovaj pristup može naići na izazove sa hardverom ili zavisnostima arhitekture, ali prenos fajl sistema korena ili određenih binarnih fajlova na uređaj sa odgovarajućom arhitekturom i endianstvom, poput Raspberry Pi-ja, ili na prethodno izgrađenu virtuelnu mašinu, može olakšati dalje testiranje.

### Emulacija pojedinačnih binarnih fajlova

Za ispitivanje pojedinačnih programa, ključno je identifikovati endianstvo programa i arhitekturu CPU-a.

#### Primer sa MIPS arhitekturom

Za emulaciju binarnog fajla sa MIPS arhitekturom, može se koristiti komanda:
```bash
file ./squashfs-root/bin/busybox
```
I za instaliranje potrebnih alata za emulaciju:
```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```
### Emulacija arhitekture ARM

Za ARM binarne datoteke, proces je sličan, pri čemu se koristi emulator `qemu-arm` za emulaciju.

### Emulacija celog sistema

Alati poput [Firmadyne](https://github.com/firmadyne/firmadyne), [Firmware Analysis Toolkit](https://github.com/attify/firmware-analysis-toolkit) i drugi olakšavaju potpunu emulaciju firmware-a, automatizujući proces i pomažući u dinamičkoj analizi.

## Praktična analiza u pokretu

U ovoj fazi se koristi stvarno ili emulisano okruženje uređaja za analizu. Bitno je održavati pristup ljuske operativnom sistemu i fajl sistemu. Emulacija možda neće savršeno oponašati hardverske interakcije, što može zahtevati povremena ponovna pokretanja emulacije. Analiza treba da ponovo pregleda fajl sistem, iskoristi izložene web stranice i mrežne servise, istraži ranjivosti bootloader-a. Testiranje integriteta firmware-a je ključno za identifikaciju potencijalnih ranjivosti vrata.

## Tehnike analize u toku izvršavanja

Analiza u toku izvršavanja podrazumeva interakciju sa procesom ili binarnom datotekom u svom radnom okruženju, koristeći alate poput gdb-multiarch, Frida i Ghidra za postavljanje prekidača i identifikaciju ranjivosti kroz ispitivanje i druge tehnike.

## Eksploatacija binarnih datoteka i dokaz koncepta

Razvoj PoC-a za identifikovane ranjivosti zahteva duboko razumevanje ciljne arhitekture i programiranje u jezicima nižeg nivoa. Zaštite binarnog izvršavanja u ugrađenim sistemima su retke, ali kada su prisutne, tehnike poput Return Oriented Programming (ROP) mogu biti neophodne.

## Pripremljeni operativni sistemi za analizu firmware-a

Operativni sistemi poput [AttifyOS](https://github.com/adi0x90/attifyos) i [EmbedOS](https://github.com/scriptingxss/EmbedOS) pružaju prekonfigurisana okruženja za testiranje sigurnosti firmware-a, opremljena neophodnim alatima.

## Pripremljeni operativni sistemi za analizu firmware-a

* [**AttifyOS**](https://github.com/adi0x90/attifyos): AttifyOS je distribucija namenjena pomoći pri sprovođenju procene sigurnosti i testiranju prodiranja uređaja Internet stvari (IoT). Štedi vreme pružajući prekonfigurisano okruženje sa svim neophodnim alatima.
* [**EmbedOS**](https://github.com/scriptingxss/EmbedOS): Ugrađeni operativni sistem za testiranje sigurnosti zasnovan na Ubuntu 18.04 sa unapred učitanim alatima za testiranje sigurnosti firmware-a.

## Ranjivi firmware za vežbanje

Za vežbanje otkrivanja ranjivosti u firmware-u, koristite sledeće projekte ranjivog firmware-a kao polaznu tačku.

* OWASP IoTGoat
* [https://github.com/OWASP/IoTGoat](https://github.com/OWASP/IoTGoat)
* The Damn Vulnerable Router Firmware Project
* [https://github.com/praetorian-code/DVRF](https://github.com/praetorian-code/DVRF)
* Damn Vulnerable ARM Router (DVAR)
* [https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html](https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html)
* ARM-X
* [https://github.com/therealsaumil/armx#downloads](https://github.com/therealsaumil/armx#downloads)
* Azeria Labs VM 2.0
* [https://azeria-labs.com/lab-vm-2-0/](https://azeria-labs.com/lab-vm-2-0/)
* Damn Vulnerable IoT Device (DVID)
* [https://github.com/Vulcainreo/DVID](https://github.com/Vulcainreo/DVID)

## Reference

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)
* [Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things](https://www.amazon.co.uk/Practical-IoT-Hacking-F-Chantzis/dp/1718500904)

## Obuka i sertifikacija

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)
