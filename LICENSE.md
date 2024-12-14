{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **fuata** sisi kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}


<a rel="license" href="https://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://licensebuttons.net/l/by-nc/4.0/88x31.png" /></a><br>Hakimiliki © Carlos Polop 2021. Isipokuwa pale ambapo imeelezwa vinginevyo (habari za nje zilizokopwa kwenye kitabu zinamilikiwa na waandishi wa asili), maandiko kwenye <a href="https://github.com/carlospolop/hacktricks">HACK TRICKS</a> na Carlos Polop yanatolewa chini ya <a href="https://creativecommons.org/licenses/by-nc/4.0/">Leseni ya Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)</a>.

Leseni: Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)<br>
Leseni Inayosomwa na Binadamu: https://creativecommons.org/licenses/by-nc/4.0/<br>
Masharti Kamili ya Kisheria: https://creativecommons.org/licenses/by-nc/4.0/legalcode<br>
Muundo: https://github.com/jmatsushita/Creative-Commons-4.0-Markdown/blob/master/licenses/by-nc.markdown<br>

# creative commons

# Attribution-NonCommercial 4.0 International

Creative Commons Corporation (“Creative Commons”) si kampuni ya sheria na haitoi huduma za kisheria au ushauri wa kisheria. Usambazaji wa leseni za umma za Creative Commons hauunda uhusiano wa wakili-mteja au uhusiano mwingine. Creative Commons inapatia leseni zake na habari zinazohusiana kwenye msingi wa “kama ilivyo”. Creative Commons haitoi dhamana yoyote kuhusu leseni zake, nyenzo zozote zilizotolewa chini ya masharti na hali zao, au habari yoyote inayohusiana. Creative Commons inakanusha dhima yoyote kwa uharibifu unaotokana na matumizi yao kwa kiwango kinachowezekana.

## Kutumia Leseni za Umma za Creative Commons

Leseni za umma za Creative Commons zinatoa seti ya kawaida ya masharti na hali ambazo waumbaji na wamiliki wengine wa haki wanaweza kutumia kushiriki kazi za asili za uandishi na nyenzo nyingine zinazohusishwa na hakimiliki na haki nyingine fulani zilizotajwa katika leseni ya umma hapa chini. Maoni yafuatayo ni kwa madhumuni ya habari tu, hayakamilifu, na hayaundai sehemu ya leseni zetu.

* __Maoni kwa watoaji leseni:__ Leseni zetu za umma zinakusudiwa kutumiwa na wale walioidhinishwa kutoa umma ruhusa ya kutumia nyenzo kwa njia ambazo kwa kawaida zinakatazwa na hakimiliki na haki nyingine fulani. Leseni zetu hazirudishwi. Watoaji leseni wanapaswa kusoma na kuelewa masharti na hali za leseni wanayochagua kabla ya kuitumia. Watoaji leseni wanapaswa pia kuhakikisha haki zote zinazohitajika kabla ya kutumia leseni zetu ili umma uweze kutumia nyenzo kama inavyotarajiwa. Watoaji leseni wanapaswa kuweka wazi nyenzo yoyote isiyo chini ya leseni. Hii inajumuisha nyenzo nyingine zilizo na leseni ya CC, au nyenzo zinazotumiwa chini ya ubaguzi au kikomo cha hakimiliki. [Maoni zaidi kwa watoaji leseni](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensors).

* __Maoni kwa umma:__ Kwa kutumia moja ya leseni zetu za umma, mtoaji leseni anampa umma ruhusa ya kutumia nyenzo iliyotolewa chini ya masharti na hali zilizotajwa. Ikiwa ruhusa ya mtoaji leseni si ya lazima kwa sababu yoyote – kwa mfano, kwa sababu ya ubaguzi wowote unaotumika au kikomo kwa hakimiliki – basi matumizi hayo hayataregulishwa na leseni. Leseni zetu zinatoa tu ruhusa chini ya hakimiliki na haki nyingine fulani ambazo mtoaji leseni ana mamlaka ya kutoa. Matumizi ya nyenzo iliyotolewa yanaweza bado kuwa na kikomo kwa sababu nyingine, ikiwa ni pamoja na kwa sababu wengine wana hakimiliki au haki nyingine katika nyenzo hiyo. Mtoaji leseni anaweza kufanya maombi maalum, kama kuomba kwamba mabadiliko yote yawe na alama au kuelezwa. Ingawa si lazima kwa leseni zetu, unashauriwa kuheshimu maombi hayo pale inapowezekana. [Maoni zaidi kwa umma](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensees).

# Leseni ya Umma ya Creative Commons Attribution-NonCommercial 4.0 International

Kwa kutumia Haki za Leseni (zilizofafanuliwa hapa chini), Unakubali na kukubali kufungwa na masharti na hali za Leseni hii ya Umma ya Creative Commons Attribution-NonCommercial 4.0 International ("Leseni ya Umma"). Kwa kiwango ambacho Leseni hii ya Umma inaweza kutafsiriwa kama mkataba, Unapewa Haki za Leseni kwa kuzingatia kukubali kwako masharti na hali hizi, na Mtoaji leseni anakupa haki hizo kwa kuzingatia faida ambazo Mtoaji leseni anapata kwa kufanya Nyenzo za Leseni zipatikane chini ya masharti na hali hizi.

## Sehemu ya 1 – Maanani.

a. __Nyenzo zilizobadilishwa__ zinamaanisha nyenzo zinazohusishwa na Hakimiliki na Haki Zinazofanana ambazo zinatokana na au zinategemea Nyenzo za Leseni na ambazo Nyenzo za Leseni zimewekwa tafsiri, kubadilishwa, kuandaliwa, kubadilishwa, au vinginevyo kubadilishwa kwa njia inayohitaji ruhusa chini ya Hakimiliki na Haki Zinazofanana zinazoshikiliwa na Mtoaji leseni. Kwa madhumuni ya Leseni hii ya Umma, ambapo Nyenzo za Leseni ni kazi ya muziki, utendaji, au rekodi ya sauti, Nyenzo zilizobadilishwa daima zinatengenezwa ambapo Nyenzo za Leseni zimeunganishwa kwa wakati na picha inayoenda.

b. __Leseni ya Mbadala__ inamaanisha leseni unayoomba kwa Hakimiliki na Haki Zinazofanana katika michango yako kwa Nyenzo zilizobadilishwa kulingana na masharti na hali za Leseni hii ya Umma.

c. __Hakimiliki na Haki Zinazofanana__ zinamaanisha hakimiliki na/au haki zinazofanana kwa karibu na hakimiliki ikiwa ni pamoja, bila kikomo, utendaji, matangazo, rekodi za sauti, na Haki za Hifadhidata za Sui Generis, bila kujali jinsi haki hizo zinavyotajwa au kuainishwa. Kwa madhumuni ya Leseni hii ya Umma, haki zilizotajwa katika Sehemu ya 2(b)(1)-(2) si Hakimiliki na Haki Zinazofanana.

d. __Hatua za Kiteknolojia za Ufanisi__ zinamaanisha hatua hizo ambazo, kwa kukosekana kwa mamlaka sahihi, haziwezi kupuuziliwa mbali chini ya sheria zinazotimiza wajibu chini ya Kifungu cha 11 cha Mkataba wa Hakimiliki wa WIPO ulioanzishwa tarehe 20 Desemba 1996, na/au makubaliano mengine ya kimataifa yanayofanana.

e. __Ubaguzi na Kikomo__ inamaanisha matumizi ya haki, biashara ya haki, na/au ubaguzi mwingine au kikomo kwa Hakimiliki na Haki Zinazofanana kinachotumika kwa matumizi yako ya Nyenzo za Leseni.

f. __Nyenzo za Leseni__ zinamaanisha kazi ya kisanii au ya kifasihi, hifadhidata, au nyenzo nyingine ambazo Mtoaji leseni alitumia Leseni hii ya Umma.

g. __Haki za Leseni__ zinamaanisha haki zinazokupwa chini ya masharti na hali za Leseni hii ya Umma, ambazo zinapunguzwa kwa Hakimiliki zote na Haki Zinazofanana zinazotumika kwa matumizi yako ya Nyenzo za Leseni na ambazo Mtoaji leseni ana mamlaka ya kutoa leseni.

h. __Mtoaji leseni__ inamaanisha mtu(mtu) au shirika(ta) linalotoa haki chini ya Leseni hii ya Umma.

i. __NonCommercial__ inamaanisha si kwa ajili ya faida ya kibiashara au malipo ya fedha. Kwa madhumuni ya Leseni hii ya Umma, kubadilishana Nyenzo za Leseni kwa nyenzo nyingine zinazohusishwa na Hakimiliki na Haki Zinazofanana kwa njia ya kushiriki faili za kidijitali au njia nyingine zinazofanana ni NonCommercial ikiwa hakuna malipo ya fedha yanayohusiana na kubadilishana.

j. __Shiriki__ inamaanisha kutoa nyenzo kwa umma kwa njia yoyote au mchakato unaohitaji ruhusa chini ya Haki za Leseni, kama vile uzalishaji, kuonyesha hadharani, utendaji wa hadharani, usambazaji, usambazaji, mawasiliano, au uagizaji, na kufanya nyenzo zipatikane kwa umma ikiwa ni pamoja na njia ambazo wanachama wa umma wanaweza kufikia nyenzo kutoka mahali na wakati waliouchagua binafsi.

k. __Haki za Hifadhidata za Sui Generis__ zinamaanisha haki nyingine isipokuwa hakimiliki zinazotokana na Maagizo 96/9/EC ya Bunge la Ulaya na Baraza la 11 Machi 1996 kuhusu ulinzi wa kisheria wa hifadhidata, kama ilivyorekebishwa na/au kufuatwa, pamoja na haki nyingine zinazofanana mahali popote duniani.

l. __Wewe__ inamaanisha mtu au shirika linalotumia Haki za Leseni chini ya Leseni hii ya Umma. Wewe ina maana inayolingana.

## Sehemu ya 2 – Muktadha.

a. ___Utoaji wa leseni.___

1. Kwa kuzingatia masharti na hali za Leseni hii ya Umma, Mtoaji leseni kwa hapa anakupa leseni ya kimataifa, isiyo na malipo, isiyoweza kuhamishwa, isiyo ya kipekee, isiyoweza kubadilishwa ya kutumia Haki za Leseni katika Nyenzo za Leseni ili:

A. kuzalisha na Kushiriki Nyenzo za Leseni, kwa ujumla au sehemu, kwa madhumuni ya NonCommercial pekee; na

B. kutengeneza, kuzalisha, na Kushiriki Nyenzo zilizobadilishwa kwa madhumuni ya NonCommercial pekee.

2. __Ubaguzi na Kikomo.__ Ili kuepusha shaka, ambapo Ubaguzi na Kikomo vinatumika kwa matumizi yako, Leseni hii ya Umma haitatumika, na huna haja ya kufuata masharti na hali zake.

3. __Muda.__ Muda wa Leseni hii ya Umma umeelezwa katika Sehemu ya 6(a).

4. __Vyombo na muundo; mabadiliko ya kiufundi yanayoruhusiwa.__ Mtoaji leseni anakupa ruhusa ya kutumia Haki za Leseni katika vyombo vyote na muundo iwe sasa inajulikana au itakayotengenezwa baadaye, na kufanya mabadiliko ya kiufundi yanayohitajika ili kufanya hivyo. Mtoaji leseni anakanusha na/au anakubali kutoshughulikia haki yoyote au mamlaka ya kukuzuia kufanya mabadiliko ya kiufundi yanayohitajika kutumia Haki za Leseni, ikiwa ni pamoja na mabadiliko ya kiufundi yanayohitajika kupita Hatua za Kiteknolojia za Ufanisi. Kwa madhumuni ya Leseni hii ya Umma, kufanya tu mabadiliko yanayoruhusiwa na Sehemu hii 2(a)(4) kamwe hakuzalishi Nyenzo zilizobadilishwa.

5. __Wapokeaji wa chini.__

A. __Ofa kutoka kwa Mtoaji leseni – Nyenzo za Leseni.__ Kila mpokeaji wa Nyenzo za Leseni moja kwa moja anapata ofa kutoka kwa Mtoaji leseni kutumia Haki za Leseni chini ya masharti na hali za Leseni hii ya Umma.

B. __Hakuna vizuizi vya chini.__ Huwezi kutoa au kuweka masharti au hali yoyote ya ziada au tofauti, au kutekeleza Hatua za Kiteknolojia za Ufanisi kwa Nyenzo za Leseni ikiwa kufanya hivyo kunakataza matumizi ya Haki za Leseni na mpokeaji yeyote wa Nyenzo za Leseni.

6. __Hakuna uthibitisho.__ Hakuna kitu katika Leseni hii ya Umma kinachounda au kinaweza kutafsiriwa kama ruhusa ya kudai au kuashiria kwamba wewe ni, au kwamba matumizi yako ya Nyenzo za Leseni ni, kuhusishwa na, au kudhaminiwa, kuungwa mkono, au kutolewa hadhi rasmi na, Mtoaji leseni au wengine waliotajwa kupokea sifa kama ilivyoelezwa katika Sehemu ya 3(a)(1)(A)(i).

b. ___Haki nyingine.___

1. Haki za maadili, kama haki ya uadilifu, hazijatumika chini ya Leseni hii ya Umma, wala matangazo, faragha, na/au haki nyingine za kibinafsi zinazofanana; hata hivyo, kadri inavyowezekana, Mtoaji leseni anakanusha na/au anakubali kutoshughulikia haki hizo zinazoshikiliwa na Mtoaji leseni kwa kiwango kidogo kinachohitajika ili kukuwezesha kutumia Haki za Leseni, lakini si vinginevyo.

2. Haki za patent na alama za biashara hazijatumika chini ya Leseni hii ya Umma.

3. Kadri inavyowezekana, Mtoaji leseni anakanusha haki yoyote ya kukusanya malipo kutoka kwako kwa matumizi ya Haki za Leseni, iwe moja kwa moja au kupitia shirika la kukusanya chini ya mpango wowote wa leseni wa hiari au wa kuondolewa. Katika hali nyingine zote Mtoaji leseni anahifadhi haki yoyote ya kukusanya malipo hayo, ikiwa ni pamoja na wakati Nyenzo za Leseni zinapotumiwa tofauti na madhumuni ya NonCommercial.

## Sehemu ya 3 – Masharti ya Leseni.

Matumizi yako ya Haki za Leseni yanategemea masharti yafuatayo.

a. ___Sifa.___

1. Ikiwa Unashiriki Nyenzo za Leseni (ikiwemo katika mfumo uliobadilishwa), lazima:

A. uweke yafuatayo ikiwa inatolewa na Mtoaji leseni pamoja na Nyenzo za Leseni:

i. utambulisho wa muumba(muumbaji) wa Nyenzo za Leseni na wengine wote waliotajwa kupokea sifa, kwa njia yoyote inayofaa iliyotolewa na Mtoaji leseni (ikiwemo kwa jina la utani ikiwa imetajwa);

ii. tangazo la hakimiliki;

iii. tangazo linalorejelea Leseni hii ya Umma;

iv. tangazo linalorejelea kukana dhamana;

v. URI au kiungo cha Nyenzo za Leseni kadri inavyowezekana;

B. onyesha ikiwa umebadilisha Nyenzo za Leseni na uweke alama ya mabadiliko yoyote ya awali; na

C. onyesha Nyenzo za Leseni zimepewa leseni chini ya Leseni hii ya Umma, na ujumuishe maandiko ya, au URI au kiungo cha, Leseni hii ya Umma.

2. Unaweza kutimiza masharti katika Sehemu 3(a)(1) kwa njia yoyote inayofaa kulingana na vyombo, njia, na muktadha ambao Unashiriki Nyenzo za Leseni. Kwa mfano, inaweza kuwa na maana kutoa URI au kiungo kwa rasilimali inayojumuisha habari zinazohitajika.

3. Ikiwa inahitajika na Mtoaji leseni, lazima uondoe taarifa yoyote inayohitajika na Sehemu 3(a)(1)(A) kadri inavyowezekana.

4. Ikiwa Unashiriki Nyenzo zilizobadilishwa unazozalisha, Leseni ya Mbadala unayoomba haipaswi kuzuia wapokeaji wa Nyenzo zilizobadilishwa kufuata Leseni hii ya Umma.

## Sehemu ya 4 – Haki za Hifadhidata za Sui Generis.

Pale ambapo Haki za Leseni zinajumuisha Haki za Hifadhidata za Sui Generis zinazotumika kwa matumizi yako ya Nyenzo za Leseni:

a. ili kuepusha shaka, Sehemu 2(a)(1) inakupa haki ya kutoa, kutumia tena, kuzalisha, na Kushiriki yote au sehemu kubwa ya maudhui ya hifadhidata kwa madhumuni ya NonCommercial pekee;

b. ikiwa unajumuisha yote au sehemu kubwa ya maudhui ya hifadhidata katika hifadhidata ambayo una Haki za Hifadhidata za Sui Generis, basi hifadhidata ambayo una Haki za Hifadhidata za Sui Generis (lakini si maudhui yake binafsi) ni Nyenzo zilizobadilishwa; na

c. lazima ufuate masharti katika Sehemu 3(a) ikiwa Unashiriki yote au sehemu kubwa ya maudhui ya hifadhidata.

Ili kuepusha shaka, Sehemu hii 4 inakamilisha na haitabadilisha wajibu wako chini ya Leseni hii ya Umma ambapo Haki za Leseni zinajumuisha Hakimiliki na Haki Zinazofanana nyingine.

## Sehemu ya 5 – Kukana Dhamana na Kikomo cha Dhima.

a. __Ili kuepusha shaka, isipokuwa pale ambapo Mtoaji leseni amechukua hatua tofauti, kadri inavyowezekana, Mtoaji leseni anatoa Nyenzo za Leseni kama ilivyo na inapatikana, na haitoi uwakilishi au dhamana yoyote ya aina yoyote kuhusu Nyenzo za Leseni, iwe wazi, iliyofichwa, ya kisheria, au nyingine. Hii inajumuisha, bila kikomo, dhamana za umiliki, biashara, kufaa kwa madhumuni maalum, kutovunja, ukosefu wa kasoro za siri au nyingine, usahihi, au uwepo au ukosefu wa makosa, iwe yanajulikana au yanapatikana. Pale ambapo kukana dhamana hakuruhusiwi kwa sehemu au kwa sehemu, kukana huku hakutaweza kutumika kwako.__

b. __Kadri inavyowezekana, kwa hali yoyote Mtoaji leseni hatakuwa na dhima kwako kwa nadharia yoyote ya kisheria (ikiwemo, bila kikomo, uzembe) au vinginevyo kwa hasara yoyote ya moja kwa moja, maalum, isiyo ya moja kwa moja, ya bahati mbaya, ya matokeo, ya adhabu, ya mfano, au hasara nyingine, gharama, matumizi, au uharibifu unaotokana na Leseni hii ya Umma au matumizi ya Nyenzo za Leseni, hata kama Mtoaji leseni ameshawishiwa kuhusu uwezekano wa hasara hizo, gharama, matumizi, au uharibifu. Pale ambapo kikomo cha dhima hakuruhusiwi kwa sehemu au kwa sehemu, kikomo hiki hakiwezi kutumika kwako.__

c. Kukana dhamana na kikomo cha dhima kilichotolewa hapo juu kitatafsiriwa kwa njia ambayo, kadri inavyowezekana, inakaribia kukana kabisa na kuondoa dhima zote.

## Sehemu ya 6 – Muda na Kuondolewa.

a. Leseni hii ya Umma inatumika kwa muda wa Hakimiliki na Haki Zinazofanana zinazotolewa hapa. Hata hivyo, ikiwa hutatii Leseni hii ya Umma, basi haki zako chini ya Leseni hii ya Umma zitakoma moja kwa moja.

b. Pale ambapo haki yako ya kutumia Nyenzo za Leseni imekatishwa chini ya Sehemu 6(a), inarejeshwa:

1. moja kwa moja kuanzia tarehe ukiukaji unapotatuliwa, ikiwa umetatuliwa ndani ya siku 30 za kugundua ukiukaji; au

2. kwa kurejeshwa wazi na Mtoaji leseni.

Ili kuepusha shaka, Sehemu hii 6(b) haiathiri haki yoyote ambayo Mtoaji leseni anaweza kuwa nayo kutafuta fidia kwa ukiukaji wako wa Leseni hii ya Umma.

c. Ili kuepusha shaka, Mtoaji leseni anaweza pia kutoa Nyenzo za Leseni chini ya masharti au hali tofauti au kuacha kusambaza Nyenzo za Leseni wakati wowote; hata hivyo, kufanya hivyo hakutakoma Leseni hii ya Umma.

d. Sehemu za 1, 5, 6, 7, na 8 zinaendelea hata baada ya kuondolewa kwa Leseni hii ya Umma.

## Sehemu ya 7 – Masharti na Hali Nyingine.

a. Mtoaji leseni hatakuwa na wajibu wa masharti au hali yoyote ya ziada au tofauti iliyowasilishwa na Wewe isipokuwa ikubaliwe wazi.

b. Mpango wowote, kuelewana, au makubaliano kuhusu Nyenzo za Leseni ambayo hayajatolewa hapa ni tofauti na huru kutoka kwa masharti na hali za Leseni hii ya Umma.

## Sehemu ya 8 – Tafsiri.

a. Ili kuepusha shaka, Leseni hii ya Umma haitapunguza, kuzuia, kukandamiza, au kuweka masharti yoyote juu ya matumizi yoyote ya Nyenzo za Leseni ambayo yanaweza kufanywa kisheria bila ruhusa chini ya Leseni hii ya Umma.

b. Kadri inavyowezekana, ikiwa kifungu chochote cha Leseni hii ya Umma kinachukuliwa kuwa hakiwezi kutekelezeka, kitarejeshwa moja kwa moja kwa kiwango cha chini kinachohitajika ili kufanya kiweze kutekelezeka. Ikiwa kifungu hakiwezi kurekebishwa, kitakatwa kutoka Leseni hii ya Umma bila kuathiri utekelezaji wa masharti na hali zilizobaki.

c. Hakuna masharti au hali yoyote ya Leseni hii ya Umma itakayoondolewa na hakuna kushindwa kutii kutakubaliwa isipokuwa ikubaliwe wazi na Mtoaji leseni.

d. Hakuna kitu katika Leseni hii ya Umma kinachounda au kinaweza kutafsiriwa kama kikomo, au kuondolewa, kwa haki na kinga zozote zinazotumika kwa Mtoaji leseni au Wewe, ikiwa ni pamoja na kutoka kwa michakato ya kisheria ya mamlaka yoyote au mamlaka.
```
Creative Commons is not a party to its public licenses. Notwithstanding, Creative Commons may elect to apply one of its public licenses to material it publishes and in those instances will be considered the “Licensor.” Except for the limited purpose of indicating that material is shared under a Creative Commons public license or as otherwise permitted by the Creative Commons policies published at [creativecommons.org/policies](http://creativecommons.org/policies), Creative Commons does not authorize the use of the trademark “Creative Commons” or any other trademark or logo of Creative Commons without its prior written consent including, without limitation, in connection with any unauthorized modifications to any of its public licenses or any other arrangements, understandings, or agreements concerning use of licensed material. For the avoidance of doubt, this paragraph does not form part of the public licenses.

Creative Commons may be contacted at [creativecommons.org](http://creativecommons.org/).
```
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
