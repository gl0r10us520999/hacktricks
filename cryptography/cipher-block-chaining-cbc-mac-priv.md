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


# CBC

もし**クッキー**が**ユーザー名**だけで（またはクッキーの最初の部分がユーザー名で）、「**admin**」というユーザー名を偽装したい場合、ユーザー名**"bdmin"**を作成し、**最初のバイト**を**ブルートフォース**することができます。

# CBC-MAC

**暗号ブロック連鎖メッセージ認証コード**（**CBC-MAC**）は、暗号学で使用される方法です。これは、メッセージをブロックごとに暗号化し、各ブロックの暗号化が前のブロックにリンクされることで機能します。このプロセスは**ブロックの連鎖**を作成し、元のメッセージのビットを1つ変更するだけでも、暗号化されたデータの最後のブロックに予測不可能な変化をもたらすことを保証します。このような変更を行うためには、暗号化キーが必要であり、セキュリティが確保されます。

メッセージmのCBC-MACを計算するには、ゼロ初期化ベクトルでCBCモードでmを暗号化し、最後のブロックを保持します。以下の図は、秘密鍵kとブロック暗号Eを使用して、ブロックからなるメッセージのCBC-MACの計算を概略しています！[https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) 

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

CBC-MACでは通常**使用されるIVは0**です。\
これは問題です。なぜなら、2つの既知のメッセージ（`m1`と`m2`）が独立して2つの署名（`s1`と`s2`）を生成するからです。したがって：

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

次に、m1とm2を連結したメッセージ（m3）は2つの署名（s31とs32）を生成します：

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**これは暗号化のキーを知らなくても計算可能です。**

あなたが**Administrator**という名前を**8バイト**のブロックで暗号化していると想像してください：

* `Administ`
* `rator\00\00\00`

ユーザー名**Administ**（m1）を作成し、署名（s1）を取得できます。\
次に、`rator\00\00\00 XOR s1`の結果を持つユーザー名を作成できます。これにより、`E(m2 XOR s1 XOR 0)`が生成され、これはs32になります。\
今、s32を**Administrator**というフルネームの署名として使用できます。

### Summary

1. ユーザー名**Administ**（m1）の署名を取得し、それがs1です。
2. ユーザー名**rator\x00\x00\x00 XOR s1 XOR 0**の署名を取得し、それがs32です。**
3. クッキーをs32に設定すると、ユーザー**Administrator**の有効なクッキーになります。

# Attack Controlling IV

使用されるIVを制御できる場合、攻撃は非常に簡単になる可能性があります。\
クッキーが単に暗号化されたユーザー名である場合、ユーザー「**administrator**」を偽装するために、ユーザー「**Administrator**」を作成し、そのクッキーを取得できます。\
今、IVを制御できる場合、IVの最初のバイトを変更することができ、**IV\[0] XOR "A" == IV'\[0] XOR "a"**となり、ユーザー**Administrator**のクッキーを再生成できます。このクッキーは、初期**IV**を使用してユーザー**administrator**を**偽装する**のに有効です。

## References

詳細情報は[https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)を参照してください。


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
