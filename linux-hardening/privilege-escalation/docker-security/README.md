# डॉकर सुरक्षा

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) का उपयोग करें और **आसानी से वर्कफ़्लो** बनाएं और सक्रिय करें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **मूल डॉकर इंजन सुरक्षा**

**डॉकर इंजन** ने लिनक्स कर्नेल के **नेमस्पेस** और **सीग्रुप्स** का उपयोग करके कंटेनरों को विभाजित करने के लिए एक मूल सुरक्षा स्तर प्रदान किया है। अतिरिक्त सुरक्षा **क्षमताएँ छोड़ने**, **सेकॉम्प**, और **SELinux/AppArmor** के माध्यम से प्रदान की जाती है, जो कंटेनर विभाजन को बढ़ाता है। एक **ऑथ प्लगइन** उपयोगकर्ता क्रियाओं को और अधिक प्रतिबंधित कर सकता है।

![डॉकर सुरक्षा](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### डॉकर इंजन तक सुरक्षित पहुंच

डॉकर इंजन को या तो स्थानीय रूप से यूनिक्स सॉकेट के माध्यम से या HTTP का उपयोग करके दूरस्थ से पहुंचा जा सकता है। दूरस्थ पहुंच के लिए, गोपनीयता, अखंडता, और प्रमाणीकरण सुनिश्चित करने के लिए HTTPS और **TLS** का उपयोग करना महत्वपूर्ण है।

डॉकर इंजन, डिफ़ॉल्ट रूप से, `unix:///var/run/docker.sock` पर यूनिक्स सॉकेट पर सुनता है। यूबंटू सिस्टम पर, डॉकर की स्टार्टअप विकल्प `/etc/default/docker` में परिभाषित होते हैं। डॉकर API और क्लाइंट के लिए दूरस्थ पहुंच सक्षम करने के लिए, निम्नलिखित सेटिंग्स जोड़कर डॉकर डेमन को एक HTTP सॉकेट पर उजागर करें:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
हालांकि, HTTP के माध्यम से Docker डेमन को अनुसारित करना सुरक्षा संबंधित चिंताओं के कारण सिफारिश नहीं है। HTTPS का उपयोग करके कनेक्शन को सुरक्षित बनाना उत्तम है। कनेक्शन को सुरक्षित बनाने के दो मुख्य दृष्टिकोण हैं:

1. क्लाइंट सर्वर की पहचान सत्यापित करता है।
2. क्लाइंट और सर्वर एक-दूसरे की पहचान सत्यापित करते हैं।

प्रमाणपत्रों का उपयोग सर्वर की पहचान सत्यापित करने के लिए किया जाता है। दोनों विधियों के विस्तृत उदाहरणों के लिए, [**इस गाइड**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/) का संदर्भ दें।

### कंटेनर इमेज की सुरक्षा

कंटेनर इमेज को निजी या सार्वजनिक रिपॉजिटरी में संग्रहीत किया जा सकता है। Docker कंटेनर इमेजों के लिए कई संग्रहण विकल्प प्रदान करता है:

* [**Docker Hub**](https://hub.docker.com): Docker से एक सार्वजनिक रजिस्ट्री सेवा।
* [**Docker Registry**](https://github.com/docker/distribution): एक ओपन-सोर्स परियोजना जो उपयोगकर्ताओं को अपनी रजिस्ट्री होस्ट करने की अनुमति देती है।
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Docker का व्यावसायिक रजिस्ट्री प्रस्ताव, जिसमें भूमिका-आधारित उपयोगकर्ता प्रमाणीकरण और LDAP निर्देशिका सेवाओं के साथ समावेश है।

### इमेज स्कैनिंग

कंटेनर में **सुरक्षा दोष** हो सकते हैं या तो आधार इमेज के कारण हो सकते हैं या आधार इमेज पर स्थापित सॉफ्टवेयर के कारण हो सकते हैं। Docker एक परियोजना पर काम कर रहा है जिसे **नौटिलस** कहा जाता है जो कंटेनरों की सुरक्षा स्कैन करता है और सुरक्षा खोजों को सूचीबद्ध करता है। नौटिलस यह काम करता है कि प्रत्येक कंटेनर इमेज लेयर को सुरक्षा दोष रिपॉजिटरी के साथ तुलना करके सुरक्षा खोजों को पहचानें।

अधिक [**जानकारी के लिए यह पढ़ें**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

**`docker scan`** कमांड आपको विदित Docker इमेजों को स्कैन करने की अनुमति देता है उपयोगकर्ता नाम या आईडी का उपयोग करके। उदाहरण के लिए, hello-world इमेज को स्कैन करने के लिए निम्नलिखित कमांड चलाएं:
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
### Docker छवि साइनिंग

Docker छवि साइनिंग छवियों में उपयोग की जाने वाली सुरक्षा और अखंडता सुनिश्चित करती है। यहाँ एक संक्षिप्त स्पष्टीकरण है:

* **Docker सामग्री विश्वसनीयता** नोटरी परियोजना का उपयोग करता है, जो The Update Framework (TUF) पर आधारित है, छवि साइनिंग का प्रबंधन करने के लिए। अधिक जानकारी के लिए, [Notary](https://github.com/docker/notary) और [TUF](https://theupdateframework.github.io) देखें।
* Docker सामग्री विश्वसनीयता को सक्रिय करने के लिए, `export DOCKER_CONTENT_TRUST=1` सेट करें। यह सुविधा Docker संस्करण 1.10 और उसके बाद के लिए डिफ़ॉल्ट रूप से बंद है।
* इस सुविधा को सक्षम करने के साथ, केवल साइन की गई छवियाँ डाउनलोड की जा सकती हैं। प्रारंभिक छवि पुश करने के लिए मूल और टैगिंग कुंजियों के लिए पासफ़्रेज सेट करने की आवश्यकता होती है, जिसके साथ Docker भी उन्नत सुरक्षा के लिए Yubikey का समर्थन करता है। अधिक विवरण [यहाँ](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) मिल सकते हैं।
* सामग्री विश्वसनीयता सक्षम करने के साथ असाइन की गई छवि को डाउनलोड करने का प्रयास "नो ट्रस्ट डेटा फॉर लेटेस्ट" त्रुटि देता है।
* पहले के बाद छवि पुश करने के लिए, Docker छवि को साइन करने के लिए रिपॉजिटरी कुंजी का पासफ़्रेज पूछता है।

अपनी निजी कुंजियों का बैकअप करने के लिए, निम्नलिखित कमांड का उपयोग करें:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Docker होस्ट बदलते समय, संचय और रिपॉजिटरी कुंजी को संचालन बनाए रखने के लिए आवश्यक है।

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) का उपयोग करें और आसानी से विश्व के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित **कार्यप्रवाह** निर्मित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## कंटेनर सुरक्षा सुविधाएं

<details>

<summary>कंटेनर सुरक्षा सुविधाओं का सारांश</summary>

**मुख्य प्रक्रिया विभाजन सुविधाएं**

कंटेनरीकृत वातावरणों में परियोजनाओं और उनकी प्रक्रियाओं को अलग करना सुरक्षा और संसाधन प्रबंधन के लिए महत्वपूर्ण है। यहां मुख्य अवधारणाओं का एक सरलीकृत व्याख्यान है:

**नेमस्पेस**

* **उद्देश्य**: प्रक्रियाओं, नेटवर्क, और फ़ाइल सिस्टम जैसे संसाधनों का अलगाव सुनिश्चित करना। विशेष रूप से Docker में, नेमस्पेस एक कंटेनर की प्रक्रियाओं को मेज़बान और अन्य कंटेनरों से अलग रखते हैं।
* **`unshare` का उपयोग**: नए नेमस्पेस बनाने के लिए `unshare` कमांड (या अंतर्निहित सिसकॉल) का उपयोग किया जाता है, जो अलगाव की एक अतिरिक्त परत प्रदान करता है। हालांकि, Kubernetes इसे निहायत रूप से नहीं ब्लॉक करता है, Docker ब्लॉक करता है।
* **सीमा**: नए नेमस्पेस बनाने से किसी प्रक्रिया को मेज़बान की डिफ़ॉल्ट नेमस्पेस पर वापस नहीं लौटने देता है। मेज़बान नेमस्पेस में प्रवेश के लिए, आमतौर पर, `nsenter` का उपयोग करके मेज़बान के `/proc` निर्देशिका तक पहुंच की आवश्यकता होती है।

**नियंत्रण समूह (CGroups)**

* **कार्य**: प्रक्रियाओं के बीच संसाधन आवंटित करने के लिए प्राथमिक रूप से उपयोग किया जाता है।
* **सुरक्षा पहलू**: CGroups स्वयं अलगाव सुरक्षा नहीं प्रदान करते हैं, केवल `release_agent` सुविधा, जो अगर गलती से कॉन्फ़िगर की गई हो, अनधिकृत पहुंच के लिए उत्पादित किया जा सकता है।

**क्षमता छोड़ें**

* **महत्व**: प्रक्रिया विभाजन के लिए एक महत्वपूर्ण सुरक्षा सुविधा है।
* **कार्यक्षमता**: यह एक महत्वपूर्ण सुरक्षा सुविधा है जो किसी रूट प्रक्रिया को कुछ विशेष क्षमताओं को छोड़कर करने की सीमा लगाती है। यदि कोई प्रक्रिया रूट विशेषाधिकारों के साथ चलती है, तो आवश्यक क्षमताओं की कमी के कारण यह उन्हें विशेषाधिकारित क्रियाएँ करने से रोकता है, क्योंकि सिसकॉल अपर्याप्त अनुमतियों के कारण विफल हो जाएंगे।

ये हैं प्रक्रिया जो अन्य क्षमताएँ छोड़ देने के बाद बची हैं:
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

यह डॉकर में डिफ़ॉल्ट रूप से सक्यूर है। यह प्रक्रिया द्वारा कॉल किए जा सकने वाले सिसकॉल्स को और भी सीमित करने में मदद करता है।\
**डिफ़ॉल्ट डॉकर Seccomp प्रोफ़ाइल** यहाँ मिल सकता है: [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

**AppArmor**

डॉकर में एक टेम्पलेट है जिसे आप सक्षम कर सकते हैं: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

यह सुनिश्चित करेगा कि क्षमताएँ, सिसकॉल्स, फ़ाइलों और फ़ोल्डर्स तक पहुँच कम हो...

</details>

### नेमस्पेस

**नेमस्पेस** लिनक्स कर्नेल की एक विशेषता है जो कर्नेल संसाधनों को विभाजित करती है ताकि एक सेट की **प्रक्रियाएँ** एक सेट के **संसाधनों** को **देखें** जबकि **दूसरे** सेट की **प्रक्रियाएँ** एक **विभिन्न** सेट के संसाधनों को देखें। यह सुविधा एक सेट के संसाधनों और प्रक्रियाओं के लिए एक ही नेमस्पेस होने के बावजूद, वे नेमस्पेस विभिन्न संसाधनों को संदर्भित करते हैं। संसाधनों कई स्थानों पर मौजूद हो सकते हैं।

डॉकर निम्नलिखित लिनक्स कर्नेल नेमस्पेस का उपयोग करता है ताकि कंटेनर विलयन को प्राप्त कर सके:

* pid नेमस्पेस
* माउंट नेमस्पेस
* नेटवर्क नेमस्पेस
* ipc नेमस्पेस
* UTS नेमस्पेस

**नेमस्पेस के बारे में अधिक जानकारी** के लिए निम्नलिखित पृष्ठ की जाँच करें:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

लिनक्स कर्नेल सुविधा **cgroups** प्रक्रियाओं के एक सेट के बीच संसाधनों जैसे cpu, मेमोरी, आईओ, नेटवर्क बैंडविड्थ को **सीमित करने की क्षमता प्रदान करती है**। डॉकर cgroup सुविधा का उपयोग करने देता है जिससे विशिष्ट कंटेनर के लिए संसाधन नियंत्रण संभव हो।\
निम्नलिखित एक कंटेनर है जिसे यूज़र स्पेस मेमोरी को 500m तक सीमित किया गया है, कर्नेल मेमोरी को 50m तक सीमित किया गया है, cpu शेयर को 512, blkioweight को 400 किया गया है। CPU शेयर एक अनुपात है जो कंटेनर का CPU उपयोग नियंत्रित करता है। इसका डिफ़ॉल्ट मान 1024 है और 0 और 1024 के बीच रेंज है। यदि तीन कंटेनर्स का CPU शेयर 1024 है, तो हर कंटेनर CPU संसाधन संघर्ष की स्थिति में 33% तक ले सकता है। blkio-weight एक अनुपात है जो कंटेनर का IO नियंत्रित करता है। इसका डिफ़ॉल्ट मान 500 है और 10 और 1000 के बीच रेंज है।
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
किसी कंटेनर का cgroup प्राप्त करने के लिए आप निम्नलिखित कर सकते हैं:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
For more information check:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### योग्यताएँ

योग्यताएँ **मूल उपयोगकर्ता के लिए जो योग्यताएँ अनुमत हो सकती हैं, उन पर अधिक नियंत्रण प्रदान करती हैं**। Docker लिनक्स कर्नेल योग्यता सुविधा का उपयोग करता है **कंटेनर के अंदर किए जा सकने वाले ऑपरेशनों को सीमित करने** के लिए, चाहे वह किसी भी प्रकार के उपयोगकर्ता हो।

जब एक डॉकर कंटेनर चलाया जाता है, **प्रक्रिया उसे छोड़ देती है जो संक्षेपण से बाहर निकलने के लिए प्रयोग कर सकती थी संवेदनशील योग्यताएँ**। यह सुनिश्चित करने का प्रयास करता है कि प्रक्रिया संवेदनशील कार्रवाई करने और बाहर निकलने के लिए सक्षम न हो:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### डॉकर में Seccomp

यह एक सुरक्षा सुविधा है जो डॉकर को यह सीमित करने की अनुमति देती है कि **कंटेनर के अंदर उपयोग किए जा सकने वाले सिसकॉल्स** को:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### डॉकर में AppArmor

**AppArmor** एक कर्नेल एन्हांसमेंट है जो **कंटेनरों** को **सीमित** संसाधनों के **सीमित सेट** में **प्रति-प्रोग्राम प्रोफाइल** के साथ सीमित करने के लिए उपयोग किया जाता है।:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### डॉकर में SELinux

* **लेबलिंग सिस्टम**: SELinux प्रत्येक प्रक्रिया और फ़ाइल सिस्टम ऑब्जेक्ट को एक अद्वितीय लेबल सौंपता है।
* **नीति प्रवर्तन**: यह सुनिश्चित करता है कि प्रक्रिया लेबल क्या कार्रवाई कर सकता है अन्य लेबलों पर सिस्टम के भीतर।
* **कंटेनर प्रक्रिया लेबल**: जब कंटेनर इंजन कंटेनर प्रक्रियाएँ प्रारंभ करते हैं, तो उन्हें सामान्यत: एक सीमित SELinux लेबल दिया जाता है, सामान्यत: `container_t`।
* **कंटेनर के भीतर फ़ाइल लेबलिंग**: कंटेनर के भीतर फ़ाइलों को सामान्यत: `container_file_t` लेबल दिया जाता है।
* **नीति नियम**: SELinux नीति मुख्य रूप से सुनिश्चित करती है कि `container_t` लेबल वाली प्रक्रियाएँ केवल उन फ़ाइलों के साथ व्यवहार (पढ़ना, लिखना, क्रियान्वयन) कर सकती हैं जिन्हें `container_file_t` लेबल से लेबलित किया गया है।

यह तंत्र सुनिश्चित करता है कि यदि कंटेनर के भीतर कोई प्रक्रिया क्षतिग्रस्त होती है, तो उसे केवल उसी लेबल वाले वस्तुओं के साथ व्यवहार करने की सीमा लगाई जाती है, इस प्रकार ऐसी क्षति से होने वाली संभावित नुकसान को सीमित करते हैं।

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ और AuthN

डॉकर में, एक अधिकृति प्लगइन सुरक्षा में महत्वपूर्ण भूमिका निभाता है जिसे यह निर्धारित करना होता है कि डॉकर डेमन को अनुमति देने या रोकने के लिए। यह निर्णय दो मुख्य संदर्भों की जांच करके लिया जाता है:

* **प्रमाणीकरण संदर्भ**: इसमें उपयोगकर्ता के बारे में विस्तृत जानकारी शामिल है, जैसे कि वे कौन हैं और वे अपने आप को कैसे प्रमाणित कर रहे हैं।
* **कमांड संदर्भ**: यह अनुरोध से संबंधित सभी महत्वपूर्ण डेटा को शामिल करता है।

ये संदर्भ सुनिश्चित करते हैं कि केवल प्रमाणित उपयोगकर्ताओं के से वैध अनुरोध प्रसंस्कृत किए जाते हैं, जिससे डॉकर कार्रवाईयों की सुरक्षा में सुधार होता है।

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## कंटेनर से डॉस

यदि आप सही ढंग से कंटेनर के उपयोग कर सकने वाले संसाधनों की सीमा नहीं लगा रहे हैं, तो एक क्षतिग्रस्त कंटेनर मेज़बान को डॉस कर सकता है जहाँ वह चल रहा है।

* CPU डॉस
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* बैंडविड्थ डीओएस
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## दिलचस्प Docker फ्लैग

### --privileged फ्लैग

निम्नलिखित पृष्ठ पर आप **यह सीख सकते हैं कि `--privileged` फ्लैग का क्या मतलब है**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

यदि आप एक कंटेनर चला रहे हैं जहां एक हमलावर को कम विशेषाधिकार वाले उपयोगकर्ता के रूप में पहुंचने में सफलता मिलती है। यदि आपके पास एक **गलत रूप से कॉन्फ़िगर किया गया suid बाइनरी** है, तो हमलावर इसका दुरुपयोग कर सकता है और **कंटेनर के अंदर विशेषाधिकारों को उन्नत कर सकता है**। जिससे उसे इससे बाहर निकलने की अनुमति हो सकती है।

**`no-new-privileges`** विकल्प सक्षम करके कंटेनर को चलाना इस प्रकार की विशेषाधिकारों की उन्नति को **रोकेगा**।
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### अन्य
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
अधिक **`--security-opt`** विकल्पों के लिए जांच करें: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## अन्य सुरक्षा मामले

### रहस्यों का प्रबंधन: सर्वोत्तम अभ्यास

डॉकर इमेज में सीधे रहस्यों को समाहित करने या पर्यावरण चरणों का उपयोग करने से बचना महत्वपूर्ण है, क्योंकि ये विधियाँ `docker inspect` या `exec` जैसे कमांडों के माध्यम से कंटेनर तक पहुंचने वाले किसी भी व्यक्ति को आपकी संवेदनशील जानकारी को उजागर करती हैं।

**डॉकर वॉल्यूम्स** एक अधिक सुरक्षित विकल्प हैं, जिन्हें संवेदनशील जानकारी तक पहुंचने के लिए सिफारिश की जाती है। ये एक अस्थायी फ़ाइल सिस्टम के रूप में उपयोग किए जा सकते हैं, `docker inspect` और लॉगिंग के साथ जुड़े जोखिमों को कम करते हैं। हालांकि, रूट उपयोगकर्ताओं और उन लोगों तक जो कंटेनर में `exec` पहुंच रखे हैं, रहस्यों तक पहुंच सकते हैं।

**डॉकर सीक्रेट्स** संवेदनशील जानकारी को संभालने के लिए एक और अधिक सुरक्षित विधि प्रदान करते हैं। इमेज निर्माण चरण के दौरान रहस्यों की आवश्यकता होने वाले मामलों के लिए, **BuildKit** एक कुशल समाधान प्रस्तुत करता है जिसमें निर्माण समय रहस्यों का समर्थन किया जाता है, निर्माण की गति को बढ़ाता है और अतिरिक्त सुविधाएँ प्रदान करता है।

BuildKit का उपयोग करने के लिए, इसे तीन तरीकों से सक्रिय किया जा सकता है:

1. एक पर्यावरण चरण के माध्यम से: `export DOCKER_BUILDKIT=1`
2. कमांडों को उपसर्ग लगाकर: `DOCKER_BUILDKIT=1 docker build .`
3. डॉकर कॉन्फ़िगरेशन में इसे डिफ़ॉल्ट रूप से सक्षम करना: `{ "features": { "buildkit": true } }`, इसके बाद डॉकर को पुनः आरंभ करें।

BuildKit रहस्यों का उपयोग करने के लिए `--secret` विकल्प के साथ अनुमति देता है, यह सुनिश्चित करता है कि ये रहस्य इमेज निर्माण कैश या अंतिम इमेज में शामिल नहीं हैं, जैसे कि निम्नलिखित कमांड का उपयोग करके:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
संचालन करने के लिए आवश्यक रहने वाले रहस्यों के लिए, **Docker Compose और Kubernetes** मजबूत समाधान प्रदान करते हैं। Docker Compose एक `secrets` कुंजी का उपयोग सेवा परिभाषा में गोपनीय फ़ाइलों को निर्दिष्ट करने के लिए करता है, जैसा कि एक `docker-compose.yml` उदाहरण में दिखाया गया है:
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
यह कॉन्फ़िगरेशन Docker Compose के साथ सेवाएं शुरू करते समय रहस्यों का उपयोग करने की अनुमति देता है।

कुबरनेट्स परिवेशों में, रहस्यों का प्राकृतिक रूप से समर्थन किया जाता है और [Helm-Secrets](https://github.com/futuresimple/helm-secrets) जैसे उपकरणों के साथ और अधिक प्रबंधित किया जा सकता है। कुबरनेट्स का रोल आधारित पहुंच नियंत्रण (RBAC) रहस्य प्रबंधन सुरक्षा को बढ़ाता है, जैसे Docker Enterprise के समान।

### gVisor

**gVisor** एक एप्लिकेशन कर्नेल है, जो गो में लिखा गया है, जो लिनक्स सिस्टम सर्फेस का एक बड़ा हिस्सा कार्यान्वित करता है। इसमें एक [Open Container Initiative (OCI)](https://www.opencontainers.org) रनटाइम `runsc` शामिल है जो एप्लिकेशन और होस्ट कर्नेल के बीच एक **अलगाव सीमा** प्रदान करता है। `runsc` रनटाइम Docker और Kubernetes के साथ एकीकृत होता है, जिससे सैंडबॉक्स कंटेनर चलाना सरल हो जाता है।

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** एक ओपन सोर्स समुदाय है जो एक सुरक्षित कंटेनर रनटाइम बनाने के लिए काम कर रहा है जिसमें हल्के वर्चुअल मशीन्स का उपयोग करके कंटेनर की तरह अनुभव और कार्य करते हैं, लेकिन **हार्डवेयर वर्चुअलाइजेशन** प्रौद्योगिकी का उपयोग करके **मजबूत कार्यभार अलगाव** प्रदान करते हैं।

{% embed url="https://katacontainers.io/" %}

### सारांश युक्तियाँ

* **`--privileged` फ्लैग का उपयोग न करें और** [**कंटेनर के अंदर डॉकर सॉकेट माउंट**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/) **न करें।** डॉकर सॉकेट कंटेनर उत्पन्न करने की अनुमति देता है, इसलिए यह होस्ट के पूर्ण नियंत्रण में लेने का एक आसान तरीका है, उदाहरण के लिए, `--privileged` फ्लैग के साथ एक और कंटेनर चलाने के द्वारा।
* **कंटेनर के अंदर रूट के रूप में न चलाएं। एक** [**विभिन्न उपयोगकर्ता**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **का उपयोग करें और** [**उपयोगकर्ता नेमस्पेस**](https://docs.docker.com/engine/security/userns-remap/)** का उपयोग करें।** कंटेनर में रूट होस्ट के समान है जब तक उपयोगकर्ता नेमस्पेस के साथ पुनर्मार्पित नहीं किया जाता है। यह केवल, मुख्य रूप से, लिनक्स नेमस्पेस, क्षमताएँ, और सीग्रुप्स द्वारा हल्के से प्रतिबंधित है।
* [**सभी क्षमताएँ छोड़ें**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) और केवल वही जो आवश्यक है उन्हें सक्षम करें** (`--cap-add=...`)। अधिकांश कार्यभारों को कोई भी क्षमताएँ नहीं चाहिए और उन्हें जोड़ने से एक संभावित हमले की दायरा बढ़ जाती है।
* [**“no-new-privileges” सुरक्षा विकल्प का उपयोग करें**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) जिससे प्रक्रियाएं अधिक विशेषाधिकार प्राप्त न करें, उदाहरण के लिए suid बाइनरी के माध्यम से।
* [**कंटेनर के लिए उपलब्ध संसाधनों की सीमा निर्धारित करें**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)। संसाधन सीमाएँ मशीन को डीनाइल ऑफ सर्विस हमलों से सुरक्षित रख सकती हैं।
* **सेक्कॉम्प**, [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(या SELinux)** प्रोफाइल को संक्षेपित करने के लिए संयम लागू करें जो कंटेनर के लिए उपलब्ध क्रियाएँ और सिसकॉल्स को न्यूनतम आवश्यक करते हैं।
* **[आधिकारिक डॉकर छवियाँ](https://docs.docker.com/docker-hub/official\_images/) का उपयोग करें और हस्ताक्षर की आवश्यकता** या उन पर आधारित अपनी छवियाँ बनाएं। [बैकडोर](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) छवियों को विरासत में न लें या उपयोग न करें। इसके अलावा, रूट कुंजी, पासफ्रेज को एक सुरक्षित स्थान में स्टोर करें। डॉकर के पास UCP के साथ कुंजियों का प्रबंधन करने की योजनाएं हैं।
* अपनी छवियों को **नियमित रूप से पुनः निर्माण करें** ताकि **होस्ट और छवियों में सुरक्षा पैच लागू किए जा सकें।**
* अपने **रहस्यों का बुद्धिमानी से प्रबंधन** करें ताकि उन्हें अधिकारी को पहुंचना कठिन हो।
* यदि आप **डॉकर डेमन को उजागर करते हैं तो HTTPS** का उपयोग करें जिसमें क्लाइंट और सर्वर प्रमाणीकरण हो।
* अपने Dockerfile में **ADD की बजाय COPY का उपयोग करें**। ADD स्वचालित रूप से ज़िप फ़ाइलों को निकालता है और यूआरएल से फ़ाइलें कॉपी कर सकता है। COPY के पास ये क्षमताएँ नहीं हैं। जितना संभव हो, ADD का उपयोग न करें ताकि आप दूरस्थ यूआरएल और ज़िप फ़ाइलों के माध्यम से हमलों के लिए संक्रमित न हों।
* प्रत्येक माइक्रो सेवा के लिए **अलग कंटेनर** होना चाहिए
* कंटेनर की **छवियों को छोटा** रखें

## Docker Breakout / Privilege Escalation

यदि आप **एक डॉकर कंटेनर के अंदर हैं** या आपके पास **डॉकर समूह में उपयोगकर्ता तक पहुंच है**, तो आप **भागने और विशेषाधिकारों को बढ़ाने** की कोशिश कर सकते हैं:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Docker Authentication Plugin Bypass

यदि आपके पास डॉकर सॉकेट तक पहुंच है या आपके पास **डॉकर समूह में उपयोगकर्ता की पहुंच है लेकिन आपके कार्रवाई को एक डॉकर प्रमाणीकरण प्लगइन द्वारा सीमित किया जा रहा है**, तो जांचें कि क्या आप इसे **अनदेखा कर सकते हैं:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Docker को मजबूत बनाना

* उपकरण [**docker-bench-security**](https://github.com/docker/docker-bench-security) एक स्क्रिप्ट है जो उत्पादन में Docker कंटेनर डिप्लॉय करने के आस-पास की दर्जनों सामान्य श्रेष्ठ-प्रथाओं की जांच करता है। ये सभी टेस्ट स्वचालित हैं, और [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/) पर आधारित हैं।\
आपको उपकरण को डॉकर चलाने वाले होस्ट से या पर्याप्त विशेषाधिकारों वाले एक कंटेनर से चलाने की आवश्यकता है। README में इसे कैसे चलाने के बारे में जानें: [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## संदर्भ

* [https://blog.trailofbits.com/2019/07/19/understanding-docker
सीखें और प्रैक्टिस करें AWS हैकिंग: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}
