{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Treinamento HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Treinamento HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Verifique os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}


## smss.exe

**Gerenciador de Sessão**.\
A Sessão 0 inicia **csrss.exe** e **wininit.exe** (**serviços do SO**) enquanto a Sessão 1 inicia **csrss.exe** e **winlogon.exe** (**sessão de usuário**). No entanto, você deve ver **apenas um processo** desse **binário** sem filhos na árvore de processos.

Além disso, sessões diferentes de 0 e 1 podem significar que sessões RDP estão ocorrendo.


## csrss.exe

**Processo de Subsistema de Execução Cliente/Servidor**.\
Ele gerencia **processos** e **threads**, disponibiliza a **API do Windows** para outros processos e também **mapeia letras de unidade**, cria **arquivos temporários** e lida com o **processo de desligamento**.

Há um **executando na Sessão 0 e outro na Sessão 1** (então **2 processos** na árvore de processos). Outro é criado **por nova Sessão**.


## winlogon.exe

**Processo de Logon do Windows**.\
É responsável pelos **logons**/**logoffs** do usuário. Ele inicia o **logonui.exe** para solicitar nome de usuário e senha e depois chama o **lsass.exe** para verificá-los.

Em seguida, ele inicia o **userinit.exe** que é especificado em **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** com a chave **Userinit**.

Além disso, o registro anterior deve ter o **explorer.exe** na chave **Shell** ou ele pode ser abusado como um **método de persistência de malware**.


## wininit.exe

**Processo de Inicialização do Windows**. \
Ele inicia **services.exe**, **lsass.exe** e **lsm.exe** na Sessão 0. Deve haver apenas 1 processo.


## userinit.exe

**Aplicativo de Logon do Usuário**.\
Carrega o **ntduser.dat em HKCU** e inicializa o **ambiente do usuário** e executa **scripts de logon** e **GPO**.

Ele inicia o **explorer.exe**.


## lsm.exe

**Gerenciador de Sessão Local**.\
Ele trabalha com smss.exe para manipular sessões de usuário: logon/logoff, inicialização do shell, bloqueio/desbloqueio da área de trabalho, etc.

Após o W7, lsm.exe foi transformado em um serviço (lsm.dll).

Deve haver apenas 1 processo no W7 e a partir deles um serviço executando o DLL.


## services.exe

**Gerenciador de Controle de Serviço**.\
Ele **carrega** **serviços** configurados como **inicialização automática** e **drivers**.

É o processo pai de **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** e muitos outros.

Os serviços são definidos em `HKLM\SYSTEM\CurrentControlSet\Services` e esse processo mantém um banco de dados na memória das informações do serviço que podem ser consultadas pelo sc.exe.

Observe como **alguns** **serviços** serão executados em um **processo próprio** e outros serão **compartilhados em um processo svchost.exe**.

Deve haver apenas 1 processo.


## lsass.exe

**Subsistema de Autoridade de Segurança Local**.\
É responsável pela autenticação do usuário e cria os **tokens de segurança**. Ele usa pacotes de autenticação localizados em `HKLM\System\CurrentControlSet\Control\Lsa`.

Ele escreve no **log de eventos de segurança** e deve haver apenas 1 processo.

Lembre-se de que esse processo é altamente atacado para extrair senhas.


## svchost.exe

**Processo de Hospedagem de Serviço Genérico**.\
Ele hospeda vários serviços DLL em um processo compartilhado.

Normalmente, você encontrará que o **svchost.exe** é iniciado com a flag `-k`. Isso iniciará uma consulta ao registro **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** onde haverá uma chave com o argumento mencionado em -k que conterá os serviços a serem iniciados no mesmo processo.

Por exemplo: `-k UnistackSvcGroup` iniciará: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Se a **flag `-s`** também for usada com um argumento, então o svchost é solicitado a **iniciar apenas o serviço especificado** neste argumento.

Haverá vários processos de `svchost.exe`. Se algum deles **não estiver usando a flag `-k`**, isso é muito suspeito. Se você descobrir que o **services.exe não é o pai**, isso também é muito suspeito.


## taskhost.exe

Este processo atua como um host para processos em execução a partir de DLLs. Ele também carrega os serviços em execução a partir de DLLs.

No W8, isso é chamado de taskhostex.exe e no W10 de taskhostw.exe.


## explorer.exe

Este é o processo responsável pela **área de trabalho do usuário** e pela abertura de arquivos por meio de extensões de arquivo.

Deve ser gerado **apenas 1** processo por **usuário conectado**.

Isso é executado a partir do **userinit.exe** que deve ser encerrado, então **nenhum processo pai** deve aparecer para este processo.


# Capturando Processos Maliciosos

* Está sendo executado no caminho esperado? (Nenhum binário do Windows é executado a partir de uma localização temporária)
* Está se comunicando com IPs estranhos?
* Verifique as assinaturas digitais (os artefatos da Microsoft devem ser assinados)
* Está escrito corretamente?
* Está sendo executado sob o SID esperado?
* O processo pai é o esperado (se houver)?
* Os processos filhos são os esperados? (sem cmd.exe, wscript.exe, powershell.exe..?)


{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Treinamento HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Treinamento HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Verifique os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
