# macOS Launch/Environment Constraints & Trust Cache

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

## Basic Information

Ograničenja pokretanja u macOS-u su uvedena kako bi se poboljšala sigurnost **regulisanjem kako, ko i odakle se proces može pokrenuti**. Uvedena u macOS Ventura, pružaju okvir koji kategorizuje **svaki sistemski binarni fajl u različite kategorije ograničenja**, koje su definisane unutar **trust cache**, liste koja sadrži sistemske binarne fajlove i njihove odgovarajuće hash-eve. Ova ograničenja se protežu na svaki izvršni binarni fajl unutar sistema, podrazumevajući skup **pravila** koja definišu zahteve za **pokretanje određenog binarnog fajla**. Pravila obuhvataju samoprocenjivanja koja binarni fajl mora zadovoljiti, roditeljska ograničenja koja moraju biti ispunjena od strane njegovog roditeljskog procesa, i odgovorna ograničenja koja moraju poštovati druge relevantne entitete.

Mehanizam se proteže na aplikacije trećih strana putem **Environment Constraints**, počevši od macOS Sonoma, omogućavajući programerima da zaštite svoje aplikacije tako što će odrediti **skup ključeva i vrednosti za ograničenja okruženja.**

Definišete **ograničenja okruženja i biblioteka za pokretanje** u rečnicima ograničenja koje ili čuvate u **`launchd`** datotekama sa svojstvima, ili u **odvojenim datotekama sa svojstvima** koje koristite u potpisivanju koda.

Postoje 4 tipa ograničenja:

* **Samoprocenjivanja**: Ograničenja primenjena na **izvršni** binarni fajl.
* **Roditeljski proces**: Ograničenja primenjena na **roditelja procesa** (na primer **`launchd`** koji pokreće XP servis)
* **Odgovorna ograničenja**: Ograničenja primenjena na **proces koji poziva servis** u XPC komunikaciji
* **Ograničenja učitavanja biblioteka**: Koristite ograničenja učitavanja biblioteka da selektivno opišete kod koji može biti učitan

Dakle, kada proces pokuša da pokrene drugi proces — pozivajući `execve(_:_:_:)` ili `posix_spawn(_:_:_:_:_:_:)` — operativni sistem proverava da li **izvršni** fajl **zadovoljava** svoje **samoograničenje**. Takođe proverava da li **izvršni** fajl **roditeljskog** **procesa** **zadovoljava** **roditeljsko ograničenje** izvršnog fajla, i da li **izvršni** fajl **odgovornog** **procesa** **zadovoljava** **odgovorno ograničenje** izvršnog fajla. Ako bilo koje od ovih ograničenja pokretanja nije ispunjeno, operativni sistem ne pokreće program.

Ako prilikom učitavanja biblioteke bilo koji deo **ograničenja biblioteke nije tačan**, vaš proces **ne učitava** biblioteku.

## LC Categories

LC se sastoji od **činjenica** i **logičkih operacija** (i, ili..) koje kombinuju činjenice.

[**Činjenice koje LC može koristiti su dokumentovane**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints). Na primer:

* is-init-proc: Boolean vrednost koja označava da li izvršni fajl mora biti proces inicijalizacije operativnog sistema (`launchd`).
* is-sip-protected: Boolean vrednost koja označava da li izvršni fajl mora biti fajl zaštićen Sistemskom integritetnom zaštitom (SIP).
* `on-authorized-authapfs-volume:` Boolean vrednost koja označava da li je operativni sistem učitao izvršni fajl sa autorizovanog, autentifikovanog APFS volumena.
* `on-authorized-authapfs-volume`: Boolean vrednost koja označava da li je operativni sistem učitao izvršni fajl sa autorizovanog, autentifikovanog APFS volumena.
* Cryptexes volumen
* `on-system-volume:` Boolean vrednost koja označava da li je operativni sistem učitao izvršni fajl sa trenutno pokrenutog sistemskog volumena.
* Unutar /System...
* ...

Kada je Apple binarni fajl potpisan, **dodeljuje ga LC kategoriji** unutar **trust cache**.

* **iOS 16 LC kategorije** su [**obrnute i dokumentovane ovde**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056).
* Trenutne **LC kategorije (macOS 14** - Somona) su obrnute i njihove [**opisne informacije se mogu naći ovde**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53).

Na primer, Kategorija 1 je:
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: Mora biti u System ili Cryptexes volumenu.
* `launch-type == 1`: Mora biti sistemska usluga (plist u LaunchDaemons).
* `validation-category == 1`: Izvršna datoteka operativnog sistema.
* `is-init-proc`: Launchd

### Obrtanje LC Kategorija

Imate više informacija [**ovde**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints), ali u suštini, one su definisane u **AMFI (AppleMobileFileIntegrity)**, tako da treba da preuzmete Kernel Development Kit da biste dobili **KEXT**. Simboli koji počinju sa **`kConstraintCategory`** su **zanimljivi**. Ekstrakcijom ćete dobiti DER (ASN.1) kodirani tok koji ćete morati da dekodirate pomoću [ASN.1 Decoder](https://holtstrom.com/michael/tools/asn1decoder.php) ili python-asn1 biblioteke i njenog `dump.py` skripta, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master) koji će vam dati razumljiviju string.

## Ograničenja Okruženja

Ovo su Launch Ograničenja postavljena u **aplikacijama trećih strana**. Razvijač može odabrati **činjenice** i **logičke operande koje će koristiti** u svojoj aplikaciji da bi ograničio pristup samom sebi.

Moguće je enumerisati Ograničenja Okruženja aplikacije sa:
```bash
codesign -d -vvvv app.app
```
## Trust Caches

U **macOS** postoji nekoliko kešova poverenja:

* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
* **`/System/Library/Security/OSLaunchPolicyData`**

A u iOS-u izgleda da se nalazi u **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`**.

{% hint style="warning" %}
Na macOS-u koji radi na Apple Silicon uređajima, ako Apple potpisani binarni fajl nije u kešu poverenja, AMFI će odbiti da ga učita.
{% endhint %}

### Enumerating Trust Caches

Prethodni fajlovi keša poverenja su u formatu **IMG4** i **IM4P**, pri čemu je IM4P deo sa payload-om formata IMG4.

Možete koristiti [**pyimg4**](https://github.com/m1stadev/PyIMG4) za ekstrakciju payload-a iz baza podataka:

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
{% endcode %}

(Druga opcija bi mogla biti korišćenje alata [**img4tool**](https://github.com/tihmstar/img4tool), koji će raditi čak i na M1, čak i ako je verzija stara, i za x86\_64 ako ga instalirate na odgovarajućim mestima).

Sada možete koristiti alat [**trustcache**](https://github.com/CRKatri/trustcache) da dobijete informacije u čitljivom formatu:
```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```
Keš poverenja prati sledeću strukturu, tako da je **LC kategorija 4. kolona**
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
Then, you could use a script such as [**this one**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) to extract data.

From that data you can check the Apps with a **launch constraints value of `0`**, which are the ones that aren't constrained ([**check here**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) for what each value is).

## Attack Mitigations

Launch Constrains would have mitigated several old attacks by **making sure that the process won't be executed in unexpected conditions:** For example from unexpected locations or being invoked by an unexpected parent process (if only launchd should be launching it)

Moreover, Launch Constraints also **mitigates downgrade attacks.**

However, they **don't mitigate common XPC** abuses, **Electron** code injections or **dylib injections** without library validation (unless the team IDs that can load libraries are known).

### XPC Daemon Protection

In the Sonoma release, a notable point is the daemon XPC service's **responsibility configuration**. The XPC service is accountable for itself, as opposed to the connecting client being responsible. This is documented in the feedback report FB13206884. This setup might seem flawed, as it allows certain interactions with the XPC service:

- **Launching the XPC Service**: If assumed to be a bug, this setup does not permit initiating the XPC service through attacker code.
- **Connecting to an Active Service**: If the XPC service is already running (possibly activated by its original application), there are no barriers to connecting to it.

While implementing constraints on the XPC service might be beneficial by **narrowing the window for potential attacks**, it doesn't address the primary concern. Ensuring the security of the XPC service fundamentally requires **validating the connecting client effectively**. This remains the sole method to fortify the service's security. Also, it's worth noting that the mentioned responsibility configuration is currently operational, which might not align with the intended design.

### Electron Protection

Even if it's required that the application has to be **opened by LaunchService** (in the parents constraints). This can be achieved using **`open`** (which can set env variables) or using the **Launch Services API** (where env variables can be indicated).

## References

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

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
