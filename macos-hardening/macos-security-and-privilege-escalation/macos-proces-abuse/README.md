# macOS Proses Misbruik

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

## Prosesse Basiese Inligting

'n Proses is 'n instansie van 'n lopende uitvoerbare, egter prosesse voer nie kode uit nie, dit is drade. Daarom **is prosesse net houers vir lopende drade** wat die geheue, beskrywings, poorte, toestemmings...

Tradisioneel is prosesse binne ander prosesse (behalwe PID 1) begin deur **`fork`** aan te roep wat 'n presiese kopie van die huidige proses sou skep en dan sou die **kind proses** gewoonlik **`execve`** aanroep om die nuwe uitvoerbare te laai en dit uit te voer. Toe is **`vfork`** bekendgestel om hierdie proses vinniger te maak sonder enige geheue kopieer.\
Toe is **`posix_spawn`** bekendgestel wat **`vfork`** en **`execve`** in een oproep kombineer en vlaggies aanvaar:

* `POSIX_SPAWN_RESETIDS`: Herstel effektiewe id's na werklike id's
* `POSIX_SPAWN_SETPGROUP`: Stel prosesgroep affiliasie
* `POSUX_SPAWN_SETSIGDEF`: Stel sein standaard gedrag
* `POSIX_SPAWN_SETSIGMASK`: Stel sein masker
* `POSIX_SPAWN_SETEXEC`: Exec in dieselfde proses (soos `execve` met meer opsies)
* `POSIX_SPAWN_START_SUSPENDED`: Begin gesuspend
* `_POSIX_SPAWN_DISABLE_ASLR`: Begin sonder ASLR
* `_POSIX_SPAWN_NANO_ALLOCATOR:` Gebruik libmalloc se Nano toewysings
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Laat `rwx` op data segmente toe
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: Sluit alle lêer beskrywings op exec(2) standaard
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` Randomiseer hoë bits van ASLR gly

Boonop laat `posix_spawn` toe om 'n reeks van **`posix_spawnattr`** te spesifiseer wat sommige aspekte van die gespaande proses beheer, en **`posix_spawn_file_actions`** om die toestand van die beskrywings te verander.

Wanneer 'n proses sterf, stuur dit die **terugkode na die ouer proses** (as die ouer gesterf het, is die nuwe ouer PID 1) met die sein `SIGCHLD`. Die ouer moet hierdie waarde kry deur `wait4()` of `waitid()` aan te roep en totdat dit gebeur, bly die kind in 'n zombie toestand waar dit steeds gelys word maar nie hulpbronne verbruik nie.

### PIDs

PIDs, prosesidentifiseerders, identifiseer 'n unieke proses. In XNU is die **PIDs** van **64bits** wat monotonies toeneem en **nooit wrap** nie (om misbruik te voorkom).

### Proses Groepe, Sessies & Koalisies

**Prosesse** kan in **groepe** ingevoeg word om dit makliker te maak om hulle te hanteer. Byvoorbeeld, opdragte in 'n shell script sal in dieselfde prosesgroep wees sodat dit moontlik is om **hulle saam te sein** deur kill te gebruik byvoorbeeld.\
Dit is ook moontlik om **prosesse in sessies te groepeer**. Wanneer 'n proses 'n sessie begin (`setsid(2)`), word die kind prosesse binne die sessie gestel, tensy hulle hul eie sessie begin.

Koalisie is 'n ander manier om prosesse in Darwin te groepeer. 'n Proses wat by 'n koalisie aansluit, laat dit toe om poel hulpbronne te benader, 'n grootboek te deel of Jetsam te ondervind. Koalisies het verskillende rolle: Leier, XPC diens, Uitbreiding.

### Kredensiale & Persone

Elke proses hou **kredensiale** wat **sy voorregte** in die stelsel identifiseer. Elke proses sal een primêre `uid` en een primêre `gid` hê (alhoewel dit aan verskeie groepe mag behoort).\
Dit is ook moontlik om die gebruiker en groep id te verander as die binêre die `setuid/setgid` bit het.\
Daar is verskeie funksies om **nuwe uids/gidses in te stel**.

Die syscall **`persona`** bied 'n **alternatiewe** stel van **kredensiale**. Om 'n persona aan te neem, neem dit sy uid, gid en groep lidmaatskappe **tegelyk** aan. In die [**bron kode**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h) is dit moontlik om die struktuur te vind:
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## Draad Basiese Inligting

1. **POSIX Draad (pthreads):** macOS ondersteun POSIX drade (`pthreads`), wat deel is van 'n standaard threading API vir C/C++. Die implementering van pthreads in macOS is te vind in `/usr/lib/system/libsystem_pthread.dylib`, wat afkomstig is van die publiek beskikbare `libpthread` projek. Hierdie biblioteek bied die nodige funksies om drade te skep en te bestuur.
2. **Drade Skep:** Die `pthread_create()` funksie word gebruik om nuwe drade te skep. Intern roep hierdie funksie `bsdthread_create()` aan, wat 'n laer vlak stelselaanroep is wat spesifiek is vir die XNU-kern (die kern waarop macOS gebaseer is). Hierdie stelselaanroep neem verskeie vlae afgeleide van `pthread_attr` (kenmerke) wat die gedrag van die draad spesifiseer, insluitend skeduleringbeleide en stapelgrootte.
* **Standaard Stapelgrootte:** Die standaard stapelgrootte vir nuwe drade is 512 KB, wat voldoende is vir tipiese operasies, maar kan aangepas word via draadkenmerke as meer of minder ruimte benodig word.
3. **Draad Inisialiserings:** Die `__pthread_init()` funksie is van kardinale belang tydens draadopstelling, wat die `env[]` argument gebruik om omgewing veranderlikes te ontleed wat besonderhede oor die stapel se ligging en grootte kan insluit.

#### Draad Beëindiging in macOS

1. **Drade Verlaat:** Drade word tipies beëindig deur `pthread_exit()` aan te roep. Hierdie funksie laat 'n draad toe om skoon te verlaat, die nodige opruiming te doen en die draad toe te laat om 'n terugkeerwaarde aan enige aansluiters te stuur.
2. **Draad Opruiming:** By die aanroep van `pthread_exit()`, word die funksie `pthread_terminate()` aangeroep, wat die verwydering van alle geassosieerde draadstrukture hanteer. Dit deallokeer Mach draadpoorte (Mach is die kommunikasiesubstelsel in die XNU-kern) en roep `bsdthread_terminate` aan, 'n stelselaanroep wat die kernvlak strukture geassosieer met die draad verwyder.

#### Sinchronisasie Meganismes

Om toegang tot gedeelde hulpbronne te bestuur en wedren toestande te vermy, bied macOS verskeie sinchronisasie primitiewe. Hierdie is krities in multi-draad omgewings om data integriteit en stelsels stabiliteit te verseker:

1. **Mutexes:**
* **Reguliere Mutex (Handtekening: 0x4D555458):** Standaard mutex met 'n geheue voetspoor van 60 bytes (56 bytes vir die mutex en 4 bytes vir die handtekening).
* **Vinnige Mutex (Handtekening: 0x4d55545A):** Soortgelyk aan 'n regulêre mutex maar geoptimaliseer vir vinniger operasies, ook 60 bytes in grootte.
2. **Toestand Veranderlikes:**
* Gebruik vir wag op sekere toestande om te gebeur, met 'n grootte van 44 bytes (40 bytes plus 'n 4-byte handtekening).
* **Toestand Veranderlike Kenmerke (Handtekening: 0x434e4441):** Konfigurasiekenmerke vir toestand veranderlikes, grootte van 12 bytes.
3. **Eens Veranderlike (Handtekening: 0x4f4e4345):**
* Verseker dat 'n stuk inisialiseringskode slegs een keer uitgevoer word. Die grootte is 12 bytes.
4. **Lees-Skryf Slotte:**
* Laat verskeie lesers of een skrywer op 'n slag toe, wat doeltreffende toegang tot gedeelde data fasiliteer.
* **Lees Skryf Slot (Handtekening: 0x52574c4b):** Grootte van 196 bytes.
* **Lees Skryf Slot Kenmerke (Handtekening: 0x52574c41):** Kenmerke vir lees-skryf slotte, 20 bytes in grootte.

{% hint style="success" %}
Die laaste 4 bytes van daardie objekte word gebruik om oorgange te detecteer.
{% endhint %}

### Draad Plaaslike Veranderlikes (TLV)

**Draad Plaaslike Veranderlikes (TLV)** in die konteks van Mach-O lêers (die formaat vir uitvoerbare lêers in macOS) word gebruik om veranderlikes te verklaar wat spesifiek is vir **elke draad** in 'n multi-draad toepassing. Dit verseker dat elke draad sy eie aparte instansie van 'n veranderlike het, wat 'n manier bied om konflikte te vermy en data integriteit te handhaaf sonder om eksplisiete sinchronisasie meganismes soos mutexes te benodig.

In C en verwante tale kan jy 'n draad-lokale veranderlike verklaar met die **`__thread`** sleutelwoord. Hier is hoe dit in jou voorbeeld werk:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
```markdown
Hierdie snit definieer `tlv_var` as 'n draad-lokale veranderlike. Elke draad wat hierdie kode uitvoer, sal sy eie `tlv_var` hê, en veranderinge wat een draad aan `tlv_var` maak, sal nie `tlv_var` in 'n ander draad beïnvloed nie.

In die Mach-O binêre, is die data wat verband hou met draad-lokale veranderlikes georganiseer in spesifieke afdelings:

* **`__DATA.__thread_vars`**: Hierdie afdeling bevat die metadata oor die draad-lokale veranderlikes, soos hul tipes en inisialisasiestatus.
* **`__DATA.__thread_bss`**: Hierdie afdeling word gebruik vir draad-lokale veranderlikes wat nie eksplisiet geinisialiseer is nie. Dit is 'n deel van geheue wat gereserveer is vir nul-geinisialiseerde data.

Mach-O bied ook 'n spesifieke API genaamd **`tlv_atexit`** om draad-lokale veranderlikes te bestuur wanneer 'n draad verlaat. Hierdie API laat jou toe om **destruktors** te **registreer**—spesiale funksies wat draad-lokale data skoonmaak wanneer 'n draad beëindig.

### Draad Prioriteite

Om draadprioriteite te verstaan, behels dit om te kyk hoe die bedryfstelsel besluit watter drade om te laat loop en wanneer. Hierdie besluit word beïnvloed deur die prioriteitsvlak wat aan elke draad toegeken word. In macOS en Unix-agtige stelsels, word dit hanteer met konsepte soos `nice`, `renice`, en Kwaliteit van Diens (QoS) klasse.

#### Nice en Renice

1. **Nice:**
* Die `nice` waarde van 'n proses is 'n nommer wat sy prioriteit beïnvloed. Elke proses het 'n nice waarde wat wissel van -20 (die hoogste prioriteit) tot 19 (die laagste prioriteit). Die standaard nice waarde wanneer 'n proses geskep word, is tipies 0.
* 'n Laer nice waarde (nader aan -20) maak 'n proses meer "selfsugtig," wat dit meer CPU tyd gee in vergelyking met ander prosesse met hoër nice waardes.
2. **Renice:**
* `renice` is 'n opdrag wat gebruik word om die nice waarde van 'n reeds lopende proses te verander. Dit kan gebruik word om die prioriteit van prosesse dinamies aan te pas, hetsy om hul CPU tyd toewysing te verhoog of te verlaag op grond van nuwe nice waardes.
* Byvoorbeeld, as 'n proses tydelik meer CPU hulpbronne benodig, kan jy sy nice waarde verlaag met `renice`.

#### Kwaliteit van Diens (QoS) Klasse

QoS klasse is 'n meer moderne benadering tot die hantering van draadprioriteite, veral in stelsels soos macOS wat **Grand Central Dispatch (GCD)** ondersteun. QoS klasse laat ontwikkelaars toe om werk in verskillende vlakke te **kategoriseer** op grond van hul belangrikheid of dringendheid. macOS bestuur draadprioritisering outomaties op grond van hierdie QoS klasse:

1. **Gebruiker Interaktief:**
* Hierdie klas is vir take wat tans met die gebruiker interaksie het of onmiddellike resultate benodig om 'n goeie gebruikerservaring te bied. Hierdie take word die hoogste prioriteit gegee om die koppelvlak responsief te hou (bv. animasies of gebeurtenis hantering).
2. **Gebruiker Inisieer:**
* Take wat die gebruiker inisieer en onmiddellike resultate verwag, soos om 'n dokument te open of op 'n knoppie te klik wat berekeninge benodig. Hierdie is hoë prioriteit maar onder gebruiker interaktief.
3. **Nuttig:**
* Hierdie take is langlopende en toon tipies 'n vordering aanduiding (bv. lêers aflaai, data invoer). Hulle is laer in prioriteit as gebruiker-inisieer take en hoef nie onmiddellik te voltooi nie.
4. **Agtergrond:**
* Hierdie klas is vir take wat in die agtergrond werk en nie sigbaar is vir die gebruiker nie. Dit kan take wees soos indeksering, sinkronisering, of rugsteun. Hulle het die laagste prioriteit en minimale impak op stelsels se prestasie.

Deur QoS klasse te gebruik, hoef ontwikkelaars nie die presiese prioriteitsnommers te bestuur nie, maar eerder te fokus op die aard van die taak, en die stelsel optimaliseer die CPU hulpbronne dienooreenkomstig.

Boonop is daar verskillende **draad skedulering beleid** wat vloei om 'n stel skeduleringsparameters te spesifiseer wat die skeduleerder in ag sal neem. Dit kan gedoen word met `thread_policy_[set/get]`. Dit kan nuttig wees in wedlooptoestand aanvalle.

## MacOS Proses Misbruik

MacOS, soos enige ander bedryfstelsel, bied 'n verskeidenheid metodes en meganismes vir **prosesse om te interaksie, kommunikeer, en data te deel**. Terwyl hierdie tegnieke noodsaaklik is vir doeltreffende stelselfunksionering, kan dit ook misbruik word deur bedreigingsakteurs om **kwaadwillige aktiwiteite** uit te voer.

### Biblioteek Inspuiting

Biblioteek Inspuiting is 'n tegniek waarin 'n aanvaller **'n proses dwing om 'n kwaadwillige biblioteek te laai**. Sodra dit ingespuit is, loop die biblioteek in die konteks van die teiken proses, wat die aanvaller dieselfde toestemmings en toegang gee as die proses.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Funksie Haak

Funksie Haak behels **om funksie-oproepe** of boodskappe binne 'n sagtewarekode te onderskep. Deur funksies te haak, kan 'n aanvaller die **gedrag** van 'n proses **wysig**, sensitiewe data waarneem, of selfs beheer oor die uitvoeringsvloei verkry.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Inter Proses Kommunikasie

Inter Proses Kommunikasie (IPC) verwys na verskillende metodes waardeur aparte prosesse **data deel en uitruil**. Terwyl IPC fundamenteel is vir baie wettige toepassings, kan dit ook misbruik word om prosesisolering te ondermyn, sensitiewe inligting te lek, of ongeoorloofde aksies uit te voer.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Electron Toepassings Inspuiting

Electron toepassings wat met spesifieke omgewing veranderlikes uitgevoer word, kan kwesbaar wees vir proses inspuiting:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Chromium Inspuiting

Dit is moontlik om die vlae `--load-extension` en `--use-fake-ui-for-media-stream` te gebruik om 'n **man in the browser aanval** uit te voer wat dit moontlik maak om toetsaanslagen, verkeer, koekies te steel, skripte in bladsye in te spuit...:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Dirty NIB

NIB lêers **definieer gebruikerskoppelvlak (UI) elemente** en hul interaksies binne 'n toepassing. Dit kan egter **arbitraire opdragte uitvoer** en **Gatekeeper stop nie** 'n reeds uitgevoerde toepassing om uitgevoer te word as 'n **NIB lêer gewysig** word nie. Daarom kan dit gebruik word om arbitraire programme arbitraire opdragte te laat uitvoer:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Java Toepassings Inspuiting

Dit is moontlik om sekere java vermoëns (soos die **`_JAVA_OPTS`** omgewing veranderlike) te misbruik om 'n java toepassing **arbitraire kode/opdragte** uit te voer.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Net Toepassings Inspuiting

Dit is moontlik om kode in .Net toepassings in te spuit deur **die .Net debuggingsfunksionaliteit te misbruik** (nie beskerm deur macOS beskermings soos runtime hardening nie).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Perl Inspuiting

Kyk na verskillende opsies om 'n Perl skrip arbitraire kode te laat uitvoer in:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Ruby Inspuiting

Dit is ook moontlik om ruby omgewing veranderlikes te misbruik om arbitraire skripte arbitraire kode te laat uitvoer:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Python Inspuiting

As die omgewing veranderlike **`PYTHONINSPECT`** gestel is, sal die python proses in 'n python cli val sodra dit klaar is. Dit is ook moontlik om **`PYTHONSTARTUP`** te gebruik om 'n python skrip aan te dui wat aan die begin van 'n interaktiewe sessie uitgevoer moet word.\
Let egter daarop dat die **`PYTHONSTARTUP`** skrip nie uitgevoer sal word wanneer **`PYTHONINSPECT`** die interaktiewe sessie skep nie.

Ander omgewing veranderlikes soos **`PYTHONPATH`** en **`PYTHONHOME`** kan ook nuttig wees om 'n python opdrag arbitraire kode te laat uitvoer.

Let daarop dat uitvoerbare lêers wat met **`pyinstaller`** gecompileer is, nie hierdie omgewingsveranderlikes sal gebruik nie, selfs al loop hulle met 'n ingebedde python.

{% hint style="danger" %}
Oor die algemeen kon ek nie 'n manier vind om python arbitraire kode uit te voer deur omgewing veranderlikes te misbruik nie.\
Echter, die meeste mense installeer python met **Hombrew**, wat python in 'n **skryfbare ligging** vir die standaard admin gebruiker sal installeer. Jy kan dit oorneem met iets soos:
```
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Even **root** sal hierdie kode uitvoer wanneer python uitgevoer word.
{% endhint %}

## Opsporing

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) is 'n oopbron toepassing wat **prosesinjeksie** aksies kan **opspoor en blokkeer**:

* Gebruik **Omgewingsveranderlikes**: Dit sal die teenwoordigheid van enige van die volgende omgewingsveranderlikes monitor: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** en **`ELECTRON_RUN_AS_NODE`**
* Gebruik **`task_for_pid`** oproepe: Om te vind wanneer een proses die **taakpoort van 'n ander** wil kry wat toelaat om kode in die proses in te spuit.
* **Electron-apps parameters**: Iemand kan **`--inspect`**, **`--inspect-brk`** en **`--remote-debugging-port`** opdraglynargumente gebruik om 'n Electron-app in foutopsporingmodus te begin, en so kode daarin in te spuit.
* Gebruik **simboliese skakels** of **hardlinks**: Tipies is die mees algemene misbruik om **'n skakel met ons gebruikersregte te plaas**, en **dit na 'n hoër regte** plek te wys. Die opsporing is baie eenvoudig vir beide hardlink en simboliese skakels. As die proses wat die skakel skep 'n **ander regtevlak** het as die teikenlêer, skep ons 'n **waarskuwing**. Ongelukkig is dit in die geval van simboliese skakels nie moontlik om te blokkeer nie, aangesien ons nie inligting oor die bestemming van die skakel voor die skepping het nie. Dit is 'n beperking van Apple se EndpointSecurity-raamwerk.

### Oproepe gemaak deur ander prosesse

In [**hierdie blogpos**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) kan jy vind hoe dit moontlik is om die funksie **`task_name_for_pid`** te gebruik om inligting oor ander **prosesse wat kode in 'n proses inspuit** te verkry en dan inligting oor daardie ander proses te kry.

Let daarop dat om daardie funksie aan te roep, jy moet **die selfde uid** wees as die een wat die proses uitvoer of **root** (en dit keer inligting oor die proses terug, nie 'n manier om kode in te spuit nie).

## Verwysings

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsieplanne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PR's in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
