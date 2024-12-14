# Browser Artifacts

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

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Browsers Artifacts <a href="#id-3def" id="id-3def"></a>

Browser artifacts include various types of data stored by web browsers, such as navigation history, bookmarks, and cache data. These artifacts are kept in specific folders within the operating system, differing in location and name across browsers, yet generally storing similar data types.

Here's a summary of the most common browser artifacts:

* **Navigaciona istorija**: Prati posete korisnika veb sajtovima, korisno za identifikaciju poseta zlonamernim sajtovima.
* **Podaci za automatsko popunjavanje**: Predlozi zasnovani na čestim pretragama, nudeći uvide kada se kombinuju sa navigacionom istorijom.
* **Obeleživači**: Sajtovi koje je korisnik sačuvao za brzi pristup.
* **Ekstenzije i dodaci**: Ekstenzije ili dodaci za pretraživač koje je instalirao korisnik.
* **Keš**: Čuva veb sadržaj (npr. slike, JavaScript datoteke) kako bi poboljšao vreme učitavanja veb sajtova, vredno za forenzičku analizu.
* **Prijave**: Sačuvane prijavne informacije.
* **Favikoni**: Ikone povezane sa veb sajtovima, koje se pojavljuju na karticama i obeleživačima, korisne za dodatne informacije o posetama korisnika.
* **Sesije pretraživača**: Podaci vezani za otvorene sesije pretraživača.
* **Preuzimanja**: Zapisnici datoteka preuzetih putem pretraživača.
* **Podaci iz obrazaca**: Informacije unesene u veb obrasce, sačuvane za buduće predloge automatskog popunjavanja.
* **Sličice**: Pregledne slike veb sajtova.
* **Custom Dictionary.txt**: Reči koje je korisnik dodao rečniku pretraživača.

## Firefox

Firefox organizuje korisničke podatke unutar profila, koji se čuvaju na specifičnim mestima u zavisnosti od operativnog sistema:

* **Linux**: `~/.mozilla/firefox/`
* **MacOS**: `/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**: `%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

A `profiles.ini` datoteka unutar ovih direktorijuma sadrži listu korisničkih profila. Podaci svakog profila se čuvaju u fascikli nazvanoj u `Path` varijabli unutar `profiles.ini`, koja se nalazi u istom direktorijumu kao i `profiles.ini`. Ako nedostaje fascikla profila, možda je obrisana.

Unutar svake fascikle profila možete pronaći nekoliko važnih datoteka:

* **places.sqlite**: Čuva istoriju, obeleživače i preuzimanja. Alati poput [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) na Windows-u mogu pristupiti podacima o istoriji.
* Koristite specifične SQL upite za ekstrakciju informacija o istoriji i preuzimanjima.
* **bookmarkbackups**: Sadrži rezervne kopije obeleživača.
* **formhistory.sqlite**: Čuva podatke iz veb obrazaca.
* **handlers.json**: Upravljanje protokolima.
* **persdict.dat**: Reči iz prilagođenog rečnika.
* **addons.json** i **extensions.sqlite**: Informacije o instaliranim dodacima i ekstenzijama.
* **cookies.sqlite**: Skladištenje kolačića, uz [MZCookiesView](https://www.nirsoft.net/utils/mzcv.html) dostupno za inspekciju na Windows-u.
* **cache2/entries** ili **startupCache**: Podaci o kešu, dostupni putem alata kao što je [MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html).
* **favicons.sqlite**: Čuva favikone.
* **prefs.js**: Korisničke postavke i preferencije.
* **downloads.sqlite**: Starija baza podataka preuzimanja, sada integrisana u places.sqlite.
* **thumbnails**: Sličice veb sajtova.
* **logins.json**: Enkriptovane prijavne informacije.
* **key4.db** ili **key3.db**: Čuva ključeve za enkripciju osetljivih informacija.

Pored toga, proveru podešavanja pretraživača za zaštitu od phishing-a možete izvršiti pretraživanjem `browser.safebrowsing` unosa u `prefs.js`, što ukazuje na to da li su funkcije bezbednog pretraživanja omogućene ili onemogućene.

Da biste pokušali da dekriptujete glavnu lozinku, možete koristiti [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt)\
Sa sledećim skriptom i pozivom možete odrediti datoteku lozinki za brute force:

{% code title="brute.sh" %}
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
{% endcode %}

![](<../../../.gitbook/assets/image (417).png>)

## Google Chrome

Google Chrome čuva korisničke profile na specifičnim lokacijama u zavisnosti od operativnog sistema:

* **Linux**: `~/.config/google-chrome/`
* **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
* **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

Unutar ovih direktorijuma, većina korisničkih podataka može se naći u **Default/** ili **ChromeDefaultData/** folderima. Sledeće datoteke sadrže značajne podatke:

* **History**: Sadrži URL-ove, preuzimanja i ključne reči za pretragu. Na Windows-u, [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) može se koristiti za čitanje istorije. Kolona "Transition Type" ima različita značenja, uključujući klikove korisnika na linkove, otkucane URL-ove, slanje obrazaca i ponovna učitavanja stranica.
* **Cookies**: Čuva kolačiće. Za inspekciju, dostupna je [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html).
* **Cache**: Drži keširane podatke. Da bi se izvršila inspekcija, korisnici Windows-a mogu koristiti [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html).
* **Bookmarks**: Korisnički obeleživači.
* **Web Data**: Sadrži istoriju obrazaca.
* **Favicons**: Čuva favicon-e sajtova.
* **Login Data**: Uključuje podatke za prijavu kao što su korisnička imena i lozinke.
* **Current Session**/**Current Tabs**: Podaci o trenutnoj sesiji pretraživanja i otvorenim karticama.
* **Last Session**/**Last Tabs**: Informacije o sajtovima aktivnim tokom poslednje sesije pre nego što je Chrome zatvoren.
* **Extensions**: Direktorijumi za ekstenzije i dodatke pretraživača.
* **Thumbnails**: Čuva sličice sajtova.
* **Preferences**: Datoteka bogata informacijama, uključujući podešavanja za dodatke, ekstenzije, iskačuće prozore, obaveštenja i još mnogo toga.
* **Browser’s built-in anti-phishing**: Da biste proverili da li su zaštita od phishing-a i zaštita od malvera omogućene, pokrenite `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`. Potražite `{"enabled: true,"}` u izlazu.

## **SQLite DB Data Recovery**

Kao što možete primetiti u prethodnim sekcijama, i Chrome i Firefox koriste **SQLite** baze podataka za čuvanje podataka. Moguće je **oporaviti obrisane unose koristeći alat** [**sqlparse**](https://github.com/padfoot999/sqlparse) **ili** [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases).

## **Internet Explorer 11**

Internet Explorer 11 upravlja svojim podacima i metapodacima na različitim lokacijama, pomažući u razdvajanju sačuvanih informacija i njihovih odgovarajućih detalja za lak pristup i upravljanje.

### Metadata Storage

Metapodaci za Internet Explorer čuvaju se u `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` (gde je VX V01, V16 ili V24). Pored toga, datoteka `V01.log` može pokazati razlike u vremenu modifikacije sa `WebcacheVX.data`, što ukazuje na potrebu za popravkom koristeći `esentutl /r V01 /d`. Ovi metapodaci, smešteni u ESE bazi podataka, mogu se oporaviti i inspekciji pomoću alata kao što su photorec i [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html). Unutar tabele **Containers**, može se uočiti specifične tabele ili kontejneri gde je svaki segment podataka smešten, uključujući detalje o kešu za druge Microsoft alate kao što je Skype.

### Cache Inspection

Alat [IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) omogućava inspekciju keša, zahtevajući lokaciju foldera za ekstrakciju podataka iz keša. Metapodaci za keš uključuju ime datoteke, direktorijum, broj pristupa, URL izvor i vremenske oznake koje označavaju vreme kreiranja, pristupa, modifikacije i isteka keša.

### Cookies Management

Kolačići se mogu istraživati koristeći [IECookiesView](https://www.nirsoft.net/utils/iecookies.html), sa metapodacima koji obuhvataju imena, URL-ove, brojeve pristupa i razne vremenske detalje. Trajni kolačići se čuvaju u `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies`, dok se sesijski kolačići nalaze u memoriji.

### Download Details

Metapodaci o preuzimanjima su dostupni putem [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html), sa specifičnim kontejnerima koji sadrže podatke kao što su URL, tip datoteke i lokacija preuzimanja. Fizičke datoteke se mogu naći pod `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory`.

### Browsing History

Da biste pregledali istoriju pretraživanja, može se koristiti [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html), zahtevajući lokaciju ekstraktovanih datoteka istorije i konfiguraciju za Internet Explorer. Metapodaci ovde uključuju vremena modifikacije i pristupa, zajedno sa brojevima pristupa. Datoteke istorije se nalaze u `%userprofile%\Appdata\Local\Microsoft\Windows\History`.

### Typed URLs

Otucani URL-ovi i njihova vremena korišćenja čuvaju se u registru pod `NTUSER.DAT` na `Software\Microsoft\InternetExplorer\TypedURLs` i `Software\Microsoft\InternetExplorer\TypedURLsTime`, prateći poslednjih 50 URL-ova koje je korisnik uneo i njihova poslednja vremena unosa.

## Microsoft Edge

Microsoft Edge čuva korisničke podatke u `%userprofile%\Appdata\Local\Packages`. Putanje za različite tipove podataka su:

* **Profile Path**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
* **History, Cookies, and Downloads**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
* **Settings, Bookmarks, and Reading List**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
* **Cache**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
* **Last Active Sessions**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Safari podaci se čuvaju na `/Users/$User/Library/Safari`. Ključne datoteke uključuju:

* **History.db**: Sadrži tabele `history_visits` i `history_items` sa URL-ovima i vremenskim oznakama poseta. Koristite `sqlite3` za upite.
* **Downloads.plist**: Informacije o preuzetim datotekama.
* **Bookmarks.plist**: Čuva obeležene URL-ove.
* **TopSites.plist**: Najčešće posećeni sajtovi.
* **Extensions.plist**: Lista ekstenzija za Safari pretraživač. Koristite `plutil` ili `pluginkit` za preuzimanje.
* **UserNotificationPermissions.plist**: Domeni kojima je dozvoljeno slanje obaveštenja. Koristite `plutil` za analizu.
* **LastSession.plist**: Kartice iz poslednje sesije. Koristite `plutil` za analizu.
* **Browser’s built-in anti-phishing**: Proverite koristeći `defaults read com.apple.Safari WarnAboutFraudulentWebsites`. Odgovor 1 označava da je funkcija aktivna.

## Opera

Opera podaci se nalaze u `/Users/$USER/Library/Application Support/com.operasoftware.Opera` i deli format Chrome-a za istoriju i preuzimanja.

* **Browser’s built-in anti-phishing**: Proverite tako što ćete proveriti da li je `fraud_protection_enabled` u datoteci Preferences postavljeno na `true` koristeći `grep`.

Ove putanje i komande su ključne za pristup i razumevanje podataka o pretraživanju koje čuvaju različiti web pretraživači.

## References

* [https://nasbench.medium.com/web-browsers-forensics-7e99940c579a](https://nasbench.medium.com/web-browsers-forensics-7e99940c579a)
* [https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/](https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/)
* [https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file](https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file)
* **Book: OS X Incident Response: Scripting and Analysis By Jaron Bradley pag 123**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

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
