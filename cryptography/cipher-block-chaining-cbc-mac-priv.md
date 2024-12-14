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


# CBC

Se il **cookie** è **solo** il **nome utente** (o la prima parte del cookie è il nome utente) e vuoi impersonare il nome utente "**admin**". Allora, puoi creare il nome utente **"bdmin"** e **bruteforce** il **primo byte** del cookie.

# CBC-MAC

**Cipher block chaining message authentication code** (**CBC-MAC**) è un metodo utilizzato nella crittografia. Funziona prendendo un messaggio e crittografandolo blocco per blocco, dove la crittografia di ogni blocco è collegata a quella precedente. Questo processo crea una **catena di blocchi**, assicurando che cambiare anche un singolo bit del messaggio originale porterà a un cambiamento imprevedibile nell'ultimo blocco di dati crittografati. Per effettuare o invertire tale cambiamento, è necessaria la chiave di crittografia, garantendo la sicurezza.

Per calcolare il CBC-MAC del messaggio m, si crittografa m in modalità CBC con un vettore di inizializzazione zero e si conserva l'ultimo blocco. La figura seguente schizza il calcolo del CBC-MAC di un messaggio composto da blocchi![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) utilizzando una chiave segreta k e un cifrario a blocchi E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerabilità

Con CBC-MAC di solito il **IV utilizzato è 0**.\
Questo è un problema perché 2 messaggi noti (`m1` e `m2`) genereranno indipendentemente 2 firme (`s1` e `s2`). Quindi:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Poi un messaggio composto da m1 e m2 concatenati (m3) genererà 2 firme (s31 e s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Il che è possibile calcolare senza conoscere la chiave della crittografia.**

Immagina di crittografare il nome **Administrator** in blocchi di **8byte**:

* `Administ`
* `rator\00\00\00`

Puoi creare un nome utente chiamato **Administ** (m1) e recuperare la firma (s1).\
Poi, puoi creare un nome utente chiamato il risultato di `rator\00\00\00 XOR s1`. Questo genererà `E(m2 XOR s1 XOR 0)` che è s32.\
Ora, puoi usare s32 come la firma del nome completo **Administrator**.

### Riepilogo

1. Ottieni la firma del nome utente **Administ** (m1) che è s1
2. Ottieni la firma del nome utente **rator\x00\x00\x00 XOR s1 XOR 0** che è s32**.**
3. Imposta il cookie su s32 e sarà un cookie valido per l'utente **Administrator**.

# Attacco Controllando IV

Se puoi controllare l'IV utilizzato, l'attacco potrebbe essere molto facile.\
Se i cookie sono solo il nome utente crittografato, per impersonare l'utente "**administrator**" puoi creare l'utente "**Administrator**" e otterrai il suo cookie.\
Ora, se puoi controllare l'IV, puoi cambiare il primo byte dell'IV in modo che **IV\[0] XOR "A" == IV'\[0] XOR "a"** e rigenerare il cookie per l'utente **Administrator.** Questo cookie sarà valido per **impersonare** l'utente **administrator** con l'**IV** iniziale.

## Riferimenti

Maggiore informazione in [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
