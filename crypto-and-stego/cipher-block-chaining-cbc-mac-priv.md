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


# CBC

As die **cookie** **net** die **gebruikersnaam** is (of die eerste deel van die cookie is die gebruikersnaam) en jy wil die gebruikersnaam "**admin**" naboots. Dan kan jy die gebruikersnaam **"bdmin"** skep en die **eerste byte** van die cookie **bruteforce**.

# CBC-MAC

**Cipher block chaining message authentication code** (**CBC-MAC**) is 'n metode wat in kriptografie gebruik word. Dit werk deur 'n boodskap te neem en dit blok vir blok te enkripteer, waar elke blok se enkripsie aan die een voor dit gekoppel is. Hierdie proses skep 'n **ketting van blokke**, wat verseker dat die verandering van selfs 'n enkele bit van die oorspronklike boodskap sal lei tot 'n onvoorspelbare verandering in die laaste blok van versleutelde data. Om so 'n verandering te maak of om dit om te keer, is die enkripsiesleutel nodig, wat sekuriteit verseker.

Om die CBC-MAC van boodskap m te bereken, enkripteer mens m in CBC-modus met nul inisialisasievector en hou die laaste blok. Die volgende figuur skets die berekening van die CBC-MAC van 'n boodskap wat uit blokke bestaan![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) met 'n geheime sleutel k en 'n blokversleuteling E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Kwetsbaarheid

Met CBC-MAC is die **IV wat gebruik word gewoonlik 0**.\
Dit is 'n probleem omdat 2 bekende boodskappe (`m1` en `m2`) onafhanklik 2 handtekeninge (`s1` en `s2`) sal genereer. So:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Dan sal 'n boodskap wat uit m1 en m2 saamgevoeg (m3) bestaan, 2 handtekeninge (s31 en s32) genereer:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Wat moontlik is om te bereken sonder om die sleutel van die enkripsie te ken.**

Stel jou voor jy enkripteer die naam **Administrator** in **8bytes** blokke:

* `Administ`
* `rator\00\00\00`

Jy kan 'n gebruikersnaam genaamd **Administ** (m1) skep en die handtekening (s1) terugkry.\
Dan kan jy 'n gebruikersnaam skep wat die resultaat van `rator\00\00\00 XOR s1` is. Dit sal `E(m2 XOR s1 XOR 0)` genereer wat s32 is.\
Nou kan jy s32 as die handtekening van die volle naam **Administrator** gebruik.

### Samevatting

1. Kry die handtekening van gebruikersnaam **Administ** (m1) wat s1 is
2. Kry die handtekening van gebruikersnaam **rator\x00\x00\x00 XOR s1 XOR 0** is s32**.**
3. Stel die cookie op s32 en dit sal 'n geldige cookie wees vir die gebruiker **Administrator**.

# Aanval Beheer IV

As jy die gebruikte IV kan beheer, kan die aanval baie maklik wees.\
As die koekies net die gebruikersnaam is wat geënkripteer is, om die gebruiker "**administrator**" na te boots, kan jy die gebruiker "**Administrator**" skep en jy sal sy cookie kry.\
Nou, as jy die IV kan beheer, kan jy die eerste byte van die IV verander sodat **IV\[0] XOR "A" == IV'\[0] XOR "a"** en die cookie vir die gebruiker **Administrator** hergenerer. Hierdie cookie sal geldig wees om die gebruiker **administrator** met die aanvanklike **IV** na te boots.

## Verwysings

Meer inligting in [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
