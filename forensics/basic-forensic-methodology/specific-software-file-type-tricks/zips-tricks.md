# ZIPs tricks

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

**コマンドラインツール**は、**zipファイル**の診断、修復、及びクラッキングに不可欠です。以下は主なユーティリティです：

- **`unzip`**: zipファイルが解凍できない理由を明らかにします。
- **`zipdetails -v`**: zipファイルフォーマットフィールドの詳細な分析を提供します。
- **`zipinfo`**: zipファイルの内容を抽出せずにリストします。
- **`zip -F input.zip --out output.zip`** および **`zip -FF input.zip --out output.zip`**: 壊れたzipファイルの修復を試みます。
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: zipパスワードのブルートフォースクラッキングツールで、約7文字までのパスワードに効果的です。

[Zipファイルフォーマット仕様](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)は、zipファイルの構造と標準に関する包括的な詳細を提供します。

パスワード保護されたzipファイルは、内部のファイル名やファイルサイズを**暗号化しない**ことに注意が必要です。このセキュリティの欠陥は、RARや7zファイルには共有されていません。さらに、古いZipCrypto方式で暗号化されたzipファイルは、圧縮ファイルの未暗号化コピーが利用可能な場合、**平文攻撃**に対して脆弱です。この攻撃は、既知の内容を利用してzipのパスワードをクラッキングします。この脆弱性は[HackThisの記事](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files)で詳述され、[この学術論文](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf)でさらに説明されています。しかし、**AES-256**暗号化で保護されたzipファイルは、この平文攻撃に対して免疫があり、機密データのために安全な暗号化方法を選択する重要性を示しています。

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

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
