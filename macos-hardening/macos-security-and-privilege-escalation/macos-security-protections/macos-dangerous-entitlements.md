# macOS Dangerous Entitlements & TCC perms

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

{% hint style="warning" %}
Napomena: entitlements koji počinju sa **`com.apple`** nisu dostupni trećim stranama, samo Apple može da ih dodeli.
{% endhint %}

## High

### `com.apple.rootless.install.heritable`

Entitlement **`com.apple.rootless.install.heritable`** omogućava **obiđi SIP**. Proverite [ovo za više informacija](macos-sip.md#com.apple.rootless.install.heritable).

### **`com.apple.rootless.install`**

Entitlement **`com.apple.rootless.install`** omogućava **obiđi SIP**. Proverite [ovo za više informacija](macos-sip.md#com.apple.rootless.install).

### **`com.apple.system-task-ports` (ranije nazvan `task_for_pid-allow`)**

Ovaj entitlement omogućava dobijanje **task porta za bilo koji** proces, osim za kernel. Proverite [**ovo za više informacija**](../macos-proces-abuse/macos-ipc-inter-process-communication/).

### `com.apple.security.get-task-allow`

Ovaj entitlement omogućava drugim procesima sa **`com.apple.security.cs.debugger`** entitlement da dobiju task port procesa koji pokreće binarni fajl sa ovim entitlementom i **ubace kod u njega**. Proverite [**ovo za više informacija**](../macos-proces-abuse/macos-ipc-inter-process-communication/).

### `com.apple.security.cs.debugger`

Aplikacije sa Debugging Tool Entitlement mogu pozvati `task_for_pid()` da dobiju validan task port za nesignirane i treće strane aplikacije sa `Get Task Allow` entitlement postavljenim na `true`. Međutim, čak i sa entitlementom za debugging alat, debugger **ne može dobiti task portove** procesa koji **nemaju `Get Task Allow` entitlement**, i koji su stoga zaštićeni System Integrity Protection. Proverite [**ovo za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger).

### `com.apple.security.cs.disable-library-validation`

Ovaj entitlement omogućava **učitavanje frameworka, plug-inova ili biblioteka bez da budu potpisani od strane Apple-a ili potpisani sa istim Team ID** kao glavni izvršni fajl, tako da napadač može zloupotrebiti učitavanje neke proizvoljne biblioteke da ubaci kod. Proverite [**ovo za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation).

### `com.apple.private.security.clear-library-validation`

Ovaj entitlement je veoma sličan **`com.apple.security.cs.disable-library-validation`** ali **umesto** da **direktno onemogući** validaciju biblioteka, omogućava procesu da **pozove `csops` sistemski poziv da ga onemogući**.\
Proverite [**ovo za više informacija**](https://theevilbit.github.io/posts/com.apple.private.security.clear-library-validation/).

### `com.apple.security.cs.allow-dyld-environment-variables`

Ovaj entitlement omogućava **korišćenje DYLD promenljivih okruženja** koje se mogu koristiti za ubacivanje biblioteka i koda. Proverite [**ovo za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables).

### `com.apple.private.tcc.manager` ili `com.apple.rootless.storage`.`TCC`

[**Prema ovom blogu**](https://objective-see.org/blog/blog\_0x4C.html) **i** [**ovom blogu**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/), ovi entitlements omogućavaju **modifikaciju** **TCC** baze podataka.

### **`system.install.apple-software`** i **`system.install.apple-software.standar-user`**

Ovi entitlements omogućavaju **instalaciju softvera bez traženja dozvola** od korisnika, što može biti korisno za **povećanje privilegija**.

### `com.apple.private.security.kext-management`

Entitlement potreban za traženje od **kernela da učita kernel ekstenziju**.

### **`com.apple.private.icloud-account-access`**

Entitlement **`com.apple.private.icloud-account-access`** omogućava komunikaciju sa **`com.apple.iCloudHelper`** XPC servisom koji će **obezbediti iCloud tokene**.

**iMovie** i **Garageband** su imali ovaj entitlement.

Za više **informacija** o eksploatu za **dobijanje iCloud tokena** iz tog entitlementa proverite predavanje: [**#OBTS v5.0: "Šta se dešava na vašem Mac-u, ostaje na Apple-ovom iCloud-u?!" - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### `com.apple.private.tcc.manager.check-by-audit-token`

TODO: Ne znam šta ovo omogućava

### `com.apple.private.apfs.revert-to-snapshot`

TODO: U [**ovom izveštaju**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **se pominje da bi ovo moglo biti korišćeno za** ažuriranje SSV-zaštićenog sadržaja nakon ponovnog pokretanja. Ako znate kako, pošaljite PR, molim vas!

### `com.apple.private.apfs.create-sealed-snapshot`

TODO: U [**ovom izveštaju**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **se pominje da bi ovo moglo biti korišćeno za** ažuriranje SSV-zaštićenog sadržaja nakon ponovnog pokretanja. Ako znate kako, pošaljite PR, molim vas!

### `keychain-access-groups`

Ovaj entitlement lista **keychain** grupa kojima aplikacija ima pristup:
```xml
<key>keychain-access-groups</key>
<array>
<string>ichat</string>
<string>apple</string>
<string>appleaccount</string>
<string>InternetAccounts</string>
<string>IMCore</string>
</array>
```
### **`kTCCServiceSystemPolicyAllFiles`**

Daje **potpuni pristup disku** dozvole, jedna od najviših TCC dozvola koje možete imati.

### **`kTCCServiceAppleEvents`**

Omogućava aplikaciji da šalje događaje drugim aplikacijama koje se obično koriste za **automatizaciju zadataka**. Kontrolisanjem drugih aplikacija, može zloupotrebiti dozvole koje su dodeljene tim drugim aplikacijama.

Kao što je da ih natera da traže od korisnika njegovu lozinku:

{% code overflow="wrap" %}
```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```
{% endcode %}

Ili ih naterati da izvrše **arbitrarne radnje**.

### **`kTCCServiceEndpointSecurityClient`**

Omogućava, između ostalog, da **piše u TCC bazu podataka korisnika**.

### **`kTCCServiceSystemPolicySysAdminFiles`**

Omogućava da **promeni** **`NFSHomeDirectory`** atribut korisnika koji menja putanju njegove početne fascikle i tako omogućava da **obiđe TCC**.

### **`kTCCServiceSystemPolicyAppBundles`**

Omogućava modifikaciju fajlova unutar aplikacija (unutar app.app), što je **podrazumevano zabranjeno**.

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Moguće je proveriti ko ima ovaj pristup u _System Settings_ > _Privacy & Security_ > _App Management._

### `kTCCServiceAccessibility`

Proces će moći da **zloupotrebi macOS pristupne funkcije**, što znači da će, na primer, moći da pritisne tastere. Tako bi mogao da zatraži pristup za kontrolu aplikacije kao što je Finder i odobri dijalog sa ovom dozvolom.

## Medium

### `com.apple.security.cs.allow-jit`

Ova dozvola omogućava da se **kreira memorija koja je zapisiva i izvršna** prosleđivanjem `MAP_JIT` oznake `mmap()` sistemskoj funkciji. Proverite [**ovo za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit).

### `com.apple.security.cs.allow-unsigned-executable-memory`

Ova dozvola omogućava da se **prepiše ili zakrpi C kod**, koristi dugo zastareli **`NSCreateObjectFileImageFromMemory`** (koji je fundamentalno nesiguran), ili koristi **DVDPlayback** okvir. Proverite [**ovo za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory).

{% hint style="danger" %}
Uključivanje ove dozvole izlaže vašu aplikaciju uobičajenim ranjivostima u jezicima koji nisu sigurni za memoriju. Pažljivo razmotrite da li vaša aplikacija treba ovu izuzetak.
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

Ova dozvola omogućava da se **modifikuju sekcije sopstvenih izvršnih fajlova** na disku kako bi se prisilno izašlo. Proverite [**ovo za više informacija**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection).

{% hint style="danger" %}
Dozvola za onemogućavanje zaštite izvršne memorije je ekstremna dozvola koja uklanja fundamentalnu sigurnosnu zaštitu iz vaše aplikacije, omogućavajući napadaču da prepiše izvršni kod vaše aplikacije bez otkrivanja. Preferirajte uže dozvole ako je moguće.
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

### `com.apple.private.nullfs_allow`

Ova dozvola omogućava montiranje nullfs fajl sistema (zabranjeno podrazumevano). Alat: [**mount\_nullfs**](https://github.com/JamaicanMoose/mount\_nullfs/tree/master).

### `kTCCServiceAll`

Prema ovom blog postu, ova TCC dozvola obično se nalazi u formi:
```
[Key] com.apple.private.tcc.allow-prompting
[Value]
[Array]
[String] kTCCServiceAll
```
Dozvolite procesu da **zatraži sve TCC dozvole**.

### **`kTCCServicePostEvent`**
{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
</details>
