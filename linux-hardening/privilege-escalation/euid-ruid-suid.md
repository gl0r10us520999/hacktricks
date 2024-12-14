# euid, ruid, suid

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

<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Deepen your expertise in **Mobile Security** with 8kSec Academy. Master iOS and Android security through our self-paced courses and get certified:

{% embed url="https://academy.8ksec.io/" %}


### User Identification Variables

- **`ruid`**: **stvarni ID korisnika** označava korisnika koji je pokrenuo proces.
- **`euid`**: Poznat kao **efektivni ID korisnika**, predstavlja identitet korisnika koji sistem koristi za utvrđivanje privilegija procesa. Generalno, `euid` odražava `ruid`, osim u slučajevima kao što je izvršavanje SetUID binarnog fajla, gde `euid` preuzima identitet vlasnika fajla, čime se dodeljuju specifične operativne dozvole.
- **`suid`**: Ovaj **sačuvani ID korisnika** je ključan kada proces sa visokim privilegijama (obično pokrenut kao root) treba privremeno da se odrekne svojih privilegija da bi izvršio određene zadatke, samo da bi kasnije povratio svoj prvobitni povišeni status.

#### Important Note
Proces koji ne radi pod root-om može samo da modifikuje svoj `euid` da odgovara trenutnom `ruid`, `euid` ili `suid`.

### Understanding set*uid Functions

- **`setuid`**: Suprotno prvobitnim pretpostavkama, `setuid` prvenstveno modifikuje `euid` umesto `ruid`. Konkretno, za privilegovane procese, usklađuje `ruid`, `euid` i `suid` sa određenim korisnikom, često root, efektivno učvršćujući ove ID-ove zbog nadjačavajućeg `suid`. Detaljne informacije mogu se naći na [setuid man stranici](https://man7.org/linux/man-pages/man2/setuid.2.html).
- **`setreuid`** i **`setresuid`**: Ove funkcije omogućavaju suptilno podešavanje `ruid`, `euid` i `suid`. Međutim, njihove mogućnosti zavise od nivoa privilegija procesa. Za procese koji nisu root, modifikacije su ograničene na trenutne vrednosti `ruid`, `euid` i `suid`. Nasuprot tome, root procesi ili oni sa `CAP_SETUID` privilegijom mogu dodeliti proizvoljne vrednosti ovim ID-ovima. Više informacija može se dobiti sa [setresuid man stranice](https://man7.org/linux/man-pages/man2/setresuid.2.html) i [setreuid man stranice](https://man7.org/linux/man-pages/man2/setreuid.2.html).

Ove funkcionalnosti nisu dizajnirane kao mehanizam bezbednosti, već da olakšaju predviđeni operativni tok, kao kada program preuzima identitet drugog korisnika menjajući svoj efektivni ID korisnika.

Važno je napomenuti da, iako `setuid` može biti uobičajen izbor za podizanje privilegija na root (pošto usklađuje sve ID-ove sa root), razlikovanje između ovih funkcija je ključno za razumevanje i manipulaciju ponašanjem ID-ova korisnika u različitim scenarijima.

### Program Execution Mechanisms in Linux

#### **`execve` System Call**
- **Functionality**: `execve` pokreće program, određen prvim argumentom. Prihvaća dva niz argumenta, `argv` za argumente i `envp` za okruženje.
- **Behavior**: Zadržava memorijski prostor pozivaoca, ali osvežava stek, heap i podatkovne segmente. Kod programa se zamenjuje novim programom.
- **User ID Preservation**:
- `ruid`, `euid` i dodatni grupni ID-ovi ostaju nepromenjeni.
- `euid` može imati suptilne promene ako novi program ima postavljen SetUID bit.
- `suid` se ažurira iz `euid` nakon izvršenja.
- **Documentation**: Detaljne informacije mogu se naći na [`execve` man stranici](https://man7.org/linux/man-pages/man2/execve.2.html).

#### **`system` Function**
- **Functionality**: Za razliku od `execve`, `system` kreira podproces koristeći `fork` i izvršava komandu unutar tog podprocesa koristeći `execl`.
- **Command Execution**: Izvršava komandu putem `sh` sa `execl("/bin/sh", "sh", "-c", command, (char *) NULL);`.
- **Behavior**: Pošto je `execl` oblik `execve`, funkcioniše slično, ali u kontekstu novog podprocesa.
- **Documentation**: Dalje uvide možete dobiti sa [`system` man stranice](https://man7.org/linux/man-pages/man3/system.3.html).

#### **Behavior of `bash` and `sh` with SUID**
- **`bash`**:
- Ima `-p` opciju koja utiče na to kako se tretiraju `euid` i `ruid`.
- Bez `-p`, `bash` postavlja `euid` na `ruid` ako se prvobitno razlikuju.
- Sa `-p`, prvobitni `euid` se čuva.
- Više detalja može se naći na [`bash` man stranici](https://linux.die.net/man/1/bash).
- **`sh`**:
- Ne poseduje mehanizam sličan `-p` u `bash`.
- Ponašanje u vezi sa ID-ovima korisnika nije eksplicitno pomenuto, osim pod `-i` opcijom, naglašavajući očuvanje jednakosti `euid` i `ruid`.
- Dodatne informacije su dostupne na [`sh` man stranici](https://man7.org/linux/man-pages/man1/sh.1p.html).

Ovi mehanizmi, različiti u svojoj operaciji, nude raznovrsne opcije za izvršavanje i prelazak između programa, sa specifičnim nijansama u načinu na koji se upravlja i čuva ID korisnika.

### Testing User ID Behaviors in Executions

Examples taken from https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail, check it for further information

#### Case 1: Using `setuid` with `system`

**Objective**: Razumevanje efekta `setuid` u kombinaciji sa `system` i `bash` kao `sh`.

**C Code**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
system("id");
return 0;
}
```
**Kompilacija i Dozvole:**
```bash
oxdf@hacky$ gcc a.c -o /mnt/nfsshare/a;
oxdf@hacky$ chmod 4755 /mnt/nfsshare/a
```

```bash
bash-4.2$ $ ./a
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Analiza:**

* `ruid` i `euid` počinju kao 99 (nobody) i 1000 (frank) respektivno.
* `setuid` usklađuje oba na 1000.
* `system` izvršava `/bin/bash -c id` zbog symlink-a sa sh na bash.
* `bash`, bez `-p`, prilagođava `euid` da odgovara `ruid`, što rezultira time da su oba 99 (nobody).

#### Slučaj 2: Korišćenje setreuid sa system

**C Kod**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setreuid(1000, 1000);
system("id");
return 0;
}
```
**Kompilacija i Dozvole:**
```bash
oxdf@hacky$ gcc b.c -o /mnt/nfsshare/b; chmod 4755 /mnt/nfsshare/b
```
**Izvršenje i Rezultat:**
```bash
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Анализа:**

* `setreuid` поставља и ruid и euid на 1000.
* `system` позива bash, који одржава корисничке ID-ове због њихове једнакости, ефективно функционишући као frank.

#### Случај 3: Користећи setuid са execve
Циљ: Истраживање интеракције између setuid и execve.
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/usr/bin/id", NULL, NULL);
return 0;
}
```
**Izvršenje i Rezultat:**
```bash
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Анализа:**

* `ruid` остаје 99, али је euid постављен на 1000, у складу са ефектом setuid-а.

**C Код Пример 2 (Позивање Bash):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/bin/bash", NULL, NULL);
return 0;
}
```
**Izvršenje i Rezultat:**
```bash
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Анализа:**

* Иако је `euid` постављен на 1000 помоћу `setuid`, `bash` ресетује euid на `ruid` (99) због одсуства `-p`.

**C Код Пример 3 (Користећи bash -p):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
setuid(1000);
execve(paramList[0], paramList, NULL);
return 0;
}
```
**Izvršenje i Rezultat:**
```bash
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=100
```
## References
* [https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail)


<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Produbite svoje znanje o **Mobilnoj Bezbednosti** sa 8kSec Akademijom. Savladajte iOS i Android bezbednost kroz naše kurseve koji se mogu pratiti sopstvenim tempom i dobijite sertifikat:

{% embed url="https://academy.8ksec.io/" %}


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
