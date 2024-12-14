# FZ - Sub-GHz

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


## Intro <a href="#kfpn7" id="kfpn7"></a>

Flipper Zeroは、**300-928 MHzの範囲の無線周波数を受信および送信**できる内蔵モジュールを備えており、リモコンを読み取り、保存し、エミュレートできます。これらのコントロールは、ゲート、バリア、無線ロック、リモートコントロールスイッチ、ワイヤレスドアベル、スマートライトなどとのインタラクションに使用されます。Flipper Zeroは、あなたのセキュリティが侵害されているかどうかを学ぶのに役立ちます。

<figure><img src="../../../.gitbook/assets/image (714).png" alt=""><figcaption></figcaption></figure>

## Sub-GHz hardware <a href="#kfpn7" id="kfpn7"></a>

Flipper Zeroは、[﻿](https://www.st.com/en/nfc/st25r3916.html#overview)﻿[CC1101チップ](https://www.ti.com/lit/ds/symlink/cc1101.pdf)に基づく内蔵のサブ1 GHzモジュールとラジオアンテナを備えており（最大範囲は50メートル）、CC1101チップとアンテナは、300-348 MHz、387-464 MHz、および779-928 MHzの周波数帯域で動作するように設計されています。

<figure><img src="../../../.gitbook/assets/image (923).png" alt=""><figcaption></figcaption></figure>

## Actions

### Frequency Analyser

{% hint style="info" %}
リモコンが使用している周波数を見つける方法
{% endhint %}

分析中、Flipper Zeroは周波数設定で利用可能なすべての周波数で信号強度（RSSI）をスキャンしています。Flipper Zeroは、RSSI値が最も高い周波数を表示し、信号強度が-90 [dBm](https://en.wikipedia.org/wiki/DBm)より高い場合に表示します。

リモコンの周波数を特定するには、次の手順を実行します：

1. リモコンをFlipper Zeroの左側に非常に近く置きます。
2. **メインメニュー** **→ Sub-GHz**に移動します。
3. **Frequency Analyzer**を選択し、分析したいリモコンのボタンを押し続けます。
4. 画面に表示される周波数値を確認します。

### Read

{% hint style="info" %}
使用されている周波数に関する情報を見つける（使用されている周波数を見つける別の方法）
{% endhint %}

**Read**オプションは、指定された変調で**設定された周波数をリスニング**します：デフォルトは433.92 AMです。**読み取り中に何かが見つかった場合、**画面に**情報が表示されます**。この情報は、将来信号を再現するために使用できます。

Readを使用中は、**左ボタン**を押して**設定する**ことができます。\
この時点で、**4つの変調**（AM270、AM650、FM328、FM476）と**いくつかの関連周波数**が保存されています：

<figure><img src="../../../.gitbook/assets/image (947).png" alt=""><figcaption></figcaption></figure>

**興味のある周波数を設定できますが、**リモコンが使用している可能性のある周波数が**不明な場合は、HoppingをONに設定**（デフォルトはOFF）し、Flipperがそれをキャプチャして周波数を設定するために必要な情報を提供するまでボタンを何度も押してください。

{% hint style="danger" %}
周波数を切り替えるには時間がかかるため、切り替え時に送信された信号が見逃される可能性があります。信号の受信を改善するために、Frequency Analyzerによって決定された固定周波数を設定してください。
{% endhint %}

### **Read Raw**

{% hint style="info" %}
設定された周波数で信号を盗む（再生する）
{% endhint %}

**Read Raw**オプションは、リスニング周波数で送信された信号を**記録**します。これを使用して、信号を**盗み**、**繰り返す**ことができます。

デフォルトでは、**Read RawもAM650の433.92で動作します**が、Readオプションで興味のある信号が**異なる周波数/変調にあることがわかった場合は、左ボタンを押してそれを変更することもできます**（Read Rawオプション内で）。

### Brute-Force

ガレージドアで使用されるプロトコルがわかっている場合、**すべてのコードを生成し、Flipper Zeroで送信することが可能です。**これは、一般的なガレージのタイプをサポートする例です：[**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)

### Add Manually

{% hint style="info" %}
設定されたプロトコルのリストから信号を追加する
{% endhint %}

#### [サポートされているプロトコルのリスト](https://docs.flipperzero.one/sub-ghz/add-new-remote) <a href="#id-3iglu" id="id-3iglu"></a>

| Princeton\_433（ほとんどの静的コードシステムで動作） | 433.92 | 静的  |
| --------------------------------------------------------------- | ------ | ------- |
| Nice Flo 12bit\_433                                             | 433.92 | 静的  |
| Nice Flo 24bit\_433                                             | 433.92 | 静的  |
| CAME 12bit\_433                                                 | 433.92 | 静的  |
| CAME 24bit\_433                                                 | 433.92 | 静的  |
| Linear\_300                                                     | 300.00 | 静的  |
| CAME TWEE                                                       | 433.92 | 静的  |
| Gate TX\_433                                                    | 433.92 | 静的  |
| DoorHan\_315                                                    | 315.00 | 動的  |
| DoorHan\_433                                                    | 433.92 | 動的  |
| LiftMaster\_315                                                 | 315.00 | 動的  |
| LiftMaster\_390                                                 | 390.00 | 動的  |
| Security+2.0\_310                                               | 310.00 | 動的  |
| Security+2.0\_315                                               | 315.00 | 動的  |
| Security+2.0\_390                                               | 390.00 | 動的  |

### サポートされているSub-GHzベンダー

[https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors)のリストを確認してください。

### 地域別のサポートされている周波数

[https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies)のリストを確認してください。

### Test

{% hint style="info" %}
保存された周波数のdBmsを取得する
{% endhint %}

## Reference

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

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
