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


Katika jibu la ping TTL:\
127 = Windows\
254 = Cisco\
Mengine, baadhi ya linux

$1$- md5\
$2$au $2a$ - Blowfish\
$5$- sha256\
$6$- sha512

Ikiwa hujui kilicho nyuma ya huduma, jaribu kufanya ombi la HTTP GET.

**UDP Scans**\
nc -nv -u -z -w 1 \<IP> 160-16

Paket ya UDP isiyo na maudhui inatumwa kwa bandari maalum. Ikiwa bandari ya UDP iko wazi, hakuna jibu litatumwa kutoka kwa mashine lengwa. Ikiwa bandari ya UDP imefungwa, paket ya ICMP isiyoweza kufikiwa inapaswa kutumwa kutoka kwa mashine lengwa.\

Kuchunguza bandari za UDP mara nyingi hakutegemeki, kwani moto wa kuzuia na route zinaweza kuondoa paket za ICMP.\
Hii inaweza kusababisha matokeo ya uwongo katika uchunguzi wako, na utaona mara kwa mara\
uchunguzi wa bandari za UDP ukionyesha bandari zote za UDP zikiwa wazi kwenye mashine iliyochunguzwa.\
o Wengi wa wachunguzi wa bandari hawachunguze bandari zote zinazopatikana, na kwa kawaida wana orodha iliyowekwa ya “bandari za kuvutia” ambazo zinachunguzwa.

# CTF - Tricks

Katika **Windows** tumia **Winzip** kutafuta faili.\
**Mito mbadala ya data**: _dir /r | find ":$DATA"_\
```
binwalk --dd=".*" <file> #Extract everything
binwalk -M -e -d=10000 suspicious.pdf #Extract, look inside extracted files and continue extracing (depth of 10000)
```
## Crypto

**featherduster**\


**Basae64**(6—>8) —> 0...9, a...z, A…Z,+,/\
**Base32**(5 —>8) —> A…Z, 2…7\
**Base85** (Ascii85, 7—>8) —> 0...9, a...z, A...Z, ., -, :, +, =, ^, !, /, \*, ?, &, <, >, (, ), \[, ], {, }, @, %, $, #\
**Uuencode** --> Anza na "_begin \<mode> \<filename>_" na herufi za ajabu\
**Xxencoding** --> Anza na "_begin \<mode> \<filename>_" na B64\
\
**Vigenere** (uchambuzi wa mara kwa mara) —> [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)\
**Scytale** (mabadiliko ya wahusika) —> [https://www.dcode.fr/scytale-cipher](https://www.dcode.fr/scytale-cipher)

**25x25 = QR**

factordb.com\
rsatool

Snow --> Ficha ujumbe kwa kutumia nafasi na tab

# Characters

%E2%80%AE => Wahusika wa RTL (andika payloads kwa nyuma)


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
