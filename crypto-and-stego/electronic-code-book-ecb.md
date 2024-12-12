{% hint style="success" %}
Aprenda e pratique Hacking AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Treinamento AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Treinamento GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Verifique os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

# ECB

(ECB) Electronic Code Book - esquema de criptografia simétrica que **substitui cada bloco do texto claro** pelo **bloco de texto cifrado**. É o esquema de criptografia **mais simples**. A ideia principal é **dividir** o texto claro em **blocos de N bits** (depende do tamanho do bloco de dados de entrada, algoritmo de criptografia) e então criptografar (descriptografar) cada bloco de texto claro usando a única chave.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

O uso do ECB tem múltiplas implicações de segurança:

* **Blocos da mensagem criptografada podem ser removidos**
* **Blocos da mensagem criptografada podem ser movidos**

# Detecção da vulnerabilidade

Imagine que você faz login em um aplicativo várias vezes e **sempre recebe o mesmo cookie**. Isso ocorre porque o cookie do aplicativo é **`<username>|<password>`**.\
Então, você gera dois novos usuários, ambos com a **mesma senha longa** e **quase** o **mesmo** **nome de usuário**.\
Você descobre que os **blocos de 8B** onde a **informação de ambos os usuários** é a mesma são **iguais**. Então, você imagina que isso pode ser porque está sendo usado o **ECB**.

Como no exemplo a seguir. Observe como esses **2 cookies decodificados** têm várias vezes o bloco **`\x23U\xE45K\xCB\x21\xC8`**.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Isso ocorre porque o **nome de usuário e senha desses cookies continha várias vezes a letra "a"** (por exemplo). Os **blocos** que são **diferentes** são blocos que continham **pelo menos 1 caractere diferente** (talvez o delimitador "|" ou alguma diferença necessária no nome de usuário).

Agora, o atacante só precisa descobrir se o formato é `<nome de usuário><delimitador><senha>` ou `<senha><delimitador><nome de usuário>`. Para fazer isso, ele pode simplesmente **gerar vários nomes de usuário** com **nomes de usuário e senhas semelhantes e longos até encontrar o formato e o comprimento do delimitador:**

| Comprimento do nome de usuário: | Comprimento da senha: | Comprimento do nome de usuário + senha: | Comprimento do cookie (após decodificação): |
| ------------------------------- | --------------------- | --------------------------------------- | ------------------------------------------- |
| 2                               | 2                     | 4                                       | 8                                           |
| 3                               | 3                     | 6                                       | 8                                           |
| 3                               | 4                     | 7                                       | 8                                           |
| 4                               | 4                     | 8                                       | 16                                          |
| 7                               | 7                     | 14                                      | 16                                          |

# Exploração da vulnerabilidade

## Removendo blocos inteiros

Conhecendo o formato do cookie (`<nome de usuário>|<senha>`), para se passar pelo nome de usuário `admin`, crie um novo usuário chamado `aaaaaaaaadmin` e obtenha o cookie e decodifique-o:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Podemos ver o padrão `\x23U\xE45K\xCB\x21\xC8` criado anteriormente com o nome de usuário que continha apenas `a`.\
Em seguida, você pode remover o primeiro bloco de 8B e obter um cookie válido para o nome de usuário `admin`:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Movendo blocos

Em muitos bancos de dados, é a mesma coisa procurar por `WHERE username='admin';` ou por `WHERE username='admin    ';` _(Note os espaços extras)_

Assim, outra maneira de se passar pelo usuário `admin` seria:

* Gerar um nome de usuário que: `len(<username>) + len(<delimiter) % len(block)`. Com um tamanho de bloco de `8B`, você pode gerar um nome de usuário chamado: `username       `, com o delimitador `|` o pedaço `<username><delimiter>` irá gerar 2 blocos de 8Bs.
* Em seguida, gerar uma senha que preencherá um número exato de blocos contendo o nome de usuário que queremos nos passar e espaços, como: `admin   `

O cookie deste usuário será composto por 3 blocos: os 2 primeiros são os blocos do nome de usuário + delimitador e o terceiro da senha (que está fingindo ser o nome de usuário): `username       |admin   `

**Então, basta substituir o primeiro bloco pelo último e estaremos nos passando pelo usuário `admin`: `admin          |username`**

## Referências

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))
