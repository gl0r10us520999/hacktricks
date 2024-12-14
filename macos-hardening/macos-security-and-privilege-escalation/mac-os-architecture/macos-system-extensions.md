# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

A differenza delle Kernel Extensions, **le System Extensions vengono eseguite nello spazio utente** anziché nello spazio del kernel, riducendo il rischio di un crash di sistema a causa di un malfunzionamento dell'estensione.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Ci sono tre tipi di system extensions: **DriverKit** Extensions, **Network** Extensions e **Endpoint Security** Extensions.

### **DriverKit Extensions**

DriverKit è un sostituto delle kernel extensions che **forniscono supporto hardware**. Consente ai driver di dispositivo (come USB, Serial, NIC e HID) di essere eseguiti nello spazio utente anziché nello spazio del kernel. Il framework DriverKit include **versioni nello spazio utente di alcune classi dell'I/O Kit**, e il kernel inoltra gli eventi normali dell'I/O Kit allo spazio utente, offrendo un ambiente più sicuro per l'esecuzione di questi driver.

### **Network Extensions**

Le Network Extensions forniscono la possibilità di personalizzare i comportamenti di rete. Ci sono diversi tipi di Network Extensions:

* **App Proxy**: Questo viene utilizzato per creare un client VPN che implementa un protocollo VPN personalizzato orientato al flusso. Ciò significa che gestisce il traffico di rete in base alle connessioni (o flussi) piuttosto che ai singoli pacchetti.
* **Packet Tunnel**: Questo viene utilizzato per creare un client VPN che implementa un protocollo VPN personalizzato orientato ai pacchetti. Ciò significa che gestisce il traffico di rete in base ai singoli pacchetti.
* **Filter Data**: Questo viene utilizzato per filtrare i "flussi" di rete. Può monitorare o modificare i dati di rete a livello di flusso.
* **Filter Packet**: Questo viene utilizzato per filtrare i singoli pacchetti di rete. Può monitorare o modificare i dati di rete a livello di pacchetto.
* **DNS Proxy**: Questo viene utilizzato per creare un provider DNS personalizzato. Può essere utilizzato per monitorare o modificare le richieste e le risposte DNS.

## Endpoint Security Framework

L'Endpoint Security è un framework fornito da Apple in macOS che offre un insieme di API per la sicurezza del sistema. È destinato all'uso da parte di **fornitori di sicurezza e sviluppatori per costruire prodotti che possono monitorare e controllare l'attività del sistema** per identificare e proteggere contro attività dannose.

Questo framework fornisce una **collezione di API per monitorare e controllare l'attività del sistema**, come esecuzioni di processi, eventi del file system, eventi di rete e del kernel.

Il nucleo di questo framework è implementato nel kernel, come una Kernel Extension (KEXT) situata in **`/System/Library/Extensions/EndpointSecurity.kext`**. Questo KEXT è composto da diversi componenti chiave:

* **EndpointSecurityDriver**: Questo funge da "punto di ingresso" per l'estensione del kernel. È il principale punto di interazione tra il sistema operativo e il framework di Endpoint Security.
* **EndpointSecurityEventManager**: Questo componente è responsabile dell'implementazione dei ganci del kernel. I ganci del kernel consentono al framework di monitorare gli eventi di sistema intercettando le chiamate di sistema.
* **EndpointSecurityClientManager**: Questo gestisce la comunicazione con i client nello spazio utente, tenendo traccia di quali client sono connessi e devono ricevere notifiche di eventi.
* **EndpointSecurityMessageManager**: Questo invia messaggi e notifiche di eventi ai client nello spazio utente.

Gli eventi che il framework di Endpoint Security può monitorare sono categorizzati in:

* Eventi di file
* Eventi di processo
* Eventi di socket
* Eventi del kernel (come il caricamento/scaricamento di un'estensione del kernel o l'apertura di un dispositivo I/O Kit)

### Architettura del Framework di Endpoint Security

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**La comunicazione nello spazio utente** con il framework di Endpoint Security avviene attraverso la classe IOUserClient. Vengono utilizzate due diverse sottoclassi, a seconda del tipo di chiamante:

* **EndpointSecurityDriverClient**: Questo richiede il diritto `com.apple.private.endpoint-security.manager`, che è detenuto solo dal processo di sistema `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Questo richiede il diritto `com.apple.developer.endpoint-security.client`. Questo verrebbe tipicamente utilizzato da software di sicurezza di terze parti che deve interagire con il framework di Endpoint Security.

Le Endpoint Security Extensions:**`libEndpointSecurity.dylib`** è la libreria C che le system extensions utilizzano per comunicare con il kernel. Questa libreria utilizza l'I/O Kit (`IOKit`) per comunicare con il KEXT di Endpoint Security.

**`endpointsecurityd`** è un demone di sistema chiave coinvolto nella gestione e nel lancio delle system extensions di sicurezza degli endpoint, in particolare durante il processo di avvio iniziale. **Solo le system extensions** contrassegnate con **`NSEndpointSecurityEarlyBoot`** nel loro file `Info.plist` ricevono questo trattamento di avvio anticipato.

Un altro demone di sistema, **`sysextd`**, **valida le system extensions** e le sposta nelle posizioni di sistema appropriate. Quindi chiede al demone pertinente di caricare l'estensione. Il **`SystemExtensions.framework`** è responsabile per l'attivazione e la disattivazione delle system extensions.

## Bypassare l'ESF

L'ESF è utilizzato da strumenti di sicurezza che cercheranno di rilevare un red teamer, quindi qualsiasi informazione su come questo potrebbe essere evitato suona interessante.

### CVE-2021-30965

Il fatto è che l'applicazione di sicurezza deve avere **permessi di accesso completo al disco**. Quindi, se un attaccante potesse rimuovere ciò, potrebbe impedire l'esecuzione del software:
```bash
tccutil reset All
```
Per **maggiori informazioni** su questo bypass e quelli correlati, controlla il talk [#OBTS v5.0: "Il tallone d'Achille di EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Alla fine, questo è stato risolto dando il nuovo permesso **`kTCCServiceEndpointSecurityClient`** all'app di sicurezza gestita da **`tccd`** in modo che `tccutil` non cancelli i suoi permessi impedendole di funzionare.

## Riferimenti

* [**OBTS v3.0: "Sicurezza e Insicurezza degli Endpoint" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Impara e pratica Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos su github.

</details>
{% endhint %}
