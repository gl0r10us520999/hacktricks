# Weaponizing Distroless

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

## What is Distroless

Distrolessコンテナは、**特定のアプリケーションを実行するために必要な依存関係のみを含む**コンテナの一種であり、必要のない追加のソフトウェアやツールは含まれていません。これらのコンテナは、**軽量**かつ**安全**であることを目的としており、不要なコンポーネントを削除することで**攻撃面を最小限に抑える**ことを目指しています。

Distrolessコンテナは、**セキュリティと信頼性が最も重要な**プロダクション環境でよく使用されます。

**Distrolessコンテナのいくつかの例**は次のとおりです：

* **Google**が提供： [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* **Chainguard**が提供： [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Weaponizing Distroless

Distrolessコンテナを武器化する目的は、**distrolessによって示される制限**（システム内の一般的なバイナリの欠如）や、**読み取り専用**や**実行不可**などのコンテナに一般的に見られる保護にもかかわらず、**任意のバイナリやペイロードを実行できる**ようにすることです。

### Through memory

2023年のある時点で...

### Via Existing binaries

#### openssl

****[**この投稿では、**](https://www.form3.tech/engineering/content/exploiting-distroless-images)バイナリ**`openssl`**がこれらのコンテナに頻繁に見られることが説明されており、これは**コンテナ内で実行されるソフトウェアに必要**である可能性があります。
