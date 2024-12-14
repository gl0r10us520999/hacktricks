# macOS Office Sandbox Bypasses

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

### Word Sandbox bypass via Launch Agents

Programu inatumia **Sandbox maalum** kwa kutumia haki **`com.apple.security.temporary-exception.sbpl`** na sandbox hii maalum inaruhusu kuandika faili popote mradi jina la faili lianze na `~$`: `(require-any (require-all (vnode-type REGULAR-FILE) (regex #"(^|/)~$[^/]+$")))`

Hivyo, kutoroka ilikuwa rahisi kama **kuandika `plist`** LaunchAgent katika `~/Library/LaunchAgents/~$escape.plist`.

Check the [**original report here**](https://www.mdsec.co.uk/2018/08/escaping-the-sandbox-microsoft-office-on-macos/).

### Word Sandbox bypass via Login Items and zip

Kumbuka kwamba kutoka kutoroka kwa kwanza, Word inaweza kuandika faili za kawaida ambazo jina lake linaanza na `~$` ingawa baada ya patch ya vuln iliyopita haikuwezekana kuandika katika `/Library/Application Scripts` au katika `/Library/LaunchAgents`.

Iligundulika kwamba kutoka ndani ya sandbox inawezekana kuunda **Kitu cha Kuingia** (programu ambazo zitatekelezwa wakati mtumiaji anapoingia). Hata hivyo, programu hizi **hazitaweza kutekelezwa isipokuwa** zime **notarized** na **haiwezekani kuongeza args** (hivyo huwezi tu kuendesha shell ya kinyume kwa kutumia **`bash`**).

Kutoka kwa kutoroka kwa Sandbox iliyopita, Microsoft ilizima chaguo la kuandika faili katika `~/Library/LaunchAgents`. Hata hivyo, iligundulika kwamba ikiwa utaweka **faili ya zip kama Kitu cha Kuingia** `Archive Utility` itachambua tu **zip** katika eneo lake la sasa. Hivyo, kwa sababu kwa kawaida folda `LaunchAgents` kutoka `~/Library` haijaundwa, ilikuwa inawezekana **kuzipa plist katika `LaunchAgents/~$escape.plist`** na **kuiweka** faili ya zip katika **`~/Library`** ili wakati wa kufungua itafikia mahali pa kudumu.

Check the [**original report here**](https://objective-see.org/blog/blog\_0x4B.html).

### Word Sandbox bypass via Login Items and .zshenv

(Kumbuka kwamba kutoka kutoroka kwa kwanza, Word inaweza kuandika faili za kawaida ambazo jina lake linaanza na `~$`).

Hata hivyo, mbinu iliyopita ilikuwa na kikomo, ikiwa folda **`~/Library/LaunchAgents`** ipo kwa sababu programu nyingine iliiunda, ingekuwa na makosa. Hivyo, mnyororo tofauti wa Kitu cha Kuingia uligundulika kwa hili.

Mshambuliaji angeweza kuunda faili **`.bash_profile`** na **`.zshenv`** zikiwa na payload ya kutekeleza na kisha kuzipa na **kuandika zip katika** folda ya mtumiaji wa wahanga: **`~/~$escape.zip`**.

Kisha, ongeza faili ya zip kwenye **Kitu cha Kuingia** na kisha programu ya **`Terminal`**. Wakati mtumiaji anapoingia tena, faili ya zip itachambuliwa katika faili za watumiaji, ikipunguza **`.bash_profile`** na **`.zshenv`** na hivyo, terminal itatekeleza moja ya faili hizi (kulingana na ikiwa bash au zsh inatumika).

Check the [**original report here**](https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c).

### Word Sandbox Bypass with Open and env variables

Kutoka kwa michakato ya sandboxed bado inawezekana kuita michakato mingine kwa kutumia **`open`** utility. Zaidi ya hayo, michakato hii itakimbia **ndani ya sandbox yao wenyewe**.

Iligundulika kwamba utility ya open ina chaguo la **`--env`** kuendesha programu na **mabadiliko maalum**. Hivyo, ilikuwa inawezekana kuunda **faili ya `.zshenv`** ndani ya folda **ndani** ya **sandbox** na kutumia `open` na `--env` kuweka **`HOME` variable** kwa folda hiyo ikifungua programu hiyo ya `Terminal`, ambayo itatekeleza faili ya `.zshenv` (kwa sababu fulani ilikuwa pia inahitajika kuweka variable `__OSINSTALL_ENVIROMENT`).

Check the [**original report here**](https://perception-point.io/blog/technical-analysis-of-cve-2021-30864/).

### Word Sandbox Bypass with Open and stdin

Utility ya **`open`** pia ilisaidia param ya **`--stdin`** (na baada ya kutoroka kwa awali haikuwezekana tena kutumia `--env`).

Jambo ni kwamba hata kama **`python`** ilitiwa saini na Apple, **haitatekeleza** script yenye sifa ya **`quarantine`**. Hata hivyo, ilikuwa inawezekana kupitisha script kutoka stdin hivyo haitakagua ikiwa ilikuwa imewekwa karantini au la:&#x20;

1. Drop a **`~$exploit.py`** file with arbitrary Python commands.
2. Run _open_ **`–stdin='~$exploit.py' -a Python`**, which runs the Python app with our dropped file serving as its standard input. Python happily runs our code, and since it’s a child process of _launchd_, it isn’t bound to Word’s sandbox rules.

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
