{% hint style="success" %}
Impara e pratica l'hacking di AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'hacking di GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Sostieni HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repository di Github.

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# Timestamp

Un attaccante potrebbe essere interessato a **modificare i timestamp dei file** per evitare di essere rilevato.\
È possibile trovare i timestamp all'interno del MFT negli attributi `$STANDARD_INFORMATION` __ e __ `$FILE_NAME`.

Entrambi gli attributi hanno 4 timestamp: **Modifica**, **accesso**, **creazione** e **modifica del registro MFT** (MACE o MACB).

**Esplora risorse di Windows** e altri strumenti mostrano le informazioni da **`$STANDARD_INFORMATION`**.

## TimeStomp - Strumento anti-forense

Questo strumento **modifica** le informazioni sui timestamp all'interno di **`$STANDARD_INFORMATION`** **ma non** le informazioni all'interno di **`$FILE_NAME`**. Pertanto, è possibile **identificare** **attività sospette**.

## Usnjrnl

Il **USN Journal** (Update Sequence Number Journal) è una caratteristica del NTFS (sistema di file di Windows NT) che tiene traccia delle modifiche del volume. Lo strumento [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) consente di esaminare queste modifiche.

![](<../../.gitbook/assets/image (449).png>)

Nell'immagine precedente è mostrato l'**output** dello **strumento** dove si possono osservare alcune **modifiche effettuate** al file.

## $LogFile

**Tutte le modifiche dei metadati a un sistema di file vengono registrate** in un processo noto come [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead_logging). I metadati registrati sono conservati in un file chiamato `**$LogFile**`, situato nella directory radice di un sistema di file NTFS. Strumenti come [LogFileParser](https://github.com/jschicht/LogFileParser) possono essere utilizzati per analizzare questo file e identificare le modifiche.

![](<../../.gitbook/assets/image (450).png>)

Ancora una volta, nell'output dello strumento è possibile vedere che **alcune modifiche sono state effettuate**.

Utilizzando lo stesso strumento è possibile identificare a **quale ora i timestamp sono stati modificati**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Ora di creazione del file
* ATIME: Ora di modifica del file
* MTIME: Modifica del registro MFT del file
* RTIME: Ora di accesso al file

## Confronto tra `$STANDARD_INFORMATION` e `$FILE_NAME`

Un altro modo per identificare file modificati in modo sospetto sarebbe confrontare l'ora su entrambi gli attributi cercando **discrepanze**.

## Nanosecondi

I timestamp di **NTFS** hanno una **precisione** di **100 nanosecondi**. Quindi, trovare file con timestamp come 2010-10-10 10:10:**00.000:0000 è molto sospetto**.

## SetMace - Strumento anti-forense

Questo strumento può modificare entrambi gli attributi `$STARNDAR_INFORMATION` e `$FILE_NAME`. Tuttavia, a partire da Windows Vista, è necessario un sistema operativo live per modificare queste informazioni.

# Nascondere dati

NFTS utilizza un cluster e la dimensione minima delle informazioni. Ciò significa che se un file occupa un cluster e mezzo, il **restante mezzo non verrà mai utilizzato** fino a quando il file non viene eliminato. Quindi, è possibile **nascondere dati in questo spazio vuoto**.

Ci sono strumenti come slacker che consentono di nascondere dati in questo spazio "nascosto". Tuttavia, un'analisi del `$logfile` e `$usnjrnl` può mostrare che sono stati aggiunti alcuni dati:

![](<../../.gitbook/assets/image (452).png>)

Quindi, è possibile recuperare lo spazio vuoto utilizzando strumenti come FTK Imager. Nota che questo tipo di strumento può salvare il contenuto oscurato o addirittura crittografato.

# UsbKill

Questo è uno strumento che **spegnerà il computer se viene rilevata qualsiasi modifica nelle porte USB**.\
Un modo per scoprirlo sarebbe ispezionare i processi in esecuzione e **esaminare ogni script python in esecuzione**.

# Distribuzioni Linux Live

Queste distribuzioni sono **eseguite all'interno della memoria RAM**. L'unico modo per rilevarle è **nel caso in cui il file system NTFS sia montato con permessi di scrittura**. Se è montato solo con permessi di lettura, non sarà possibile rilevare l'intrusione.

# Cancellazione sicura

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Configurazione di Windows

È possibile disabilitare diversi metodi di registrazione di Windows per rendere molto più difficile l'indagine forense.

## Disabilita Timestamps - UserAssist

Si tratta di una chiave di registro che mantiene le date e le ore in cui ogni eseguibile è stato avviato dall'utente.

Disabilitare UserAssist richiede due passaggi:

1. Impostare due chiavi di registro, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` e `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, entrambe a zero per segnalare che desideriamo disabilitare UserAssist.
2. Cancella i sottoalberi del registro che assomigliano a `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

## Disabilita Timestamps - Prefetch

Questo salverà informazioni sulle applicazioni eseguite con l'obiettivo di migliorare le prestazioni del sistema Windows. Tuttavia, questo può essere utile anche per le pratiche forensi.

* Esegui `regedit`
* Seleziona il percorso del file `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Fai clic con il pulsante destro su entrambi `EnablePrefetcher` e `EnableSuperfetch`
* Seleziona Modifica su ciascuno di questi per cambiare il valore da 1 (o 3) a 0
* Riavvia

## Disabilita Timestamps - Last Access Time

Ogni volta che una cartella viene aperta da un volume NTFS su un server Windows NT, il sistema impiega tempo per **aggiornare un campo timestamp su ciascuna cartella elencata**, chiamato last access time. Su un volume NTFS molto utilizzato, questo può influire sulle prestazioni.

1. Apri l'Editor del Registro di sistema (Regedit.exe).
2. Naviga fino a `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Cerca `NtfsDisableLastAccessUpdate`. Se non esiste, aggiungi questo DWORD e impostane il valore su 1, che disabiliterà il processo.
4. Chiudi l'Editor del Registro di sistema e riavvia il server.
## Eliminare la cronologia USB

Tutte le **voci dei dispositivi USB** sono memorizzate nel Registro di Windows sotto la chiave di registro **USBSTOR** che contiene sottochiavi create ogni volta che si collega un dispositivo USB al PC o al laptop. È possibile trovare questa chiave qui H`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Eliminando questo** si eliminerà la cronologia USB.\
È anche possibile utilizzare lo strumento [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) per assicurarsi di averli eliminati (e per eliminarli).

Un altro file che salva informazioni sugli USB è il file `setupapi.dev.log` all'interno di `C:\Windows\INF`. Anche questo dovrebbe essere eliminato.

## Disabilitare le Copie Shadow

**Elenca** le copie shadow con `vssadmin list shadowstorage`\
Per **eliminarle** eseguire `vssadmin delete shadow`

È anche possibile eliminarle tramite GUI seguendo i passaggi proposti in [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Per disabilitare le copie shadow [passaggi da qui](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows):

1. Aprire il programma Servizi digitando "servizi" nella casella di ricerca di testo dopo aver cliccato sul pulsante di avvio di Windows.
2. Dalla lista, trovare "Volume Shadow Copy", selezionarlo e quindi accedere alle Proprietà facendo clic con il pulsante destro del mouse.
3. Scegliere Disabilitato dal menu a discesa "Tipo di avvio" e quindi confermare la modifica facendo clic su Applica e OK.

È anche possibile modificare la configurazione dei file che verranno copiati nella copia shadow nel registro `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

## Sovrascrivere i file eliminati

* È possibile utilizzare uno strumento **Windows**: `cipher /w:C` Questo indicherà a cipher di rimuovere tutti i dati dallo spazio disco non utilizzato disponibile nel drive C.
* È anche possibile utilizzare strumenti come [**Eraser**](https://eraser.heidi.ie)

## Eliminare i log degli eventi di Windows

* Windows + R --> eventvwr.msc --> Espandi "Log di Windows" --> Fare clic con il pulsante destro su ogni categoria e selezionare "Cancella log"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Disabilitare i log degli eventi di Windows

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* All'interno della sezione dei servizi disabilitare il servizio "Log eventi di Windows"
* `WEvtUtil.exec clear-log` o `WEvtUtil.exe cl`

## Disabilitare $UsnJrnl

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


{% hint style="success" %}
Impara e pratica l'Hacking su AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'Hacking su GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>
{% endhint %}
