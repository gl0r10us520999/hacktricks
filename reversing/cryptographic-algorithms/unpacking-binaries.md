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


# パックされたバイナリの識別

* **文字列の欠如**: パックされたバイナリにはほとんど文字列がないことが一般的です。
* **未使用の文字列**: また、マルウェアが商業用パッカーを使用している場合、相互参照のない多くの文字列が見つかることが一般的です。これらの文字列が存在しても、バイナリがパックされていないことを意味するわけではありません。
* バイナリをパックするために使用されたパッカーを特定するために、いくつかのツールを使用することもできます：
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# 基本的な推奨事項

* **IDAでパックされたバイナリを下から上に**分析し始めます。アンパッカーはアンパックされたコードが終了すると終了するため、アンパッカーが最初にアンパックされたコードに実行を渡すことは考えにくいです。
* **レジスタ**や**メモリ**の**領域**への**JMP**や**CALL**を探します。また、**引数とアドレスをプッシュしてから`retn`を呼び出す関数**を探します。なぜなら、その場合、関数の戻りはスタックにプッシュされたアドレスを呼び出す可能性があるからです。
* `VirtualAlloc`に**ブレークポイント**を設定します。これは、プログラムがアンパックされたコードを書き込むためのメモリ内のスペースを割り当てます。「ユーザーコードまで実行」またはF8を使用して、関数を実行した後に**EAX内の値を取得**します。そして「**ダンプ内のそのアドレスを追跡**」します。アンパックされたコードが保存される領域であるかもしれません。
* **`VirtualAlloc`**に**「40」**という値を引数として渡すことは、読み取り+書き込み+実行を意味します（実行が必要なコードがここにコピーされる予定です）。
* コードを**アンパックしている間**、**算術演算**や**`memcopy`**や**`Virtual`**`Alloc`のような関数への**いくつかの呼び出し**を見つけることが一般的です。もし、算術演算のみを行い、場合によっては`memcopy`を行う関数にいる場合、**関数の終わりを見つける**（おそらくレジスタへのJMPまたはCALL）**か**、少なくとも**最後の関数への呼び出しを見つけて実行する**ことをお勧めします。なぜなら、そのコードは興味深くないからです。
* コードをアンパックしている間、**メモリ領域が変更されるたびに注意**してください。メモリ領域の変更は、**アンパックコードの開始を示す**可能性があります。Process Hackerを使用してメモリ領域を簡単にダンプできます（プロセス --> プロパティ --> メモリ）。
* コードをアンパックしようとしているとき、**アンパックされたコードで作業しているかどうかを知る良い方法**（その場合は単にダンプできます）は、**バイナリの文字列を確認すること**です。ある時点でジャンプを行い（おそらくメモリ領域を変更）、**はるかに多くの文字列が追加されたことに気付いた場合**、**アンパックされたコードで作業していることがわかります**。\
ただし、パッカーにすでに多くの文字列が含まれている場合は、「http」という単語を含む文字列の数を確認し、この数が増加するかどうかを確認できます。
* メモリの領域から実行可能ファイルをダンプするとき、[PE-bear](https://github.com/hasherezade/pe-bear-releases/releases)を使用していくつかのヘッダーを修正できます。


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
