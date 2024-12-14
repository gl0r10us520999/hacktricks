# CGroups

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Basiese Inligting

**Linux Beheer Groepe**, of **cgroups**, is 'n kenmerk van die Linux-kern wat die toewysing, beperking en prioritisering van stelselhulpbronne soos CPU, geheue en skyf I/O onder prosesgroepe toelaat. Hulle bied 'n mekanisme vir **die bestuur en isolering van hulpbronverbruik** van prosesversamelings, wat voordelig is vir doeleindes soos hulpbronbeperking, werkladingisolering en hulpbronprioritisering onder verskillende prosesgroepe.

Daar is **twee weergawes van cgroups**: weergawe 1 en weergawe 2. Albei kan gelyktydig op 'n stelsel gebruik word. Die primêre onderskeid is dat **cgroups weergawe 2** 'n **hiërargiese, boomagtige struktuur** bekendstel, wat meer genuanseerde en gedetailleerde hulpbronverdeling onder prosesgroepe moontlik maak. Boonop bring weergawe 2 verskeie verbeterings, insluitend:

Benewens die nuwe hiërargiese organisasie, het cgroups weergawe 2 ook **verskeie ander veranderinge en verbeterings** bekendgestel, soos ondersteuning vir **nuwe hulpbronbeheerders**, beter ondersteuning vir ouer toepassings, en verbeterde prestasie.

Algeheel bied cgroups **weergawe 2 meer funksies en beter prestasie** as weergawe 1, maar laasgenoemde kan steeds in sekere scenario's gebruik word waar kompatibiliteit met ouer stelsels 'n bekommernis is.

Jy kan die v1 en v2 cgroups vir enige proses lys deur na sy cgroup-lêer in /proc/\<pid> te kyk. Jy kan begin deur na jou skulp se cgroups te kyk met hierdie opdrag:
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
The output structure is as follows:

* **Nommer 2–12**: cgroups v1, met elke lyn wat 'n ander cgroup verteenwoordig. Beheerders vir hierdie is langs die nommer gespesifiseer.
* **Nommer 1**: Ook cgroups v1, maar slegs vir bestuursdoeleindes (gestel deur, bv., systemd), en het nie 'n beheerder nie.
* **Nommer 0**: Verteenwoordig cgroups v2. Geen beheerders word gelys nie, en hierdie lyn is eksklusief op stelsels wat slegs cgroups v2 draai.
* Die **name is hiërargies**, wat soos lêerpaaie lyk, wat die struktuur en verhouding tussen verskillende cgroups aandui.
* **Name soos /user.slice of /system.slice** spesifiseer die kategorisering van cgroups, met user.slice tipies vir aanmeldsessies wat deur systemd bestuur word en system.slice vir stelseldienste.

### Viewing cgroups

Die lêerstelsel word tipies gebruik om toegang te verkry tot **cgroups**, wat afwyk van die Unix-sisteemoproepinterface wat tradisioneel vir kerninteraksies gebruik word. Om 'n shell se cgroup-konfigurasie te ondersoek, moet 'n mens die **/proc/self/cgroup** lêer nagaan, wat die shell se cgroup onthul. Dan, deur na die **/sys/fs/cgroup** (of **`/sys/fs/cgroup/unified`**) gids te navigeer en 'n gids te vind wat die cgroup se naam deel, kan 'n mens verskeie instellings en hulpbronverbruikinligting wat relevant is vir die cgroup, waarneem.

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

Die sleutelinterfaiselêers vir cgroups is met **cgroup** voorafgegaan. Die **cgroup.procs** lêer, wat met standaardopdragte soos cat gesien kan word, lys die prosesse binne die cgroup. 'n Ander lêer, **cgroup.threads**, sluit draad-inligting in.

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Cgroups wat shells bestuur, sluit tipies twee beheerders in wat geheuegebruik en prosesgetal reguleer. Om met 'n beheerder te kommunikeer, moet lêers met die beheerder se voorvoegsel geraadpleeg word. Byvoorbeeld, **pids.current** sou geraadpleeg word om die aantal drade in die cgroup te bepaal.

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

Die aanduiding van **max** in 'n waarde dui op die afwesigheid van 'n spesifieke limiet vir die cgroup. Dit is egter moontlik dat, weens die hiërargiese aard van cgroups, limiete opgelê kan word deur 'n cgroup op 'n laer vlak in die gids hiërargie.

### Manipulating and Creating cgroups

Prosesse word aan cgroups toegeken deur **hulle Proses ID (PID) na die `cgroup.procs` lêer te skryf**. Dit vereis wortelprivileges. Byvoorbeeld, om 'n proses by te voeg:
```bash
echo [pid] > cgroup.procs
```
Net so, **om cgroup-attribuutte te wysig, soos om 'n PID-limiet in te stel**, word dit gedoen deur die verlangde waarde na die relevante lêer te skryf. Om 'n maksimum van 3,000 PIDs vir 'n cgroup in te stel:
```bash
echo 3000 > pids.max
```
**Die skep van nuwe cgroups** behels die maak van 'n nuwe subgids binne die cgroup hiërargie, wat die kern aanmoedig om outomaties die nodige koppelvlaklêers te genereer. Alhoewel cgroups sonder aktiewe prosesse met `rmdir` verwyder kan word, wees bewus van sekere beperkings:

* **Prosesse kan slegs in blaar cgroups geplaas word** (d.w.s., die mees geneste in 'n hiërargie).
* **'n cgroup kan nie 'n kontroleerder hê wat in sy ouer ontbreek nie**.
* **Kontroleerders vir kind cgroups moet eksplisiet verklaar word** in die `cgroup.subtree_control` lêer. Byvoorbeeld, om CPU en PID kontroleerders in 'n kind cgroup in te skakel:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
Die **root cgroup** is 'n uitsondering op hierdie reëls, wat direkte prosesplasing toelaat. Dit kan gebruik word om prosesse uit systemd bestuur te verwyder.

**Monitering van CPU-gebruik** binne 'n cgroup is moontlik deur die `cpu.stat` lêer, wat die totale CPU-tyd wat verbruik is, vertoon, nuttig vir die opsporing van gebruik oor 'n diens se subprocesses:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>CPU-gebruik statistieke soos getoon in die cpu.stat lêer</p></figcaption></figure>

## Verwysings

* **Boek: Hoe Linux Werk, 3de Uitgawe: Wat Elke Supergebruiker Moet Weet Deur Brian Ward**

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
