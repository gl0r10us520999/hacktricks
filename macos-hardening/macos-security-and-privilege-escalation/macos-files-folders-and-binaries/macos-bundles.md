# macOS Bundles

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

## 基本情報

macOSのバンドルは、アプリケーション、ライブラリ、その他の必要なファイルを含むさまざまなリソースのコンテナとして機能し、Finderでは`*.app`ファイルのような単一のオブジェクトとして表示されます。最も一般的に遭遇するバンドルは`.app`バンドルですが、`.framework`、`.systemextension`、および`.kext`のような他のタイプも広く存在します。

### バンドルの重要なコンポーネント

バンドル内、特に`<application>.app/Contents/`ディレクトリ内には、さまざまな重要なリソースが格納されています：

* **\_CodeSignature**: このディレクトリは、アプリケーションの整合性を確認するために重要なコード署名の詳細を保存します。コード署名情報は、次のコマンドを使用して確認できます： %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: ユーザーの操作に応じて実行されるアプリケーションの実行可能バイナリを含みます。
* **Resources**: 画像、文書、インターフェースの説明（nib/xibファイル）など、アプリケーションのユーザーインターフェースコンポーネントのリポジトリです。
* **Info.plist**: アプリケーションの主な構成ファイルとして機能し、システムがアプリケーションを適切に認識し、相互作用するために重要です。

#### Info.plistの重要なキー

`Info.plist`ファイルはアプリケーション構成の基盤であり、次のようなキーを含んでいます：

* **CFBundleExecutable**: `Contents/MacOS`ディレクトリにある主な実行可能ファイルの名前を指定します。
* **CFBundleIdentifier**: アプリケーションのグローバル識別子を提供し、macOSによるアプリケーション管理で広く使用されます。
* **LSMinimumSystemVersion**: アプリケーションを実行するために必要なmacOSの最小バージョンを示します。

### バンドルの探索

`Safari.app`のようなバンドルの内容を探索するには、次のコマンドを使用できます： `bash ls -lR /Applications/Safari.app/Contents`

この探索により、`_CodeSignature`、`MacOS`、`Resources`のようなディレクトリや、`Info.plist`のようなファイルが明らかになり、それぞれがアプリケーションのセキュリティからユーザーインターフェースおよび操作パラメータの定義まで、独自の目的を果たします。

#### 追加のバンドルディレクトリ

一般的なディレクトリに加えて、バンドルには次のようなものも含まれる場合があります：

* **Frameworks**: アプリケーションによって使用されるバンドルフレームワークを含みます。フレームワークは、追加のリソースを持つdylibsのようなものです。
* **PlugIns**: アプリケーションの機能を強化するプラグインや拡張のためのディレクトリです。
* **XPCServices**: アプリケーションによって使用されるプロセス外通信のためのXPCサービスを保持します。

この構造により、すべての必要なコンポーネントがバンドル内にカプセル化され、モジュール式で安全なアプリケーション環境が促進されます。

`Info.plist`キーとその意味に関する詳細情報については、Appleの開発者ドキュメントが広範なリソースを提供しています：[Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html)。

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
