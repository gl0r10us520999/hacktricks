# Cheat Engine

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

[**Cheat Engine**](https://www.cheatengine.org/downloads.php) ni programu muhimu ya kupata mahali ambapo thamani muhimu zimehifadhiwa ndani ya kumbukumbu ya mchezo unaoendelea na kuzibadilisha.\
Unaposhusha na kuendesha, unapata **mafunzo** ya jinsi ya kutumia chombo hicho. Ikiwa unataka kujifunza jinsi ya kutumia chombo hicho, inashauriwa kukamilisha.

## Unatafuta nini?

![](<../../.gitbook/assets/image (762).png>)

Chombo hiki ni muhimu sana kupata **mahali ambapo thamani fulani** (kawaida ni nambari) **imehifadhiwa katika kumbukumbu** ya programu.\
**Kawaida nambari** huhifadhiwa katika **4bytes** fomu, lakini unaweza pia kuzikuta katika **double** au **float** fomati, au unaweza kutaka kutafuta kitu **tofauti na nambari**. Kwa sababu hiyo unahitaji kuwa na uhakika unachagua kile unachotaka **kutafuta**:

![](<../../.gitbook/assets/image (324).png>)

Pia unaweza kuashiria **aina tofauti** za **tafiti**:

![](<../../.gitbook/assets/image (311).png>)

Unaweza pia kuangalia kisanduku ili **kusitisha mchezo wakati wa kuskania kumbukumbu**:

![](<../../.gitbook/assets/image (1052).png>)

### Hotkeys

Katika _**Edit --> Settings --> Hotkeys**_ unaweza kuweka **hotkeys** tofauti kwa madhumuni tofauti kama **kusitisha** **mchezo** (ambayo ni muhimu sana ikiwa kwa wakati fulani unataka kuskania kumbukumbu). Chaguzi nyingine zinapatikana:

![](<../../.gitbook/assets/image (864).png>)

## Kubadilisha thamani

Mara tu unapokuwa **umepata** mahali ambapo **thamani** unayo **tafuta** (zaidi kuhusu hii katika hatua zinazofuata) unaweza **kuibadilisha** kwa kubofya mara mbili, kisha kubofya mara mbili kwenye thamani yake:

![](<../../.gitbook/assets/image (563).png>)

Na hatimaye **kuweka alama** ili kupata mabadiliko kufanyika katika kumbukumbu:

![](<../../.gitbook/assets/image (385).png>)

**Mabadiliko** kwa **kumbukumbu** yatakuwa **yamewekwa** mara moja (kumbuka kwamba hadi mchezo usitumie thamani hii tena thamani **haitasasishwa katika mchezo**).

## Kutafuta thamani

Hivyo, tunaenda kudhani kwamba kuna thamani muhimu (kama maisha ya mtumiaji wako) ambayo unataka kuboresha, na unatafuta thamani hii katika kumbukumbu)

### Kupitia mabadiliko yanayojulikana

Tukidhani unatafuta thamani 100, unafanya **scan** ukitafuta thamani hiyo na unapata mengi ya kufanana:

![](<../../.gitbook/assets/image (108).png>)

Kisha, unafanya kitu ili **thamani ibadilike**, na un **asitisha** mchezo na **ufanye** **scan inayofuata**:

![](<../../.gitbook/assets/image (684).png>)

Cheat Engine itatafuta **thamani** ambazo **zilipita kutoka 100 hadi thamani mpya**. Hongera, umepata **anwani** ya thamani uliyokuwa unatafuta, sasa unaweza kuibadilisha.\
&#xNAN;_Ikiwa bado una thamani kadhaa, fanya kitu kubadilisha tena thamani hiyo, na fanya "scan inayofuata" ili kuchuja anwani._

### Thamani isiyojulikana, mabadiliko yanayojulikana

Katika hali ambapo **hujui thamani** lakini unajua **jinsi ya kuifanya ibadilike** (na hata thamani ya mabadiliko) unaweza kutafuta nambari yako.

Hivyo, anza kwa kufanya scan ya aina "**Thamani ya mwanzo isiyojulikana**":

![](<../../.gitbook/assets/image (890).png>)

Kisha, fanya thamani ibadilike, onyesha **jinsi** **thamani** **ilibadilika** (katika kesi yangu ilipungua kwa 1) na fanya **scan inayofuata**:

![](<../../.gitbook/assets/image (371).png>)

Utawasilishwa **na thamani zote ambazo zilibadilishwa kwa njia iliyochaguliwa**:

![](<../../.gitbook/assets/image (569).png>)

Mara tu unapokuwa umepata thamani yako, unaweza kuibadilisha.

Kumbuka kwamba kuna **mabadiliko mengi yanayowezekana** na unaweza kufanya hatua hizi **kadri unavyotaka** ili kuchuja matokeo:

![](<../../.gitbook/assets/image (574).png>)

### Anwani ya Kumbukumbu ya Nasibu - Kupata msimbo

Hadi sasa tumefundishwa jinsi ya kupata anwani inayohifadhi thamani, lakini kuna uwezekano mkubwa kwamba katika **utekelezaji tofauti wa mchezo anwani hiyo iko katika sehemu tofauti za kumbukumbu**. Hivyo hebu tujifunze jinsi ya kila wakati kupata anwani hiyo.

Tumia baadhi ya hila zilizotajwa, pata anwani ambapo mchezo wako wa sasa unahifadhi thamani muhimu. Kisha (ukisimamisha mchezo ikiwa unataka) fanya **kubofya kulia** kwenye **anwani** iliyopatikana na uchague "**Jua ni nani anayefikia anwani hii**" au "**Jua ni nani anayandika kwenye anwani hii**":

![](<../../.gitbook/assets/image (1067).png>)

**Chaguo la kwanza** ni muhimu kujua **sehemu** za **msimbo** zinazotumia **anwani hii** (ambayo ni muhimu kwa mambo zaidi kama **kujua wapi unaweza kubadilisha msimbo** wa mchezo).\
**Chaguo la pili** ni **maalum zaidi**, na litakuwa na msaada zaidi katika kesi hii kwani tunavutiwa kujua **kutoka wapi thamani hii inaandikwa**.

Mara tu unapochagua moja ya chaguzi hizo, **debugger** itakuwa **imeunganishwa** na programu na dirisha jipya **bila maudhui** litajitokeza. Sasa, **cheza** **mchezo** na **badilisha** hiyo **thamani** (bila kuanzisha upya mchezo). **Dirisha** linapaswa kuwa **limejaa** na **anwani** zinazobadilisha **thamani**:

![](<../../.gitbook/assets/image (91).png>)

Sasa kwamba umepata anwani inayobadilisha thamani unaweza **kubadilisha msimbo kwa mapenzi yako** (Cheat Engine inakuruhusu kuibadilisha kwa NOPs haraka):

![](<../../.gitbook/assets/image (1057).png>)

Hivyo, sasa unaweza kuibadilisha ili msimbo usiathiri nambari yako, au uathiri kila wakati kwa njia chanya.

### Anwani ya Kumbukumbu ya Nasibu - Kupata kiashiria

Kufuata hatua zilizopita, pata mahali ambapo thamani unayovutiwa nayo iko. Kisha, kwa kutumia "**Jua ni nani anayandika kwenye anwani hii**" pata ni anwani gani inayoandika thamani hii na ubofye mara mbili ili kupata mtazamo wa disassembly:

![](<../../.gitbook/assets/image (1039).png>)

Kisha, fanya scan mpya **ukitafuta thamani ya hex kati ya "\[]"** (thamani ya $edx katika kesi hii):

![](<../../.gitbook/assets/image (994).png>)

(_Ikiwa kadhaa zinaonekana kawaida unahitaji ile yenye anwani ndogo zaidi_)\
Sasa, tumepata **kiashiria ambacho kitakuwa kinabadilisha thamani tunayotaka**.

Bofya kwenye "**Ongeza Anwani kwa Mikono**":

![](<../../.gitbook/assets/image (990).png>)

Sasa, bofya kwenye kisanduku cha "Kiashiria" na ongeza anwani iliyopatikana katika kisanduku cha maandiko (katika hali hii, anwani iliyopatikana katika picha iliyopita ilikuwa "Tutorial-i386.exe"+2426B0):

![](<../../.gitbook/assets/image (392).png>)

(Kumbuka jinsi "Anwani" ya kwanza inajazwa kiotomatiki kutoka kwa anwani ya kiashiria unayoingiza)

Bofya OK na kiashiria kipya kitaundwa:

![](<../../.gitbook/assets/image (308).png>)

Sasa, kila wakati unabadilisha thamani hiyo unakuwa **unabadilisha thamani muhimu hata kama anwani ya kumbukumbu ambapo thamani hiyo iko ni tofauti.**

### Uingizaji wa Msimbo

Uingizaji wa msimbo ni mbinu ambapo unatia kipande cha msimbo katika mchakato wa lengo, na kisha unarudisha utekelezaji wa msimbo ili kupita kupitia msimbo wako ulioandikwa (kama kukupa alama badala ya kuziondoa).

Hivyo, fikiria umepata anwani inayopunguza 1 kwa maisha ya mchezaji wako:

![](<../../.gitbook/assets/image (203).png>)

Bofya onyesha disassembler ili kupata **msimbo wa disassemble**.\
Kisha, bofya **CTRL+a** ili kuanzisha dirisha la Auto assemble na uchague _**Template --> Code Injection**_

![](<../../.gitbook/assets/image (902).png>)

Jaza **anwani ya amri unayotaka kubadilisha** (hii kawaida inajazwa kiotomatiki):

![](<../../.gitbook/assets/image (744).png>)

Kigezo kitaundwa:

![](<../../.gitbook/assets/image (944).png>)

Hivyo, ingiza msimbo wako mpya wa assembly katika sehemu ya "**newmem**" na ondolea msimbo wa asili kutoka kwa "**originalcode**" ikiwa hutaki itekelezwe\*\*.\*\* Katika mfano huu msimbo uliotiwa ndani utaongeza alama 2 badala ya kupunguza 1:

![](<../../.gitbook/assets/image (521).png>)

**Bofya kwenye tekeleza na kadhalika na msimbo wako unapaswa kuingizwa katika programu ukibadilisha tabia ya kazi hiyo!**

## **Marejeleo**

* **Mafunzo ya Cheat Engine, kamilisha ili kujifunza jinsi ya kuanza na Cheat Engine**
