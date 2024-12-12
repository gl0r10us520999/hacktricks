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

## Integridade do Firmware

O **firmware personalizado e/ou binários compilados podem ser carregados para explorar falhas de integridade ou verificação de assinatura**. Os seguintes passos podem ser seguidos para a compilação de um shell bind de backdoor:

1. O firmware pode ser extraído usando firmware-mod-kit (FMK).
2. A arquitetura do firmware alvo e a ordem de bytes devem ser identificadas.
3. Um compilador cruzado pode ser construído usando Buildroot ou outros métodos adequados para o ambiente.
4. A backdoor pode ser construída usando o compilador cruzado.
5. A backdoor pode ser copiada para o diretório /usr/bin do firmware extraído.
6. O binário QEMU apropriado pode ser copiado para o rootfs do firmware extraído.
7. A backdoor pode ser emulada usando chroot e QEMU.
8. A backdoor pode ser acessada via netcat.
9. O binário QEMU deve ser removido do rootfs do firmware extraído.
10. O firmware modificado pode ser reempacotado usando FMK.
11. O firmware com backdoor pode ser testado emulando-o com a ferramenta de análise de firmware (FAT) e conectando-se ao IP e porta da backdoor alvo usando netcat.

Se um shell root já foi obtido através de análise dinâmica, manipulação do bootloader ou testes de segurança de hardware, binários maliciosos pré-compilados, como implantes ou shells reversos, podem ser executados. Ferramentas automatizadas de payload/implante, como o framework Metasploit e 'msfvenom', podem ser aproveitadas usando os seguintes passos:

1. A arquitetura do firmware alvo e a ordem de bytes devem ser identificadas.
2. Msfvenom pode ser usado para especificar o payload alvo, IP do host atacante, número da porta de escuta, tipo de arquivo, arquitetura, plataforma e o arquivo de saída.
3. O payload pode ser transferido para o dispositivo comprometido e garantir que ele tenha permissões de execução.
4. O Metasploit pode ser preparado para lidar com solicitações de entrada iniciando o msfconsole e configurando as configurações de acordo com o payload.
5. O shell reverso meterpreter pode ser executado no dispositivo comprometido.
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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
