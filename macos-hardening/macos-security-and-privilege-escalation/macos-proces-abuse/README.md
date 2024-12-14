# macOS Process Abuse

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

## Informazioni di base sui processi

Un processo è un'istanza di un eseguibile in esecuzione, tuttavia i processi non eseguono codice, questi sono thread. Pertanto **i processi sono solo contenitori per thread in esecuzione** che forniscono memoria, descrittori, porte, permessi...

Tradizionalmente, i processi venivano avviati all'interno di altri processi (eccetto PID 1) chiamando **`fork`**, che creava una copia esatta del processo corrente e poi il **processo figlio** generalmente chiamava **`execve`** per caricare il nuovo eseguibile e eseguirlo. Poi, **`vfork`** è stato introdotto per rendere questo processo più veloce senza alcuna copia di memoria.\
Successivamente, è stato introdotto **`posix_spawn`** combinando **`vfork`** e **`execve`** in un'unica chiamata e accettando flag:

* `POSIX_SPAWN_RESETIDS`: Reimposta gli ID effettivi agli ID reali
* `POSIX_SPAWN_SETPGROUP`: Imposta l'affiliazione al gruppo di processi
* `POSUX_SPAWN_SETSIGDEF`: Imposta il comportamento predefinito del segnale
* `POSIX_SPAWN_SETSIGMASK`: Imposta la maschera del segnale
* `POSIX_SPAWN_SETEXEC`: Esegue nello stesso processo (come `execve` con più opzioni)
* `POSIX_SPAWN_START_SUSPENDED`: Avvia sospeso
* `_POSIX_SPAWN_DISABLE_ASLR`: Avvia senza ASLR
* `_POSIX_SPAWN_NANO_ALLOCATOR:` Usa l'allocatore Nano di libmalloc
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Consente `rwx` sui segmenti di dati
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: Chiude tutte le descrizioni di file su exec(2) per impostazione predefinita
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` Randomizza i bit alti dello slide ASLR

Inoltre, `posix_spawn` consente di specificare un array di **`posix_spawnattr`** che controlla alcuni aspetti del processo generato, e **`posix_spawn_file_actions`** per modificare lo stato dei descrittori.

Quando un processo muore, invia il **codice di ritorno al processo padre** (se il padre è morto, il nuovo padre è PID 1) con il segnale `SIGCHLD`. Il padre deve ottenere questo valore chiamando `wait4()` o `waitid()` e fino a quel momento il figlio rimane in uno stato zombie dove è ancora elencato ma non consuma risorse.

### PIDs

I PIDs, identificatori di processo, identificano un processo unico. In XNU i **PID** sono di **64 bit** e aumentano in modo monotono e **non si avvolgono mai** (per evitare abusi).

### Gruppi di processi, sessioni e coalizioni

**I processi** possono essere inseriti in **gruppi** per facilitarne la gestione. Ad esempio, i comandi in uno script shell saranno nello stesso gruppo di processi, quindi è possibile **segnalarli insieme** utilizzando kill, ad esempio.\
È anche possibile **raggruppare i processi in sessioni**. Quando un processo avvia una sessione (`setsid(2)`), i processi figli vengono inseriti all'interno della sessione, a meno che non avviino la propria sessione.

La coalizione è un altro modo per raggruppare i processi in Darwin. Un processo che si unisce a una coalizione gli consente di accedere alle risorse del pool, condividendo un libro mastro o affrontando Jetsam. Le coalizioni hanno ruoli diversi: Leader, servizio XPC, Estensione.

### Credenziali e persone

Ogni processo detiene **credenziali** che **identificano i suoi privilegi** nel sistema. Ogni processo avrà un `uid` primario e un `gid` primario (anche se potrebbe appartenere a più gruppi).\
È anche possibile cambiare l'ID utente e l'ID gruppo se il binario ha il bit `setuid/setgid`.\
Ci sono diverse funzioni per **impostare nuovi uids/gids**.

La syscall **`persona`** fornisce un **insieme alternativo** di **credenziali**. Adottare una persona assume il suo uid, gid e le appartenenze ai gruppi **tutte insieme**. Nel [**codice sorgente**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h) è possibile trovare la struct:
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## Informazioni di Base sui Thread

1. **POSIX Threads (pthreads):** macOS supporta i thread POSIX (`pthreads`), che fanno parte di un'API di threading standard per C/C++. L'implementazione di pthreads in macOS si trova in `/usr/lib/system/libsystem_pthread.dylib`, che proviene dal progetto `libpthread` disponibile pubblicamente. Questa libreria fornisce le funzioni necessarie per creare e gestire i thread.
2. **Creazione di Thread:** La funzione `pthread_create()` viene utilizzata per creare nuovi thread. Internamente, questa funzione chiama `bsdthread_create()`, che è una chiamata di sistema a livello inferiore specifica per il kernel XNU (il kernel su cui si basa macOS). Questa chiamata di sistema prende vari flag derivati da `pthread_attr` (attributi) che specificano il comportamento del thread, comprese le politiche di scheduling e la dimensione dello stack.
* **Dimensione Predefinita dello Stack:** La dimensione predefinita dello stack per i nuovi thread è di 512 KB, che è sufficiente per operazioni tipiche ma può essere regolata tramite gli attributi del thread se è necessario più o meno spazio.
3. **Inizializzazione del Thread:** La funzione `__pthread_init()` è cruciale durante la configurazione del thread, utilizzando l'argomento `env[]` per analizzare le variabili di ambiente che possono includere dettagli sulla posizione e sulla dimensione dello stack.

#### Terminazione del Thread in macOS

1. **Uscita dai Thread:** I thread vengono tipicamente terminati chiamando `pthread_exit()`. Questa funzione consente a un thread di uscire in modo pulito, eseguendo la pulizia necessaria e permettendo al thread di inviare un valore di ritorno a eventuali joiner.
2. **Pulizia del Thread:** Al momento della chiamata a `pthread_exit()`, viene invocata la funzione `pthread_terminate()`, che gestisce la rimozione di tutte le strutture di thread associate. Dealloca le porte di thread Mach (Mach è il sottosistema di comunicazione nel kernel XNU) e chiama `bsdthread_terminate`, una syscall che rimuove le strutture a livello di kernel associate al thread.

#### Meccanismi di Sincronizzazione

Per gestire l'accesso alle risorse condivise e evitare condizioni di gara, macOS fornisce diversi primitivi di sincronizzazione. Questi sono critici negli ambienti multi-threading per garantire l'integrità dei dati e la stabilità del sistema:

1. **Mutex:**
* **Mutex Regolare (Firma: 0x4D555458):** Mutex standard con un'impronta di memoria di 60 byte (56 byte per il mutex e 4 byte per la firma).
* **Mutex Veloce (Firma: 0x4d55545A):** Simile a un mutex regolare ma ottimizzato per operazioni più veloci, anch'esso di 60 byte di dimensione.
2. **Variabili di Condizione:**
* Utilizzate per attendere che si verifichino determinate condizioni, con una dimensione di 44 byte (40 byte più una firma di 4 byte).
* **Attributi della Variabile di Condizione (Firma: 0x434e4441):** Attributi di configurazione per le variabili di condizione, di dimensione 12 byte.
3. **Variabile Once (Firma: 0x4f4e4345):**
* Garantisce che un pezzo di codice di inizializzazione venga eseguito solo una volta. La sua dimensione è di 12 byte.
4. **Lock di Lettura-Scrittura:**
* Consente a più lettori o a uno scrittore alla volta, facilitando l'accesso efficiente ai dati condivisi.
* **Lock di Lettura-Scrittura (Firma: 0x52574c4b):** Dimensionato a 196 byte.
* **Attributi del Lock di Lettura-Scrittura (Firma: 0x52574c41):** Attributi per i lock di lettura-scrittura, di 20 byte di dimensione.

{% hint style="success" %}
Gli ultimi 4 byte di quegli oggetti vengono utilizzati per rilevare overflow.
{% endhint %}

### Variabili Locali al Thread (TLV)

**Variabili Locali al Thread (TLV)** nel contesto dei file Mach-O (il formato per gli eseguibili in macOS) vengono utilizzate per dichiarare variabili specifiche per **ogni thread** in un'applicazione multi-threaded. Questo garantisce che ogni thread abbia la propria istanza separata di una variabile, fornendo un modo per evitare conflitti e mantenere l'integrità dei dati senza la necessità di meccanismi di sincronizzazione espliciti come i mutex.

In C e nei linguaggi correlati, puoi dichiarare una variabile locale al thread utilizzando la parola chiave **`__thread`**. Ecco come funziona nel tuo esempio:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
Questo frammento definisce `tlv_var` come una variabile locale al thread. Ogni thread che esegue questo codice avrà il proprio `tlv_var`, e le modifiche apportate da un thread a `tlv_var` non influenzeranno `tlv_var` in un altro thread.

Nel binario Mach-O, i dati relativi alle variabili locali al thread sono organizzati in sezioni specifiche:

* **`__DATA.__thread_vars`**: Questa sezione contiene i metadati sulle variabili locali al thread, come i loro tipi e lo stato di inizializzazione.
* **`__DATA.__thread_bss`**: Questa sezione è utilizzata per le variabili locali al thread che non sono esplicitamente inizializzate. È una parte della memoria riservata per i dati inizializzati a zero.

Mach-O fornisce anche un'API specifica chiamata **`tlv_atexit`** per gestire le variabili locali al thread quando un thread termina. Questa API consente di **registrare distruttori**—funzioni speciali che puliscono i dati locali al thread quando un thread termina.

### Priorità dei Thread

Comprendere le priorità dei thread implica esaminare come il sistema operativo decide quali thread eseguire e quando. Questa decisione è influenzata dal livello di priorità assegnato a ciascun thread. In macOS e nei sistemi simili a Unix, questo è gestito utilizzando concetti come `nice`, `renice` e classi di Qualità del Servizio (QoS).

#### Nice e Renice

1. **Nice:**
* Il valore `nice` di un processo è un numero che influisce sulla sua priorità. Ogni processo ha un valore nice che varia da -20 (la massima priorità) a 19 (la minima priorità). Il valore nice predefinito quando un processo viene creato è tipicamente 0.
* Un valore nice più basso (più vicino a -20) rende un processo più "egoista", concedendogli più tempo CPU rispetto ad altri processi con valori nice più alti.
2. **Renice:**
* `renice` è un comando utilizzato per cambiare il valore nice di un processo già in esecuzione. Questo può essere utilizzato per regolare dinamicamente la priorità dei processi, aumentando o diminuendo la loro allocazione di tempo CPU in base ai nuovi valori nice.
* Ad esempio, se un processo ha bisogno di più risorse CPU temporaneamente, potresti abbassare il suo valore nice usando `renice`.

#### Classi di Qualità del Servizio (QoS)

Le classi QoS sono un approccio più moderno per gestire le priorità dei thread, in particolare in sistemi come macOS che supportano **Grand Central Dispatch (GCD)**. Le classi QoS consentono agli sviluppatori di **categorizzare** il lavoro in diversi livelli in base alla loro importanza o urgenza. macOS gestisce automaticamente la priorità dei thread in base a queste classi QoS:

1. **Interattivo per l'Utente:**
* Questa classe è per i compiti che stanno attualmente interagendo con l'utente o richiedono risultati immediati per fornire una buona esperienza utente. Questi compiti ricevono la massima priorità per mantenere l'interfaccia reattiva (ad es., animazioni o gestione degli eventi).
2. **Iniziato dall'Utente:**
* Compiti che l'utente inizia e si aspetta risultati immediati, come aprire un documento o fare clic su un pulsante che richiede calcoli. Questi hanno alta priorità ma sono inferiori a quelli interattivi per l'utente.
3. **Utilità:**
* Questi compiti sono a lungo termine e mostrano tipicamente un indicatore di progresso (ad es., download di file, importazione di dati). Hanno una priorità inferiore rispetto ai compiti iniziati dall'utente e non devono finire immediatamente.
4. **In Background:**
* Questa classe è per i compiti che operano in background e non sono visibili all'utente. Questi possono essere compiti come indicizzazione, sincronizzazione o backup. Hanno la priorità più bassa e un impatto minimo sulle prestazioni del sistema.

Utilizzando le classi QoS, gli sviluppatori non devono gestire i numeri di priorità esatti, ma piuttosto concentrarsi sulla natura del compito, e il sistema ottimizza le risorse CPU di conseguenza.

Inoltre, ci sono diverse **politiche di pianificazione dei thread** che fluiscono per specificare un insieme di parametri di pianificazione che il pianificatore prenderà in considerazione. Questo può essere fatto utilizzando `thread_policy_[set/get]`. Questo potrebbe essere utile negli attacchi di condizione di gara.

## Abuso dei Processi in MacOS

MacOS, come qualsiasi altro sistema operativo, fornisce una varietà di metodi e meccanismi per **l'interazione, la comunicazione e la condivisione dei dati** tra i processi. Sebbene queste tecniche siano essenziali per il funzionamento efficiente del sistema, possono anche essere abusate da attori malevoli per **eseguire attività dannose**.

### Iniezione di Librerie

L'iniezione di librerie è una tecnica in cui un attaccante **costringe un processo a caricare una libreria malevola**. Una volta iniettata, la libreria viene eseguita nel contesto del processo target, fornendo all'attaccante le stesse autorizzazioni e accesso del processo.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Hooking di Funzioni

L'hooking di funzioni implica **intercettare le chiamate di funzione** o i messaggi all'interno di un codice software. Hookando le funzioni, un attaccante può **modificare il comportamento** di un processo, osservare dati sensibili o persino ottenere il controllo sul flusso di esecuzione.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Comunicazione tra Processi

La comunicazione tra processi (IPC) si riferisce a diversi metodi con cui processi separati **condividono e scambiano dati**. Sebbene l'IPC sia fondamentale per molte applicazioni legittime, può anche essere abusato per sovvertire l'isolamento dei processi, rivelare informazioni sensibili o eseguire azioni non autorizzate.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Iniezione di Applicazioni Electron

Le applicazioni Electron eseguite con variabili ambientali specifiche potrebbero essere vulnerabili all'iniezione di processi:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Iniezione di Chromium

È possibile utilizzare i flag `--load-extension` e `--use-fake-ui-for-media-stream` per eseguire un **attacco man in the browser** che consente di rubare sequenze di tasti, traffico, cookie, iniettare script nelle pagine...:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Dirty NIB

I file NIB **definiscono gli elementi dell'interfaccia utente (UI)** e le loro interazioni all'interno di un'applicazione. Tuttavia, possono **eseguire comandi arbitrari** e **Gatekeeper non impedisce** a un'applicazione già eseguita di essere eseguita se un **file NIB è modificato**. Pertanto, potrebbero essere utilizzati per far eseguire programmi arbitrari che eseguono comandi arbitrari:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Iniezione di Applicazioni Java

È possibile abusare di alcune capacità di Java (come la variabile ambientale **`_JAVA_OPTS`**) per far eseguire a un'applicazione Java **codice/comandi arbitrari**.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### Iniezione di Applicazioni .Net

È possibile iniettare codice nelle applicazioni .Net **abusando della funzionalità di debug di .Net** (non protetta dalle protezioni di macOS come l'irrobustimento a runtime).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Iniezione di Perl

Controlla diverse opzioni per far eseguire a uno script Perl codice arbitrario in:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Iniezione di Ruby

È anche possibile abusare delle variabili ambientali di Ruby per far eseguire script arbitrari codice arbitrario:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Iniezione di Python

Se la variabile ambientale **`PYTHONINSPECT`** è impostata, il processo python entrerà in una CLI python una volta terminato. È anche possibile utilizzare **`PYTHONSTARTUP`** per indicare uno script python da eseguire all'inizio di una sessione interattiva.\
Tuttavia, nota che lo script **`PYTHONSTARTUP`** non verrà eseguito quando **`PYTHONINSPECT`** crea la sessione interattiva.

Altre variabili ambientali come **`PYTHONPATH`** e **`PYTHONHOME`** potrebbero essere utili per far eseguire a un comando python codice arbitrario.

Nota che gli eseguibili compilati con **`pyinstaller`** non utilizzeranno queste variabili ambientali anche se vengono eseguiti utilizzando un python incorporato.

{% hint style="danger" %}
In generale, non sono riuscito a trovare un modo per far eseguire a python codice arbitrario abusando delle variabili ambientali.\
Tuttavia, la maggior parte delle persone installa python utilizzando **Homebrew**, che installerà python in una **posizione scrivibile** per l'utente admin predefinito. Puoi dirottarlo con qualcosa come:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Anche **root** eseguirà questo codice quando esegue python.  
{% endhint %}

## Rilevamento

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) è un'applicazione open source che può **rilevare e bloccare le azioni di iniezione di processo**:

* Utilizzando **Variabili Ambientali**: Monitorerà la presenza di una delle seguenti variabili ambientali: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** e **`ELECTRON_RUN_AS_NODE`**
* Utilizzando le chiamate **`task_for_pid`**: Per scoprire quando un processo vuole ottenere il **task port di un altro** che consente di iniettare codice nel processo.
* **Parametri delle app Electron**: Qualcuno può utilizzare l'argomento della riga di comando **`--inspect`**, **`--inspect-brk`** e **`--remote-debugging-port`** per avviare un'app Electron in modalità di debug, e quindi iniettare codice in essa.
* Utilizzando **symlink** o **hardlink**: Tipicamente l'abuso più comune è **posizionare un link con i nostri privilegi utente**, e **puntarlo a una posizione con privilegi superiori**. Il rilevamento è molto semplice sia per hardlink che per symlink. Se il processo che crea il link ha un **livello di privilegio diverso** rispetto al file di destinazione, creiamo un **alert**. Sfortunatamente, nel caso dei symlink, il blocco non è possibile, poiché non abbiamo informazioni sulla destinazione del link prima della creazione. Questa è una limitazione del framework EndpointSecurity di Apple.

### Chiamate effettuate da altri processi

In [**questo post del blog**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) puoi trovare come è possibile utilizzare la funzione **`task_name_for_pid`** per ottenere informazioni su altri **processi che iniettano codice in un processo** e quindi ottenere informazioni su quell'altro processo.

Nota che per chiamare quella funzione devi avere lo **stesso uid** di quello che esegue il processo o essere **root** (e restituisce informazioni sul processo, non un modo per iniettare codice).

## Riferimenti

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

{% hint style="success" %}
Impara e pratica AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos su github.

</details>
{% endhint %}
