# macOS MDM

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

**Ili kujifunza kuhusu macOS MDMs angalia:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Msingi

### **Muhtasari wa MDM (Usimamizi wa Vifaa vya Mkononi)**

[Usimamizi wa Vifaa vya Mkononi](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) unatumika kwa kusimamia vifaa mbalimbali vya mwisho kama vile simu za mkononi, kompyuta za mkononi, na vidonge. Hasa kwa majukwaa ya Apple (iOS, macOS, tvOS), inahusisha seti ya vipengele maalum, APIs, na mazoea. Uendeshaji wa MDM unategemea seva ya MDM inayofaa, ambayo inaweza kuwa inapatikana kibiashara au ya chanzo wazi, na lazima iunge mkono [Protokali ya MDM](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Vidokezo muhimu ni pamoja na:

* Udhibiti wa kati juu ya vifaa.
* Kutegemea seva ya MDM inayofuata protokali ya MDM.
* Uwezo wa seva ya MDM kutuma amri mbalimbali kwa vifaa, kwa mfano, kufuta data kwa mbali au kufunga usanidi.

### **Msingi wa DEP (Mpango wa Usajili wa Vifaa)**

[Mpango wa Usajili wa Vifaa](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) unaotolewa na Apple unarahisisha uunganisho wa Usimamizi wa Vifaa vya Mkononi (MDM) kwa kuwezesha usanidi wa sifuri wa kugusa kwa vifaa vya iOS, macOS, na tvOS. DEP inafanya mchakato wa usajili kuwa wa kiotomatiki, ikiruhusu vifaa kuwa na kazi mara moja kutoka kwenye sanduku, kwaingawa kwaingilia kati kidogo kutoka kwa mtumiaji au msimamizi. Vipengele muhimu ni pamoja na:

* Inaruhusu vifaa kujiandikisha kwa uhuru na seva ya MDM iliyowekwa awali wakati wa kuanzishwa kwa mara ya kwanza.
* Kimsingi ni faida kwa vifaa vipya, lakini pia inatumika kwa vifaa vinavyopitia usanidi upya.
* Inarahisisha usanidi rahisi, ikifanya vifaa kuwa tayari kwa matumizi ya shirika haraka.

### **Kuzingatia Usalama**

Ni muhimu kutambua kwamba urahisi wa usajili unaotolewa na DEP, ingawa ni wa manufaa, unaweza pia kuleta hatari za usalama. Ikiwa hatua za kinga hazitekelezwi ipasavyo kwa usajili wa MDM, washambuliaji wanaweza kutumia mchakato huu rahisi kujiandikisha kifaa chao kwenye seva ya MDM ya shirika, wakijifanya kuwa kifaa cha kampuni.

{% hint style="danger" %}
**Tahadhari ya Usalama**: Usajili wa DEP ulio rahisishwa unaweza kuruhusu usajili usioidhinishwa wa kifaa kwenye seva ya MDM ya shirika ikiwa hatua sahihi za kinga hazipo.
{% endhint %}

### Msingi Ni nini SCEP (Protokali ya Usajili wa Cheti Rahisi)?

* Protokali ya zamani, iliyoundwa kabla ya TLS na HTTPS kuwa maarufu.
* Inatoa wateja njia iliyo sanifishwa ya kutuma **Ombi la Kusaini Cheti** (CSR) kwa lengo la kupata cheti. Mteja ataomba seva impe cheti kilichosainiwa.

### Ni nini Profaili za Usanidi (pia inajulikana kama mobileconfigs)?

* Njia rasmi ya Apple ya **kuweka/kulazimisha usanidi wa mfumo.**
* Muundo wa faili ambao unaweza kuwa na payload nyingi.
* Imejengwa kwa orodha za mali (aina ya XML).
* “inaweza kusainiwa na kufichwa ili kuthibitisha asili yao, kuhakikisha uadilifu wao, na kulinda maudhui yao.” Msingi — Ukurasa wa 70, Mwongozo wa Usalama wa iOS, Januari 2018.

## Protokali

### MDM

* Mchanganyiko wa APNs (**seva za Apple**) + API ya RESTful (**seva za wauzaji wa MDM**)
* **Mawasiliano** hutokea kati ya **kifaa** na seva inayohusishwa na **bidhaa ya usimamizi wa kifaa**
* **Amri** hutolewa kutoka kwa MDM kwenda kwa kifaa katika **kamusi za plist zilizokodishwa**
* Kote **HTTPS**. Seva za MDM zinaweza kuwa (na kawaida huwa) zimepangwa.
* Apple inampa muuzaji wa MDM **cheti cha APNs** kwa uthibitisho

### DEP

* **API 3**: 1 kwa wauzaji, 1 kwa wauzaji wa MDM, 1 kwa utambulisho wa kifaa (isiyoandikwa):
* API inayoitwa [DEP "huduma ya wingu"](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Hii inatumika na seva za MDM kuunganisha profaili za DEP na vifaa maalum.
* [API ya DEP inayotumiwa na Wauzaji Waliothibitishwa na Apple](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html) kujiandikisha vifaa, kuangalia hali ya usajili, na kuangalia hali ya muamala.
* API ya DEP ya kibinafsi isiyoandikwa. Hii inatumika na Vifaa vya Apple kuomba profaili yao ya DEP. Kwenye macOS, binary ya `cloudconfigurationd` inawajibika kwa mawasiliano kupitia API hii.
* Ya kisasa zaidi na **inategemea JSON** (kinyume na plist)
* Apple inampa muuzaji wa MDM **token ya OAuth**

**DEP "huduma ya wingu" API**

* RESTful
* sambaza rekodi za kifaa kutoka Apple hadi seva ya MDM
* sambaza “profaili za DEP” kwa Apple kutoka kwa seva ya MDM (zitakazopelekwa na Apple kwa kifaa baadaye)
* Profaili ya DEP ina:
* URL ya seva ya muuzaji wa MDM
* Cheti za ziada za kuaminika kwa URL ya seva (kuweka pin hiari)
* Mipangilio ya ziada (kwa mfano, ni skrini zipi za kupita katika Msaidizi wa Usanidi)

## Nambari ya Serial

Vifaa vya Apple vilivyotengenezwa baada ya mwaka 2010 kwa ujumla vina **nambari za serial za alphanumeric 12**, ambapo **nambari tatu za kwanza zinaonyesha mahali pa utengenezaji**, mbili zinazofuata zinaashiria **mwaka** na **wiki** ya utengenezaji, nambari tatu zinazofuata zinatoa **kitambulisho** **maalum**, na **nambari nne** za mwisho zinaonyesha **nambari ya mfano**.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## Hatua za usajili na usimamizi

1. Uundaji wa rekodi ya kifaa (Muuzaji, Apple): Rekodi ya kifaa kipya inaundwa
2. Ugawaji wa rekodi ya kifaa (Mteja): Kifaa kinapewa seva ya MDM
3. Usawazishaji wa rekodi ya kifaa (Muuzaji wa MDM): MDM inasawazisha rekodi za kifaa na kusukuma profaili za DEP kwa Apple
4. Kuangalia DEP (Kifaa): Kifaa kinapata profaili yake ya DEP
5. Urejeleaji wa Profaili (Kifaa)
6. Usanidi wa Profaili (Kifaa) a. ikijumuisha MDM, SCEP na payloads za CA za mizizi
7. Kutolewa kwa amri za MDM (Kifaa)

![](<../../../.gitbook/assets/image (694).png>)

Faili `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` inatoa kazi ambazo zinaweza kuzingatiwa kama **"hatua" za juu** za mchakato wa usajili.

### Hatua ya 4: Kuangalia DEP - Kupata Rekodi ya Uanzishaji

Sehemu hii ya mchakato hutokea wakati **mtumiaji anapoanzisha Mac kwa mara ya kwanza** (au baada ya kufutwa kabisa)

![](<../../../.gitbook/assets/image (1044).png>)

au wakati wa kutekeleza `sudo profiles show -type enrollment`

* Kuamua **kama kifaa kina uwezo wa DEP**
* Rekodi ya Uanzishaji ni jina la ndani la **"profaili" ya DEP**
* Huanzia mara tu kifaa kinapounganishwa kwenye Mtandao
* Inasukumwa na **`CPFetchActivationRecord`**
* Imeanzishwa na **`cloudconfigurationd`** kupitia XPC. **"Msaidizi wa Usanidi"** (wakati kifaa kinapoanzishwa kwa mara ya kwanza) au amri ya **`profiles`** itawasiliana na **daemon** hii ili kupata rekodi ya uanzishaji.
* LaunchDaemon (daima inafanya kazi kama root)

Inafuata hatua chache kupata Rekodi ya Uanzishaji inayofanywa na **`MCTeslaConfigurationFetcher`**. Mchakato huu unatumia usimbuaji unaoitwa **Absinthe**

1. Pata **cheti**
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. **Anzisha** hali kutoka kwa cheti (**`NACInit`**)
1. Inatumia data mbalimbali maalum za kifaa (yaani **Nambari ya Serial kupitia `IOKit`**)
3. Pata **funguo ya kikao**
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. Kuanzisha kikao (**`NACKeyEstablishment`**)
5. Fanya ombi
1. POST kwa [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) ukituma data `{ "action": "RequestProfileConfiguration", "sn": "" }`
2. Payload ya JSON imefichwa kwa kutumia Absinthe (**`NACSign`**)
3. Maombi yote kupitia HTTPs, cheti za mizizi zilizojengwa ndani zinatumika

![](<../../../.gitbook/assets/image (566) (1).png>)

Jibu ni kamusi ya JSON yenye data muhimu kama:

* **url**: URL ya mwenyeji wa muuzaji wa MDM kwa profaili ya uanzishaji
* **cheti za ankara**: Orodha ya cheti za DER zinazotumika kama ankara za kuaminika

### **Hatua ya 5: Urejeleaji wa Profaili**

![](<../../../.gitbook/assets/image (444).png>)

* Ombi lilitumwa kwa **url iliyotolewa katika profaili ya DEP**.
* **Cheti za ankara** zinatumika ili **kuthibitisha uaminifu** ikiwa zimetolewa.
* Kumbuka: mali ya **anchor\_certs** ya profaili ya DEP
* **Ombi ni .plist rahisi** yenye utambulisho wa kifaa
* Mifano: **UDID, toleo la OS**.
* Imeandikwa CMS, imekodishwa DER
* Imeandikwa kwa kutumia **cheti cha utambulisho wa kifaa (kutoka APNS)**
* **Mnyororo wa cheti** unajumuisha **CA ya Kifaa cha Apple iPhone** iliyokwisha muda

![](<../../../.gitbook/assets/image (567) (1) (2) (2) (2) (2) (2) (2) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (2).png>)

### Hatua ya 6: Usanidi wa Profaili

* Mara tu inapopatikana, **profaili inahifadhiwa kwenye mfumo**
* Hatua hii huanza kiotomatiki (ikiwa katika **msaidizi wa usanidi**)
* Inasukumwa na **`CPInstallActivationProfile`**
* Imeanzishwa na mdmclient kupitia XPC
* LaunchDaemon (kama root) au LaunchAgent (kama mtumiaji), kulingana na muktadha
* Profaili za usanidi zina payload nyingi za kusanidi
* Mfumo huu una usanidi wa msingi wa plugin kwa ajili ya kusanidi profaili
* Kila aina ya payload inahusishwa na plugin
* Inaweza kuwa XPC (katika mfumo) au Cocoa ya kawaida (katika ManagedClient.app)
* Mfano:
* Payload za Cheti hutumia CertificateService.xpc

Kwa kawaida, **profaili ya uanzishaji** inayotolewa na muuzaji wa MDM itajumuisha **payloads zifuatazo**:

* `com.apple.mdm`: ili **kujiandikisha** kifaa katika MDM
* `com.apple.security.scep`: ili kutoa kwa usalama **cheti cha mteja** kwa kifaa.
* `com.apple.security.pem`: ili **kusanidi cheti za CA zinazokubalika** kwenye Keychain ya Mfumo wa kifaa.
* Kusanidi payload ya MDM ni sawa na **kuangalia MDM katika nyaraka**
* Payload **ina mali muhimu**:
*
* URL ya Kuangalia MDM (**`CheckInURL`**)
* URL ya Kuangalia Amri za MDM (**`ServerURL`**) + mada ya APNs kuichochea
* Ili kusanidi payload ya MDM, ombi litatumwa kwa **`CheckInURL`**
* Imeanzishwa katika **`mdmclient`**
* Payload ya MDM inaweza kutegemea payload nyingine
* Inaruhusu **maombi kuunganishwa na vyeti maalum**:
* Mali: **`CheckInURLPinningCertificateUUIDs`**
* Mali: **`ServerURLPinningCertificateUUIDs`**
* Imetolewa kupitia payload ya PEM
* Inaruhusu kifaa kupewa cheti cha utambulisho:
* Mali: IdentityCertificateUUID
* Imetolewa kupitia payload ya SCEP

### **Hatua ya 7: Kusikiliza amri za MDM**

* Baada ya kuangalia MDM kukamilika, muuzaji anaweza **kutuma arifa za kusukuma kwa kutumia APNs**
* Mara baada ya kupokea, inashughulikiwa na **`mdmclient`**
* Ili kupiga kura kwa amri za MDM, ombi litatumwa kwa ServerURL
* Inatumia payload ya MDM iliyowekwa awali:
* **`ServerURLPinningCertificateUUIDs`** kwa ombi la kuweka pin
* **`IdentityCertificateUUID`** kwa cheti cha mteja wa TLS

## Mashambulizi

### Kujiandikisha Vifaa katika Mashirika Mengine

Kama ilivyosemwa awali, ili kujaribu kujiandikisha kifaa katika shirika **ni nambari ya Serial pekee inayomilikiwa na Shirika hilo inayohitajika**. Mara kifaa kinapojisajili, mashirika kadhaa yataweka data nyeti kwenye kifaa kipya: vyeti, programu, nywila za WiFi, usanidi wa VPN [na kadhalika](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Kwa hivyo, hii inaweza kuwa njia hatari kwa washambuliaji ikiwa mchakato wa usajili haujalindwa ipasavyo:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

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
