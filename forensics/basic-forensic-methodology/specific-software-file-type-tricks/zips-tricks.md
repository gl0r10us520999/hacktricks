# ZIPs tricks

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}

**Ferramentas de linha de comando** para gerenciar **arquivos zip** são essenciais para diagnosticar, reparar e quebrar arquivos zip. Aqui estão algumas utilidades chave:

- **`unzip`**: Revela por que um arquivo zip pode não descompactar.
- **`zipdetails -v`**: Oferece análise detalhada dos campos do formato de arquivo zip.
- **`zipinfo`**: Lista o conteúdo de um arquivo zip sem extraí-lo.
- **`zip -F input.zip --out output.zip`** e **`zip -FF input.zip --out output.zip`**: Tentam reparar arquivos zip corrompidos.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Uma ferramenta para quebra de senhas zip por força bruta, eficaz para senhas de até cerca de 7 caracteres.

A [especificação do formato de arquivo Zip](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) fornece detalhes abrangentes sobre a estrutura e os padrões dos arquivos zip.

É crucial notar que arquivos zip protegidos por senha **não criptografam nomes de arquivos ou tamanhos de arquivos** dentro, uma falha de segurança que não é compartilhada com arquivos RAR ou 7z que criptografam essas informações. Além disso, arquivos zip criptografados com o método mais antigo ZipCrypto são vulneráveis a um **ataque de texto simples** se uma cópia não criptografada de um arquivo comprimido estiver disponível. Este ataque aproveita o conteúdo conhecido para quebrar a senha do zip, uma vulnerabilidade detalhada no [artigo do HackThis](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) e explicada mais detalhadamente [neste artigo acadêmico](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). No entanto, arquivos zip protegidos com criptografia **AES-256** são imunes a esse ataque de texto simples, demonstrando a importância de escolher métodos de criptografia seguros para dados sensíveis.

## Referências
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}
