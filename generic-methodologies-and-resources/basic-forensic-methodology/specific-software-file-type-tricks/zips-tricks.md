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

**Zana za mistari wa amri** kwa usimamizi wa **zip files** ni muhimu kwa ajili ya kugundua, kurekebisha, na kuvunja zip files. Hapa kuna zana muhimu:

- **`unzip`**: Inaonyesha kwa nini zip file inaweza isifunguke.
- **`zipdetails -v`**: Inatoa uchambuzi wa kina wa maeneo ya muundo wa zip file.
- **`zipinfo`**: Inataja maudhui ya zip file bila kuyatoa.
- **`zip -F input.zip --out output.zip`** na **`zip -FF input.zip --out output.zip`**: Jaribu kurekebisha zip files zilizoharibika.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: Zana ya kuvunja nenosiri la zip kwa nguvu, inafanya kazi vizuri kwa nenosiri hadi karibu herufi 7.

[Maelezo ya muundo wa zip file](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) inatoa maelezo ya kina kuhusu muundo na viwango vya zip files.

Ni muhimu kutambua kwamba zip files zilizo na nenosiri **hazifichi majina ya faili au ukubwa wa faili** ndani, kasoro ya usalama ambayo haipatikani kwa RAR au 7z files ambazo zinaficha taarifa hii. Zaidi ya hayo, zip files zilizofichwa kwa njia ya zamani ya ZipCrypto zinaweza kuathirika na **shambulio la maandiko wazi** ikiwa nakala isiyo na usalama ya faili iliyoshinikizwa inapatikana. Shambulio hili linatumia maudhui yanayojulikana kuvunja nenosiri la zip, udhaifu huu umeelezwa kwa kina katika [makala ya HackThis](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) na kufafanuliwa zaidi katika [karatasi hii ya kitaaluma](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). Hata hivyo, zip files zilizolindwa kwa **AES-256** encryption hazihusiki na shambulio hili la maandiko wazi, ikionyesha umuhimu wa kuchagua mbinu salama za encryption kwa data nyeti.

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
