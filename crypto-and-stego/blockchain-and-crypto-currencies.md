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


## 基本概念

- **スマートコントラクト**は、特定の条件が満たされたときにブロックチェーン上で実行されるプログラムとして定義され、中介者なしで合意の実行を自動化します。
- **分散型アプリケーション (dApps)**は、スマートコントラクトに基づいて構築され、ユーザーフレンドリーなフロントエンドと透明で監査可能なバックエンドを特徴とします。
- **トークンとコイン**は区別され、コインはデジタルマネーとして機能し、トークンは特定の文脈での価値や所有権を表します。
- **ユーティリティトークン**はサービスへのアクセスを付与し、**セキュリティトークン**は資産の所有権を示します。
- **DeFi**は分散型金融を意味し、中央集権的な権限なしで金融サービスを提供します。
- **DEX**と**DAO**はそれぞれ分散型取引所プラットフォームと分散型自律組織を指します。

## コンセンサスメカニズム

コンセンサスメカニズムは、ブロックチェーン上での安全で合意されたトランザクションの検証を確保します：
- **プルーフ・オブ・ワーク (PoW)**は、トランザクションの検証に計算能力を依存します。
- **プルーフ・オブ・ステーク (PoS)**は、バリデーターが一定量のトークンを保持することを要求し、PoWに比べてエネルギー消費を削減します。

## ビットコインの基本

### トランザクション

ビットコインのトランザクションは、アドレス間での資金の移動を含みます。トランザクションはデジタル署名を通じて検証され、プライベートキーの所有者のみが転送を開始できることを保証します。

#### 主要コンポーネント：

- **マルチシグネチャトランザクション**は、トランザクションを承認するために複数の署名を必要とします。
- トランザクションは**入力**（資金の出所）、**出力**（宛先）、**手数料**（マイナーに支払われる）、および**スクリプト**（トランザクションルール）で構成されます。

### ライトニングネットワーク

ビットコインのスケーラビリティを向上させることを目的としており、チャネル内で複数のトランザクションを可能にし、最終的な状態のみをブロックチェーンにブロードキャストします。

## ビットコインのプライバシーの懸念

プライバシー攻撃、例えば**共通入力所有権**や**UTXO変更アドレス検出**は、トランザクションパターンを悪用します。**ミキサー**や**CoinJoin**のような戦略は、ユーザー間のトランザクションリンクを隠すことで匿名性を向上させます。

## ビットコインを匿名で取得する方法

方法には現金取引、マイニング、ミキサーの使用が含まれます。**CoinJoin**は複数のトランザクションを混ぜて追跡を複雑にし、**PayJoin**はCoinJoinを通常のトランザクションとして偽装してプライバシーを高めます。


# ビットコインのプライバシー攻撃

# ビットコインのプライバシー攻撃の概要

ビットコインの世界では、トランザクションのプライバシーとユーザーの匿名性はしばしば懸念の対象となります。攻撃者がビットコインのプライバシーを侵害する一般的な方法のいくつかを簡潔にまとめました。

## **共通入力所有権の仮定**

異なるユーザーの入力が単一のトランザクションに結合されることは一般的に稀であり、複雑さが関与します。したがって、**同じトランザクション内の2つの入力アドレスは、しばしば同じ所有者に属すると仮定されます**。

## **UTXO変更アドレス検出**

UTXO、または**未使用トランザクション出力**は、トランザクション内で完全に消費されなければなりません。もしその一部だけが別のアドレスに送信されると、残りは新しい変更アドレスに送られます。観察者はこの新しいアドレスが送信者に属すると仮定し、プライバシーが侵害されます。

### 例
これを軽減するために、ミキシングサービスや複数のアドレスを使用することで所有権を隠すことができます。

## **ソーシャルネットワークとフォーラムの露出**

ユーザーは時々自分のビットコインアドレスをオンラインで共有し、**アドレスを所有者にリンクさせることが容易になります**。

## **トランザクショングラフ分析**

トランザクションはグラフとして視覚化でき、資金の流れに基づいてユーザー間の潜在的な接続を明らかにします。

## **不必要な入力ヒューリスティック (最適変更ヒューリスティック)**

このヒューリスティックは、複数の入力と出力を持つトランザクションを分析して、どの出力が送信者に戻る変更であるかを推測することに基づいています。

### 例
```bash
2 btc --> 4 btc
3 btc     1 btc
```
If adding more inputs makes the change output larger than any single input, it can confuse the heuristic.

## **強制アドレス再利用**

攻撃者は以前に使用されたアドレスに少額を送信し、受取人が将来の取引でこれらを他の入力と組み合わせることを期待して、アドレスをリンクさせることを狙います。

### 正しいウォレットの動作
ウォレットは、プライバシーの漏洩を防ぐために、すでに使用された空のアドレスで受け取ったコインを使用することを避けるべきです。

## **その他のブロックチェーン分析技術**

- **正確な支払い額:** お釣りのない取引は、同じユーザーが所有する2つのアドレス間のものである可能性が高いです。
- **丸い数字:** 取引における丸い数字は支払いを示唆し、非丸い出力はお釣りである可能性が高いです。
- **ウォレットフィンガープリンティング:** 異なるウォレットは独自の取引作成パターンを持ち、アナリストが使用されたソフトウェアやお釣りのアドレスを特定できるようにします。
- **金額とタイミングの相関:** 取引の時間や金額を開示することで、取引が追跡可能になることがあります。

## **トラフィック分析**

ネットワークトラフィックを監視することで、攻撃者は取引やブロックをIPアドレスにリンクさせ、ユーザーのプライバシーを侵害する可能性があります。これは、あるエンティティが多くのビットコインノードを運営している場合に特に当てはまり、取引を監視する能力が向上します。

## もっと
プライバシー攻撃と防御の包括的なリストについては、[Bitcoin Privacy on Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy)を訪れてください。

# 匿名ビットコイン取引

## 匿名でビットコインを取得する方法

- **現金取引**: 現金でビットコインを取得すること。
- **現金の代替**: ギフトカードを購入し、それをオンラインでビットコインと交換すること。
- **マイニング**: ビットコインを得る最もプライベートな方法はマイニングであり、特に一人で行う場合は、マイニングプールがマイナーのIPアドレスを知っている可能性があるためです。[マイニングプール情報](https://en.bitcoin.it/wiki/Pooled_mining)
- **盗難**: 理論的には、ビットコインを盗むことも匿名で取得する方法の一つですが、これは違法であり推奨されません。

## ミキシングサービス

ミキシングサービスを使用することで、ユーザーは**ビットコインを送信**し、**異なるビットコインを受け取る**ことができ、元の所有者を追跡することが難しくなります。しかし、これはサービスがログを保持せず、実際にビットコインを返すことを信頼する必要があります。代替のミキシングオプションにはビットコインカジノがあります。

## CoinJoin

**CoinJoin**は、異なるユーザーからの複数の取引を1つに統合し、入力と出力を一致させようとする人にとってプロセスを複雑にします。その効果にもかかわらず、ユニークな入力と出力のサイズを持つ取引は依然として追跡される可能性があります。

CoinJoinを使用した可能性のある取引の例には、`402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a`と`85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`が含まれます。

詳細については、[CoinJoin](https://coinjoin.io/en)を訪れてください。Ethereumの同様のサービスについては、マイナーからの資金で取引を匿名化する[Tornado Cash](https://tornado.cash)をチェックしてください。

## PayJoin

CoinJoinのバリアントである**PayJoin**（またはP2EP）は、2つの当事者（例：顧客と商人）間の取引を通常の取引として偽装し、CoinJoinの特徴的な等しい出力を持たないため、非常に検出が難しくなります。これにより、取引監視機関が使用する一般的な入力所有権のヒューリスティックを無効にする可能性があります。
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transactions like the above could be PayJoin, enhancing privacy while remaining indistinguishable from standard bitcoin transactions.

**PayJoinの利用は、従来の監視手法を大きく妨害する可能性があり、**取引のプライバシーを追求する上で有望な発展です。


# 暗号通貨におけるプライバシーのベストプラクティス

## **ウォレット同期技術**

プライバシーとセキュリティを維持するためには、ウォレットをブロックチェーンと同期させることが重要です。2つの方法が際立っています：

- **フルノード**: ブロックチェーン全体をダウンロードすることで、フルノードは最大限のプライバシーを確保します。過去に行われたすべての取引がローカルに保存され、敵がユーザーが関心を持つ取引やアドレスを特定することは不可能です。
- **クライアントサイドブロックフィルタリング**: この方法は、ブロックチェーン内の各ブロックにフィルターを作成し、ウォレットがネットワークの観察者に特定の関心を露呈することなく関連する取引を特定できるようにします。軽量ウォレットはこれらのフィルターをダウンロードし、ユーザーのアドレスと一致する場合にのみフルブロックを取得します。

## **匿名性のためのTorの利用**

ビットコインがピアツーピアネットワークで動作するため、ネットワークとやり取りする際にIPアドレスを隠すためにTorの使用が推奨されます。

## **アドレスの再利用防止**

プライバシーを守るためには、取引ごとに新しいアドレスを使用することが重要です。アドレスを再利用すると、同じ主体に取引を結びつけることでプライバシーが損なわれる可能性があります。現代のウォレットはその設計によりアドレスの再利用を避けるようにしています。

## **取引プライバシーの戦略**

- **複数の取引**: 支払いをいくつかの取引に分割することで、取引額を不明瞭にし、プライバシー攻撃を防ぎます。
- **お釣りの回避**: お釣りの出力が不要な取引を選択することで、プライバシーを向上させ、お釣り検出手法を混乱させます。
- **複数のお釣り出力**: お釣りを避けることができない場合でも、複数のお釣り出力を生成することでプライバシーを改善できます。

# **モネロ：匿名性の灯台**

モネロはデジタル取引における絶対的な匿名性の必要性に応え、高いプライバシー基準を設定しています。

# **イーサリアム：ガスと取引**

## **ガスの理解**

ガスは、イーサリアム上での操作を実行するために必要な計算努力を測定し、**gwei**で価格が設定されています。たとえば、2,310,000 gwei（または0.00231 ETH）の取引は、ガス制限と基本料金が含まれ、マイナーへのインセンティブとしてチップが付与されます。ユーザーは過剰支払いを避けるために最大料金を設定でき、余剰は返金されます。

## **取引の実行**

イーサリアムの取引には送信者と受信者が関与し、どちらもユーザーまたはスマートコントラクトのアドレスである可能性があります。取引には手数料が必要で、マイニングされなければなりません。取引における重要な情報には、受信者、送信者の署名、価値、オプションのデータ、ガス制限、手数料が含まれます。特に、送信者のアドレスは署名から推測されるため、取引データに含める必要はありません。

これらのプラクティスとメカニズムは、プライバシーとセキュリティを優先しながら暗号通貨に関与しようとする人々にとって基礎的なものです。


## 参考文献

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)


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
