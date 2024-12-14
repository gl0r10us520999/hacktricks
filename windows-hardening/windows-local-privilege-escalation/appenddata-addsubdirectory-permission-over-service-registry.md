{% hint style="success" %}
Impara e pratica il hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica il hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos su github.

</details>
{% endhint %}


**Il post originale è** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## Riepilogo

Due chiavi di registro sono state trovate scrivibili dall'utente attuale:

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

È stato suggerito di controllare i permessi del servizio **RpcEptMapper** utilizzando la **GUI di regedit**, specificamente la scheda **Permessi Efficaci** nella finestra **Impostazioni di Sicurezza Avanzate**. Questo approccio consente di valutare i permessi concessi a specifici utenti o gruppi senza la necessità di esaminare ogni voce di controllo accessi (ACE) singolarmente.

Uno screenshot mostrava i permessi assegnati a un utente a basso privilegio, tra cui il permesso **Crea Sottocchiave** era notevole. Questo permesso, noto anche come **AppendData/AddSubdirectory**, corrisponde ai risultati dello script.

È stata notata l'incapacità di modificare direttamente alcuni valori, ma la capacità di creare nuove sottocchiavi. Un esempio evidenziato è stato un tentativo di alterare il valore **ImagePath**, che ha portato a un messaggio di accesso negato.

Nonostante queste limitazioni, è stata identificata una potenzialità di escalation dei privilegi attraverso la possibilità di sfruttare la sottocchiave **Performance** all'interno della struttura di registro del servizio **RpcEptMapper**, una sottocchiave non presente per impostazione predefinita. Questo potrebbe consentire la registrazione di DLL e il monitoraggio delle prestazioni.

È stata consultata la documentazione sulla sottocchiave **Performance** e il suo utilizzo per il monitoraggio delle prestazioni, portando allo sviluppo di una DLL proof-of-concept. Questa DLL, che dimostra l'implementazione delle funzioni **OpenPerfData**, **CollectPerfData** e **ClosePerfData**, è stata testata tramite **rundll32**, confermando il suo successo operativo.

L'obiettivo era costringere il **servizio RPC Endpoint Mapper** a caricare la DLL Performance creata. Le osservazioni hanno rivelato che l'esecuzione di query di classi WMI relative ai Dati di Prestazione tramite PowerShell ha portato alla creazione di un file di log, consentendo l'esecuzione di codice arbitrario nel contesto **LOCAL SYSTEM**, concedendo così privilegi elevati.

Sono state sottolineate la persistenza e le potenziali implicazioni di questa vulnerabilità, evidenziando la sua rilevanza per le strategie di post-sfruttamento, movimento laterale e evasione dei sistemi antivirus/EDR.

Sebbene la vulnerabilità sia stata inizialmente divulgata involontariamente tramite lo script, è stato enfatizzato che il suo sfruttamento è limitato a versioni obsolete di Windows (ad es., **Windows 7 / Server 2008 R2**) e richiede accesso locale.
