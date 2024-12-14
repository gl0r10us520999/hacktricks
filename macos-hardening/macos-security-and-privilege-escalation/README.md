# macOS Security & Privilege Escalation

{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy)サーバーに参加して、経験豊富なハッカーやバグバウンティハンターとコミュニケーションを取りましょう！

**ハッキングの洞察**\
ハッキングのスリルと課題に深く掘り下げたコンテンツに参加する

**リアルタイムハックニュース**\
リアルタイムのニュースと洞察を通じて、急速に進化するハッキングの世界に遅れずについていく

**最新の発表**\
新しいバグバウンティの開始や重要なプラットフォームの更新について情報を得る

**[**Discord**](https://discord.com/invite/N3FrSbmwdy)に参加して、今日からトップハッカーとコラボレーションを始めましょう！**

## 基本的なMacOS

macOSに不慣れな場合は、macOSの基本を学び始めるべきです：

* 特殊なmacOS **ファイルと権限：**

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* 一般的なmacOS **ユーザー**

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* **カーネル**の**アーキテクチャ**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* 一般的なmacOS **ネットワークサービスとプロトコル**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **オープンソース** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* `tar.gz`をダウンロードするには、[https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/)のようなURLを[https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)に変更します。

### MacOS MDM

企業では、**macOS**システムはMDMで**管理される**可能性が非常に高いです。したがって、攻撃者の視点からは**それがどのように機能するか**を知ることが興味深いです：

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - 検査、デバッグ、ファジング

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## MacOS セキュリティ保護

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## 攻撃面

### ファイル権限

**rootとして実行されているプロセスが**ユーザーによって制御可能なファイルを書き込む場合、ユーザーはこれを悪用して**権限を昇格させる**ことができます。\
これは以下の状況で発生する可能性があります：

* 使用されたファイルがすでにユーザーによって作成されている（ユーザーが所有）
* 使用されたファイルがグループのためにユーザーによって書き込み可能
* 使用されたファイルがユーザーが所有するディレクトリ内にある（ユーザーがファイルを作成できる）
* 使用されたファイルがrootが所有するディレクトリ内にあるが、ユーザーがグループのために書き込みアクセスを持っている（ユーザーがファイルを作成できる）

**rootによって使用される**ファイルを**作成できる**ことは、ユーザーがその**内容を利用する**ことを可能にし、さらには**シンボリックリンク/ハードリンク**を作成して別の場所を指すこともできます。

この種の脆弱性については、**脆弱な`.pkg`インストーラーを確認することを忘れないでください**：

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### ファイル拡張子とURLスキームアプリハンドラー

ファイル拡張子によって登録された奇妙なアプリは悪用される可能性があり、異なるアプリケーションが特定のプロトコルを開くために登録されることがあります。

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## macOS TCC / SIP 権限昇格

macOSでは、**アプリケーションやバイナリが**フォルダや設定にアクセスするための権限を持つことができ、他のものよりも特権を持つことがあります。

したがって、macOSマシンを成功裏に侵害したい攻撃者は、**TCC権限を昇格させる**必要があります（または、必要に応じて**SIPをバイパスする**こともあります）。

これらの権限は通常、アプリケーションが署名されている**権利**の形で与えられるか、アプリケーションがいくつかのアクセスを要求し、**ユーザーがそれらを承認した後**に**TCCデータベース**に見つかります。プロセスがこれらの権限を取得する別の方法は、**その権限を持つプロセスの子プロセス**であることです。これらの権限は通常**継承されます**。

これらのリンクをたどって、[**TCCで権限を昇格させる**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses)、[**TCCをバイパスする**](macos-security-protections/macos-tcc/macos-tcc-bypasses/)方法や、過去に[**SIPがバイパスされた**](macos-security-protections/macos-sip.md#sip-bypasses)方法を見つけてください。

## macOS 伝統的権限昇格

もちろん、レッドチームの視点からは、rootに昇格することにも興味があるはずです。以下の投稿をチェックして、いくつかのヒントを得てください：

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}

## macOS コンプライアンス

* [https://github.com/usnistgov/macos\_security](https://github.com/usnistgov/macos_security)

## 参考文献

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy)サーバーに参加して、経験豊富なハッカーやバグバウンティハンターとコミュニケーションを取りましょう！

**ハッキングの洞察**\
ハッキングのスリルと課題に深く掘り下げたコンテンツに参加する

**リアルタイムハックニュース**\
リアルタイムのニュースと洞察を通じて、急速に進化するハッキングの世界に遅れずについていく

**最新の発表**\
新しいバグバウンティの開始や重要なプラットフォームの更新について情報を得る

**[**Discord**](https://discord.com/invite/N3FrSbmwdy)に参加して、今日からトップハッカーとコラボレーションを始めましょう！**

{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}
