# Sensitive Mounts

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

L'esposizione di `/proc` e `/sys` senza una corretta isolamento dei namespace introduce significativi rischi per la sicurezza, inclusi l'ampliamento della superficie di attacco e la divulgazione di informazioni. Queste directory contengono file sensibili che, se mal configurati o accessibili da un utente non autorizzato, possono portare a fuga dal container, modifica dell'host o fornire informazioni che facilitano ulteriori attacchi. Ad esempio, montare in modo errato `-v /proc:/host/proc` può eludere la protezione di AppArmor a causa della sua natura basata su percorso, lasciando `/host/proc` non protetto.

**Puoi trovare ulteriori dettagli su ciascuna potenziale vulnerabilità in** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## procfs Vulnerabilities

### `/proc/sys`

Questa directory consente l'accesso per modificare le variabili del kernel, di solito tramite `sysctl(2)`, e contiene diverse sottodirectory di interesse:

#### **`/proc/sys/kernel/core_pattern`**

* Descritto in [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Consente di definire un programma da eseguire alla generazione di un file di core con i primi 128 byte come argomenti. Questo può portare all'esecuzione di codice se il file inizia con una pipe `|`.
*   **Esempio di test e sfruttamento**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Test di accesso in scrittura
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Imposta gestore personalizzato
sleep 5 && ./crash & # Attiva gestore
```

#### **`/proc/sys/kernel/modprobe`**

* Dettagliato in [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Contiene il percorso per il caricatore di moduli del kernel, invocato per caricare i moduli del kernel.
*   **Esempio di controllo accesso**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Controlla accesso a modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Riferito in [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Un flag globale che controlla se il kernel va in panico o invoca l'oom killer quando si verifica una condizione OOM.

#### **`/proc/sys/fs`**

* Secondo [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), contiene opzioni e informazioni sul file system.
* L'accesso in scrittura può abilitare vari attacchi di denial-of-service contro l'host.

#### **`/proc/sys/fs/binfmt_misc`**

* Consente di registrare interpreti per formati binari non nativi basati sul loro numero magico.
* Può portare a escalation dei privilegi o accesso a shell root se `/proc/sys/fs/binfmt_misc/register` è scrivibile.
* Sfruttamento e spiegazione rilevanti:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Tutorial approfondito: [Video link](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Altri in `/proc`

#### **`/proc/config.gz`**

* Può rivelare la configurazione del kernel se `CONFIG_IKCONFIG_PROC` è abilitato.
* Utile per gli attaccanti per identificare vulnerabilità nel kernel in esecuzione.

#### **`/proc/sysrq-trigger`**

* Consente di invocare comandi Sysrq, potenzialmente causando riavvii immediati del sistema o altre azioni critiche.
*   **Esempio di riavvio dell'host**:

```bash
echo b > /proc/sysrq-trigger # Riavvia l'host
```

#### **`/proc/kmsg`**

* Espone i messaggi del buffer di anello del kernel.
* Può aiutare negli exploit del kernel, perdite di indirizzi e fornire informazioni sensibili sul sistema.

#### **`/proc/kallsyms`**

* Elenca i simboli esportati dal kernel e i loro indirizzi.
* Essenziale per lo sviluppo di exploit del kernel, specialmente per superare KASLR.
* Le informazioni sugli indirizzi sono limitate con `kptr_restrict` impostato su `1` o `2`.
* Dettagli in [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Interfaccia con il dispositivo di memoria del kernel `/dev/mem`.
* Storicamente vulnerabile ad attacchi di escalation dei privilegi.
* Maggiori informazioni su [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Rappresenta la memoria fisica del sistema in formato ELF core.
* La lettura può rivelare contenuti di memoria del sistema host e di altri container.
* La grande dimensione del file può portare a problemi di lettura o crash del software.
* Uso dettagliato in [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Interfaccia alternativa per `/dev/kmem`, rappresenta la memoria virtuale del kernel.
* Consente lettura e scrittura, quindi modifica diretta della memoria del kernel.

#### **`/proc/mem`**

* Interfaccia alternativa per `/dev/mem`, rappresenta la memoria fisica.
* Consente lettura e scrittura, la modifica di tutta la memoria richiede la risoluzione degli indirizzi virtuali in fisici.

#### **`/proc/sched_debug`**

* Restituisce informazioni sulla pianificazione dei processi, eludendo le protezioni del namespace PID.
* Espone nomi di processi, ID e identificatori di cgroup.

#### **`/proc/[pid]/mountinfo`**

* Fornisce informazioni sui punti di montaggio nel namespace di montaggio del processo.
* Espone la posizione del `rootfs` o dell'immagine del container.

### `/sys` Vulnerabilities

#### **`/sys/kernel/uevent_helper`**

* Utilizzato per gestire i `uevents` dei dispositivi del kernel.
* Scrivere in `/sys/kernel/uevent_helper` può eseguire script arbitrari al verificarsi di `uevent`.
*   **Esempio di sfruttamento**: %%%bash

#### Crea un payload

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Trova il percorso host dal montaggio OverlayFS per il container

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Imposta uevent\_helper su helper malevolo

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Attiva un uevent

echo change > /sys/class/mem/null/uevent

#### Legge l'output

cat /output %%%

#### **`/sys/class/thermal`**

* Controlla le impostazioni di temperatura, potenzialmente causando attacchi DoS o danni fisici.

#### **`/sys/kernel/vmcoreinfo`**

* Rilascia indirizzi del kernel, compromettendo potenzialmente KASLR.

#### **`/sys/kernel/security`**

* Contiene l'interfaccia `securityfs`, che consente la configurazione dei moduli di sicurezza Linux come AppArmor.
* L'accesso potrebbe consentire a un container di disabilitare il proprio sistema MAC.

#### **`/sys/firmware/efi/vars` e `/sys/firmware/efi/efivars`**

* Espone interfacce per interagire con le variabili EFI in NVRAM.
* Una configurazione errata o sfruttamento può portare a laptop bloccati o macchine host non avviabili.

#### **`/sys/kernel/debug`**

* `debugfs` offre un'interfaccia di debug "senza regole" al kernel.
* Storia di problemi di sicurezza a causa della sua natura illimitata.

### References

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

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
