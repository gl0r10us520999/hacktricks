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
{% endhint %}


# Kutambua binaries zilizopakwa

* **ukosefu wa nyuzi**: Ni kawaida kukutana na binaries zilizopakwa ambazo hazina karibu nyuzi yoyote
* Nyuzi nyingi **zisizotumika**: Pia, wakati malware inatumia aina fulani ya pakka ya kibiashara ni kawaida kukutana na nyuzi nyingi zisizo na marejeo. Hata kama nyuzi hizi zipo, hiyo haimaanishi kwamba binary haijapakwa.
* Unaweza pia kutumia zana kadhaa kujaribu kubaini ni pakka ipi ilitumika kupakia binary:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Lugha 2000](http://farrokhi.net/language/)

# Mapendekezo ya Msingi

* **Anza** kuchambua binary iliyopakwa **kutoka chini katika IDA na uende juu**. Unpackers huondoka mara tu msimbo ulioondolewa unapoondoka, hivyo ni vigumu kwa unpacker kuhamasisha utekelezaji kwa msimbo ulioondolewa mwanzoni.
* Tafuta **JMP's** au **CALLs** kwa **registers** au **mikoa** ya **kumbukumbu**. Pia tafuta **kazi zinazoweka hoja na mwelekeo wa anwani kisha kuita `retn`**, kwa sababu kurudi kwa kazi katika kesi hiyo kunaweza kuita anwani iliyowekwa tu kwenye stack kabla ya kuitwa.
* Weka **breakpoint** kwenye `VirtualAlloc` kwani hii inatoa nafasi katika kumbukumbu ambapo programu inaweza kuandika msimbo ulioondolewa. "enda kwa msimbo wa mtumiaji" au tumia F8 ili **kupata thamani ndani ya EAX** baada ya kutekeleza kazi na "**fuata anwani hiyo katika dump**". Hujui kama hiyo ndiyo mkoa ambapo msimbo ulioondolewa utaokolewa.
* **`VirtualAlloc`** ikiwa na thamani "**40**" kama hoja inamaanisha Soma+Andika+Tekeleza (msimbo fulani unaohitaji utekelezaji utaandikwa hapa).
* **Wakati wa kuondoa** msimbo ni kawaida kukutana na **simu kadhaa** kwa **operesheni za hesabu** na kazi kama **`memcopy`** au **`Virtual`**`Alloc`. Ikiwa unajikuta katika kazi ambayo kwa wazi inafanya tu operesheni za hesabu na labda `memcopy`, mapendekezo ni kujaribu **kupata mwisho wa kazi** (labda JMP au simu kwa register fulani) **au** angalau **simu kwa kazi ya mwisho** na uende kwa hiyo kwani msimbo si wa kuvutia.
* Wakati wa kuondoa msimbo **kumbuka** kila wakati unapo **badilisha mkoa wa kumbukumbu** kwani mabadiliko ya mkoa wa kumbukumbu yanaweza kuashiria **kuanza kwa msimbo wa kuondoa**. Unaweza kwa urahisi dump mkoa wa kumbukumbu ukitumia Process Hacker (mchakato --> mali --> kumbukumbu).
* Wakati wa kujaribu kuondoa msimbo njia nzuri ya **kujua kama tayari unafanya kazi na msimbo ulioondolewa** (hivyo unaweza tu kuudump) ni **kuangalia nyuzi za binary**. Ikiwa katika hatua fulani unafanya jump (labda kubadilisha mkoa wa kumbukumbu) na unagundua kwamba **nyuzi nyingi zaidi zimeongezwa**, basi unaweza kujua **unafanya kazi na msimbo ulioondolewa**.\
Hata hivyo, ikiwa pakka tayari ina nyuzi nyingi unaweza kuona ni nyuzi ngapi zina neno "http" na kuona ikiwa nambari hii inaongezeka.
* Unapodump executable kutoka mkoa wa kumbukumbu unaweza kurekebisha baadhi ya vichwa kwa kutumia [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases).

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
</details>
{% endhint %}
