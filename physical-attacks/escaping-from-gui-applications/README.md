# Proverite moguće akcije unutar GUI aplikacije

**Zajednički dijalozi** su opcije za **čuvanje datoteke**, **otvaranje datoteke**, izbor fonta, boje... Većina njih će **ponuditi punu funkcionalnost Explorera**. To znači da ćete moći pristupiti funkcionalnostima Explorera ako možete pristupiti ovim opcijama:

* Zatvori/Zatvori kao
* Otvori/Otvori sa
* Štampaj
* Izvezi/Uvezi
* Pretraži
* Skeniraj

Trebalo bi da proverite da li možete:

* Modifikovati ili kreirati nove datoteke
* Kreirati simboličke linkove
* Dobiti pristup ograničenim oblastima
* Izvršiti druge aplikacije

## Izvršavanje komandi

Možda **koristeći opciju `Otvori sa`** možete otvoriti/izvršiti neku vrstu shell-a.

### Windows

Na primer _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ pronađite više binarnih datoteka koje se mogu koristiti za izvršavanje komandi (i obavljanje neočekivanih akcija) ovde: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

### \*NIX __

_bash, sh, zsh..._ Više ovde: [https://gtfobins.github.io/](https://gtfobins.github.io)

# Windows

## Zaobilaženje putanjskih ograničenja

* **Promenljive okruženja**: Postoji mnogo promenljivih okruženja koje upućuju na neku putanju
* **Ostali protokoli**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Simbolički linkovi**
* **Prečice**: CTRL+N (otvori novu sesiju), CTRL+R (izvrši komande), CTRL+SHIFT+ESC (Upravnik zadataka), Windows+E (otvori explorer), CTRL-B, CTRL-I (Omiljeni), CTRL-H (Istorija), CTRL-L, CTRL-O (Datoteka/Otvori dijalog), CTRL-P (Štampanje dijalog), CTRL-S (Sačuvaj kao)
* Skriveni Administrativni meni: CTRL-ALT-F8, CTRL-ESC-F9
* **Shell URIs**: _shell:Administrativni alati, shell:Biblioteka dokumenata, shell:Biblioteke, shell:Profil korisnika, shell:Lični, shell:Pretraži domaću fasciklu, shell:Sistem, shell:Mrežne fascikle, shell:Pošalji, shell:Profili korisnika, shell:Zajednički administrativni alati, shell:Moj računar, shell:Internet fascikla_
* **UNC putanje**: Putanje za povezivanje sa deljenim fasciklama. Trebalo bi da pokušate da se povežete na C$ lokalne mašine ("\\\127.0.0.1\c$\Windows\System32")
* **Više UNC putanja:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

## Preuzmite svoje binarne datoteke

Konzola: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Explorer: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Registry editor: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

## Pristupanje datotečnom sistemu iz pregledača

| PUTANJA              | PUTANJA            | PUTANJA             | PUTANJA              |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

## Prečice

* Sticky Keys – Pritisnite SHIFT 5 puta
* Mouse Keys – SHIFT+ALT+NUMLOCK
* High Contrast – SHIFT+ALT+PRINTSCN
* Toggle Keys – Držite NUMLOCK 5 sekundi
* Filter Keys – Držite desni SHIFT 12 sekundi
* WINDOWS+F1 – Windows pretraga
* WINDOWS+D – Prikaži radnu površinu
* WINDOWS+E – Pokreni Windows Explorer
* WINDOWS+R – Pokreni
* WINDOWS+U – Centar za pristupačnost
* WINDOWS+F – Pretraži
* SHIFT+F10 – Kontekstualni meni
* CTRL+SHIFT+ESC – Upravnik zadataka
* CTRL+ALT+DEL – Splash ekran na novijim verzijama Windows-a
* F1 – Pomoć F3 – Pretraži
* F6 – Adresna traka
* F11 – Prebaci u pun ekran unutar Internet Explorera
* CTRL+H – Istorija Internet Explorera
* CTRL+T – Internet Explorer – Nova kartica
* CTRL+N – Internet Explorer – Nova stranica
* CTRL+O – Otvori datoteku
* CTRL+S – Sačuvaj CTRL+N – Nova RDP / Citrix

## Swipe

* Prevucite s leve strane na desnu da biste videli sve otvorene Windows, minimizirajući KIOSK aplikaciju i direktno pristupajući celom OS-u;
* Prevucite s desne strane na levu da biste otvorili Centar za akcije, minimizirajući KIOSK aplikaciju i direktno pristupajući celom OS-u;
* Prevucite od gornjeg ruba da biste učinili naslovnu traku vidljivom za aplikaciju otvorenu u režimu punog ekrana;
* Prevucite nagore od dna da biste prikazali traku zadataka u aplikaciji punog ekrana.

## Internet Explorer trikovi

### 'Image Toolbar'

To je alatna traka koja se pojavljuje u gornjem levom uglu slike kada se klikne. Moći ćete da Sačuvate, Štampate, Pošaljete e-poštu, Otvorite "Moje slike" u Exploreru. Kiosk treba da koristi Internet Explorer.

### Shell protokol

Ukucajte ove URL-ove da biste dobili prikaz Explorera:

* `shell:Administrativni alati`
* `shell:Biblioteka dokumenata`
* `shell:Biblioteke`
* `shell:Profil korisnika`
* `shell:Lični`
* `shell:Pretraži domaću fasciklu`
* `shell:Mrežne fascikle`
* `shell:Pošalji`
* `shell:Profili korisnika`
* `shell:Zajednički administrativni alati`
* `shell:Moj računar`
* `shell:Internet fascikla`
* `Shell:Profil`
* `Shell:ProgramFiles`
* `Shell:Sistem`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Kontrolna tabla
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Moj računar
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Moja mrežna mesta
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

## Prikaži ekstenzije datoteka

Proverite ovu stranicu za više informacija: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

# Trikovi za pretraživače

Backup iKat verzije:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\

Kreirajte zajednički dijalog koristeći JavaScript i pristupite file explorer-u: `document.write('<input/type=file>')`
Izvor: https://medium.com/@Rend_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

# iPad

## Gestikulacije i dugmad

* Prevucite nagore sa četiri (ili pet) prsta / Dvostruki dodir na dugme Home: Da biste videli prikaz multitaskinga i promenili aplikaciju

* Prevucite na jednu ili drugu stranu sa četiri ili pet prsta: Da biste prešli na sledeću/poslednju aplikaciju

* Stisnite ekran sa pet prstiju / Dodirnite dugme Home / Prevucite nagore sa 1 prstom sa dna ekrana u brzom pokretu: Da biste pristupili Home

* Prevucite jednim prstom sa dna ekrana samo 1-2 inča (sporo): Dock će se pojaviti

* Prevucite nagore sa gornjeg dela ekrana sa 1 prstom: Da biste videli obaveštenja

* Prevucite nagore sa 1 prstom u gornjem desnom uglu ekrana: Da biste videli kontrolni centar iPad Pro-a

* Prevucite 1 prstom sa leve strane ekrana 1-2 inča: Da biste videli prikaz dana

* Brzo prevucite 1 prstom iz centra ekrana na desno ili levo: Da biste prešli na sledeću/poslednju aplikaciju

* Pritisnite i držite dugme On/**Off**/Sleep u gornjem desnom uglu **iPad-a +** Pomaknite klizač za **isključivanje** skroz udesno: Da biste isključili

* Pritisnite dugme On/**Off**/Sleep u gornjem desnom uglu **iPad-a i dugme Home nekoliko sekundi**: Da biste prisilili teško isključivanje

* Pritisnite dugme On/**Off**/Sleep u gornjem desnom uglu **iPad-a i dugme Home brzo**: Da biste napravili snimak ekrana koji će se pojaviti u donjem levom uglu ekrana. Pritisnite oba dugmeta u isto vreme vrlo kratko, jer ako ih držite nekoliko sekundi, izvršiće se teško isključivanje.

## Prečice

Trebalo bi da imate iPad tastaturu ili USB adapter za tastaturu. Samo prečice koje bi mogle pomoći u izlasku iz aplikacije biće prikazane ovde.

| Taster | Ime         |
| --- | ------------ |
| ⌘   | Komanda      |
| ⌥   | Opcija (Alt) |
| ⇧   | Shift        |
| ↩   | Povratak       |
| ⇥   | Tab          |
| ^   | Kontrola      |
| ←   | Leva strelica   |
| →   | Desna strelica  |
| ↑   | Gornja strelica     |
| ↓   | Donja strelica   |

### Sistem prečice

Ove prečice su za vizuelne postavke i postavke zvuka, u zavisnosti od korišćenja iPad-a.

| Prečica | Akcija                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Smanji ekran                                                                    |
| F2       | Povećaj ekran                                                                |
| F7       | Vratite se na prethodnu pesmu                                                                  |
| F8       | Pusti/pausa                                                                     |
| F9       | Preskoči pesmu                                                                      |
| F10      | Isključi                                                                           |
| F11      | Smanji jačinu zvuka                                                                |
| F12      | Povećaj jačinu zvuka                                                                |
| ⌘ Space  | Prikaži listu dostupnih jezika; da biste izabrali jedan, ponovo dodirnite razmaknicu. |

### Navigacija iPad-a

| Prečica                                           | Akcija                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Idi na Home                                              |
| ⌘⇧H (Command-Shift-H)                              | Idi na Home                                              |
| ⌘ (Space)                                          | Otvori Spotlight                                          |
| ⌘⇥ (Command-Tab)                                   | Lista poslednjih deset korišćenih aplikacija                                 |
| ⌘\~                                                | Idi na poslednju aplikaciju                                       |
| ⌘⇧3 (Command-Shift-3)                              | Snimak ekrana (pluta u donjem levom uglu da sačuva ili deluje na njega) |
| ⌘⇧4                                                | Snimak ekrana i otvori ga u editoru                    |
| Pritisnite i držite ⌘                                   | Lista prečica dostupnih za aplikaciju                 |
| ⌘⌥D (Command-Option/Alt-D)                         | Prikazuje dock                                      |
| ^⌥H (Control-Option-H)                             | Dugme Home                                             |
| ^⌥H H (Control-Option-H-H)                         | Prikaži traku multitaskinga                                      |
| ^⌥I (Control-Option-i)                             | Izbor stavke                                            |
| Escape                                             | Dugme nazad                                             |
| → (Desna strelica)                                    | Sledeća stavka                                               |
| ← (Leva strelica)                                     | Prethodna stavka                                           |
| ↑↓ (Gornja strelica, Donja strelica)                          | Istovremeno dodirnite izabranu stavku                        |
| ⌥ ↓ (Opcija-Donja strelica)                            | Pomeri se nadole                                             |
| ⌥↑ (Opcija-Gornja strelica)                               | Pomeri se nagore                                               |
| ⌥← ili ⌥→ (Opcija-Leva strelica ili Opcija-Desna strelica) | Pomeri se levo ili desno                                    |
| ^⌥S (Control-Option-S)                             | Uključi ili isključi VoiceOver govor                         |
| ⌘⇧⇥ (Command-Shift-Tab)                            | Prebaci se na prethodnu aplikaciju                              |
| ⌘⇥ (Command-Tab)                                   | Prebaci se nazad na originalnu aplikaciju                         |
| ←+→, zatim Opcija + ← ili Opcija+→                   | Navigirajte kroz Dock                                   |

### Safari prečice

| Prečica                | Akcija                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Command-L)          | Otvori lokaciju                                    |
| ⌘T                      | Otvori novu karticu                                   |
| ⌘W                      | Zatvori trenutnu karticu                            |
| ⌘R                      | Osveži trenutnu karticu                          |
| ⌘.                      | Prekini učitavanje trenutne kartice                     |
| ^⇥                      | Prebaci se na sledeću karticu                           |
| ^⇧⇥ (Control-Shift-Tab) | Prebaci se na prethodnu karticu                         |
| ⌘L                      | Izaberi tekstualni unos/URL polje da ga izmeniš     |
| ⌘⇧T (Command-Shift-T)   | Otvori poslednju zatvorenu karticu (može se koristiti više puta) |
| ⌘\[                     | Vraća se na prethodnu stranicu u tvojoj istoriji pretraživanja      |
| ⌘]                      | Ide napred na sledeću stranicu u tvojoj istoriji pretraživanja   |
| ⌘⇧R                     | Aktiviraj režim čitača                             |

### Mail prečice

| Prečica                   | Akcija                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | Otvori lokaciju                |
| ⌘T                         | Otvori novu karticu               |
| ⌘W                         | Zatvori trenutnu karticu        |
| ⌘R                         | Osveži trenutnu karticu      |
| ⌘.                         | Prekini učitavanje trenutne kartice |
| ⌘⌥F (Command-Option/Alt-F) | Pretraži u tvojoj pošti       |

# Reference

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)
