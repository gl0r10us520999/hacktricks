# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

Kinyume na Kernel Extensions, **System Extensions zinaendesha katika nafasi ya mtumiaji** badala ya nafasi ya kernel, kupunguza hatari ya kuanguka kwa mfumo kutokana na hitilafu ya kiendelezi.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Kuna aina tatu za system extensions: **DriverKit** Extensions, **Network** Extensions, na **Endpoint Security** Extensions.

### **DriverKit Extensions**

DriverKit ni mbadala wa kernel extensions ambazo **zinatoa msaada wa vifaa**. Inaruhusu madereva ya vifaa (kama USB, Serial, NIC, na HID drivers) kuendesha katika nafasi ya mtumiaji badala ya nafasi ya kernel. Mfumo wa DriverKit unajumuisha **toleo la nafasi ya mtumiaji la baadhi ya madarasa ya I/O Kit**, na kernel inasambaza matukio ya kawaida ya I/O Kit kwa nafasi ya mtumiaji, ikitoa mazingira salama kwa madereva haya kuendesha.

### **Network Extensions**

Network Extensions zinatoa uwezo wa kubadilisha tabia za mtandao. Kuna aina kadhaa za Network Extensions:

* **App Proxy**: Hii inatumika kwa kuunda mteja wa VPN ambao unatekeleza itifaki ya VPN iliyobinafsishwa inayotegemea mtiririko. Hii ina maana inashughulikia trafiki ya mtandao kulingana na muunganisho (au mitiririko) badala ya pakiti za kibinafsi.
* **Packet Tunnel**: Hii inatumika kwa kuunda mteja wa VPN ambao unatekeleza itifaki ya VPN iliyobinafsishwa inayotegemea pakiti. Hii ina maana inashughulikia trafiki ya mtandao kulingana na pakiti za kibinafsi.
* **Filter Data**: Hii inatumika kwa kuchuja "mitiririko" ya mtandao. Inaweza kufuatilia au kubadilisha data za mtandao katika kiwango cha mtiririko.
* **Filter Packet**: Hii inatumika kwa kuchuja pakiti za mtandao za kibinafsi. Inaweza kufuatilia au kubadilisha data za mtandao katika kiwango cha pakiti.
* **DNS Proxy**: Hii inatumika kwa kuunda mtoa huduma wa DNS uliobinafsishwa. Inaweza kutumika kufuatilia au kubadilisha maombi na majibu ya DNS.

## Endpoint Security Framework

Endpoint Security ni mfumo ulioandaliwa na Apple katika macOS ambao unatoa seti ya APIs kwa usalama wa mfumo. Unakusudiwa kutumiwa na **watoa huduma za usalama na waendelezaji kujenga bidhaa ambazo zinaweza kufuatilia na kudhibiti shughuli za mfumo** ili kubaini na kulinda dhidi ya shughuli mbaya.

Mfumo huu unatoa **mkusanyiko wa APIs za kufuatilia na kudhibiti shughuli za mfumo**, kama vile utekelezaji wa michakato, matukio ya mfumo wa faili, matukio ya mtandao na kernel.

Msingi wa mfumo huu umewekwa katika kernel, kama Kernel Extension (KEXT) iliyoko **`/System/Library/Extensions/EndpointSecurity.kext`**. KEXT hii inajumuisha vipengele kadhaa muhimu:

* **EndpointSecurityDriver**: Hii inafanya kazi kama "nukta ya kuingia" kwa kiendelezi cha kernel. Ni nukta kuu ya mwingiliano kati ya OS na mfumo wa Endpoint Security.
* **EndpointSecurityEventManager**: Kipengele hiki kinawajibika kwa kutekeleza nanga za kernel. Nanga za kernel zinaruhusu mfumo kufuatilia matukio ya mfumo kwa kukamata wito wa mfumo.
* **EndpointSecurityClientManager**: Hii inasimamia mawasiliano na wateja wa nafasi ya mtumiaji, ikifuatilia ni wateja gani wameunganishwa na wanahitaji kupokea arifa za matukio.
* **EndpointSecurityMessageManager**: Hii inatuma ujumbe na arifa za matukio kwa wateja wa nafasi ya mtumiaji.

Matukio ambayo mfumo wa Endpoint Security unaweza kufuatilia yanagawanywa katika:

* Matukio ya faili
* Matukio ya mchakato
* Matukio ya socket
* Matukio ya kernel (kama vile kupakia/kutoa kiendelezi cha kernel au kufungua kifaa cha I/O Kit)

### Endpoint Security Framework Architecture

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Mawasiliano ya nafasi ya mtumiaji** na mfumo wa Endpoint Security hufanyika kupitia darasa la IOUserClient. Aina mbili tofauti za subclasses zinatumika, kulingana na aina ya mwitishaji:

* **EndpointSecurityDriverClient**: Hii inahitaji ruhusa ya `com.apple.private.endpoint-security.manager`, ambayo inashikiliwa tu na mchakato wa mfumo `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Hii inahitaji ruhusa ya `com.apple.developer.endpoint-security.client`. Hii kwa kawaida ingetumiwa na programu za usalama za wahusika wengine ambazo zinahitaji kuingiliana na mfumo wa Endpoint Security.

The Endpoint Security Extensions:**`libEndpointSecurity.dylib`** ni maktaba ya C ambayo system extensions hutumia kuwasiliana na kernel. Maktaba hii inatumia I/O Kit (`IOKit`) kuwasiliana na KEXT ya Endpoint Security.

**`endpointsecurityd`** ni daemon muhimu wa mfumo unaohusika na kusimamia na kuzindua system extensions za usalama wa mwisho, hasa wakati wa mchakato wa kuanzisha mapema. **Ni system extensions tu** zilizoainishwa na **`NSEndpointSecurityEarlyBoot`** katika faili yao ya `Info.plist` ndizo zinapata matibabu haya ya kuanzisha mapema.

Daemon nyingine ya mfumo, **`sysextd`**, **inasahihisha system extensions** na kuhamasisha katika maeneo sahihi ya mfumo. Kisha inaomba daemon husika kupakia kiendelezi. **`SystemExtensions.framework`** inawajibika kwa kuanzisha na kuzima system extensions.

## Bypassing ESF

ESF inatumika na zana za usalama ambazo zitajaribu kugundua red teamer, hivyo taarifa yoyote kuhusu jinsi hii inaweza kuepukwa inavutia.

### CVE-2021-30965

Jambo ni kwamba programu ya usalama inahitaji kuwa na **ruhusa za Upatikanaji wa Disk Kamili**. Hivyo ikiwa mshambuliaji anaweza kuondoa hiyo, anaweza kuzuia programu hiyo isifanye kazi:
```bash
tccutil reset All
```
Kwa **maelezo zaidi** kuhusu hii bypass na zinazohusiana, angalia mazungumzo [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Mwisho, hii ilirekebishwa kwa kutoa ruhusa mpya **`kTCCServiceEndpointSecurityClient`** kwa programu ya usalama inayosimamiwa na **`tccd`** ili `tccutil` isifute ruhusa zake na kuzuia kuendesha kwake.

## Marejeo

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
