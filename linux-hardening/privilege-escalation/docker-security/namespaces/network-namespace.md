# Mrežni Namespac

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podrška HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Osnovne Informacije

Mrežni namespace je funkcija Linux jezgra koja obezbeđuje izolaciju mrežnog steka, omogućavajući **svakom mrežnom namespace-u da ima svoju nezavisnu mrežnu konfiguraciju**, interfejse, IP adrese, tabele rutiranja i pravila vatrozida. Ova izolacija je korisna u raznim scenarijima, kao što je kontejnerizacija, gde svaki kontejner treba da ima svoju mrežnu konfiguraciju, nezavisno od drugih kontejnera i host sistema.

### Kako to funkcioniše:

1. Kada se kreira novi mrežni namespace, počinje sa **potpuno izolovanim mrežnim stekom**, sa **nema mrežnih interfejsa** osim za loopback interfejs (lo). To znači da procesi koji se izvršavaju u novom mrežnom namespace-u ne mogu komunicirati sa procesima u drugim namespace-ima ili host sistemu po defaultu.
2. **Virtuelni mrežni interfejsi**, kao što su veth parovi, mogu se kreirati i premestiti između mrežnih namespace-a. To omogućava uspostavljanje mrežne povezanosti između namespace-a ili između namespace-a i host sistema. Na primer, jedan kraj veth para može biti postavljen u mrežni namespace kontejnera, a drugi kraj može biti povezan sa **mostom** ili drugim mrežnim interfejsom u host namespace-u, obezbeđujući mrežnu povezanost kontejneru.
3. Mrežni interfejsi unutar namespace-a mogu imati svoje **vlastite IP adrese, tabele rutiranja i pravila vatrozida**, nezavisno od drugih namespace-a. To omogućava procesima u različitim mrežnim namespace-ima da imaju različite mrežne konfiguracije i funkcionišu kao da se izvršavaju na odvojenim umreženim sistemima.
4. Procesi mogu prelaziti između namespace-a koristeći `setns()` sistemski poziv, ili kreirati nove namespace-e koristeći `unshare()` ili `clone()` sistemske pozive sa `CLONE_NEWNET` zastavicom. Kada proces pređe u novi namespace ili ga kreira, počeće da koristi mrežnu konfiguraciju i interfejse povezane sa tim namespace-om.

## Laboratorija:

### Kreirajte različite Namespace-e

#### CLI
```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```
Montiranjem nove instance `/proc` datotečnog sistema ako koristite parametar `--mount-proc`, osiguravate da nova mount namespace ima **tačan i izolovan prikaz informacija o procesima specifičnim za tu namespace**.

<details>

<summary>Greška: bash: fork: Ne može da alocira memoriju</summary>

Kada se `unshare` izvrši bez `-f` opcije, dolazi do greške zbog načina na koji Linux obrađuje nove PID (ID procesa) namespace. Ključni detalji i rešenje su navedeni u nastavku:

1. **Objašnjenje problema**:
- Linux kernel omogućava procesu da kreira nove namespace koristeći `unshare` sistemski poziv. Međutim, proces koji inicira kreiranje novog PID namespace (poznat kao "unshare" proces) ne ulazi u novi namespace; samo njegovi podprocesi to čine.
- Pokretanjem `%unshare -p /bin/bash%` pokreće se `/bin/bash` u istom procesu kao `unshare`. Kao rezultat, `/bin/bash` i njegovi podprocesi su u originalnom PID namespace.
- Prvi podproces `/bin/bash` u novom namespace postaje PID 1. Kada ovaj proces izađe, pokreće čišćenje namespace-a ako nema drugih procesa, jer PID 1 ima posebnu ulogu usvajanja siročadi procesa. Linux kernel će tada onemogućiti alokaciju PID-a u tom namespace-u.

2. **Posledica**:
- Izlazak PID 1 u novom namespace-u dovodi do čišćenja `PIDNS_HASH_ADDING` oznake. To rezultira neuspehom funkcije `alloc_pid` da alocira novi PID prilikom kreiranja novog procesa, što proizvodi grešku "Ne može da alocira memoriju".

3. **Rešenje**:
- Problem se može rešiti korišćenjem `-f` opcije sa `unshare`. Ova opcija čini da `unshare` fork-uje novi proces nakon kreiranja novog PID namespace-a.
- Izvršavanje `%unshare -fp /bin/bash%` osigurava da `unshare` komanda sama postane PID 1 u novom namespace-u. `/bin/bash` i njegovi podprocesi su tada sigurno sadržani unutar ovog novog namespace-a, sprečavajući prevremeni izlazak PID 1 i omogućavajući normalnu alokaciju PID-a.

Osiguravanjem da `unshare` radi sa `-f` oznakom, novi PID namespace se ispravno održava, omogućavajući `/bin/bash` i njegove podprocese da funkcionišu bez susretanja greške u alokaciji memorije.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
# Run ifconfig or ip -a
```
### &#x20;Proverite u kojem je namespace vaš proces
```bash
ls -l /proc/self/ns/net
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/net -> 'net:[4026531840]'
```
### Pronađite sve mrežne imenske prostore

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name net -exec readlink {} \; 2>/dev/null | sort -u | grep "net:"
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name net -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Uđite unutar mrežnog prostora imena
```bash
nsenter -n TARGET_PID --pid /bin/bash
```
Takođe, možete **ući u drugi procesni prostor imena samo ako ste root**. I **ne možete** **ući** u drugo ime prostora **bez deskriptora** koji na njega pokazuje (kao što je `/proc/self/ns/net`).

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

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
