# macOS MDM

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PR's in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

**Om meer te leer oor macOS MDM's, kyk:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Basiese beginsels

### **MDM (Mobiele Toestelbestuur) Oorsig**

[Mobiele Toestelbestuur](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) word gebruik om verskeie eindgebruikertoestelle soos slimfone, skootrekenaars en tablette te bestuur. Veral vir Apple se platforms (iOS, macOS, tvOS), dit behels 'n stel gespesialiseerde funksies, API's en praktyke. Die werking van MDM hang af van 'n versoenbare MDM-bediener, wat kommersieel beskikbaar of oopbron kan wees, en moet die [MDM-protokol](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf) ondersteun. Sleutelpunte sluit in:

* Gekonsolideerde beheer oor toestelle.
* Afhangend van 'n MDM-bediener wat aan die MDM-protokol voldoen.
* Vermoë van die MDM-bediener om verskeie opdragte na toestelle te stuur, byvoorbeeld, afstandsdata-uitwissing of konfigurasie-installasie.

### **Basiese beginsels van DEP (Toestelregistrasieprogram)**

Die [Toestelregistrasieprogram](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) wat deur Apple aangebied word, stroomlyn die integrasie van Mobiele Toestelbestuur (MDM) deur nul-aanraaking konfigurasie vir iOS, macOS en tvOS toestelle te fasiliteer. DEP outomatiseer die registrasieproses, wat toestelle in staat stel om reg uit die boks te werk, met minimale gebruikers- of administratiewe ingryping. Belangrike aspekte sluit in:

* Stel toestelle in staat om outonoom te registreer met 'n vooraf gedefinieerde MDM-bediener by die aanvanklike aktivering.
* Primêr voordelig vir splinternuwe toestelle, maar ook toepaslik vir toestelle wat herkonfigureer word.
* Fasiliteer 'n eenvoudige opstelling, wat toestelle vinnig gereed maak vir organisatoriese gebruik.

### **Sekuriteitsoorweging**

Dit is belangrik om daarop te let dat die gemak van registrasie wat deur DEP verskaf word, terwyl dit voordelig is, ook sekuriteitsrisiko's kan inhou. As beskermingsmaatreëls nie voldoende afgedwing word vir MDM-registrasie nie, kan aanvallers hierdie gestroomlynde proses benut om hul toestel op die organisasie se MDM-bediener te registreer, terwyl hulle as 'n korporatiewe toestel voorgee.

{% hint style="danger" %}
**Sekuriteitswaarskuwing**: Vereenvoudigde DEP-registrasie kan moontlik ongeoorloofde toestelregistrasie op die organisasie se MDM-bediener toelaat as behoorlike beskermingsmaatreëls nie in plek is nie.
{% endhint %}

### Basiese beginsels Wat is SCEP (Eenvoudige Sertifikaatregistrasieprotokol)?

* 'n Relatief ou protokol, geskep voordat TLS en HTTPS algemeen was.
* Gee kliënte 'n gestandaardiseerde manier om 'n **Sertifikaatondertekeningsversoek** (CSR) te stuur ten einde 'n sertifikaat te verkry. Die kliënt sal die bediener vra om vir hom 'n ondertekende sertifikaat te gee.

### Wat is Konfigurasieprofiele (ook bekend as mobileconfigs)?

* Apple se amptelike manier om **stelselskonfigurasie in te stel/af te dwing.**
* Lêerformaat wat verskeie payloads kan bevat.
* Gebaseer op eiendomslyste (die XML-tipe).
* “kan onderteken en geënkripteer word om hul oorsprong te valideer, hul integriteit te verseker, en hul inhoud te beskerm.” Basiese beginsels — Bladsy 70, iOS Sekuriteitsgids, Januarie 2018.

## Protokolle

### MDM

* Kombinasie van APNs (**Apple bediener**s) + RESTful API (**MDM** **verkoper** bedieners)
* **Kommunikasie** vind plaas tussen 'n **toestel** en 'n bediener wat verband hou met 'n **toestel** **bestuur** **produk**
* **Opdragte** gelewer van die MDM na die toestel in **plist-gecodeerde woordeboeke**
* Oral oor **HTTPS**. MDM-bedieners kan (en is gewoonlik) geprik.
* Apple gee die MDM-verkoper 'n **APNs sertifikaat** vir verifikasie

### DEP

* **3 API's**: 1 vir herverkopers, 1 vir MDM-verkopers, 1 vir toestelidentiteit (nie gedokumenteer nie):
* Die sogenaamde [DEP "cloud service" API](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Dit word deur MDM-bedieners gebruik om DEP-profiele met spesifieke toestelle te assosieer.
* Die [DEP API wat deur Apple Geautoriseerde Herverkopers gebruik word](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html) om toestelle te registreer, registrasiestatus te kontroleer, en transaksie-status te kontroleer.
* Die nie-gedokumenteerde private DEP API. Dit word deur Apple Toestelle gebruik om hul DEP-profiel aan te vra. Op macOS is die `cloudconfigurationd` binêre verantwoordelik vir kommunikasie oor hierdie API.
* Meer modern en **JSON** gebaseer (teenoor plist)
* Apple gee 'n **OAuth-token** aan die MDM-verkoper

**DEP "cloud service" API**

* RESTful
* sinkroniseer toestelrekords van Apple na die MDM-bediener
* sinkroniseer “DEP-profiele” na Apple vanaf die MDM-bediener (later aan die toestel gelewer deur Apple)
* 'n DEP “profiel” bevat:
* MDM-verkoper bediener URL
* Bykomende vertroude sertifikate vir bediener URL (opsionele prik)
* Ekstra instellings (bv. watter skerms om in die Setup Assistant oor te slaan)

## Serienommer

Apple-toestelle wat na 2010 vervaardig is, het oor die algemeen **12-karakter alfanumeriese** serienommers, met die **eerste drie syfers wat die vervaardigingsligging verteenwoordig**, die volgende **twee** wat die **jaar** en **week** van vervaardiging aandui, die volgende **drie** syfers wat 'n **unieke** **identifiseerder** verskaf, en die **laaste** **vier** syfers wat die **modelnommer** verteenwoordig.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## Stappe vir registrasie en bestuur

1. Toestelrekord skep (Herverkoper, Apple): Die rekord vir die nuwe toestel word geskep
2. Toestelrekord toewys (Kliënt): Die toestel word aan 'n MDM-bediener toegeken
3. Toestelrekord sinkroniseer (MDM-verkoper): MDM sinkroniseer die toestelrekords en druk die DEP-profiele na Apple
4. DEP inligting (Toestel): Toestel ontvang sy DEP-profiel
5. Profielherwinning (Toestel)
6. Profielinstallasie (Toestel) a. insluitend MDM, SCEP en wortel CA payloads
7. MDM-opdrag uitreiking (Toestel)

![](<../../../.gitbook/assets/image (694).png>)

Die lêer `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` voer funksies uit wat beskou kan word as **hoëvlak "stappe"** van die registrasieproses.

### Stap 4: DEP inligting - Verkryging van die Aktiveringsrekord

Hierdie deel van die proses vind plaas wanneer 'n **gebruiker 'n Mac vir die eerste keer opstart** (of na 'n volledige skoonmaak)

![](<../../../.gitbook/assets/image (1044).png>)

of wanneer `sudo profiles show -type enrollment` uitgevoer word

* Bepaal **of toestel DEP geaktiveer is**
* Aktiveringsrekord is die interne naam vir **DEP “profiel”**
* Begin sodra die toestel aan die internet gekoppel is
* Gedryf deur **`CPFetchActivationRecord`**
* Geïmplementeer deur **`cloudconfigurationd`** via XPC. Die **"Setup Assistant"** (wanneer die toestel vir die eerste keer opgestart word) of die **`profiles`** opdrag sal **hierdie daemon** kontak om die aktiveringsrekord te verkry.
* LaunchDaemon (loop altyd as root)

Dit volg 'n paar stappe om die Aktiveringsrekord te verkry wat deur **`MCTeslaConfigurationFetcher`** uitgevoer word. Hierdie proses gebruik 'n enkripsie genaamd **Absinthe**

1. Verkry **sertifikaat**
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. **Begin** toestand vanaf sertifikaat (**`NACInit`**)
1. Gebruik verskeie toestelspesifieke data (d.w.s. **Serienommer via `IOKit`**)
3. Verkry **sessiesleutel**
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. Vestig die sessie (**`NACKeyEstablishment`**)
5. Maak die versoek
1. POST na [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) en stuur die data `{ "action": "RequestProfileConfiguration", "sn": "" }`
2. Die JSON payload is geënkripteer met Absinthe (**`NACSign`**)
3. Alle versoeke oor HTTPs, ingeboude wortelsertifikate word gebruik

![](<../../../.gitbook/assets/image (566) (1).png>)

Die antwoord is 'n JSON-woordeboek met belangrike data soos:

* **url**: URL van die MDM-verkoper gasheer vir die aktiveringsprofiel
* **anchor-certs**: Array van DER-sertifikate wat as vertroude ankers gebruik word

### **Stap 5: Profielherwinning**

![](<../../../.gitbook/assets/image (444).png>)

* Versoek gestuur na **url verskaf in DEP-profiel**.
* **Ankersertifikate** word gebruik om **vertroue te evalueer** indien verskaf.
* Herinnering: die **anchor\_certs** eienskap van die DEP-profiel
* **Versoek is 'n eenvoudige .plist** met toestelidentifikasie
* Voorbeelde: **UDID, OS weergawe**.
* CMS-onderteken, DER-gecodeer
* Onderteken met die **toestelidentiteitssertifikaat (van APNS)**
* **Sertifikaatchain** sluit vervalle **Apple iPhone Device CA** in

![](<../../../.gitbook/assets/image (567) (1) (2) (2) (2) (2) (2) (2) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (2).png>)

### Stap 6: Profielinstallasie

* Sodra verkry, **word profiel op die stelsel gestoor**
* Hierdie stap begin outomaties (indien in **setup assistant**)
* Gedryf deur **`CPInstallActivationProfile`**
* Geïmplementeer deur mdmclient oor XPC
* LaunchDaemon (as root) of LaunchAgent (as gebruiker), afhangende van konteks
* Konfigurasieprofiele het verskeie payloads om te installeer
* Raamwerk het 'n plugin-gebaseerde argitektuur vir die installering van profiele
* Elke payload tipe is geassosieer met 'n plugin
* Kan XPC (in raamwerk) of klassieke Cocoa (in ManagedClient.app) wees
* Voorbeeld:
* Sertifikaatpayloads gebruik CertificateService.xpc

Tipies, **aktiveringsprofiel** verskaf deur 'n MDM-verkoper sal **die volgende payloads insluit**:

* `com.apple.mdm`: om die toestel in MDM te **registreer**
* `com.apple.security.scep`: om 'n **kliëntsertifikaat** veilig aan die toestel te verskaf.
* `com.apple.security.pem`: om **vertroude CA-sertifikate** aan die toestel se Stelselsleutelhouer te installeer.
* Die installering van die MDM-payload is gelyk aan **MDM inligting in die dokumentasie**
* Payload **bevat sleutel eienskappe**:
*
* MDM Inligting URL (**`CheckInURL`**)
* MDM Opdrag Polling URL (**`ServerURL`**) + APNs onderwerp om dit te aktiveer
* Om MDM-payload te installeer, word 'n versoek gestuur na **`CheckInURL`**
* Geïmplementeer in **`mdmclient`**
* MDM-payload kan op ander payloads afhanklik wees
* Laat **versoeke toe om aan spesifieke sertifikate geprik te word**:
* Eienskap: **`CheckInURLPinningCertificateUUIDs`**
* Eienskap: **`ServerURLPinningCertificateUUIDs`**
* Gelewer via PEM payload
* Laat die toestel toe om met 'n identiteitssertifikaat toegeskryf te word:
* Eienskap: IdentityCertificateUUID
* Gelewer via SCEP payload

### **Stap 7: Luister vir MDM-opdragte**

* Nadat MDM-inligting voltooi is, kan die verkoper **drukkennisgewings uitreik met behulp van APNs**
* By ontvangs, hanteer deur **`mdmclient`**
* Om vir MDM-opdragte te poll, word 'n versoek gestuur na ServerURL
* Maak gebruik van die voorheen geïnstalleerde MDM-payload:
* **`ServerURLPinningCertificateUUIDs`** vir prik versoek
* **`IdentityCertificateUUID`** vir TLS kliëntsertifikaat

## Aanvalle

### Registrasie van Toestelle in Ander Organisasies

Soos voorheen opgemerk, om te probeer om 'n toestel in 'n organisasie te registreer, **is slegs 'n Serienommer wat aan daardie Organisasie behoort, nodig**. Sodra die toestel geregistreer is, sal verskeie organisasies sensitiewe data op die nuwe toestel installeer: sertifikate, toepassings, WiFi-wagwoorde, VPN-konfigurasies [en so aan](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Daarom kan dit 'n gevaarlike toegangspunt vir aanvallers wees as die registrasieproses nie korrek beskerm word nie:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PR's in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
