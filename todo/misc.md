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


ping応答TTL：\
127 = Windows\
254 = Cisco\
その他は、いくつかのLinux

$1$- md5\
$2$または$2a$ - Blowfish\
$5$- sha256\
$6$- sha512

サービスの背後に何があるかわからない場合は、HTTP GETリクエストを試みてください。

**UDPスキャン**\
nc -nv -u -z -w 1 \<IP> 160-16

特定のポートに空のUDPパケットが送信されます。UDPポートが開いている場合、ターゲットマシンからの応答は返されません。UDPポートが閉じている場合、ターゲットマシンからICMPポート到達不能パケットが返されるべきです。\

UDPポートスキャンはしばしば信頼性が低く、ファイアウォールやルーターがICMPパケットをドロップする可能性があります。これにより、スキャンでの偽陽性が発生し、スキャンされたマシンのすべてのUDPポートが開いていると表示されることがよくあります。\
ほとんどのポートスキャナーはすべての利用可能なポートをスキャンせず、通常はスキャンされる「興味深いポート」のプリセットリストを持っています。

# CTF - トリック

**Windows**では、**Winzip**を使用してファイルを検索します。\
**代替データストリーム**：_dir /r | find ":$DATA"_\
```
binwalk --dd=".*" <file> #Extract everything
binwalk -M -e -d=10000 suspicious.pdf #Extract, look inside extracted files and continue extracing (depth of 10000)
```
## Crypto

**featherduster**\


**Basae64**(6—>8) —> 0...9, a...z, A…Z,+,/\
**Base32**(5 —>8) —> A…Z, 2…7\
**Base85** (Ascii85, 7—>8) —> 0...9, a...z, A...Z, ., -, :, +, =, ^, !, /, \*, ?, &, <, >, (, ), \[, ], {, }, @, %, $, #\
**Uuencode** --> "_begin \<mode> \<filename>_" で始まり、奇妙な文字\
**Xxencoding** --> "_begin \<mode> \<filename>_" で始まり、B64\
\
**Vigenere** (頻度分析) —> [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)\
**Scytale** (文字のオフセット) —> [https://www.dcode.fr/scytale-cipher](https://www.dcode.fr/scytale-cipher)

**25x25 = QR**

factordb.com\
rsatool

Snow --> スペースとタブを使ってメッセージを隠す

# Characters

%E2%80%AE => RTL文字（ペイロードを逆に書く）


{% hint style="success" %}
AWSハッキングを学び、練習する:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、練習する: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください!
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを送信してください。**

</details>
{% endhint %}
