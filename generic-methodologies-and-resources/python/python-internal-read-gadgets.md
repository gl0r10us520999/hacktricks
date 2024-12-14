# Python Internal Read Gadgets

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

[**Pythonフォーマット文字列**](bypass-python-sandboxes/#python-format-string)や[**クラス汚染**](class-pollution-pythons-prototype-pollution.md)などの異なる脆弱性は、**Python内部データを読み取ることを可能にするが、コードを実行することはできない**かもしれません。したがって、ペンテスターはこれらの読み取り権限を最大限に活用して、**機密特権を取得し、脆弱性をエスカレートさせる**必要があります。

### Flask - シークレットキーの読み取り

Flaskアプリケーションのメインページには、**`app`**グローバルオブジェクトがあり、ここでこの**シークレットが設定されています**。
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
この場合、[**Pythonサンドボックスをバイパスするページ**](bypass-python-sandboxes/)から**グローバルオブジェクトにアクセスする**ために、任意のガジェットを使用してこのオブジェクトにアクセスすることが可能です。

**脆弱性が別のPythonファイルにある場合**、メインのファイルに到達するためにファイルを横断するガジェットが必要で、**グローバルオブジェクト `app.secret_key` にアクセス**してFlaskの秘密鍵を変更し、この鍵を知って[**権限を昇格させる**](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign)ことができるようになります。

このようなペイロードは[この書き込みから](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

このペイロードを使用して、**`app.secret_key`**（あなたのアプリでは名前が異なる場合があります）を変更し、新しいより多くの権限を持つフラスククッキーに署名できるようにします。

### Werkzeug - machine\_id と node uuid

[**この書き込みからのペイロードを使用することで**](https://vozec.fr/writeups/tweedle-dum-dee/)、**machine\_id** と **uuid** ノードにアクセスでき、これらは **Werkzeug pin を生成するために必要な主な秘密**です。[**この pin を使用して、/console で Python コンソールにアクセスできます。**](../../network-services-pentesting/pentesting-web/werkzeug.md) **デバッグモードが有効な場合：**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
注意してください、**サーバーのローカルパスを `app.py`** 取得するには、ウェブページでいくつかの**エラー**を生成する必要があります。これにより**パスが得られます**。
{% endhint %}

脆弱性が別のPythonファイルにある場合は、メインのPythonファイルからオブジェクトにアクセスするために前のFlaskトリックを確認してください。

{% hint style="success" %}
AWSハッキングを学び、練習する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、練習する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
