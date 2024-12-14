# ZIPs tricks

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

**Opdraglyn gereedskap** vir die bestuur van **zip lêers** is noodsaaklik vir die diagnose, herstel en kraak van zip lêers. Hier is 'n paar sleutel nutsmiddels:

- **`unzip`**: Toon waarom 'n zip lêer dalk nie ontspan nie.
- **`zipdetails -v`**: Bied gedetailleerde analise van zip lêer formaat velde.
- **`zipinfo`**: Lys die inhoud van 'n zip lêer sonder om dit te onttrek.
- **`zip -F input.zip --out output.zip`** en **`zip -FF input.zip --out output.zip`**: Probeer om beskadigde zip lêers te herstel.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: 'n Gereedskap vir brute-force kraak van zip wagwoorde, effektief vir wagwoorde tot ongeveer 7 karakters.

Die [Zip lêer formaat spesifikasie](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) bied omvattende besonderhede oor die struktuur en standaarde van zip lêers.

Dit is belangrik om op te let dat wagwoord-beskermde zip lêers **nie lêernaam of lêergrootte** binne-in enkripteer nie, 'n sekuriteitsgebrek wat nie met RAR of 7z lêers gedeel word nie, wat hierdie inligting enkripteer. Verder is zip lêers wat met die ouer ZipCrypto metode enkripteer is, kwesbaar vir 'n **plaktekstaanval** as 'n nie-enkryptiese kopie van 'n gecomprimeerde lêer beskikbaar is. Hierdie aanval benut die bekende inhoud om die zip se wagwoord te kraak, 'n kwesbaarheid wat in [HackThis se artikel](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) uiteengesit word en verder verduidelik word in [hierdie akademiese artikel](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). egter, zip lêers wat met **AES-256** enkripteer is, is immuun teen hierdie plaktekstaanval, wat die belangrikheid van die keuse van veilige enkriptemetodes vir sensitiewe data toon.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
