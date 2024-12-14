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


# カービングツール

## Autopsy

フォレンジックで画像からファイルを抽出するために最も一般的に使用されるツールは[**Autopsy**](https://www.autopsy.com/download/)です。ダウンロードしてインストールし、ファイルを取り込んで「隠れた」ファイルを見つけます。Autopsyはディスクイメージやその他の種類のイメージをサポートするように構築されていますが、単純なファイルには対応していないことに注意してください。

## Binwalk <a id="binwalk"></a>

**Binwalk**は、画像や音声ファイルなどのバイナリファイル内の埋め込まれたファイルやデータを検索するためのツールです。`apt`でインストールできますが、[ソース](https://github.com/ReFirmLabs/binwalk)はgithubにあります。
**便利なコマンド**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

もう一つの一般的なツールは**foremost**です。foremostの設定ファイルは`/etc/foremost.conf`にあります。特定のファイルを検索したい場合は、それらのコメントを外してください。何もコメントを外さなければ、foremostはデフォルトで設定されたファイルタイプを検索します。
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **スカルペル**

**スカルペル**は、**ファイルに埋め込まれたファイル**を見つけて抽出するために使用できる別のツールです。この場合、抽出したいファイルタイプを設定ファイル\(_/etc/scalpel/scalpel.conf_\)からコメント解除する必要があります。
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

このツールはkaliに含まれていますが、ここで見つけることができます: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

このツールはイメージをスキャンし、その中にある**pcaps**を**抽出**し、**ネットワーク情報（URLs、ドメイン、IPs、MACs、メール）**やその他の**ファイル**を取得します。あなたがする必要があるのは:
```text
bulk_extractor memory.img -o out_folder
```
すべての情報をナビゲートします。ツールが収集した情報（パスワード？）、パケットを分析します（読み取り[ **Pcaps分析**](../pcap-inspection/)）、奇妙なドメインを検索します（**マルウェア**や**存在しない**ドメインに関連する）。

## PhotoRec

[https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) で見つけることができます。

GUIとCLIバージョンが付属しています。PhotoRecに検索してほしい**ファイルタイプ**を選択できます。

![](../../../.gitbook/assets/image%20%28524%29.png)

# 特定のデータカービングツール

## FindAES

AESキーのスケジュールを検索することでAESキーを検索します。TrueCryptやBitLockerで使用される128、192、256ビットのキーを見つけることができます。

[こちらからダウンロード](https://sourceforge.net/projects/findaes/)。

# 補完ツール

[**viu**](https://github.com/atanunq/viu)を使用してターミナルから画像を見ることができます。  
Linuxコマンドラインツール**pdftotext**を使用してPDFをテキストに変換し、読むことができます。



{% hint style="success" %}
AWSハッキングを学び、練習する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、練習する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
