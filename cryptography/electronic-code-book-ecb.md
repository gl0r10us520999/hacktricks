{% hint style="success" %}
Leer & oefen AWS-hacking: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Opleiding AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP-hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Opleiding GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Controleer die [**inskrywingsplanne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking-truuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
{% endhint %}


# ECB

(ECB) Elektroniese Kodeboek - simmetriese enkripsieskema wat **elke blok van die teks** vervang met die **blok van kriptoteks**. Dit is die **eenvoudigste** enkripsieskema. Die hoofidee is om die teks in **blokke van N-bits** (afhangende van die grootte van die blok van insetdata, enkripsie-algoritme) te **verdeel** en dan elke blok van die teks te enkripteer (de-enkripteer) met slegs die sleutel.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Die gebruik van ECB het verskeie veiligheidsimplikasies:

* **Blokke van die gekripteerde boodskap kan verwyder word**
* **Blokke van die gekripteerde boodskap kan rondgeskuif word**

# Opname van die kwesbaarheid

Stel jou voor jy meld aan by 'n aansoek verskeie kere aan en jy kry **altyd dieselfde koekie**. Dit is omdat die koekie van die aansoek is **`<gebruikersnaam>|<wagwoord>`**.\
Dan, jy skep twee nuwe gebruikers, albei met dieselfde lang wagwoord en **bykans** dieselfde **gebruikersnaam**.\
Jy vind uit dat die **blokke van 8B** waar die **inligting van beide gebruikers** dieselfde is, **gelyk** is. Dan, jy dink dat dit dalk is omdat **ECB gebruik word**.

Soos in die volgende voorbeeld. Let op hoe hierdie **2 ontkodeerde koekies** verskeie kere die blok **`\x23U\xE45K\xCB\x21\xC8`** het.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Dit is omdat die **gebruikersnaam en wagwoord van daardie koekies verskeie kere die letter "a" bevat het** (byvoorbeeld). Die **blokke** wat **verskillend** is, is blokke wat **ten minste 1 verskillende karakter** bevat het (miskien die afskeider "|" of 'n nodige verskil in die gebruikersnaam).

Nou hoef die aanvaller net te ontdek of die formaat `<gebruikersnaam><afskeider><wagwoord>` of `<wagwoord><afskeider><gebruikersnaam>` is. Om dit te doen, kan hy net **veral verskeie gebruikersname** met **soortgelyke en lang gebruikersname en wagwoorde genereer totdat hy die formaat en die lengte van die afskeider vind:**

| Gebruikersnaam lengte: | Wagwoord lengte: | Gebruikersnaam+Wagwoord lengte: | Koekie se lengte (na dekodeer): |
| ---------------------- | ----------------- | ------------------------------- | ------------------------------- |
| 2                      | 2                 | 4                               | 8                               |
| 3                      | 3                 | 6                               | 8                               |
| 3                      | 4                 | 7                               | 8                               |
| 4                      | 4                 | 8                               | 16                              |
| 7                      | 7                 | 14                              | 16                              |

# Uitbuiting van die kwesbaarheid

## Verwydering van hele blokke

Met kennis van die formaat van die koekie (`<gebruikersnaam>|<wagwoord>`), om die gebruikersnaam `admin` te impersoneer, skep 'n nuwe gebruiker genaamd `aaaaaaaaadmin` en kry die koekie en dekodeer dit:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Ons kan die patroon `\x23U\xE45K\xCB\x21\xC8` sien wat vroeër geskep is met die gebruikersnaam wat slegs `a` bevat het.\
Dan kan jy die eerste blok van 8B verwyder en jy sal 'n geldige koekie vir die gebruikersnaam `admin` kry:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Bewegende blokke

In baie databasisse is dit dieselfde om te soek vir `WHERE username='admin';` of vir `WHERE username='admin    ';` _(Let op die ekstra spasies)_

Dus, 'n ander manier om die gebruiker `admin` te impersoneer sou wees om:

* Genereer 'n gebruikersnaam wat: `len(<username>) + len(<delimiter) % len(block)`. Met 'n blokgrootte van `8B` kan jy 'n gebruikersnaam genaamd genereer: `username       `, met die delimiter `|` sal die stuk `<username><delimiter>` 2 blokke van 8Bs genereer.
* Genereer dan 'n wagwoord wat 'n presiese aantal blokke vul wat die gebruikersnaam wat ons wil impersoneer en spasies bevat, soos: `admin   `

Die koekie van hierdie gebruiker gaan saamgestel wees uit 3 blokke: die eerste 2 is die blokke van die gebruikersnaam + delimiter en die derde een van die wagwoord (wat die gebruikersnaam naboots): `username       |admin   `

**Dan, vervang net die eerste blok met die laaste keer en sal jy die gebruiker `admin` impersoneer: `admin          |username`**

## Verwysings

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))
