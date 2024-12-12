# Pcap Inspection

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

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) は **スペイン** で最も関連性の高いサイバーセキュリティイベントであり、**ヨーロッパ** で最も重要なイベントの一つです。**技術知識の促進**を使命とし、この会議はあらゆる分野の技術とサイバーセキュリティの専門家の熱い交流の場です。

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
**PCAP** と **PCAPNG** に関する注意: PCAPファイル形式には2つのバージョンがあります; **PCAPNGは新しく、すべてのツールでサポートされているわけではありません**。他のツールで作業するために、Wiresharkや他の互換性のあるツールを使用してPCAPNGからPCAPにファイルを変換する必要があるかもしれません。
{% endhint %}

## Online tools for pcaps

* pcapのヘッダーが**壊れている**場合は、[http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)を使用して**修正**を試みてください。
* [**PacketTotal**](https://packettotal.com)でpcap内の**情報**を抽出し、**マルウェア**を検索します。
* [**www.virustotal.com**](https://www.virustotal.com)および[**www.hybrid-analysis.com**](https://www.hybrid-analysis.com)を使用して**悪意のある活動**を検索します。
* [**https://apackets.com/**](https://apackets.com/)でブラウザから**完全なpcap分析**を行います。

## Extract Information

次のツールは、統計、ファイルなどを抽出するのに役立ちます。

### Wireshark

{% hint style="info" %}
**PCAPを分析する場合、基本的にWiresharkの使い方を知っておく必要があります**
{% endhint %}

Wiresharkのトリックは以下で見つけることができます:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### [**https://apackets.com/**](https://apackets.com/)

ブラウザからのpcap分析。

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(Linuxのみ)_は、**pcap**を**分析**し、そこから情報を抽出することができます。たとえば、pcapファイルからXplicoは、各メール（POP、IMAP、SMTPプロトコル）、すべてのHTTPコンテンツ、各VoIP通話（SIP）、FTP、TFTPなどを抽出します。

**Install**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**実行**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
_**127.0.0.1:9876**_ に _**xplico:xplico**_ の資格情報でアクセスします。

次に、**新しいケース**を作成し、ケース内に**新しいセッション**を作成し、**pcap**ファイルを**アップロード**します。

### NetworkMiner

Xplicoと同様に、**pcapsからオブジェクトを分析および抽出する**ツールです。無料版があり、[**こちらからダウンロード**](https://www.netresec.com/?page=NetworkMiner)できます。**Windows**で動作します。\
このツールは、**パケットから分析された他の情報**を取得するのにも役立ち、**より迅速**に何が起こっているかを知ることができます。

### NetWitness Investigator

[**こちらからNetWitness Investigatorをダウンロード**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware)できます。**(Windowsで動作します)**\
これは、**パケットを分析**し、情報を有用な方法で整理して、**内部で何が起こっているかを知る**のに役立つ別の便利なツールです。

### [BruteShark](https://github.com/odedshimon/BruteShark)

* ユーザー名とパスワードを抽出およびエンコード (HTTP, FTP, Telnet, IMAP, SMTP...)
* 認証ハッシュを抽出し、Hashcatを使用してクラックします (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* ビジュアルネットワークダイアグラムを作成 (ネットワークノードとユーザー)
* DNSクエリを抽出
* すべてのTCPおよびUDPセッションを再構築
* ファイルカービング

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

pcap内で**何か**を**探している**場合は、**ngrep**を使用できます。以下は、主要なフィルターを使用した例です：
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### カービング

一般的なカービング技術を使用することで、pcapからファイルや情報を抽出するのに役立ちます：

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### 資格情報のキャプチャ

[https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz)のようなツールを使用して、pcapまたはライブインターフェースから資格情報を解析できます。

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/)は、**スペイン**で最も関連性の高いサイバーセキュリティイベントであり、**ヨーロッパ**で最も重要なイベントの一つです。**技術的知識の促進**を使命とし、この会議はあらゆる分野の技術とサイバーセキュリティの専門家が集まる熱い交流の場です。

{% embed url="https://www.rootedcon.com/" %}

## 脆弱性/マルウェアの確認

### Suricata

**インストールと設定**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**pcapを確認する**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap) は、以下の機能を持つツールです。

* PCAPファイルを読み込み、Httpストリームを抽出します。
* 圧縮されたストリームをgzipで解凍します。
* すべてのファイルをyaraでスキャンします。
* report.txtを作成します。
* 一致するファイルをディレクトリに保存するオプションがあります。

### Malware Analysis

既知のマルウェアのフィンガープリントを見つけられるか確認してください：

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> [Zeek](https://docs.zeek.org/en/master/about.html) は、受動的なオープンソースのネットワークトラフィックアナライザーです。多くのオペレーターは、疑わしいまたは悪意のある活動の調査をサポートするために、Zeekをネットワークセキュリティモニター（NSM）として使用しています。Zeekは、セキュリティドメインを超えたパフォーマンス測定やトラブルシューティングを含む、幅広いトラフィック分析タスクもサポートしています。

基本的に、`zeek`によって作成されたログは**pcap**ではありません。したがって、**pcap**に関する**情報**を分析するためには、**他のツール**を使用する必要があります。

### Connections Info
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
### DNS情報
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
## その他のpcap分析のトリック

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

[**RootedCON**](https://www.rootedcon.com/) は **スペイン** で最も関連性の高いサイバーセキュリティイベントであり、**ヨーロッパ** で最も重要なイベントの一つです。**技術知識の促進**を使命とし、この会議はあらゆる分野の技術とサイバーセキュリティの専門家が集まる熱い交流の場です。

{% embed url="https://www.rootedcon.com/" %}

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
