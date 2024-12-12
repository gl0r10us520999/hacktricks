# Orodha - Kuinua Haki ya Linux

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Jiunge na [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) server kuwasiliana na hackers wenye uzoefu na wawindaji wa bug bounty!

**Maoni ya Hacking**\
Shiriki na maudhui yanayoangazia msisimko na changamoto za hacking

**Habari za Hack kwa Wakati Halisi**\
Baki na habari za hivi punde katika ulimwengu wa hacking kupitia habari na maoni ya wakati halisi

**Matangazo ya Hivi Punde**\
Baki na habari kuhusu bug bounties mpya zinazozinduliwa na sasisho muhimu za jukwaa

**Jiunge nasi kwenye** [**Discord**](https://discord.com/invite/N3FrSbmwdy) na uanze kushirikiana na hackers bora leo!

### **Zana bora ya kutafuta vigezo vya kuinua haki za ndani za Linux:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [Taarifa za Mfumo](privilege-escalation/#system-information)

* [ ] Pata **taarifa za OS**
* [ ] Angalia [**PATH**](privilege-escalation/#path), kuna **folda inayoweza kuandikwa**?
* [ ] Angalia [**env variables**](privilege-escalation/#env-info), kuna maelezo nyeti yoyote?
* [ ] Tafuta [**kernel exploits**](privilege-escalation/#kernel-exploits) **ukitumia scripts** (DirtyCow?)
* [ ] **Angalia** kama [**toleo la sudo** lina udhaifu](privilege-escalation/#sudo-version)
* [ ] [**Dmesg** uthibitisho wa saini umeshindwa](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] Zaidi ya mfumo enum ([tarehe, takwimu za mfumo, taarifa za cpu, printers](privilege-escalation/#more-system-enumeration))
* [ ] [**Tathmini zaidi ya ulinzi**](privilege-escalation/#enumerate-possible-defenses)

### [Diski](privilege-escalation/#drives)

* [ ] **Orodhesha diski zilizowekwa**
* [ ] **Kuna diski isiyowekwa?**
* [ ] **Kuna creds katika fstab?**

### [**Programu Zilizowekwa**](privilege-escalation/#installed-software)

* [ ] **Angalia** [**programu muhimu**](privilege-escalation/#useful-software) **zilizowekwa**
* [ ] **Angalia** [**programu zenye udhaifu**](privilege-escalation/#vulnerable-software-installed) **zilizowekwa**

### [Mchakato](privilege-escalation/#processes)

* [ ] Je, kuna **programu isiyojulikana inayoendesha**?
* [ ] Je, kuna programu inayoendesha na **haki zaidi kuliko inavyopaswa kuwa**?
* [ ] Tafuta **exploits za michakato inayoendesha** (hasa toleo linaloendesha).
* [ ] Je, unaweza **kubadilisha binary** ya mchakato wowote unaoendesha?
* [ ] **Fuatilia michakato** na angalia kama kuna mchakato wa kuvutia unaoendesha mara kwa mara.
* [ ] Je, unaweza **kusoma** baadhi ya **kumbukumbu za mchakato** (ambapo nywila zinaweza kuhifadhiwa)?

### [Kazi za Ratiba/Cron?](privilege-escalation/#scheduled-jobs)

* [ ] Je, [**PATH**](privilege-escalation/#cron-path) inabadilishwa na cron fulani na unaweza **kuandika** ndani yake?
* [ ] Kuna [**wildcard**](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection) katika kazi ya cron?
* [ ] Baadhi ya [**script inayoweza kubadilishwa**](privilege-escalation/#cron-script-overwriting-and-symlink) inatekelezwa au iko ndani ya **folda inayoweza kubadilishwa**?
* [ ] Je, umepata kuwa baadhi ya **script** zinaweza au zinafanywa [**kutekelezwa mara kwa mara**](privilege-escalation/#frequent-cron-jobs)? (kila dakika 1, 2 au 5)

### [Huduma](privilege-escalation/#services)

* [ ] Kuna **faili ya .service inayoweza kuandikwa**?
* [ ] Kuna **binary inayoweza kuandikwa** inayotekelezwa na **huduma**?
* [ ] Kuna **folda inayoweza kuandikwa katika mfumo wa PATH**?

### [Wakati](privilege-escalation/#timers)

* [ ] Kuna **timer inayoweza kuandikwa**?

### [Sockets](privilege-escalation/#sockets)

* [ ] Kuna **faili ya .socket inayoweza kuandikwa**?
* [ ] Je, unaweza **kuwasiliana na socket yoyote**?
* [ ] **HTTP sockets** zikiwa na habari za kuvutia?

### [D-Bus](privilege-escalation/#d-bus)

* [ ] Je, unaweza **kuwasiliana na D-Bus yoyote**?

### [Mtandao](privilege-escalation/#network)

* [ ] Tathmini mtandao ili kujua uko wapi
* [ ] **Port zilizofunguliwa ambazo huwezi kufikia kabla** ya kupata shell ndani ya mashine?
* [ ] Je, unaweza **kunusa trafiki** ukitumia `tcpdump`?

### [Watumiaji](privilege-escalation/#users)

* [ ] Orodha ya watumiaji/mikundi **kuhesabu**
* [ ] Je, una **UID kubwa sana**? Je, **mashine** ina **udhaifu**?
* [ ] Je, unaweza [**kuinua haki kwa sababu ya kundi**](privilege-escalation/interesting-groups-linux-pe/) unalotegemea?
* [ ] **Data za Clipboard**?
* [ ] Sera ya Nywila?
* [ ] Jaribu **kutumia** kila **nywila inayojulikana** uliyogundua awali kuingia **na kila** **mtumiaji** anayeweza. Jaribu kuingia pia bila nywila.

### [PATH inayoweza kuandikwa](privilege-escalation/#writable-path-abuses)

* [ ] Ikiwa una **haki za kuandika juu ya folda fulani katika PATH** unaweza kuwa na uwezo wa kuinua haki

### [SUDO na amri za SUID](privilege-escalation/#sudo-and-suid)

* [ ] Je, unaweza kutekeleza **amri yoyote na sudo**? Je, unaweza kuitumia KUSOMA, KUANDIKA au KUTEKELEZA chochote kama root? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] Je, kuna **binary ya SUID inayoweza kutumika**? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] Je, [**amri za sudo** **zimepunguziliwa** na **path**? Je, unaweza **kuzidi** vizuizi](privilege-escalation/#sudo-execution-bypassing-paths)?
* [ ] [**Sudo/SUID binary bila path iliyotajwa**](privilege-escalation/#sudo-command-suid-binary-without-command-path)?
* [ ] [**SUID binary ikitaja path**](privilege-escalation/#suid-binary-with-command-path)? Pita
* [ ] [**LD\_PRELOAD vuln**](privilege-escalation/#ld_preload)
* [ ] [**Ukosefu wa maktaba ya .so katika binary ya SUID**](privilege-escalation/#suid-binary-so-injection) kutoka folda inayoweza kuandikwa?
* [ ] [**Tokens za SUDO zinazopatikana**](privilege-escalation/#reusing-sudo-tokens)? [**Je, unaweza kuunda token ya SUDO**](privilege-escalation/#var-run-sudo-ts-less-than-username-greater-than)?
* [ ] Je, unaweza [**kusoma au kubadilisha faili za sudoers**](privilege-escalation/#etc-sudoers-etc-sudoers-d)?
* [ ] Je, unaweza [**kubadilisha /etc/ld.so.conf.d/**](privilege-escalation/#etc-ld-so-conf-d)?
* [ ] [**Amri ya OpenBSD DOAS**](privilege-escalation/#doas)

### [Uwezo](privilege-escalation/#capabilities)

* [ ] Je, binary yoyote ina **uwezo usiotarajiwa**?

### [ACLs](privilege-escalation/#acls)

* [ ] Je, faili yoyote ina **ACL isiyotegemewa**?

### [Sessions za Shell Zilizofunguliwa](privilege-escalation/#open-shell-sessions)

* [ ] **screen**
* [ ] **tmux**

### [SSH](privilege-escalation/#ssh)

* [ ] **Debian** [**OpenSSL Predictable PRNG - CVE-2008-0166**](privilege-escalation/#debian-openssl-predictable-prng-cve-2008-0166)
* [ ] [**SSH Thamani za usanidi za Kuvutia**](privilege-escalation/#ssh-interesting-configuration-values)

### [Faili za Kuvutia](privilege-escalation/#interesting-files)

* [ ] **Faili za Profaili** - Soma data nyeti? Andika kwa privesc?
* [ ] **faili za passwd/shadow** - Soma data nyeti? Andika kwa privesc?
* [ ] **Angalia folda zinazovutia kwa kawaida** kwa data nyeti
* [ ] **Mahali/Picha za Ajabu,** unaweza kuwa na ufikiaji au kubadilisha faili zinazoweza kutekelezwa
* [ ] **Imebadilishwa** katika dakika za mwisho
* [ ] **Faili za Sqlite DB**
* [ ] **Faili zilizofichwa**
* [ ] **Script/Binaries katika PATH**
* [ ] **Faili za Mtandao** (nywila?)
* [ ] **Nakala za Hifadhi**?
* [ ] **Faili zinazojulikana ambazo zina nywila**: Tumia **Linpeas** na **LaZagne**
* [ ] **Utafutaji wa Kawaida**

### [**Faili Zinazoweza Kuandikwa**](privilege-escalation/#writable-files)

* [ ] **Badilisha maktaba ya python** ili kutekeleza amri zisizo za kawaida?
* [ ] Je, unaweza **kubadilisha faili za log**? **Logtotten** exploit
* [ ] Je, unaweza **kubadilisha /etc/sysconfig/network-scripts/**? Centos/Redhat exploit
* [ ] Je, unaweza [**kuandika katika faili za ini, int.d, systemd au rc.d**](privilege-escalation/#init-init-d-systemd-and-rc-d)?

### [**Mbinu Nyingine**](privilege-escalation/#other-tricks)

* [ ] Je, unaweza [**kudhulumu NFS ili kuinua haki**](privilege-escalation/#nfs-privilege-escalation)?
* [ ] Je, unahitaji [**kutoroka kutoka shell yenye vizuizi**](privilege-escalation/#escaping-from-restricted-shells)?

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Jiunge na [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) server kuwasiliana na hackers wenye uzoefu na wawindaji wa bug bounty!

**Maoni ya Hacking**\
Shiriki na maudhui yanayoangazia msisimko na changamoto za hacking

**Habari za Hack kwa Wakati Halisi**\
Baki na habari za hivi punde katika ulimwengu wa hacking kupitia habari na maoni ya wakati halisi

**Matangazo ya Hivi Punde**\
Baki na habari kuhusu bug bounties mpya zinazozinduliwa na sasisho muhimu za jukwaa

**Jiunge nasi kwenye** [**Discord**](https://discord.com/invite/N3FrSbmwdy) na uanze kushirikiana na hackers bora leo!

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
