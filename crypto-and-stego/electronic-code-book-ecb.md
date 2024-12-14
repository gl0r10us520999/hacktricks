{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}


# ECB

(ECB) Elektroniese Kodeboek - simmetriese versleuteling skema wat **elke blok van die duidelike teks vervang** deur die **blok van gesleutelde teks**. Dit is die **simpele** versleuteling skema. Die hoofidee is om die duidelike teks in **blokkies van N bits** te **verdeel** (hang af van die grootte van die blok invoerdata, versleuteling algoritme) en dan om elke blok van duidelike teks te versleutel (ontsleutel) met die enigste sleutel.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Die gebruik van ECB het verskeie sekuriteitsimplikasies:

* **Blokkies van die gesleutelde boodskap kan verwyder word**
* **Blokkies van die gesleutelde boodskap kan rondbeweeg word**

# Opsporing van die kwesbaarheid

Stel jou voor jy teken verskeie kere in op 'n toepassing en jy **kry altyd dieselfde koekie**. Dit is omdat die koekie van die toepassing **`<gebruikersnaam>|<wagwoord>`** is.\
Dan genereer jy twee nuwe gebruikers, albei met die **selfde lang wagwoord** en **amper** die **selfde** **gebruikersnaam**.\
Jy vind uit dat die **blokkies van 8B** waar die **inligting van albei gebruikers** dieselfde is, **gelyk** is. Dan stel jy jou voor dat dit dalk is omdat **ECB gebruik word**.

Soos in die volgende voorbeeld. Let op hoe hierdie **2 gedecodeerde koekies** verskeie kere die blok **`\x23U\xE45K\xCB\x21\xC8`** het.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Dit is omdat die **gebruikersnaam en wagwoord van daardie koekies verskeie kere die letter "a" bevat** (byvoorbeeld). Die **blokke** wat **verskillend** is, is blokke wat **ten minste 1 verskillende karakter** bevat (miskien die skeidingsteken "|" of 'n nodige verskil in die gebruikersnaam).

Nou moet die aanvaller net ontdek of die formaat `<gebruikersnaam><skeidingsteken><wagwoord>` of `<wagwoord><skeidingsteken><gebruikersnaam>` is. Om dit te doen, kan hy net **verskeie gebruikersname genereer** met **soortgelyke en lang gebruikersname en wagwoorde totdat hy die formaat en die lengte van die skeidingsteken vind:**

| Gebruikersnaam lengte: | Wagwoord lengte: | Gebruikersnaam+Wagwoord lengte: | Koekie se lengte (na dekodering): |
| ---------------------- | ---------------- | ------------------------------- | --------------------------------- |
| 2                      | 2                | 4                               | 8                                 |
| 3                      | 3                | 6                               | 8                                 |
| 3                      | 4                | 7                               | 8                                 |
| 4                      | 4                | 8                               | 16                                |
| 7                      | 7                | 14                              | 16                                |

# Exploitering van die kwesbaarheid

## Verwydering van hele blokke

Weetende die formaat van die koekie (`<gebruikersnaam>|<wagwoord>`), om die gebruikersnaam `admin` na te doen, skep 'n nuwe gebruiker genaamd `aaaaaaaaadmin` en kry die koekie en dekodeer dit:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
We kan die patroon `\x23U\xE45K\xCB\x21\xC8` sien wat vroeër geskep is met die gebruikersnaam wat slegs `a` bevat.\
Dan kan jy die eerste blok van 8B verwyder en jy sal 'n geldige koekie vir die gebruikersnaam `admin` kry:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Beweeg blokke

In baie databasisse is dit dieselfde om te soek vir `WHERE username='admin';` of vir `WHERE username='admin    ';` _(Let op die ekstra spaties)_

So, 'n ander manier om die gebruiker `admin` na te boots, sou wees om:

* 'n gebruikersnaam te genereer wat: `len(<username>) + len(<delimiter) % len(block)`. Met 'n blokgrootte van `8B` kan jy 'n gebruikersnaam genereer wat genoem word: `username       `, met die afskeid `|` sal die stuk `<username><delimiter>` 2 blokke van 8Bs genereer.
* Dan, genereer 'n wagwoord wat 'n presiese aantal blokke sal vul wat die gebruikersnaam bevat wat ons wil naboots en spaties, soos: `admin   `

Die koekie van hierdie gebruiker gaan bestaan uit 3 blokke: die eerste 2 is die blokke van die gebruikersnaam + afskeid en die derde een van die wagwoord (wat die gebruikersnaam vervals): `username       |admin   `

**Vervang dan net die eerste blok met die laaste keer en jy sal die gebruiker `admin` naboots: `admin          |username`**

## Verwysings

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
