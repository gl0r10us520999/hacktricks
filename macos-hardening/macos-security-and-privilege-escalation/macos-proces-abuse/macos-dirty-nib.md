# macOS Dirty NIB

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

**For further detail about the technique check the original post from:** [**https://blog.xpnsec.com/dirtynib/**](https://blog.xpnsec.com/dirtynib/) and the following post by [**https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/**](https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/)**.** Here is a summary:

### NIBファイルとは

Nib（NeXT Interface Builderの略）ファイルは、Appleの開発エコシステムの一部であり、アプリケーション内の**UI要素**とその相互作用を定義するために使用されます。これらはウィンドウやボタンなどのシリアライズされたオブジェクトを含み、ランタイムで読み込まれます。現在も使用されていますが、Appleはより包括的なUIフローの視覚化のためにStoryboardを推奨しています。

主要なNibファイルは、アプリケーションの`Info.plist`ファイル内の**`NSMainNibFile`**の値で参照され、アプリケーションの`main`関数で実行される**`NSApplicationMain`**関数によって読み込まれます。

### ダーティNibインジェクションプロセス

#### NIBファイルの作成と設定

1. **初期設定**:
* XCodeを使用して新しいNIBファイルを作成します。
* インターフェースにオブジェクトを追加し、そのクラスを`NSAppleScript`に設定します。
* ユーザー定義のランタイム属性を介して初期の`source`プロパティを設定します。
2. **コード実行ガジェット**:
* この設定により、必要に応じてAppleScriptを実行できます。
* `Apple Script`オブジェクトをアクティブにするボタンを統合し、特に`executeAndReturnError:`セレクタをトリガーします。
3. **テスト**:
* テスト用のシンプルなApple Script:

```bash
set theDialogText to "PWND"
display dialog theDialogText
```
* XCodeデバッガーで実行し、ボタンをクリックしてテストします。

#### アプリケーションのターゲット（例：Pages）

1. **準備**:
* ターゲットアプリ（例：Pages）を別のディレクトリ（例：`/tmp/`）にコピーします。
* Gatekeeperの問題を回避し、アプリをキャッシュするためにアプリを起動します。
2. **NIBファイルの上書き**:
* 既存のNIBファイル（例：About Panel NIB）を作成したDirtyNIBファイルで置き換えます。
3. **実行**:
* アプリと対話して実行をトリガーします（例：`About`メニュー項目を選択）。

#### 概念実証：ユーザーデータへのアクセス

* ユーザーの同意なしに、AppleScriptを修正して写真などのユーザーデータにアクセスし、抽出します。

### コードサンプル：悪意のある.xibファイル

* 任意のコードを実行することを示す[**悪意のある.xibファイルのサンプル**](https://gist.github.com/xpn/16bfbe5a3f64fedfcc1822d0562636b4)にアクセスして確認します。

### その他の例

投稿[https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/](https://sector7.computest.nl/post/2024-04-bringing-process-injection-into-view-exploiting-all-macos-apps-using-nib-files/)では、ダーティNibの作成方法に関するチュートリアルを見つけることができます。&#x20;

### 起動制約への対処

* 起動制約は、予期しない場所（例：`/tmp`）からのアプリの実行を妨げます。
* 起動制約によって保護されていないアプリを特定し、NIBファイルインジェクションのターゲットにすることが可能です。

### 追加のmacOS保護

macOS Sonoma以降、アプリバンドル内の変更が制限されています。ただし、以前の方法は次のように行われました：

1. アプリを別の場所（例：`/tmp/`）にコピーします。
2. 初期の保護を回避するために、アプリバンドル内のディレクトリの名前を変更します。
3. Gatekeeperに登録するためにアプリを実行した後、アプリバンドルを変更します（例：MainMenu.nibをDirty.nibに置き換えます）。
4. ディレクトリの名前を元に戻し、アプリを再実行してインジェクトされたNIBファイルを実行します。

**注意**: 最近のmacOSのアップデートにより、Gatekeeperキャッシュ後にアプリバンドル内のファイル変更が防止され、この脆弱性は無効化されました。

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
