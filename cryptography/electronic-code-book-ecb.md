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


# ECB

(ECB) Kitabu cha Kanuni za Kielektroniki - mpango wa usimbuaji wa simetriki ambao **unabadilisha kila block ya maandiko wazi** kwa **block ya maandiko yaliyosimbwa**. Ni mpango wa usimbuaji **rahisi zaidi**. Wazo kuu ni **kugawanya** maandiko wazi katika **blocks za N bits** (inategemea ukubwa wa block ya data ya ingizo, algorithm ya usimbuaji) na kisha kusimbua (kufungua) kila block ya maandiko wazi kwa kutumia funguo pekee.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Kutumia ECB kuna athari nyingi za usalama:

* **Blocks kutoka kwa ujumbe uliofungwa zinaweza kuondolewa**
* **Blocks kutoka kwa ujumbe uliofungwa zinaweza kuhamishwa**

# Ugunduzi wa udhaifu

Fikiria unapoingia kwenye programu mara kadhaa na **daima unapata cookie ile ile**. Hii ni kwa sababu cookie ya programu ni **`<username>|<password>`**.\
Kisha, unaunda watumiaji wapya wawili, wote wakiwa na **nenosiri refu sawa** na **karibu** **jina la mtumiaji** **sawa**.\
Unagundua kwamba **blocks za 8B** ambapo **info ya watumiaji wote wawili** ni sawa ni **sawa**. Kisha, unafikiria kwamba hii inaweza kuwa kwa sababu **ECB inatumika**.

Kama katika mfano ufuatao. Angalia jinsi hizi **2 cookies zilizofunguliwa** zina block kadhaa **`\x23U\xE45K\xCB\x21\xC8`**.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Hii ni kwa sababu **jina la mtumiaji na nenosiri la vidakuzi hivyo vilikuwa na herufi "a" mara kadhaa** (kwa mfano). **Vizuizi** ambavyo ni **tofauti** ni vizuizi vilivyokuwa na **angalau herufi 1 tofauti** (labda mkataba "|" au tofauti muhimu katika jina la mtumiaji).

Sasa, mshambuliaji anahitaji tu kugundua kama muundo ni `<username><delimiter><password>` au `<password><delimiter><username>`. Ili kufanya hivyo, anaweza tu **kuunda majina kadhaa ya watumiaji** yenye **majina marefu na yanayofanana na nenosiri hadi apate muundo na urefu wa mkataba:**

| Urefu wa jina la mtumiaji: | Urefu wa nenosiri: | Urefu wa Jina la mtumiaji + Nenosiri: | Urefu wa Cookie (baada ya kufichua): |
| --------------------------- | ------------------ | ------------------------------------ | ----------------------------------- |
| 2                           | 2                  | 4                                    | 8                                   |
| 3                           | 3                  | 6                                    | 8                                   |
| 3                           | 4                  | 7                                    | 8                                   |
| 4                           | 4                  | 8                                    | 16                                  |
| 7                           | 7                  | 14                                   | 16                                  |

# Ukatili wa udhaifu

## Kuondoa vizuizi vyote

Kujua muundo wa cookie (`<username>|<password>`), ili kujifanya kuwa jina la mtumiaji `admin` tengeneza mtumiaji mpya anayeitwa `aaaaaaaaadmin` na pata cookie na uifichue:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Tunaweza kuona muundo `\x23U\xE45K\xCB\x21\xC8` ulioundwa hapo awali na jina la mtumiaji lililokuwa na `a` pekee.\
Kisha, unaweza kuondoa block ya kwanza ya 8B na utapata cookie halali kwa jina la mtumiaji `admin`:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Kuhamasisha vizuizi

Katika hifadhidata nyingi, ni sawa kutafuta `WHERE username='admin';` au `WHERE username='admin    ';` _(Kumbuka nafasi za ziada)_

Hivyo, njia nyingine ya kujifanya kuwa mtumiaji `admin` ingekuwa:

* Kuunda jina la mtumiaji ambalo: `len(<username>) + len(<delimiter) % len(block)`. Kwa ukubwa wa block wa `8B` unaweza kuunda jina la mtumiaji linaloitwa: `username       `, na delimiter `|` kipande `<username><delimiter>` kitazalisha vizuizi 2 vya 8Bs.
* Kisha, kuunda nenosiri ambalo litajaza idadi sahihi ya vizuizi vinavyomwonyesha jina la mtumiaji tunataka kujifanya na nafasi, kama: `admin   `

Keki ya mtumiaji huyu itaundwa na vizuizi 3: vya kwanza 2 ni vizuizi vya jina la mtumiaji + delimiter na cha tatu ni nenosiri (ambacho kinajifanya kuwa jina la mtumiaji): `username       |admin   `

**Kisha, badilisha tu block ya kwanza na ya mwisho na utakuwa unajifanya kuwa mtumiaji `admin`: `admin          |username`**

## Marejeo

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


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
