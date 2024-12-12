# CGroups

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Osnovne informacije

**Linux Control Groups**, ili **cgroups**, su funkcija Linux jezgra koja omogućava dodelu, ograničavanje i prioritizaciju resursa sistema poput CPU-a, memorije i disk I/O među grupama procesa. Pružaju mehanizam za **upravljanje i izolaciju korišćenja resursa** kolekcija procesa, korisnih za svrhe poput ograničavanja resursa, izolacije radnih opterećenja i prioritizacije resursa među različitim grupama procesa.

Postoje **dve verzije cgroups-a**: verzija 1 i verzija 2. Obe mogu istovremeno biti korišćene na sistemu. Osnovna razlika je što **cgroups verzija 2** uvodi **hijerarhijsku, stablo-sličnu strukturu**, omogućavajući detaljniju distribuciju resursa među grupama procesa. Pored toga, verzija 2 donosi različita poboljšanja, uključujući:

Pored nove hijerarhijske organizacije, cgroups verzija 2 takođe je uvela **nekoliko drugih promena i poboljšanja**, kao što su podrška za **nove kontrolere resursa**, bolja podrška za legacy aplikacije i poboljšana performansa.

Ukupno, cgroups **verzija 2 nudi više funkcija i bolju performansu** od verzije 1, ali ova poslednja se i dalje može koristiti u određenim scenarijima gde je kompatibilnost sa starijim sistemima od značaja.

Možete videti v1 i v2 cgroups za bilo koji proces gledanjem njegovog cgroup fajla u /proc/\<pid>. Možete početi tako što ćete pogledati cgroups vaše ljuske ovom komandom:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
### Pregledanje cgroups

Struktura izlaza je kako sledi:

* **Brojevi 2–12**: cgroups v1, pri čemu svaka linija predstavlja drugačiji cgroup. Kontroleri za ove su navedeni pored broja.
* **Broj 1**: Takođe cgroups v1, ali isključivo u svrhe upravljanja (postavljen od strane, npr., systemd-a), i nedostaje kontroler.
* **Broj 0**: Predstavlja cgroups v2. Kontroleri nisu navedeni, i ova linija je ekskluzivna za sisteme koji koriste samo cgroups v2.
* **Imena su hijerarhijska**, slična putanjama datoteka, ukazujući na strukturu i odnos između različitih cgroups.
* **Imena poput /user.slice ili /system.slice** specificiraju kategorizaciju cgroups, pri čemu je user.slice obično za sesije prijavljivanja koje upravlja systemd, a system.slice za sistemski servis.

Slika: ![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

Ključne datoteke interfejsa za cgroups imaju prefiks **cgroup**. Datoteka **cgroup.procs**, koja se može pregledati standardnim komandama poput cat, nabraja procese unutar cgroup-a. Druga datoteka, **cgroup.threads**, uključuje informacije o nitima.

Slika: ![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Cgroups koji upravljaju ljuskama obično obuhvataju dva kontrolera koji regulišu upotrebu memorije i broj procesa. Za interakciju sa kontrolerom, treba se konsultovati datoteke koje nose prefiks kontrolera. Na primer, **pids.current** bi se koristio da bi se utvrdio broj niti u cgroup-u.

Slika: ![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

Indikacija **max** u vrednosti sugeriše odsustvo specifičnog ograničenja za cgroup. Međutim, zbog hijerarhijske prirode cgroups-a, ograničenja mogu biti nametnuta od strane cgroup-a na nižem nivou u hijerarhiji direktorijuma.

### Manipulacija i Kreiranje cgroups-a

Procesi se dodeljuju cgroups-ima tako što se **upisuje njihov ID procesa (PID) u datoteku `cgroup.procs`**. Ovo zahteva privilegije root-a. Na primer, da dodate proces:
```bash
echo [pid] > cgroup.procs
```
Slično tome, **izmena cgroup atributa, poput postavljanja PID ograničenja**, vrši se upisivanjem željene vrednosti u odgovarajući fajl. Da biste postavili maksimum od 3.000 PID-ova za cgroup:
```bash
echo 3000 > pids.max
```
**Kreiranje novih cgroups** uključuje pravljenje novog poddirektorijuma unutar hijerarhije cgroup-a, što podstiče jezgro da automatski generiše neophodne interfejs fajlove. Iako se cgroups bez aktivnih procesa mogu ukloniti pomoću `rmdir`, budite svesni određenih ograničenja:

* **Procesi mogu biti smešteni samo u listne cgroups** (tj. najugnježdenije u hijerarhiji).
* **Cgroup ne može imati kontroler koji nedostaje u svom roditelju**.
* **Kontroleri za pod-cgroups moraju biti eksplicitno deklarisani** u fajlu `cgroup.subtree_control`. Na primer, da biste omogućili kontrolere CPU i PID u pod-cgroup-u:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
**Root cgroup** je izuzetak od ovih pravila, omogućavajući direktno postavljanje procesa. Ovo se može koristiti za uklanjanje procesa iz systemd upravljanja.

**Pratiti upotrebu CPU-a** unutar cgroup-a je moguće putem datoteke `cpu.stat`, koja prikazuje ukupno vreme CPU-a koje je potrošeno, korisno za praćenje upotrebe preko podprocesa servisa:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>Statistika upotrebe CPU-a prikazana u datoteci cpu.stat</p></figcaption></figure>

## Reference

* **Knjiga: Kako Linux funkcioniše, 3. izdanje: Šta svaki superkorisnik treba da zna, autor Brian Ward**
