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


# ECB

(ECB) Livro de Código Eletrônico - esquema de criptografia simétrica que **substitui cada bloco do texto claro** pelo **bloco de texto cifrado**. É o esquema de criptografia **mais simples**. A ideia principal é **dividir** o texto claro em **blocos de N bits** (depende do tamanho do bloco de dados de entrada, algoritmo de criptografia) e então criptografar (descriptografar) cada bloco do texto claro usando a única chave.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Usar ECB tem múltiplas implicações de segurança:

* **Blocos da mensagem criptografada podem ser removidos**
* **Blocos da mensagem criptografada podem ser movidos**

# Detecção da vulnerabilidade

Imagine que você faz login em um aplicativo várias vezes e **sempre recebe o mesmo cookie**. Isso ocorre porque o cookie do aplicativo é **`<username>|<password>`**.\
Então, você gera dois novos usuários, ambos com a **mesma senha longa** e **quase** o **mesmo** **nome de usuário**.\
Você descobre que os **blocos de 8B** onde a **info de ambos os usuários** é a mesma são **iguais**. Então, você imagina que isso pode ser porque **ECB está sendo usado**.

Como no exemplo a seguir. Observe como esses **2 cookies decodificados** têm várias vezes o bloco **`\x23U\xE45K\xCB\x21\xC8`**.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Isso ocorre porque o **nome de usuário e a senha desses cookies continham várias vezes a letra "a"** (por exemplo). Os **blocos** que são **diferentes** são blocos que continham **pelo menos 1 caractere diferente** (talvez o delimitador "|" ou alguma diferença necessária no nome de usuário).

Agora, o atacante só precisa descobrir se o formato é `<username><delimiter><password>` ou `<password><delimiter><username>`. Para fazer isso, ele pode apenas **gerar vários nomes de usuário** com **nomes de usuário e senhas longos e semelhantes até encontrar o formato e o comprimento do delimitador:**

| Comprimento do nome de usuário: | Comprimento da senha: | Comprimento do nome de usuário+senha: | Comprimento do cookie (após decodificação): |
| ------------------------------- | --------------------- | ------------------------------------- | ------------------------------------------- |
| 2                               | 2                     | 4                                     | 8                                           |
| 3                               | 3                     | 6                                     | 8                                           |
| 3                               | 4                     | 7                                     | 8                                           |
| 4                               | 4                     | 8                                     | 16                                          |
| 7                               | 7                     | 14                                    | 16                                          |

# Exploração da vulnerabilidade

## Removendo blocos inteiros

Sabendo o formato do cookie (`<username>|<password>`), para se passar pelo nome de usuário `admin`, crie um novo usuário chamado `aaaaaaaaadmin` e obtenha o cookie e decodifique-o:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Podemos ver o padrão `\x23U\xE45K\xCB\x21\xC8` criado anteriormente com o nome de usuário que continha apenas `a`.\
Então, você pode remover o primeiro bloco de 8B e obterá um cookie válido para o nome de usuário `admin`:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Movendo blocos

Em muitos bancos de dados, é o mesmo procurar por `WHERE username='admin';` ou por `WHERE username='admin    ';` _(Note os espaços extras)_

Assim, outra maneira de se passar pelo usuário `admin` seria:

* Gerar um nome de usuário que: `len(<username>) + len(<delimiter) % len(block)`. Com um tamanho de bloco de `8B`, você pode gerar um nome de usuário chamado: `username       `, com o delimitador `|`, o bloco `<username><delimiter>` gerará 2 blocos de 8Bs.
* Em seguida, gerar uma senha que preencherá um número exato de blocos contendo o nome de usuário que queremos imitar e espaços, como: `admin   `

O cookie deste usuário será composto por 3 blocos: os primeiros 2 são os blocos do nome de usuário + delimitador e o terceiro da senha (que está falsificando o nome de usuário): `username       |admin   `

**Então, basta substituir o primeiro bloco pelo último e você estará se passando pelo usuário `admin`: `admin          |username`**

## Referências

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


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
