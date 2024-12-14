# macOS Defensive Apps

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Firewalls

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): 各プロセスによって行われるすべての接続を監視します。モード（接続を静かに許可、接続を静かに拒否し警告）に応じて、新しい接続が確立されるたびに**警告を表示**します。また、この情報をすべて見るための非常に良いGUIがあります。
* [**LuLu**](https://objective-see.org/products/lulu.html): Objective-Seeのファイアウォール。これは、疑わしい接続に対して警告を出す基本的なファイアウォールです（GUIはありますが、Little Snitchのものほど派手ではありません）。

## Persistence detection

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): **マルウェアが持続している可能性のある**いくつかの場所を検索するObjective-Seeのアプリケーションです（これは一回限りのツールで、監視サービスではありません）。
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): KnockKnockのように、持続性を生成するプロセスを監視します。

## Keyloggers detection

* [**ReiKey**](https://objective-see.org/products/reikey.html): キーボードの「イベントタップ」をインストールする**キーロガー**を見つけるためのObjective-Seeのアプリケーションです。
