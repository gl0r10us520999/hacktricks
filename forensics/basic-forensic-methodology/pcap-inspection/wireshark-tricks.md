# Wireshark truuks

## Wireshark truuks

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is 'n **donkerweb**-aangedrewe soekenjin wat **gratis** funksies bied om te kontroleer of 'n maatskappy of sy kliënte deur **steelmalware** gekompromitteer is.

Die primêre doel van WhiteIntel is om rekening-oorneem te bekamp en losgeldaanvalle as gevolg van inligtingsteelmalware te voorkom.

Jy kan hul webwerf besoek en hul enjin **gratis** probeer by:

{% embed url="https://whiteintel.io" %}

---

## Verbeter jou Wireshark-vaardighede

### Tutoriale

Die volgende tutoriale is wonderlik om 'n paar koel basiese truuks te leer:

* [https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
* [https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* [https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* [https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)

### Geanaliseerde Inligting

**Ekspertinligting**

Deur te klik op _**Analiseer** --> **Ekspertinligting**_ sal jy 'n **oorsig** kry van wat in die **geanaliseerde** pakkies gebeur:

![](<../../../.gitbook/assets/image (570).png>)

**Opgeloste Adresse**

Onder _**Statistiek --> Opgeloste Adresse**_ kan jy verskeie **inligting** vind wat deur Wireshark "**opgelos**" is, soos poort/vervoer na protokol, MAC na die vervaardiger, ens. Dit is interessant om te weet wat betrokke is in die kommunikasie.

![](<../../../.gitbook/assets/image (571).png>)

**Protokolhiërargie**

Onder _**Statistiek --> Protokolhiërargie**_ kan jy die **protokolle** vind wat betrokke is by die kommunikasie en inligting daaroor.

![](<../../../.gitbook/assets/image (572).png>)

**Gesprekke**

Onder _**Statistiek --> Gesprekke**_ kan jy 'n **opsomming van die gesprekke** in die kommunikasie vind en inligting daaroor.

![](<../../../.gitbook/assets/image (573).png>)

**Eindpunte**

Onder _**Statistiek --> Eindpunte**_ kan jy 'n **opsomming van die eindpunte** in die kommunikasie vind en inligting oor elkeen van hulle.

![](<../../../.gitbook/assets/image (575).png>)

**DNS-inligting**

Onder _**Statistiek --> DNS**_ kan jy statistieke vind oor die vasgevangde DNS-versoek.

![](<../../../.gitbook/assets/image (577).png>)

**I/O-grafiek**

Onder _**Statistiek --> I/O-grafiek**_ kan jy 'n **grafiek van die kommunikasie** vind.

![](<../../../.gitbook/assets/image (574).png>)

### Filters

Hier kan jy Wireshark-filter vind afhangende van die protokol: [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)\
Ander interessante filters:

* `(http.request or ssl.handshake.type == 1) and !(udp.port eq 1900)`
* HTTP en aanvanklike HTTPS-verkeer
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002) and !(udp.port eq 1900)`
* HTTP en aanvanklike HTTPS-verkeer + TCP SYN
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002 or dns) and !(udp.port eq 1900)`
* HTTP en aanvanklike HTTPS-verkeer + TCP SYN + DNS-versoeke

### Soek

As jy wil **soek** vir **inhoud** binne die **pakkies** van die sessies, druk _CTRL+f_. Jy kan nuwe lae by die hoofinligtingstafel (No., Tyd, Bron, ens.) voeg deur die regterknoppie te druk en dan die kolom te wysig.

### Gratis pcap-laboratoriums

**Oefen met die gratis uitdagings van: [https://www.malware-traffic-analysis.net/](https://www.malware-traffic-analysis.net)**

## Identifiseer Domeine

Jy kan 'n kolom byvoeg wat die Host HTTP-kop wys:

![](<../../../.gitbook/assets/image (403).png>)

En 'n kolom wat die Bedienernaam byvoeg van 'n inisieerende HTTPS-verbinding (**ssl.handshake.type == 1**):

![](<../../../.gitbook/assets/image (408) (1).png>)

## Identifiseer plaaslike gasheernaam

### Vanaf DHCP

In die huidige Wireshark moet jy in plaas van `bootp` soek vir `DHCP`

![](<../../../.gitbook/assets/image (404).png>)

### Vanaf NBNS

![](<../../../.gitbook/assets/image (405).png>)

## Ontsleutel TLS

### Ontsleutel https-verkeer met bedienerprivaatsleutel

_wysig>voorkeur>protokol>ssl>_

![](<../../../.gitbook/assets/image (98).png>)

Druk _Wysig_ en voeg al die data van die bediener en die privaatsleutel by (_IP, Poort, Protokol, Sleutel lêer en wagwoord_)

### Ontsleutel https-verkeer met simmetriese sessiesleutels

Beide Firefox en Chrome het die vermoë om TLS-sessiesleutels te log, wat met Wireshark gebruik kan word om TLS-verkeer te ontsluit. Dit maak in-diepte analise van veilige kommunikasie moontlik. Meer besonderhede oor hoe om hierdie ontsleuteling uit te voer, kan gevind word in 'n gids by [Red Flag Security](https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/).

Om dit op te spoor, soek binne die omgewing vir die veranderlike `SSLKEYLOGFILE`

'N Lêer van gedeelde sleutels sal soos volg lyk:

![](<../../../.gitbook/assets/image (99).png>)

Om dit in Wireshark in te voer, gaan na \_wysig > voorkeur > protokol > ssl > en voer dit in (Pre)-Master-Secret log-lêernaam:

![](<../../../.gitbook/assets/image (100).png>)
## ADB kommunikasie

Haal 'n APK uit 'n ADB kommunikasie waar die APK gestuur is:
```python
from scapy.all import *

pcap = rdpcap("final2.pcapng")

def rm_data(data):
splitted = data.split(b"DATA")
if len(splitted) == 1:
return data
else:
return splitted[0]+splitted[1][4:]

all_bytes = b""
for pkt in pcap:
if Raw in pkt:
a = pkt[Raw]
if b"WRTE" == bytes(a)[:4]:
all_bytes += rm_data(bytes(a)[24:])
else:
all_bytes += rm_data(bytes(a))
print(all_bytes)

f = open('all_bytes.data', 'w+b')
f.write(all_bytes)
f.close()
```
## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is a **dark-web** fueled search engine that offers **free** functionalities to check if a company or its customers have been **compromised** by **stealer malwares**.

Hul primêre doel van WhiteIntel is om rekening-oorneemings en lospryse-aanvalle te beveg wat voortspruit uit inligtingsteel-malware.

Jy kan hul webwerf besoek en hul enjin vir **gratis** probeer by:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
