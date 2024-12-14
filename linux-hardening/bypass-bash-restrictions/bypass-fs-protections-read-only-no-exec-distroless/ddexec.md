# DDexec / EverythingExec

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

## Context

U Linux-u, da biste pokrenuli program, on mora postojati kao datoteka, mora biti dostupan na neki način kroz hijerarhiju datotečnog sistema (to je jednostavno kako `execve()` funkcioniše). Ova datoteka može biti na disku ili u RAM-u (tmpfs, memfd), ali vam je potreban put do datoteke. To je olakšalo kontrolu onoga što se pokreće na Linux sistemu, olakšava otkrivanje pretnji i alata napadača ili sprečavanje njih da pokušaju da izvrše bilo šta svoje (_npr._ ne dozvoljavajući korisnicima bez privilegija da postavljaju izvršne datoteke bilo gde).

Ali ova tehnika je ovde da promeni sve to. Ako ne možete da pokrenete proces koji želite... **onda preuzimate već postojeći**.

Ova tehnika vam omogućava da **zaobiđete uobičajene zaštitne tehnike kao što su samo za čitanje, noexec, bela lista imena datoteka, bela lista hešova...**

## Dependencies

Završni skript zavisi od sledećih alata da bi radio, oni moraju biti dostupni u sistemu koji napadate (po defaultu ćete ih pronaći svuda):
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## Tehnika

Ako ste u mogućnosti da proizvoljno modifikujete memoriju procesa, onda možete preuzeti kontrolu nad njim. Ovo se može koristiti za preuzimanje već postojećeg procesa i zamenu sa drugim programom. To možemo postići ili korišćenjem `ptrace()` sistemskog poziva (što zahteva da imate mogućnost izvršavanja sistemskih poziva ili da imate gdb dostupan na sistemu) ili, što je zanimljivije, pisanjem u `/proc/$pid/mem`.

Datoteka `/proc/$pid/mem` je jedan-na-jedan mapiranje celog adresnog prostora procesa (_npr._ od `0x0000000000000000` do `0x7ffffffffffff000` u x86-64). To znači da je čitanje ili pisanje u ovu datoteku na offsetu `x` isto kao čitanje ili modifikovanje sadržaja na virtuelnoj adresi `x`.

Sada imamo četiri osnovna problema sa kojima se suočavamo:

* Uopšte, samo root i vlasnik programa datoteke mogu da je modifikuju.
* ASLR.
* Ako pokušamo da čitamo ili pišemo na adresu koja nije mapirana u adresnom prostoru programa, dobićemo I/O grešku.

Ovi problemi imaju rešenja koja, iako nisu savršena, su dobra:

* Većina shell interpretera omogućava kreiranje deskriptora datoteka koji će zatim biti nasledni od strane podprocesa. Možemo kreirati fd koji pokazuje na `mem` datoteku shelle sa dozvolama za pisanje... tako da podprocesi koji koriste taj fd će moći da modifikuju memoriju shelle.
* ASLR čak nije ni problem, možemo proveriti `maps` datoteku shelle ili bilo koju drugu iz procfs kako bismo dobili informacije o adresnom prostoru procesa.
* Tako da treba da `lseek()` preko datoteke. Iz shelle to ne može biti urađeno osim korišćenjem infamoznog `dd`.

### Detaljnije

Koraci su relativno laki i ne zahtevaju nikakvu vrstu stručnosti da ih razumete:

* Parsirajte binarni fajl koji želite da pokrenete i loader da biste saznali koja mapiranja su potrebna. Zatim kreirajte "shell" kod koji će, u širokom smislu, izvršiti iste korake koje kernel preduzima pri svakom pozivu `execve()`:
* Kreirajte navedena mapiranja.
* Učitajte binarne fajlove u njih.
* Postavite dozvole.
* Na kraju, inicijalizujte stek sa argumentima za program i postavite pomoćni vektor (potreban loaderu).
* Skočite u loader i pustite ga da uradi ostalo (učita biblioteke potrebne programu).
* Dobijte iz `syscall` datoteke adresu na koju će se proces vratiti nakon izvršavanja sistemskog poziva.
* Prepišite to mesto, koje će biti izvršivo, našim shell kodom (kroz `mem` možemo modifikovati nepisive stranice).
* Prosledite program koji želite da pokrenete na stdin procesa (biće `read()` od strane pomenutog "shell" koda).
* U ovom trenutku, na loaderu je da učita potrebne biblioteke za naš program i skoči u njega.

**Pogledajte alat na** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

Postoji nekoliko alternativa za `dd`, od kojih je jedna, `tail`, trenutno podrazumevani program koji se koristi za `lseek()` kroz `mem` datoteku (što je bio jedini cilj korišćenja `dd`). Te alternative su:
```bash
tail
hexdump
cmp
xxd
```
Podešavanjem promenljive `SEEKER` možete promeniti korišćenog tražioca, _npr._:
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Ako pronađete još jednog validnog tražioca koji nije implementiran u skripti, još uvek ga možete koristiti postavljanjem promenljive `SEEKER_ARGS`:
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Blockirajte ovo, EDR-ovi.

## Reference
* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

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
