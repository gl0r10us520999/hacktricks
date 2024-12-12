<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


# मूल जानकारी

**Seccomp** या सुरक्षित कंप्यूटिंग मोड, संक्षेप में, एक लिनक्स कर्नेल की विशेषता है जो **सिस्टम कॉल फिल्टर** के रूप में कार्य कर सकती है।\
Seccomp में 2 मोड होते हैं।

**seccomp** (सुरक्षित कंप्यूटिंग मोड के लिए संक्षिप्त) एक कंप्यूटर सुरक्षा सुविधा है **लिनक्स** **कर्नेल** में। seccomp एक प्रक्रिया को "सुरक्षित" स्थिति में एकतरफा संक्रमण करने की अनुमति देता है जहां **यह केवल निम्नलिखित सिस्टम कॉल कर सकता है** `exit()`, `sigreturn()`, `read()` और `write()` **पहले से खुले** फाइल डिस्क्रिप्टर्स के लिए। यदि यह कोई अन्य सिस्टम कॉल करने का प्रयास करता है, तो **कर्नेल** **प्रक्रिया** को SIGKILL या SIGSYS के साथ **समाप्त** कर देगा। इस अर्थ में, यह सिस्टम के संसाधनों को वर्चुअलाइज़ नहीं करता है लेकिन प्रक्रिया को उनसे पूरी तरह से अलग कर देता है।

seccomp मोड को `prctl(2)` सिस्टम कॉल का उपयोग करके `PR_SET_SECCOMP` तर्क के साथ **सक्षम किया जाता है**, या (लिनक्स कर्नेल 3.17 के बाद से) `seccomp(2)` सिस्टम कॉल के माध्यम से। seccomp मोड को पहले `/proc/self/seccomp` फाइल में लिखकर सक्षम किया जाता था, लेकिन इस विधि को `prctl()` के पक्ष में हटा दिया गया। कुछ कर्नेल संस्करणों में, seccomp `RDTSC` x86 निर्देश को अक्षम कर देता है, जो चालू होने के बाद से बीते प्रोसेसर चक्रों की संख्या लौटाता है, जिसका उपयोग उच्च-परिशुद्धता टाइमिंग के लिए किया जाता है।

**seccomp-bpf** seccomp का एक विस्तार है जो **सिस्टम कॉल की फिल्टरिंग को एक कॉन्फ़िगरेबल पॉलिसी का उपयोग करके अनुमति देता है** जिसे Berkeley Packet Filter नियमों का उपयोग करके लागू किया जाता है। इसका उपयोग OpenSSH और vsftpd के साथ-साथ Google Chrome/Chromium वेब ब्राउज़रों द्वारा Chrome OS और लिनक्स पर किया जाता है। (इस संबंध में seccomp-bpf पुराने systrace के समान कार्यक्षमता प्राप्त करता है, लेकिन अधिक लचीलापन और उच्च प्रदर्शन के साथ, जो लिनक्स के लिए अब समर्थित नहीं लगता है।)

## **मूल/सख्त मोड**

इस मोड में **Seccomp** केवल सिस्टम कॉल्स `exit()`, `sigreturn()`, `read()` और `write()` को पहले से खुले फाइल डिस्क्रिप्टर्स के लिए अनुमति देता है। यदि कोई अन्य सिस्टम कॉल किया जाता है, तो प्रक्रिया को SIGKILL का उपयोग करके मार दिया जाता है।

{% code title="seccomp_strict.c" %}
```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>

//From https://sysdig.com/blog/selinux-seccomp-falco-technical-discussion/
//gcc seccomp_strict.c -o seccomp_strict

int main(int argc, char **argv)
{
int output = open("output.txt", O_WRONLY);
const char *val = "test";

//enables strict seccomp mode
printf("Calling prctl() to set seccomp strict mode...\n");
prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);

//This is allowed as the file was already opened
printf("Writing to an already open file...\n");
write(output, val, strlen(val)+1);

//This isn't allowed
printf("Trying to open file for reading...\n");
int input = open("output.txt", O_RDONLY);

printf("You will not see this message--the process will be killed first\n");
}
```
## Seccomp-bpf

यह मोड **सिस्टम कॉल्स को एक कॉन्फ़िगरेबल पॉलिसी का उपयोग करके फ़िल्टर करने की अनुमति देता है** जिसे Berkeley Packet Filter नियमों का उपयोग करके लागू किया जाता है।

{% code title="seccomp_bpf.c" %}
```c
#include <seccomp.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

//https://security.stackexchange.com/questions/168452/how-is-sandboxing-implemented/175373
//gcc seccomp_bpf.c -o seccomp_bpf -lseccomp

void main(void) {
/* initialize the libseccomp context */
scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);

/* allow exiting */
printf("Adding rule : Allow exit_group\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

/* allow getting the current pid */
//printf("Adding rule : Allow getpid\n");
//seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(getpid), 0);

printf("Adding rule : Deny getpid\n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(getpid), 0);
/* allow changing data segment size, as required by glibc */
printf("Adding rule : Allow brk\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(brk), 0);

/* allow writing up to 512 bytes to fd 1 */
printf("Adding rule : Allow write upto 512 bytes to FD 1\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 2,
SCMP_A0(SCMP_CMP_EQ, 1),
SCMP_A2(SCMP_CMP_LE, 512));

/* if writing to any other fd, return -EBADF */
printf("Adding rule : Deny write to any FD except 1 \n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(write), 1,
SCMP_A0(SCMP_CMP_NE, 1));

/* load and enforce the filters */
printf("Load rules and enforce \n");
seccomp_load(ctx);
seccomp_release(ctx);
//Get the getpid is denied, a weird number will be returned like
//this process is -9
printf("this process is %d\n", getpid());
}
```
{% endcode %}

# Docker में Seccomp

**Seccomp-bpf** का समर्थन **Docker** द्वारा किया जाता है ताकि कंटेनरों से **syscalls** को प्रभावी ढंग से सीमित किया जा सके, जिससे सतह क्षेत्र में कमी आती है। आप **डिफ़ॉल्ट रूप से अवरुद्ध syscalls** को यहां पा सकते हैं [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/) और **डिफ़ॉल्ट seccomp प्रोफ़ाइल** यहां पाई जा सकती है [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json).\
आप एक docker कंटेनर को **विभिन्न seccomp** नीति के साथ चला सकते हैं:
```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```
यदि आप उदाहरण के लिए किसी **container** को कुछ **syscall** जैसे `uname` का उपयोग करने से **रोकना** चाहते हैं, तो आप [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) से डिफ़ॉल्ट प्रोफ़ाइल डाउनलोड कर सकते हैं और बस **सूची से `uname` स्ट्रिंग को हटा दें**।\
यदि आप यह सुनिश्चित करना चाहते हैं कि **कोई बाइनरी डॉकर कंटेनर के अंदर काम न करे**, तो आप strace का उपयोग करके बाइनरी द्वारा उपयोग किए जा रहे syscalls की सूची बना सकते हैं और फिर उन्हें रोक सकते हैं।\
निम्नलिखित उदाहरण में `uname` के **syscalls** का पता लगाया गया है:
```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```
{% hint style="info" %}
यदि आप **Docker का उपयोग केवल एक एप्लिकेशन लॉन्च करने के लिए कर रहे हैं**, तो आप इसे **`strace`** के साथ **प्रोफाइल** कर सकते हैं और **केवल उन syscalls को अनुमति दे सकते हैं** जिनकी इसे आवश्यकता है
{% endhint %}

## Docker में इसे निष्क्रिय करें

इस फ्लैग के साथ एक कंटेनर लॉन्च करें: **`--security-opt seccomp=unconfined`**


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करना चाहते हैं**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
