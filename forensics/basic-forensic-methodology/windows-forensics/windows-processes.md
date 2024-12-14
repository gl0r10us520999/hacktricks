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

**Session Manager**.\
La Sessione 0 avvia **csrss.exe** e **wininit.exe** (**servizi** **OS**) mentre la Sessione 1 avvia **csrss.exe** e **winlogon.exe** (**sessione** **utente**). Tuttavia, dovresti vedere **solo un processo** di quel **binario** senza figli nell'albero dei processi.

Inoltre, sessioni diverse da 0 e 1 possono significare che si stanno verificando sessioni RDP.


## csrss.exe

**Client/Server Run Subsystem Process**.\
Gestisce **processi** e **thread**, rende disponibile l'**API** **Windows** per altri processi e mappa anche le **lettere di unità**, crea **file temporanei** e gestisce il **processo** di **spegnimento**.

C'è un **processo in esecuzione nella Sessione 0 e un altro nella Sessione 1** (quindi **2 processi** nell'albero dei processi). Un altro viene creato **per ogni nuova Sessione**.


## winlogon.exe

**Windows Logon Process**.\
È responsabile per il **logon**/**logoff** dell'utente. Avvia **logonui.exe** per chiedere nome utente e password e poi chiama **lsass.exe** per verificarli.

Poi avvia **userinit.exe** che è specificato in **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** con la chiave **Userinit**.

Inoltre, il registro precedente dovrebbe avere **explorer.exe** nella chiave **Shell** o potrebbe essere abusato come un **metodo di persistenza malware**.


## wininit.exe

**Windows Initialization Process**. \
Avvia **services.exe**, **lsass.exe** e **lsm.exe** nella Sessione 0. Dovrebbe esserci solo 1 processo.


## userinit.exe

**Userinit Logon Application**.\
Carica il **ntduser.dat in HKCU** e inizializza l'**ambiente** **utente** e esegue **script di logon** e **GPO**.

Avvia **explorer.exe**.


## lsm.exe

**Local Session Manager**.\
Lavora con smss.exe per manipolare le sessioni utente: logon/logoff, avvio della shell, blocco/sblocco del desktop, ecc.

Dopo W7, lsm.exe è stato trasformato in un servizio (lsm.dll).

Dovrebbe esserci solo 1 processo in W7 e da esso un servizio che esegue la DLL.


## services.exe

**Service Control Manager**.\
**Carica** **servizi** configurati come **auto-avvio** e **driver**.

È il processo padre di **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** e molti altri.

I servizi sono definiti in `HKLM\SYSTEM\CurrentControlSet\Services` e questo processo mantiene un DB in memoria delle informazioni sui servizi che possono essere interrogate da sc.exe.

Nota come **alcuni** **servizi** verranno eseguiti in un **processo proprio** e altri condivideranno un processo **svchost.exe**.

Dovrebbe esserci solo 1 processo.


## lsass.exe

**Local Security Authority Subsystem**.\
È responsabile per l'**autenticazione** dell'utente e crea i **token** di **sicurezza**. Utilizza pacchetti di autenticazione situati in `HKLM\System\CurrentControlSet\Control\Lsa`.

Scrive nel **registro eventi di sicurezza** e dovrebbe esserci solo 1 processo.

Tieni presente che questo processo è altamente attaccato per estrarre password.


## svchost.exe

**Generic Service Host Process**.\
Ospita più servizi DLL in un unico processo condiviso.

Di solito, troverai che **svchost.exe** viene avviato con il flag `-k`. Questo avvierà una query al registro **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** dove ci sarà una chiave con l'argomento menzionato in -k che conterrà i servizi da avviare nello stesso processo.

Ad esempio: `-k UnistackSvcGroup` avvierà: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Se il **flag `-s`** viene utilizzato anche con un argomento, allora svchost viene chiesto di **avviare solo il servizio specificato** in questo argomento.

Ci saranno diversi processi di `svchost.exe`. Se uno di essi **non utilizza il flag `-k`**, allora è molto sospetto. Se scopri che **services.exe non è il padre**, è anche molto sospetto.


## taskhost.exe

Questo processo funge da host per i processi in esecuzione da DLL. Carica anche i servizi che vengono eseguiti da DLL.

In W8 questo è chiamato taskhostex.exe e in W10 taskhostw.exe.


## explorer.exe

Questo è il processo responsabile per il **desktop dell'utente** e per l'apertura di file tramite estensioni di file.

**Solo 1** processo dovrebbe essere avviato **per ogni utente connesso.**

Questo viene eseguito da **userinit.exe** che dovrebbe essere terminato, quindi **nessun padre** dovrebbe apparire per questo processo.


# Catching Malicious Processes

* Sta girando dal percorso previsto? (Nessun binario Windows gira da una posizione temporanea)
* Sta comunicando con IP strani?
* Controlla le firme digitali (gli artefatti Microsoft dovrebbero essere firmati)
* È scritto correttamente?
* Sta girando sotto il SID previsto?
* È il processo padre quello previsto (se presente)?
* I processi figli sono quelli attesi? (no cmd.exe, wscript.exe, powershell.exe..?)


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
