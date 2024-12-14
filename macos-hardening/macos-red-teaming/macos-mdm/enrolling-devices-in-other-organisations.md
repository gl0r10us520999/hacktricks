# Kujiandikisha Vifaa katika Mashirika Mengine

{% hint style="success" %}
Jifunze & fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze & fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Utangulizi

Kama [**ilivyosemwa awali**](./#what-is-mdm-mobile-device-management)**,** ili kujaribu kujiandikisha kifaa katika shirika **ni nambari ya serial pekee inayomilikiwa na Shirika hilo inahitajika**. Mara kifaa kinapojisajili, mashirika kadhaa yataweka data nyeti kwenye kifaa kipya: vyeti, programu, nywila za WiFi, mipangilio ya VPN [na kadhalika](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Hivyo, hii inaweza kuwa njia hatari ya kuingia kwa washambuliaji ikiwa mchakato wa usajili haujalindwa ipasavyo.

**Ifuatayo ni muhtasari wa utafiti [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe). Angalia kwa maelezo zaidi ya kiufundi!**

## Muonekano wa Uchambuzi wa DEP na MDM Binary

Utafiti huu unachunguza binaries zinazohusiana na Programu ya Kujiandikisha Vifaa (DEP) na Usimamizi wa Vifaa vya Mkononi (MDM) kwenye macOS. Vipengele muhimu ni pamoja na:

- **`mdmclient`**: Inawasiliana na seva za MDM na kuanzisha ukaguzi wa DEP kwenye toleo la macOS kabla ya 10.13.4.
- **`profiles`**: Inasimamia Profaili za Mipangilio, na kuanzisha ukaguzi wa DEP kwenye toleo la macOS 10.13.4 na baadaye.
- **`cloudconfigurationd`**: Inasimamia mawasiliano ya DEP API na inapata profaili za Kujiandikisha Vifaa.

Ukaguzi wa DEP unatumia kazi za `CPFetchActivationRecord` na `CPGetActivationRecord` kutoka kwa mfumo wa ndani wa Profaili za Mipangilio ili kupata Rekodi ya Uanzishaji, huku `CPFetchActivationRecord` ikishirikiana na `cloudconfigurationd` kupitia XPC.

## Uhandisi wa Kurudi wa Protokali ya Tesla na Mpango wa Absinthe

Ukaguzi wa DEP unahusisha `cloudconfigurationd` kutuma payload ya JSON iliyosainiwa na iliyosimbwa kwa _iprofiles.apple.com/macProfile_. Payload hiyo inajumuisha nambari ya serial ya kifaa na hatua "RequestProfileConfiguration". Mpango wa usimbaji unaotumika unajulikana kwa ndani kama "Absinthe". Kufichua mpango huu ni ngumu na kunahusisha hatua nyingi, ambazo zilisababisha kuchunguza njia mbadala za kuingiza nambari za serial zisizo za kawaida katika ombi la Rekodi ya Uanzishaji.

## Kuweka Proxy kwa Maombi ya DEP

Jaribio la kukamata na kubadilisha maombi ya DEP kwa _iprofiles.apple.com_ kwa kutumia zana kama Charles Proxy lilikwamishwa na usimbaji wa payload na hatua za usalama za SSL/TLS. Hata hivyo, kuwezesha usanidi wa `MCCloudConfigAcceptAnyHTTPSCertificate` kunaruhusu kupita uthibitishaji wa cheti cha seva, ingawa asili ya payload iliyosimbwa bado inazuia kubadilisha nambari ya serial bila funguo ya kufichua.

## Kuweka Vifaa vya Mfumo Vinavyoshirikiana na DEP

Kuweka vifaa vya mfumo kama `cloudconfigurationd` kunahitaji kuzima Ulinzi wa Uadilifu wa Mfumo (SIP) kwenye macOS. Ikiwa SIP imezimwa, zana kama LLDB zinaweza kutumika kuunganishwa na michakato ya mfumo na labda kubadilisha nambari ya serial inayotumika katika mwingiliano wa DEP API. Njia hii inpreferiwa kwani inakwepa changamoto za haki na saini ya msimbo.

**Kutatua Uhandisi wa Binary:**
Kubadilisha payload ya ombi la DEP kabla ya serialization ya JSON katika `cloudconfigurationd` ilionekana kuwa na ufanisi. Mchakato huo ulijumuisha:

1. Kuunganisha LLDB na `cloudconfigurationd`.
2. Kutafuta mahali ambapo nambari ya serial ya mfumo inapatikana.
3. Kuingiza nambari ya serial isiyo ya kawaida kwenye kumbukumbu kabla ya payload kusimbwa na kutumwa.

Njia hii iliruhusu kupata profaili kamili za DEP kwa nambari za serial zisizo za kawaida, ikionyesha udhaifu wa uwezekano.

### Kuandaa Uhandisi kwa Python

Mchakato wa unyakuzi ulifanywa kiotomatiki kwa kutumia Python na API ya LLDB, na kufanya iwezekanavyo kuingiza nambari za serial zisizo za kawaida na kupata profaili zinazolingana za DEP.

### Athari Zinazoweza Kutokana na Udhaifu wa DEP na MDM

Utafiti huo ulionyesha wasiwasi mkubwa wa usalama:

1. **Ufunuo wa Taarifa**: Kwa kutoa nambari ya serial iliyosajiliwa na DEP, taarifa nyeti za shirika zilizomo katika profaili ya DEP zinaweza kupatikana.
{% hint style="success" %}
Jifunze & fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze & fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
