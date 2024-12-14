# Pcap Inspection

{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) ist die relevanteste Cybersecurity-Veranstaltung in **Spanien** und eine der wichtigsten in **Europa**. Mit **der Mission, technisches Wissen zu fördern**, ist dieser Kongress ein brodelnder Treffpunkt für Technologie- und Cybersecurity-Profis in jeder Disziplin.

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
Eine Anmerkung zu **PCAP** vs **PCAPNG**: Es gibt zwei Versionen des PCAP-Dateiformats; **PCAPNG ist neuer und wird nicht von allen Tools unterstützt**. Möglicherweise müssen Sie eine Datei von PCAPNG in PCAP mit Wireshark oder einem anderen kompatiblen Tool konvertieren, um sie in einigen anderen Tools verwenden zu können.
{% endhint %}

## Online-Tools für pcaps

* Wenn der Header Ihres pcaps **beschädigt** ist, sollten Sie versuchen, ihn mit: [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php) zu **reparieren**.
* Extrahieren Sie **Informationen** und suchen Sie nach **Malware** in einem pcap in [**PacketTotal**](https://packettotal.com).
* Suchen Sie nach **bösartiger Aktivität** mit [**www.virustotal.com**](https://www.virustotal.com) und [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com).

## Informationen extrahieren

Die folgenden Tools sind nützlich, um Statistiken, Dateien usw. zu extrahieren.

### Wireshark

{% hint style="info" %}
**Wenn Sie ein PCAP analysieren möchten, müssen Sie im Grunde wissen, wie man Wireshark verwendet.**
{% endhint %}

Sie finden einige Wireshark-Tricks in:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(nur Linux)_ kann **ein** **pcap** analysieren und Informationen daraus extrahieren. Zum Beispiel extrahiert Xplico aus einer pcap-Datei jede E-Mail (POP, IMAP und SMTP-Protokolle), alle HTTP-Inhalte, jeden VoIP-Anruf (SIP), FTP, TFTP usw.

**Installieren**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**Ausführen**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
Zugriff auf _**127.0.0.1:9876**_ mit den Anmeldeinformationen _**xplico:xplico**_

Erstellen Sie dann einen **neuen Fall**, erstellen Sie eine **neue Sitzung** innerhalb des Falls und **laden Sie die pcap**-Datei hoch.

### NetworkMiner

Wie Xplico ist es ein Tool, um **pcaps zu analysieren und Objekte zu extrahieren**. Es hat eine kostenlose Edition, die Sie **hier herunterladen** können [**hier**](https://www.netresec.com/?page=NetworkMiner). Es funktioniert mit **Windows**.\
Dieses Tool ist auch nützlich, um **andere Informationen aus den Paketen zu analysieren**, um schneller zu verstehen, was passiert ist.

### NetWitness Investigator

Sie können [**NetWitness Investigator von hier herunterladen**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware) **(Es funktioniert unter Windows)**.\
Dies ist ein weiteres nützliches Tool, das **die Pakete analysiert** und die Informationen auf nützliche Weise sortiert, um **zu wissen, was im Inneren passiert**.

### [BruteShark](https://github.com/odedshimon/BruteShark)

* Extrahieren und Kodieren von Benutzernamen und Passwörtern (HTTP, FTP, Telnet, IMAP, SMTP...)
* Authentifizierungshashes extrahieren und mit Hashcat knacken (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* Erstellen eines visuellen Netzwerkdiagramms (Netzwerkknoten & Benutzer)
* DNS-Abfragen extrahieren
* Alle TCP- und UDP-Sitzungen rekonstruieren
* File Carving

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

Wenn Sie **nach** **etwas** im pcap **suchen**, können Sie **ngrep** verwenden. Hier ist ein Beispiel mit den Hauptfiltern:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### Carving

Die Verwendung gängiger Carving-Techniken kann nützlich sein, um Dateien und Informationen aus dem pcap zu extrahieren:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### Capturing credentials

Sie können Tools wie [https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz) verwenden, um Anmeldeinformationen aus einem pcap oder einer Live-Schnittstelle zu parsen.

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) ist die relevanteste Cybersecurity-Veranstaltung in **Spanien** und eine der wichtigsten in **Europa**. Mit **der Mission, technisches Wissen zu fördern**, ist dieser Kongress ein brodelnder Treffpunkt für Technologie- und Cybersecurity-Profis in jeder Disziplin.

{% embed url="https://www.rootedcon.com/" %}

## Check Exploits/Malware

### Suricata

**Installieren und einrichten**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**Überprüfen Sie pcap**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap) ist ein Tool, das

* Eine PCAP-Datei liest und Http-Streams extrahiert.
* gzip komprimierte Streams deflate.
* Jede Datei mit yara scannt.
* Einen report.txt schreibt.
* Optional übereinstimmende Dateien in ein Verzeichnis speichert.

### Malware-Analyse

Überprüfen Sie, ob Sie einen Fingerabdruck einer bekannten Malware finden können:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> [Zeek](https://docs.zeek.org/en/master/about.html) ist ein passiver, Open-Source-Netzwerkverkehrsanalysator. Viele Betreiber verwenden Zeek als Netzwerk-Sicherheitsmonitor (NSM), um Untersuchungen zu verdächtigen oder bösartigen Aktivitäten zu unterstützen. Zeek unterstützt auch eine Vielzahl von Verkehrsanalysaufgaben über den Sicherheitsbereich hinaus, einschließlich Leistungsbewertung und Fehlersuche.

Im Grunde genommen sind die von `zeek` erstellten Protokolle keine **pcaps**. Daher müssen Sie **andere Tools** verwenden, um die Protokolle zu analysieren, in denen die **Informationen** über die pcaps enthalten sind.

### Verbindungsinformationen
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
### DNS-Informationen
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
## Andere pcap-Analyse-Tricks

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

[**RootedCON**](https://www.rootedcon.com/) ist die relevanteste Cybersecurity-Veranstaltung in **Spanien** und eine der wichtigsten in **Europa**. Mit **der Mission, technisches Wissen zu fördern**, ist dieser Kongress ein brodelnder Treffpunkt für Technologie- und Cybersecurity-Profis in jeder Disziplin.

{% embed url="https://www.rootedcon.com/" %}

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
