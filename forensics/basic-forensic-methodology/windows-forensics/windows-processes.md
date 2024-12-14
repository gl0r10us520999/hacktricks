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


## smss.exe

**Gerenciador de Sessão**.\
A Sessão 0 inicia **csrss.exe** e **wininit.exe** (**serviços do SO**) enquanto a Sessão 1 inicia **csrss.exe** e **winlogon.exe** (**sessão do usuário**). No entanto, você deve ver **apenas um processo** desse **binário** sem filhos na árvore de processos.

Além disso, sessões diferentes de 0 e 1 podem significar que sessões RDP estão ocorrendo.


## csrss.exe

**Processo do Subsistema Cliente/Servidor**.\
Ele gerencia **processos** e **threads**, torna a **API do Windows** disponível para outros processos e também **mapeia letras de unidade**, cria **arquivos temporários** e lida com o **processo de desligamento**.

Há um **executando na Sessão 0 e outro na Sessão 1** (portanto, **2 processos** na árvore de processos). Outro é criado **por nova Sessão**.


## winlogon.exe

**Processo de Logon do Windows**.\
É responsável pelo **logon**/**logoff** do usuário. Ele inicia **logonui.exe** para solicitar nome de usuário e senha e, em seguida, chama **lsass.exe** para verificá-los.

Em seguida, ele inicia **userinit.exe**, que é especificado em **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** com a chave **Userinit**.

Além disso, o registro anterior deve ter **explorer.exe** na chave **Shell** ou pode ser abusado como um **método de persistência de malware**.


## wininit.exe

**Processo de Inicialização do Windows**. \
Ele inicia **services.exe**, **lsass.exe** e **lsm.exe** na Sessão 0. Deve haver apenas 1 processo.


## userinit.exe

**Aplicativo de Logon Userinit**.\
Carrega o **ntduser.dat em HKCU** e inicializa o **ambiente do usuário** e executa **scripts de logon** e **GPO**.

Ele inicia **explorer.exe**.


## lsm.exe

**Gerenciador de Sessão Local**.\
Trabalha com smss.exe para manipular sessões de usuário: Logon/logoff, início de shell, bloqueio/desbloqueio de desktop, etc.

Após o W7, lsm.exe foi transformado em um serviço (lsm.dll).

Deve haver apenas 1 processo no W7 e, a partir dele, um serviço executando a DLL.


## services.exe

**Gerenciador de Controle de Serviços**.\
Ele **carrega** **serviços** configurados como **início automático** e **drivers**.

É o processo pai de **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** e muitos mais.

Os serviços são definidos em `HKLM\SYSTEM\CurrentControlSet\Services` e este processo mantém um banco de dados em memória de informações de serviços que podem ser consultadas pelo sc.exe.

Note como **alguns** **serviços** estarão rodando em um **processo próprio** e outros estarão **compartilhando um processo svchost.exe**.

Deve haver apenas 1 processo.


## lsass.exe

**Subsistema da Autoridade de Segurança Local**.\
É responsável pela **autenticação** do usuário e pela criação dos **tokens de segurança**. Utiliza pacotes de autenticação localizados em `HKLM\System\CurrentControlSet\Control\Lsa`.

Escreve no **log de eventos de Segurança** e deve haver apenas 1 processo.

Tenha em mente que este processo é altamente atacado para despejar senhas.


## svchost.exe

**Processo Genérico de Hospedagem de Serviços**.\
Hospeda múltiplos serviços DLL em um único processo compartilhado.

Normalmente, você encontrará que **svchost.exe** é iniciado com a flag `-k`. Isso lançará uma consulta ao registro **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** onde haverá uma chave com o argumento mencionado em -k que conterá os serviços a serem lançados no mesmo processo.

Por exemplo: `-k UnistackSvcGroup` lançará: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Se a **flag `-s`** também for usada com um argumento, então o svchost é solicitado a **lançar apenas o serviço especificado** neste argumento.

Haverá vários processos de `svchost.exe`. Se algum deles **não estiver usando a flag `-k`**, então isso é muito suspeito. Se você descobrir que **services.exe não é o pai**, isso também é muito suspeito.


## taskhost.exe

Este processo atua como um host para processos executando a partir de DLLs. Ele também carrega os serviços que estão sendo executados a partir de DLLs.

No W8, isso é chamado de taskhostex.exe e no W10 taskhostw.exe.


## explorer.exe

Este é o processo responsável pelo **desktop do usuário** e pela abertura de arquivos via extensões de arquivo.

**Apenas 1** processo deve ser gerado **por usuário logado.**

Isso é executado a partir de **userinit.exe**, que deve ser encerrado, então **nenhum pai** deve aparecer para este processo.


# Capturando Processos Maliciosos

* Está rodando a partir do caminho esperado? (Nenhum binário do Windows roda de local temporário)
* Está se comunicando com IPs estranhos?
* Verifique assinaturas digitais (artefatos da Microsoft devem ser assinados)
* Está escrito corretamente?
* Está rodando sob o SID esperado?
* O processo pai é o esperado (se houver)?
* Os processos filhos são os esperados? (sem cmd.exe, wscript.exe, powershell.exe..?)
