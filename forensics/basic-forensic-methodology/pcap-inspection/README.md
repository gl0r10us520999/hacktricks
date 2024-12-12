# Pregled Pcap datoteka

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Obuka AWS Crveni Tim Stručnjak (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Obuka GCP Crveni Tim Stručnjak (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/ssponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti **kibernetičke bezbednosti** u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i bezbednosnih stručnjaka u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
Napomena o **PCAP** vs **PCAPNG**: postoje dve verzije PCAP formata datoteka; **PCAPNG je noviji i nije podržan od svih alata**. Možda ćete morati da konvertujete datoteku iz PCAPNG u PCAP koristeći Wireshark ili neki drugi kompatibilni alat, kako biste mogli da radite sa njom u nekim drugim alatima.
{% endhint %}

## Online alati za pcap datoteke

* Ako je zaglavlje vaše pcap datoteke **oštećeno**, trebalo bi da pokušate da ga **popravite** koristeći: [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)
* Izvucite **informacije** i tražite **malver** unutar pcap datoteke na [**PacketTotal**](https://packettotal.com)
* Tražite **zlonamerne aktivnosti** koristeći [**www.virustotal.com**](https://www.virustotal.com) i [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com)

## Izdvajanje informacija

Sledeći alati su korisni za izdvajanje statistika, datoteka, itd.

### Wireshark

{% hint style="info" %}
**Ako ćete analizirati Famekor Wi** usingen**, ukivanć** usingounder  leastcesfind theima-ref for ivoć** usingounder : ** Wi**:anal example theimaref theive URLomerefs Wi**:anal example theima-ref for  yourte url Wi theiteore you theimaref the duaoive % URLec** usingoundive %est Wi relevant for ** Wi relevant for ** Wi relevant for ** Wi relevant for ** Wi relevant for ** Wi relevant for ** you Wi relevant for ** Wi relevant for ** Wi theau.** usingand a his for theau example the. xtop your online training, and the [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repositories.

</details>
{% endhint %}

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti **kibernetičke bezbednosti** u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i bezbednosnih stručnjaka u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
Napomena o **PCAP** vs **PCAPNG**: postoje dve verzije PCAP formata datoteka; **PCAPNG je noviji i nije podržan od svih alata**. Možda ćete morati da konvertujete datoteku iz PCAPNG u PCAP koristeći Wireshark ili neki drugi kompatibilni alat, kako biste mogli da radite sa njom u nekim drugim alatima.
{% endhint %}

## Online alati za pcap datoteke

* Ako je zaglavlje vaše pcap datoteke **oštećeno**, trebalo bi da pokušate da ga **popravite** koristeći: [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)
* Izvucite **informacije** i tražite **malver** unutar pcap datoteke na [**PacketTotal**](https://packettotal.com)
* Tražite **zlonamerne aktivnosti** koristeći [**www.virustotal.com**](https://www.virustotal.com) i [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com)

## Izdvajanje informacija

Sledeći alati su korisni za izdvajanje statistika, datoteka, itd.

### Wireshark

{% hint style="info" %}
**Ako ćete analizirati PCAP datoteku, osnovno je da znate kako koristiti Wireshark**
{% endhint %}

Možete pronaći neke trikove za Wireshark u:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(samo za linux)_ može **analizirati** pcap datoteku i izvući informacije iz nje. Na primer, iz pcap datoteke Xplico iz  izvlači svaki email (POP, IMAP i SMTP protokoli), sve HTTP sadržaje, svaki VoIP poziv (SIP), FTP, TFTP, i tako dalje.

**Instalacija**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**Pokreni**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
Pristupite _**127.0.0.1:9876**_ sa pristupnim podacima _**xplico:xplico**_

Zatim kreirajte **novi slučaj**, napravite **novu sesiju** unutar slučaja i **učitajte pcap** datoteku.

### NetworkMiner

Kao i Xplico, ovo je alat za **analizu i izdvajanje objekata iz pcap-ova**. Ima besplatno izdanje koje možete **preuzeti** [**ovde**](https://www.netresec.com/?page=NetworkMiner). Radi sa **Windows**-om.\
Ovaj alat je takođe koristan za dobijanje **ostalih informacija analiziranih** iz paketa kako biste mogli znati šta se dešavalo na **brži** način.

### NetWitness Investigator

Možete preuzeti [**NetWitness Investigator ovde**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware) **(Radi na Windows-u)**.\
Ovo je još jedan koristan alat koji **analizira pakete** i sortira informacije na koristan način kako biste **znali šta se dešava unutra**.

### [BruteShark](https://github.com/odedshimon/BruteShark)

* Izdvajanje i enkodiranje korisničkih imena i lozinki (HTTP, FTP, Telnet, IMAP, SMTP...)
* Izdvajanje autentifikacionih heševa i njihovo pucanje korišćenjem Hashcat-a (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* Izgradnja vizuelnog dijagrama mreže (Mrežni čvorovi & korisnici)
* Izdvajanje DNS upita
* Rekonstrukcija svih TCP & UDP sesija
  * Rezanje datoteka

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

Ako **tražite** nešto unutar pcap datoteke,**možete** koristiti **ngrep**. Evo primera korišćenja osnovnih filtera:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### Rezanje

Korišćenje uobičajenih tehnika rezanja može biti korisno za izvlačenje datoteka i informacija iz pcap datoteke:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### Snimanje akreditacija

Možete koristiti alate poput [https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz) za parsiranje akreditacija iz pcap datoteke ili sa live interfejsa.

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti **kibernetičke bezbednosti** u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i stručnjaka za kibernetičku bezbednost u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

## Provera Eksploatacija/Malvera

### Suricata

**Instalacija i podešavanje**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**Provera pcap-a**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap) je alatka koja

* Čita PCAP datoteku i izvlači HTTP tokove.
* gzip dekompresuje bilo koje kompresovane tokove
* Skenira svaku datoteku pomoću yara
* Piše izveštaj.txt
* Opciono čuva podudarne datoteke u direktorijumu

### Analiza Malvera

Proverite da li možete pronaći bilo kakav otisak poznatog malvera:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> [Zeek](https://docs.zeek.org/en/master/about.html) je pasivni, open-source analizator saobraćaja na mreži. Mnogi operateri koriste Zeek kao Monitor za bezbednost mreže (NSM) kako bi podržali istrage sumnjive ili zlonamerne aktivnosti. Zeek takođe podržava širok spektar zadataka analize saobraćaja izvan domena bezbednosti, uključujući merenje performansi i rešavanje problema.

U osnovi, zapisi kreirani od strane `zeek` nisu **pcap** datoteke. Stoga će vam biti potrebni **drugi alati** za analizu zapisa gde se nalaze **informacije** o pcap datotekama. 

### Informacije o konekcijama
```bash
#Get info about longest connections (add "grep udp" to see only udp traffic)
#The longest connection might be of malware (constant reverse shell?)
cat conn.log | zeek-cut id.orig_h id.orig_p id.resp_h id.resp_p proto service duration | sort -nrk 7 | head -n 10

10.55.100.100   49778   65.52.108.225   443     tcp     -       86222.365445
10.55.100.107   56099   111.221.29.113  443     tcp     -       86220.126151
10.55.100.110   60168   40.77.229.82    443     tcp     -       86160.119664


#Improve the metrics by summing up the total duration time for connections that have the same destination IP and Port.
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto duration | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += $5 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head -n 10

10.55.100.100   65.52.108.225   443     tcp     86222.4
10.55.100.107   111.221.29.113  443     tcp     86220.1
10.55.100.110   40.77.229.82    443     tcp     86160.1

#Get the number of connections summed up per each line
cat conn.log | zeek-cut id.orig_h id.resp_h duration | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2] += $3; count[$1 FS $2] += 1 } END{ for (key in arr) printf "%s%s%s%s%s\n", key, FS, count[key], FS, arr[key] }' | sort -nrk 4 | head -n 10

10.55.100.100   65.52.108.225   1       86222.4
10.55.100.107   111.221.29.113  1       86220.1
10.55.100.110   40.77.229.82    134       86160.1

#Check if any IP is connecting to 1.1.1.1
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto service | grep '1.1.1.1' | sort | uniq -c

#Get number of connections per source IP, dest IP and dest Port
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += 1 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head -n 10


# RITA
#Something similar can be done with the tool rita
rita show-long-connections -H --limit 10 zeek_logs

+---------------+----------------+--------------------------+----------------+
|   SOURCE IP   | DESTINATION IP | DSTPORT:PROTOCOL:SERVICE |    DURATION    |
+---------------+----------------+--------------------------+----------------+
| 10.55.100.100 | 65.52.108.225  | 443:tcp:-                | 23h57m2.3655s  |
| 10.55.100.107 | 111.221.29.113 | 443:tcp:-                | 23h57m0.1262s  |
| 10.55.100.110 | 40.77.229.82   | 443:tcp:-                | 23h56m0.1197s  |

#Get connections info from rita
rita show-beacons zeek_logs | head -n 10
Score,Source IP,Destination IP,Connections,Avg Bytes,Intvl Range,Size Range,Top Intvl,Top Size,Top Intvl Count,Top Size Count,Intvl Skew,Size Skew,Intvl Dispersion,Size Dispersion
1,192.168.88.2,165.227.88.15,108858,197,860,182,1,89,53341,108319,0,0,0,0
1,10.55.100.111,165.227.216.194,20054,92,29,52,1,52,7774,20053,0,0,0,0
0.838,10.55.200.10,205.251.194.64,210,69,29398,4,300,70,109,205,0,0,0,0
```
### DNS informacije
```bash
#Get info about each DNS request performed
cat dns.log | zeek-cut -c id.orig_h query qtype_name answers

#Get the number of times each domain was requested and get the top 10
cat dns.log | zeek-cut query | sort | uniq | rev | cut -d '.' -f 1-2 | rev | sort | uniq -c | sort -nr | head -n 10

#Get all the IPs
cat dns.log | zeek-cut id.orig_h query | grep 'example\.com' | cut -f 1 | sort | uniq -c

#Sort the most common DNS record request (should be A)
cat dns.log | zeek-cut qtype_name | sort | uniq -c | sort -nr

#See top DNS domain requested with rita
rita show-exploded-dns -H --limit 10 zeek_logs
```
## Ostale trikove analize pcap-a

{% content-ref url="dnscat-exfiltration.md" %}
[dnscat-exfiltration.md](dnscat-exfiltration.md)
{% endcontent-ref %}

{% content-ref url="wifi-pcap-analysis.md" %}
[wifi-pcap-analysis.md](wifi-pcap-analysis.md)
{% endcontent-ref %}

{% content-ref url="usb-keystrokes.md" %}
[usb-keystrokes.md](usb-keystrokes.md)
{% endcontent-ref %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) je najrelevantniji događaj u oblasti **kibernetičke bezbednosti** u **Španiji** i jedan od najvažnijih u **Evropi**. Sa **misijom promovisanja tehničkog znanja**, ovaj kongres je ključno mesto susreta tehnoloških i stručnjaka za kibernetičku bezbednost u svakoj disciplini.

{% embed url="https://www.rootedcon.com/" %}

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
