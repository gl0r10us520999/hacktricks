# ZIPs tricks

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

**Alati za komandnu liniju** za upravljanje **zip datotekama** su neophodni za dijagnostikovanje, popravku i razbijanje zip datoteka. Evo nekoliko ključnih alata:

- **`unzip`**: Otkriva zašto zip datoteka možda ne može da se raspakuje.
- **`zipdetails -v`**: Pruža detaljnu analizu polja formata zip datoteke.
- **`zipinfo`**: Navodi sadržaj zip datoteke bez vađenja.
- **`zip -F input.zip --out output.zip`** i **`zip -FF input.zip --out output.zip`**: Pokušavaju da poprave oštećene zip datoteke.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Alat za brute-force razbijanje zip lozinki, efikasan za lozinke do oko 7 karaktera.

[Specifikacija formata zip datoteka](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) pruža sveobuhvatne detalje o strukturi i standardima zip datoteka.

Važno je napomenuti da zip datoteke zaštićene lozinkom **ne enkriptuju imena datoteka ili veličine datoteka** unutar, što je sigurnosni propust koji RAR ili 7z datoteke ne dele, jer enkriptuju te informacije. Pored toga, zip datoteke enkriptovane starijom metodom ZipCrypto su ranjive na **napad u običnom tekstu** ako je dostupna neenkriptovana kopija kompresovane datoteke. Ovaj napad koristi poznati sadržaj za razbijanje zip lozinke, ranjivost koja je detaljno objašnjena u [HackThis članku](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) i dodatno objašnjena u [ovoj akademskoj studiji](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). Međutim, zip datoteke zaštićene **AES-256** enkripcijom su imune na ovaj napad u običnom tekstu, što pokazuje važnost izbora sigurnih metoda enkripcije za osetljive podatke.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

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
