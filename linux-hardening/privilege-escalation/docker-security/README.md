# Docker Sekuriteit

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Opleiding AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Opleiding GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Gebruik [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) om maklik te bou en **werkvloei te outomatiseer** wat deur die wêreld se **mees gevorderde** gemeenskapstoestelle aangedryf word.\
Kry Toegang Vandag:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **Basiese Docker Engine Sekuriteit**

Die **Docker engine** gebruik die Linux-kern se **Namespaces** en **Cgroups** om houers te isoleer, wat 'n basiese laag van sekuriteit bied. Addisionele beskerming word verskaf deur **Capabilities dropping**, **Seccomp**, en **SELinux/AppArmor**, wat houer-isolasie verbeter. 'n **auth plugin** kan verder gebruikersaksies beperk.

![Docker Sekuriteit](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Veilige Toegang tot Docker Engine

Die Docker engine kan plaaslik via 'n Unix-soket of op afstand met HTTP toegang verkry. Vir afstandstoegang is dit noodsaaklik om HTTPS en **TLS** te gebruik om vertroulikheid, integriteit en outentisering te verseker.

Die Docker engine luister standaard op die Unix-soket by `unix:///var/run/docker.sock`. Op Ubuntu-stelsels word Docker se opstartopsies in `/etc/default/docker` gedefinieer. Om afstandstoegang tot die Docker API en kliënt te aktiveer, stel die Docker-daemon bloot deur die volgende instellings by te voeg:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
However, exposing the Docker daemon over HTTP is not recommended due to security concerns. It's advisable to secure connections using HTTPS. There are two main approaches to securing the connection:

1. Die kliënt verifieer die bediener se identiteit.
2. Beide die kliënt en bediener verifieer mekaar se identiteit.

Certificates are utilized to confirm a server's identity. For detailed examples of both methods, refer to [**this guide**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### Security of Container Images

Container images can be stored in either private or public repositories. Docker offers several storage options for container images:

* [**Docker Hub**](https://hub.docker.com): 'n Publieke registrasiediens van Docker.
* [**Docker Registry**](https://github.com/docker/distribution): 'n Oopbronprojek wat gebruikers toelaat om hul eie registrasie te huisves.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Docker se kommersiële registrasie-aanbod, wat rolgebaseerde gebruikersverifikasie en integrasie met LDAP-gidsdienste bied.

### Image Scanning

Containers can have **security vulnerabilities** either because of the base image or because of the software installed on top of the base image. Docker is working on a project called **Nautilus** that does security scan of Containers and lists the vulnerabilities. Nautilus works by comparing the each Container image layer with vulnerability repository to identify security holes.

For more [**information read this**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

The **`docker scan`** command allows you to scan existing Docker images using the image name or ID. For example, run the following command to scan the hello-world image:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Docker Image Signing

Docker beeldondertekening verseker die sekuriteit en integriteit van beelde wat in houers gebruik word. Hier is 'n saamgeperste verduideliking:

* **Docker Content Trust** gebruik die Notary-projek, gebaseer op The Update Framework (TUF), om beeldondertekening te bestuur. Vir meer inligting, sien [Notary](https://github.com/docker/notary) en [TUF](https://theupdateframework.github.io).
* Om Docker inhoudsvertroue te aktiveer, stel `export DOCKER_CONTENT_TRUST=1` in. Hierdie funksie is standaard afgeskakel in Docker weergawe 1.10 en later.
* Met hierdie funksie geaktiveer, kan slegs ondertekende beelde afgelaai word. Die aanvanklike beeldduw vereis die instelling van wagwoorde vir die wortel- en etiketteringssleutels, met Docker wat ook Yubikey ondersteun vir verbeterde sekuriteit. Meer besonderhede kan [hier](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) gevind word.
* Pogings om 'n ongetekende beeld te trek met inhoudsvertroue geaktiveer, lei tot 'n "No trust data for latest" fout.
* Vir beeldduwings na die eerste, vra Docker vir die wagwoord van die repository-sleutel om die beeld te onderteken.

Om jou private sleutels te rugsteun, gebruik die opdrag:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Wanneer jy Docker-gashere verander, is dit nodig om die wortel- en repository-sleutels te skuif om bedrywighede te handhaaf.

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Gebruik [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) om maklik te bou en **werkvloei** te **automate** wat deur die wêreld se **mees gevorderde** gemeenskapstools aangedryf word.\
Kry Toegang Vandag:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## Houers Sekuriteitskenmerke

<details>

<summary>Opsomming van Houer Sekuriteitskenmerke</summary>

**Hoofproses Isolasiekenmerke**

In gecontaineriseerde omgewings is dit van kardinale belang om projekte en hul prosesse te isoleer vir sekuriteit en hulpbronbestuur. Hier is 'n vereenvoudigde verduideliking van sleutelkonsepte:

**Namespaces**

* **Doel**: Verseker isolasie van hulpbronne soos prosesse, netwerk en lêerstelsels. Veral in Docker, hou namespaces 'n houer se prosesse apart van die gasheer en ander houers.
* **Gebruik van `unshare`**: Die `unshare` opdrag (of die onderliggende syscall) word gebruik om nuwe namespaces te skep, wat 'n bykomende laag van isolasie bied. egter, terwyl Kubernetes dit nie inherent blokkeer nie, doen Docker dit.
* **Beperking**: Die skep van nuwe namespaces laat nie 'n proses toe om na die gasheer se standaard namespaces terug te keer nie. Om in die gasheer namespaces te dring, sal 'n mens tipies toegang tot die gasheer se `/proc` gids benodig, met `nsenter` vir toegang.

**Beheer Groepe (CGroups)**

* **Funksie**: Primêr gebruik vir die toewysing van hulpbronne onder prosesse.
* **Sekuriteitsaspek**: CGroups self bied nie isolasie sekuriteit nie, behalwe vir die `release_agent` kenmerk, wat, indien verkeerd geconfigureer, potensieel misbruik kan word vir ongeoorloofde toegang.

**Vermindering van Vermoëns**

* **Belangrikheid**: Dit is 'n noodsaaklike sekuriteitskenmerk vir prosesisolatie.
* **Funksionaliteit**: Dit beperk die aksies wat 'n wortelproses kan uitvoer deur sekere vermoëns te laat val. Selfs al loop 'n proses met wortelregte, keer die gebrek aan die nodige vermoëns dit om bevoorregte aksies uit te voer, aangesien die syscalls sal misluk weens onvoldoende toestemmings.

Dit is die **oorblywende vermoëns** nadat die proses die ander laat val het:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

Dit is standaard in Docker geaktiveer. Dit help om **nog meer die syscalls** wat die proses kan aanroep te beperk.\
Die **standaard Docker Seccomp-profiel** kan gevind word op [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

Docker het 'n sjabloon wat jy kan aktiveer: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Dit sal toelaat om vermoëns, syscalls, toegang tot lêers en vouers te verminder...

</details>

### Namespaces

**Namespaces** is 'n kenmerk van die Linux-kern wat **kernhulpbronne** **verdeel** sodat een stel **prosesse** **een stel** **hulpbronne** sien terwyl **'n ander** stel **prosesse** 'n **verskillende** stel hulpbronne sien. Die kenmerk werk deur die samelewing van die selfde namespace vir 'n stel hulpbronne en prosesse, maar daardie namespaces verwys na onderskeie hulpbronne. Hulpbronne kan in verskeie ruimtes bestaan.

Docker maak gebruik van die volgende Linux-kern Namespaces om Containere isolasie te bereik:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

Vir **meer inligting oor die namespaces** kyk na die volgende bladsy:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Die Linux-kern kenmerk **cgroups** bied die vermoë om **hulpbronne soos cpu, geheue, io, netwerkbandwydte onder** 'n stel prosesse te **beperk**. Docker laat toe om Containere te skep met behulp van die cgroup kenmerk wat hulpbronbeheer vir die spesifieke Container toelaat.\
Hieronder is 'n Container geskep met gebruikersruimte geheue beperk tot 500m, kern geheue beperk tot 50m, cpu-aandeel tot 512, blkioweight tot 400. CPU-aandeel is 'n verhouding wat die Container se CPU-gebruik beheer. Dit het 'n standaardwaarde van 1024 en 'n reeks tussen 0 en 1024. As drie Containere dieselfde CPU-aandeel van 1024 het, kan elke Container tot 33% van die CPU neem in die geval van CPU-hulpbronkompetisie. blkio-weight is 'n verhouding wat die Container se IO beheer. Dit het 'n standaardwaarde van 500 en 'n reeks tussen 10 en 1000.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Om die cgroup van 'n houer te kry, kan jy doen:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
For more information check:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Vermoëns

Vermoëns laat **finer control oor die vermoëns wat toegelaat kan word** vir die root gebruiker. Docker gebruik die Linux-kern vermoënskenmerk om **die operasies wat binne 'n Container gedoen kan word, te beperk** ongeag die tipe gebruiker.

Wanneer 'n docker-container gedraai word, **laat die proses sensitiewe vermoëns wat die proses kan gebruik om uit die isolasie te ontsnap, val**. Dit probeer verseker dat die proses nie sensitiewe aksies kan uitvoer en ontsnap nie:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp in Docker

Dit is 'n sekuriteitskenmerk wat Docker toelaat om **die syscalls** wat binne die container gebruik kan word, te beperk:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor in Docker

**AppArmor** is 'n kernverbetering om **containers** te beperk tot 'n **beperkte** stel **hulpbronne** met **per-program profiele**.:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux in Docker

* **Etiketstelsel**: SELinux ken 'n unieke etiket aan elke proses en lêersysteemobjek toe.
* **Beleidstoepassing**: Dit handhaaf sekuriteitsbeleide wat definieer watter aksies 'n proses etiket op ander etikette binne die stelsel kan uitvoer.
* **Container Proses Etikette**: Wanneer container enjinse container prosesse inisieer, word hulle gewoonlik 'n beperkte SELinux etiket, algemeen `container_t`, toegeken.
* **Lêer Etikettering binne Containers**: Lêers binne die container word gewoonlik as `container_file_t` geëtiketteer.
* **Beleid Reëls**: Die SELinux-beleid verseker hoofsaaklik dat prosesse met die `container_t` etiket slegs kan interaksie hê (lees, skryf, uitvoer) met lêers wat as `container_file_t` geëtiketteer is.

Hierdie meganisme verseker dat selfs al is 'n proses binne 'n container gecompromitteer, dit beperk is tot interaksie slegs met voorwerpe wat die ooreenstemmende etikette het, wat die potensiële skade van sulke kompromies aansienlik beperk.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

In Docker speel 'n magtiging-plug-in 'n belangrike rol in sekuriteit deur te besluit of versoeke aan die Docker-daemon toegelaat of geblokkeer moet word. Hierdie besluit word geneem deur twee sleutelkontexte te ondersoek:

* **Verifikasiekonteks**: Dit sluit omvattende inligting oor die gebruiker in, soos wie hulle is en hoe hulle hulself geverifieer het.
* **Opdragkonteks**: Dit bestaan uit alle relevante data rakende die versoek wat gemaak word.

Hierdie kontekste help verseker dat slegs wettige versoeke van geverifieerde gebruikers verwerk word, wat die sekuriteit van Docker-operasies verbeter.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## DoS vanaf 'n container

As jy nie behoorlik die hulpbronne wat 'n container kan gebruik, beperk nie, kan 'n gecompromitteerde container DoS die gasheer waar dit draai.

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* Bandwidth DoS
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## Interessante Docker Vlaggies

### --privileged vlag

In die volgende bladsy kan jy leer **wat die `--privileged` vlag impliseer**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

As jy 'n houer bestuur waar 'n aanvaller daarin slaag om toegang te verkry as 'n lae voorreg gebruiker. As jy 'n **verkeerd geconfigureerde suid binêre** het, kan die aanvaller dit misbruik en **voorregte binne** die houer verhoog. Dit kan hom in staat stel om daaruit te ontsnap.

Om die houer met die **`no-new-privileges`** opsie geaktiveer te bestuur, sal **hierdie soort voorregverhoging voorkom**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Ander
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
For more **`--security-opt`** options check: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Ander Sekuriteitsoorwegings

### Bestuur van Geheime: Beste Praktyke

Dit is van kardinale belang om te vermy om geheime direk in Docker-beelde in te sluit of om omgewing veranderlikes te gebruik, aangesien hierdie metodes jou sensitiewe inligting blootstel aan enigiemand met toegang tot die houer deur opdragte soos `docker inspect` of `exec`.

**Docker volumes** is 'n veiliger alternatief, wat aanbeveel word vir die toegang tot sensitiewe inligting. Hulle kan as 'n tydelike lêerstelsel in geheue gebruik word, wat die risiko's wat verband hou met `docker inspect` en logging verminder. egter, wortelgebruikers en diegene met `exec` toegang tot die houer mag steeds toegang tot die geheime hê.

**Docker geheime** bied 'n selfs veiliger metode vir die hantering van sensitiewe inligting. Vir voorbeelde wat geheime tydens die beeldbou fase benodig, bied **BuildKit** 'n doeltreffende oplossing met ondersteuning vir bou-tyd geheime, wat die bou spoed verbeter en addisionele funksies bied.

Om BuildKit te benut, kan dit op drie maniere geaktiveer word:

1. Deur 'n omgewing veranderlike: `export DOCKER_BUILDKIT=1`
2. Deur opdragte te prefix: `DOCKER_BUILDKIT=1 docker build .`
3. Deur dit standaard in die Docker-konfigurasie in te skakel: `{ "features": { "buildkit": true } }`, gevolg deur 'n Docker herstart.

BuildKit stel die gebruik van bou-tyd geheime met die `--secret` opsie moontlik, wat verseker dat hierdie geheime nie in die beeldbou kas of die finale beeld ingesluit word nie, met 'n opdrag soos:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Vir geheime wat nodig is in 'n lopende houer, **Docker Compose en Kubernetes** bied robuuste oplossings. Docker Compose gebruik 'n `secrets` sleutel in die diensdefinisie om geheime lêers te spesifiseer, soos getoon in 'n `docker-compose.yml` voorbeeld:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
This configuration allows for the use of secrets when starting services with Docker Compose.

In Kubernetes environments, secrets are natively supported and can be further managed with tools like [Helm-Secrets](https://github.com/futuresimple/helm-secrets). Kubernetes' Role Based Access Controls (RBAC) enhances secret management security, similar to Docker Enterprise.

### gVisor

**gVisor** is 'n toepassingskern, geskryf in Go, wat 'n substansiële gedeelte van die Linux-stelselsurface implementeer. Dit sluit 'n [Open Container Initiative (OCI)](https://www.opencontainers.org) runtime in genaamd `runsc` wat 'n **isolasiegrens tussen die toepassing en die gasheerkern** bied. Die `runsc` runtime integreer met Docker en Kubernetes, wat dit eenvoudig maak om sandboxed houers te laat loop.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** is 'n oopbron-gemeenskap wat werk om 'n veilige houer runtime te bou met liggewig virtuele masjiene wat soos houers voel en presteer, maar **sterker werklading-isolasie bied deur middel van hardewarevirtualisering** tegnologie as 'n tweede verdedigingslaag.

{% embed url="https://katacontainers.io/" %}

### Summary Tips

* **Moet nie die `--privileged` vlag gebruik of 'n** [**Docker-soket binne die houer**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)** monteer.** Die docker soket laat toe om houers te spawn, so dit is 'n maklike manier om volle beheer oor die gasheer te neem, byvoorbeeld deur 'n ander houer met die `--privileged` vlag te laat loop.
* Moet **nie as root binne die houer loop nie. Gebruik 'n** [**ander gebruiker**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) **en** [**gebruikersnamespaces**](https://docs.docker.com/engine/security/userns-remap/)**.** Die root in die houer is dieselfde as op die gasheer tensy dit met gebruikersnamespaces herverdeel word. Dit is slegs liggies beperk deur, hoofsaaklik, Linux namespaces, vermoëns, en cgroups.
* [**Laat alle vermoëns val**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) en stel slegs diegene wat benodig word in** (`--cap-add=...`). Baie werklading benodig nie enige vermoëns nie en om dit by te voeg verhoog die omvang van 'n potensiële aanval.
* [**Gebruik die “no-new-privileges” sekuriteitsopsie**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) om te voorkom dat prosesse meer voorregte verkry, byvoorbeeld deur suid-binaries.
* [**Beperk hulpbronne beskikbaar aan die houer**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Hulpbronlimiete kan die masjien beskerm teen ontkenning van diensaanvalle.
* **Pas** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(of SELinux)** profiele aan om die aksies en syscalls wat beskikbaar is vir die houer tot die minimum vereiste te beperk.
* **Gebruik** [**amptelike docker beelde**](https://docs.docker.com/docker-hub/official_images/) **en vereis handtekeninge** of bou jou eie gebaseer daarop. Moet nie [terugdeure](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) beelde erf of gebruik nie. Stoor ook root sleutels, wagwoord in 'n veilige plek. Docker het planne om sleutels met UCP te bestuur.
* **Herbou** jou beelde **gereeld** om **sekuriteitsopdaterings op die gasheer en beelde toe te pas.**
* Bestuur jou **secrets verstandig** sodat dit moeilik is vir die aanvaller om toegang daartoe te verkry.
* As jy **die docker daemon blootstel, gebruik HTTPS** met kliënt- en bedienerverifikasie.
* In jou Dockerfile, **gee voorkeur aan COPY eerder as ADD**. ADD ontleed outomaties gecomprimeerde lêers en kan lêers van URL's kopieer. COPY het nie hierdie vermoëns nie. Vermy ADD waar moontlik sodat jy nie kwesbaar is vir aanvalle deur middel van afgeleë URL's en Zip-lêers nie.
* Het **afsonderlike houers vir elke mikro-diens**
* **Moet nie ssh** binne die houer plaas nie, “docker exec” kan gebruik word om na die houer te ssh.
* Het **kleiner** houer **beelde**

## Docker Breakout / Privilege Escalation

As jy **binne 'n docker houer** is of jy het toegang tot 'n gebruiker in die **docker groep**, kan jy probeer om te **ontsnap en voorregte te verhoog**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Docker Authentication Plugin Bypass

As jy toegang het tot die docker soket of toegang het tot 'n gebruiker in die **docker groep maar jou aksies word beperk deur 'n docker auth plugin**, kyk of jy dit kan **omseil:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Hardening Docker

* Die hulpmiddel [**docker-bench-security**](https://github.com/docker/docker-bench-security) is 'n skrif wat vir dosyne algemene beste praktyke rondom die ontplooiing van Docker-houers in produksie nagaan. Die toetse is almal geoutomatiseer, en is gebaseer op die [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/).\
Jy moet die hulpmiddel vanaf die gasheer wat docker draai of vanaf 'n houer met genoeg voorregte uitvoer. Vind uit **hoe om dit in die README te loop:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## References

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/\_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux\_namespaces](https://en.wikipedia.org/wiki/Linux\_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins\_authorization](https://docs.docker.com/engine/extend/plugins\_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

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
