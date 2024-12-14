# Firmware Analysis

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

## **Introduction**

Firmware ni programu muhimu inayowezesha vifaa kufanya kazi ipasavyo kwa kusimamia na kuwezesha mawasiliano kati ya vipengele vya vifaa na programu ambayo watumiaji wanashirikiana nayo. Inahifadhiwa katika kumbukumbu ya kudumu, kuhakikisha kwamba kifaa kinaweza kufikia maagizo muhimu tangu wakati kinapowashwa, na kusababisha uzinduzi wa mfumo wa uendeshaji. Kuchunguza na labda kubadilisha firmware ni hatua muhimu katika kubaini udhaifu wa usalama.

## **Gathering Information**

**Kukusanya taarifa** ni hatua ya awali muhimu katika kuelewa muundo wa kifaa na teknolojia zinazotumiwa. Mchakato huu unahusisha kukusanya data kuhusu:

* Mifumo ya CPU na mfumo wa uendeshaji unaotumika
* Maelezo ya bootloader
* Mpangilio wa vifaa na karatasi za data
* Vipimo vya msingi wa msimbo na maeneo ya chanzo
* Maktaba za nje na aina za leseni
* Historia za sasisho na vyeti vya udhibiti
* Mchoro wa usanifu na mtiririko
* Tathmini za usalama na udhaifu ulioainishwa

Kwa kusudi hili, zana za **intelligence ya chanzo wazi (OSINT)** ni muhimu, kama vile uchambuzi wa vipengele vyovyote vya programu za chanzo wazi vinavyopatikana kupitia michakato ya ukaguzi wa mikono na otomatiki. Zana kama [Coverity Scan](https://scan.coverity.com) na [Semmle’s LGTM](https://lgtm.com/#explore) hutoa uchambuzi wa statiki bure ambao unaweza kutumika kugundua masuala yanayoweza kutokea.

## **Acquiring the Firmware**

Kupata firmware kunaweza kufanywa kwa njia mbalimbali, kila moja ikiwa na ngazi yake ya ugumu:

* **Moja kwa moja** kutoka kwa chanzo (waendelezaji, watengenezaji)
* **Kujenga** kutoka kwa maelekezo yaliyotolewa
* **Kupakua** kutoka kwenye tovuti rasmi za msaada
* Kutumia **Google dork** maswali ya kutafuta faili za firmware zilizohifadhiwa
* Kufikia **hifadhi ya wingu** moja kwa moja, kwa kutumia zana kama [S3Scanner](https://github.com/sa7mon/S3Scanner)
* Kukamata **sasisho** kupitia mbinu za mtu katikati
* **Kutoa** kutoka kwa kifaa kupitia muunganisho kama **UART**, **JTAG**, au **PICit**
* **Sniffing** kwa maombi ya sasisho ndani ya mawasiliano ya kifaa
* Kutambua na kutumia **nukta za sasisho zilizowekwa**
* **Dumping** kutoka kwa bootloader au mtandao
* **Kuondoa na kusoma** chip ya uhifadhi, wakati kila kitu kingine kimeshindwa, kwa kutumia zana sahihi za vifaa

## Analyzing the firmware

Sasa kwamba una **firmware**, unahitaji kutoa taarifa kuhusu hiyo ili kujua jinsi ya kuitendea. Zana tofauti unazoweza kutumia kwa hiyo:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```
Ikiwa hujapata mengi na zana hizo, angalia **entropy** ya picha kwa kutumia `binwalk -E <bin>`, ikiwa entropy ni ya chini, basi haiwezekani kuwa imefungwa. Ikiwa entropy ni ya juu, inawezekana imefungwa (au imepandwa kwa njia fulani).

Zaidi ya hayo, unaweza kutumia zana hizi kutoa **faili zilizojumuishwa ndani ya firmware**:

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

Au [**binvis.io**](https://binvis.io/#/) ([code](https://code.google.com/archive/p/binvis/)) kuchunguza faili hiyo.

### Kupata Mfumo wa Faili

Kwa zana zilizotajwa hapo awali kama `binwalk -ev <bin>` unapaswa kuwa umeweza **kutoa mfumo wa faili**.\
Binwalk kawaida hutoa ndani ya **kabrasha lililo na jina la aina ya mfumo wa faili**, ambayo kawaida ni moja ya yafuatayo: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs.

#### Utoaji wa Mfumo wa Faili kwa Mikono

Wakati mwingine, binwalk **haitakuwa na byte ya kichawi ya mfumo wa faili katika saini zake**. Katika kesi hizi, tumia binwalk ili **kupata offset ya mfumo wa faili na kuchonga mfumo wa faili uliofinywa** kutoka kwa binary na **kutoa kwa mikono** mfumo wa faili kulingana na aina yake kwa kutumia hatua zilizo hapa chini.
```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```
Kimbia amri hii **dd** ikichonga mfumo wa faili wa Squashfs.
```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```
Mbinu mbadala, amri ifuatayo inaweza pia kutekelezwa.

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* Kwa squashfs (iliyotumika katika mfano hapo juu)

`$ unsquashfs dir.squashfs`

Faili zitakuwa katika "`squashfs-root`" directory baada ya hapo.

* Faili za CPIO archive

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* Kwa mifumo ya faili ya jffs2

`$ jefferson rootfsfile.jffs2`

* Kwa mifumo ya faili ya ubifs yenye NAND flash

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`

## Kuchambua Firmware

Mara firmware inapopatikana, ni muhimu kuichambua ili kuelewa muundo wake na uwezekano wa udhaifu. Mchakato huu unahusisha kutumia zana mbalimbali kuchambua na kutoa data muhimu kutoka kwa picha ya firmware.

### Zana za Uchambuzi wa Awali

Seti ya amri imetolewa kwa ukaguzi wa awali wa faili ya binary (inayorejelewa kama `<bin>`). Amri hizi husaidia katika kubaini aina za faili, kutoa nyuzi, kuchambua data ya binary, na kuelewa maelezo ya sehemu na mfumo wa faili:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #prints offsets in hexadecimal
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head #useful for finding signatures in the header
fdisk -lu <bin> #lists partitions and filesystems, if there are multiple
```
Ili kutathmini hali ya usimbaji wa picha, **entropy** inakaguliwa kwa `binwalk -E <bin>`. Entropy ya chini inaashiria ukosefu wa usimbaji, wakati entropy ya juu inaonyesha uwezekano wa usimbaji au ufinyazi.

Kwa ajili ya kutoa **faili zilizojumuishwa**, zana na rasilimali kama vile nyaraka za **file-data-carving-recovery-tools** na **binvis.io** kwa ajili ya ukaguzi wa faili zinapendekezwa.

### Kutolewa kwa Faili za Mfumo

Kwa kutumia `binwalk -ev <bin>`, mtu anaweza kawaida kutoa mfumo wa faili, mara nyingi katika saraka iliyopewa jina la aina ya mfumo wa faili (mfano, squashfs, ubifs). Hata hivyo, wakati **binwalk** inashindwa kutambua aina ya mfumo wa faili kutokana na kukosekana kwa byte za uchawi, kutoa kwa mikono kunahitajika. Hii inahusisha kutumia `binwalk` kutafuta offset ya mfumo wa faili, ikifuatiwa na amri ya `dd` ili kuchonga mfumo wa faili:
```bash
$ binwalk DIR850L_REVB.bin

$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs
```
Baadaye, kulingana na aina ya mfumo wa faili (kwa mfano, squashfs, cpio, jffs2, ubifs), amri tofauti hutumiwa kutoa maudhui kwa mikono.

### Uchambuzi wa Mfumo wa Faili

Mara mfumo wa faili unapokuwa umetolewa, utafutaji wa kasoro za usalama huanza. Kipaumbele kinatolewa kwa daemons za mtandao zisizo salama, akidi za ndani, mwisho wa API, kazi za seva za sasisho, msimbo usioandikwa, skripti za kuanzisha, na binaries zilizokusanywa kwa uchambuzi wa mbali.

**Mikoa muhimu** na **vitu** vya kukagua ni pamoja na:

* **etc/shadow** na **etc/passwd** kwa ajili ya akidi za watumiaji
* Vyeti vya SSL na funguo katika **etc/ssl**
* Faili za usanidi na skripti kwa ajili ya uwezekano wa udhaifu
* Binaries zilizojumuishwa kwa uchambuzi zaidi
* Seva za wavuti za vifaa vya IoT na binaries

Zana kadhaa husaidia katika kufichua taarifa nyeti na udhaifu ndani ya mfumo wa faili:

* [**LinPEAS**](https://github.com/carlospolop/PEASS-ng) na [**Firmwalker**](https://github.com/craigz28/firmwalker) kwa ajili ya utafutaji wa taarifa nyeti
* [**The Firmware Analysis and Comparison Tool (FACT)**](https://github.com/fkie-cad/FACT\_core) kwa ajili ya uchambuzi wa kina wa firmware
* [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer), [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep), [**ByteSweep-go**](https://gitlab.com/bytesweep/bytesweep-go), na [**EMBA**](https://github.com/e-m-b-a/emba) kwa ajili ya uchambuzi wa statiki na wa dinamik

### Ukaguzi wa Usalama kwenye Binaries Zilizokusanywa

Msimbo wa chanzo na binaries zilizokusanywa zinazopatikana katika mfumo wa faili lazima zichunguzwe kwa udhaifu. Zana kama **checksec.sh** kwa binaries za Unix na **PESecurity** kwa binaries za Windows husaidia kubaini binaries zisizo na ulinzi ambazo zinaweza kutumika.

## Kuiga Firmware kwa Uchambuzi wa Dinamik

Mchakato wa kuiga firmware unaruhusu **uchambuzi wa dinamik** ama wa uendeshaji wa kifaa au programu binafsi. Njia hii inaweza kukutana na changamoto za utegemezi wa vifaa au usanifu, lakini kuhamasisha mfumo wa faili wa mzizi au binaries maalum kwa kifaa chenye usanifu na endianness inayolingana, kama vile Raspberry Pi, au kwa mashine halisi iliyojengwa mapema, kunaweza kuwezesha majaribio zaidi.

### Kuiga Binaries Binafsi

Kwa ajili ya kuchunguza programu moja, kubaini endianness ya programu na usanifu wa CPU ni muhimu.

#### Mfano na Usanifu wa MIPS

Ili kuiga binary ya usanifu wa MIPS, mtu anaweza kutumia amri:
```bash
file ./squashfs-root/bin/busybox
```
Na ili kufunga zana za emulering zinazohitajika:
```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```
For MIPS (big-endian), `qemu-mips` inatumika, na kwa binaries za little-endian, `qemu-mipsel` itakuwa chaguo.

#### ARM Architecture Emulation

Kwa binaries za ARM, mchakato ni sawa, na emulator `qemu-arm` inatumika kwa emulation.

### Full System Emulation

Zana kama [Firmadyne](https://github.com/firmadyne/firmadyne), [Firmware Analysis Toolkit](https://github.com/attify/firmware-analysis-toolkit), na zingine, zinawezesha emulation kamili ya firmware, zikifanya mchakato kuwa wa kiotomatiki na kusaidia katika uchambuzi wa dynamic.

## Dynamic Analysis in Practice

Katika hatua hii, mazingira halisi au yaliyotengenezwa yanatumika kwa uchambuzi. Ni muhimu kudumisha ufikiaji wa shell kwa OS na filesystem. Emulation inaweza isifanane kikamilifu na mwingiliano wa vifaa, ikihitaji kuanzishwa tena kwa emulation mara kwa mara. Uchambuzi unapaswa kutembelea filesystem, kutumia kurasa za wavuti zilizofichuliwa na huduma za mtandao, na kuchunguza udhaifu wa bootloader. Mijadala ya uadilifu wa firmware ni muhimu ili kubaini udhaifu wa backdoor unaoweza kuwepo.

## Runtime Analysis Techniques

Uchambuzi wa wakati wa kukimbia unahusisha kuingiliana na mchakato au binary katika mazingira yake ya uendeshaji, kwa kutumia zana kama gdb-multiarch, Frida, na Ghidra kwa kuweka breakpoints na kubaini udhaifu kupitia fuzzing na mbinu nyingine.

## Binary Exploitation and Proof-of-Concept

Kuendeleza PoC kwa udhaifu ulioainishwa kunahitaji uelewa wa kina wa usanifu wa lengo na programu katika lugha za kiwango cha chini. Ulinzi wa binary runtime katika mifumo iliyojumuishwa ni nadra, lakini inapokuwepo, mbinu kama Return Oriented Programming (ROP) zinaweza kuwa muhimu.

## Prepared Operating Systems for Firmware Analysis

Mifumo ya uendeshaji kama [AttifyOS](https://github.com/adi0x90/attifyos) na [EmbedOS](https://github.com/scriptingxss/EmbedOS) hutoa mazingira yaliyoandaliwa tayari kwa ajili ya mtihani wa usalama wa firmware, yakiwa na zana muhimu.

## Prepared OSs to analyze Firmware

* [**AttifyOS**](https://github.com/adi0x90/attifyos): AttifyOS ni distro iliyokusudiwa kukusaidia kufanya tathmini ya usalama na mtihani wa penetration wa vifaa vya Internet of Things (IoT). Inakuokoa muda mwingi kwa kutoa mazingira yaliyoandaliwa tayari na zana zote muhimu zilizopakuliwa.
* [**EmbedOS**](https://github.com/scriptingxss/EmbedOS): Mfumo wa uendeshaji wa mtihani wa usalama wa embedded unaotegemea Ubuntu 18.04 uliojaa zana za mtihani wa usalama wa firmware.

## Vulnerable firmware to practice

Ili kufanya mazoezi ya kugundua udhaifu katika firmware, tumia miradi ifuatayo ya firmware yenye udhaifu kama hatua ya kuanzia.

* OWASP IoTGoat
* [https://github.com/OWASP/IoTGoat](https://github.com/OWASP/IoTGoat)
* Mradi wa Damn Vulnerable Router Firmware
* [https://github.com/praetorian-code/DVRF](https://github.com/praetorian-code/DVRF)
* Damn Vulnerable ARM Router (DVAR)
* [https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html](https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html)
* ARM-X
* [https://github.com/therealsaumil/armx#downloads](https://github.com/therealsaumil/armx#downloads)
* Azeria Labs VM 2.0
* [https://azeria-labs.com/lab-vm-2-0/](https://azeria-labs.com/lab-vm-2-0/)
* Damn Vulnerable IoT Device (DVID)
* [https://github.com/Vulcainreo/DVID](https://github.com/Vulcainreo/DVID)

## References

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)
* [Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things](https://www.amazon.co.uk/Practical-IoT-Hacking-F-Chantzis/dp/1718500904)

## Trainning and Cert

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)

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
