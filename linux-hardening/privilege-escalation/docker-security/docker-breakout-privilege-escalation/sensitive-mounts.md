# Sensitive Mounts

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Ufunuo wa `/proc` na `/sys` bila kutengwa kwa majina sahihi huleta hatari kubwa za usalama, ikiwa ni pamoja na kuongezeka kwa uso wa shambulio na ufichuzi wa taarifa. Maktaba hizi zina faili nyeti ambazo, ikiwa zimepangwa vibaya au kufikiwa na mtumiaji asiyeidhinishwa, zinaweza kusababisha kutoroka kwa kontena, mabadiliko ya mwenyeji, au kutoa taarifa zinazosaidia mashambulizi zaidi. Kwa mfano, kuunganisha vibaya `-v /proc:/host/proc` kunaweza kupita ulinzi wa AppArmor kutokana na asili yake ya msingi wa njia, na kuacha `/host/proc` bila ulinzi.

**Unaweza kupata maelezo zaidi ya kila hatari inayoweza kutokea katika** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## procfs Vulnerabilities

### `/proc/sys`

Maktaba hii inaruhusu ufikiaji wa kubadilisha vigezo vya kernel, kawaida kupitia `sysctl(2)`, na ina subdirectories kadhaa za wasiwasi:

#### **`/proc/sys/kernel/core_pattern`**

* Imeelezwa katika [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Inaruhusu kufafanua programu ya kutekeleza wakati wa uzalishaji wa core-file na bytes 128 za kwanza kama hoja. Hii inaweza kusababisha utekelezaji wa msimbo ikiwa faili inaanza na bomba `|`.
*   **Mfano wa Kujaribu na Kutumia**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Jaribu ufikiaji wa kuandika
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Weka mpangaji maalum
sleep 5 && ./crash & # Trigger handler
```

#### **`/proc/sys/kernel/modprobe`**

* Imeelezwa katika [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Ina njia ya mpangaji wa moduli za kernel, inayotumika kwa kupakia moduli za kernel.
*   **Mfano wa Kuangalia Ufikiaji**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Angalia ufikiaji wa modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Imeelezwa katika [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Bendera ya kimataifa inayodhibiti ikiwa kernel inapaswa kujiingiza au kuanzisha OOM killer wakati hali ya OOM inatokea.

#### **`/proc/sys/fs`**

* Kulingana na [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), ina chaguzi na taarifa kuhusu mfumo wa faili.
* Ufikiaji wa kuandika unaweza kuwezesha mashambulizi mbalimbali ya kukataa huduma dhidi ya mwenyeji.

#### **`/proc/sys/fs/binfmt_misc`**

* Inaruhusu kujiandikisha kwa wakalimani wa muundo wa binary usio wa asili kulingana na nambari yao ya uchawi.
* Inaweza kusababisha kupanda kwa haki au ufikiaji wa root shell ikiwa `/proc/sys/fs/binfmt_misc/register` inaweza kuandikwa.
* Uthibitisho wa husika na maelezo:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Mafunzo ya kina: [Video link](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Wengine katika `/proc`

#### **`/proc/config.gz`**

* Inaweza kufichua usanidi wa kernel ikiwa `CONFIG_IKCONFIG_PROC` imewezeshwa.
* Inatumika kwa washambuliaji kubaini udhaifu katika kernel inayotumika.

#### **`/proc/sysrq-trigger`**

* Inaruhusu kuanzisha amri za Sysrq, ambayo inaweza kusababisha upya wa mfumo mara moja au hatua nyingine muhimu.
*   **Mfano wa Kuanzisha Upya Mwenyeji**:

```bash
echo b > /proc/sysrq-trigger # Inarejesha mwenyeji
```

#### **`/proc/kmsg`**

* Inafichua ujumbe wa buffer ya ring ya kernel.
* Inaweza kusaidia katika mashambulizi ya kernel, uvujaji wa anwani, na kutoa taarifa nyeti za mfumo.

#### **`/proc/kallsyms`**

* Inataja alama za kernel zilizotolewa na anwani zao.
* Muhimu kwa maendeleo ya mashambulizi ya kernel, hasa kwa kushinda KASLR.
* Taarifa za anwani zimepunguzika ikiwa `kptr_restrict` imewekwa kuwa `1` au `2`.
* Maelezo katika [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Inafanya kazi na kifaa cha kumbukumbu ya kernel `/dev/mem`.
* Kihistoria ilikuwa na udhaifu wa mashambulizi ya kupanda kwa haki.
* Zaidi kwenye [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Inawakilisha kumbukumbu ya kimwili ya mfumo katika muundo wa ELF core.
* Kusoma kunaweza kufichua maudhui ya kumbukumbu ya mfumo wa mwenyeji na kontena nyingine.
* Ukubwa mkubwa wa faili unaweza kusababisha matatizo ya kusoma au kuanguka kwa programu.
* Matumizi ya kina katika [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Kiolesura mbadala kwa `/dev/kmem`, kinawakilisha kumbukumbu ya virtual ya kernel.
* Inaruhusu kusoma na kuandika, hivyo kubadilisha moja kwa moja kumbukumbu ya kernel.

#### **`/proc/mem`**

* Kiolesura mbadala kwa `/dev/mem`, kinawakilisha kumbukumbu ya kimwili.
* Inaruhusu kusoma na kuandika, kubadilisha kumbukumbu yote kunahitaji kutatua anwani za virtual hadi kimwili.

#### **`/proc/sched_debug`**

* Inarudisha taarifa za kupanga mchakato, ikipita ulinzi wa PID namespace.
* Inafichua majina ya mchakato, IDs, na vitambulisho vya cgroup.

#### **`/proc/[pid]/mountinfo`**

* Inatoa taarifa kuhusu maeneo ya kuunganisha katika namespace ya kuunganisha ya mchakato.
* Inafichua eneo la `rootfs` ya kontena au picha.

### `/sys` Vulnerabilities

#### **`/sys/kernel/uevent_helper`**

* Inatumika kwa kushughulikia `uevents` za kifaa cha kernel.
* Kuandika kwenye `/sys/kernel/uevent_helper` kunaweza kutekeleza skripti zisizo na mipaka wakati wa kuanzishwa kwa `uevent`.
*   **Mfano wa Kutumia**: %%%bash

#### Inaunda payload

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Inapata njia ya mwenyeji kutoka OverlayFS mount kwa kontena

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Inapanga uevent\_helper kwa msaidizi mbaya

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Inasababisha uevent

echo change > /sys/class/mem/null/uevent

#### Inasoma matokeo

cat /output %%%

#### **`/sys/class/thermal`**

* Inadhibiti mipangilio ya joto, ambayo inaweza kusababisha mashambulizi ya DoS au uharibifu wa kimwili.

#### **`/sys/kernel/vmcoreinfo`**

* Inafichua anwani za kernel, ambayo inaweza kuhatarisha KASLR.

#### **`/sys/kernel/security`**

* Inahifadhi kiolesura cha `securityfs`, kinachoruhusu usanidi wa Moduli za Usalama za Linux kama AppArmor.
* Ufikiaji unaweza kuwezesha kontena kuzima mfumo wake wa MAC.

#### **`/sys/firmware/efi/vars` na `/sys/firmware/efi/efivars`**

* Inafichua violesura vya kuingiliana na mabadiliko ya EFI katika NVRAM.
* Usanidi mbaya au matumizi mabaya yanaweza kusababisha kompyuta za mkononi zisizoweza kuanzishwa au mashine za mwenyeji zisizoweza kuanzishwa.

#### **`/sys/kernel/debug`**

* `debugfs` inatoa kiolesura cha "hakuna sheria" kwa ufuatiliaji wa kernel.
* Historia ya matatizo ya usalama kutokana na asili yake isiyo na mipaka.

### References

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

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
