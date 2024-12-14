# DDexec / EverythingExec

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

## Muktadha

Katika Linux ili kuendesha programu lazima iwepo kama faili, lazima iweze kupatikana kwa njia fulani kupitia mfumo wa faili (hii ndiyo jinsi `execve()` inavyofanya kazi). Faili hii inaweza kuwa kwenye diski au kwenye ram (tmpfs, memfd) lakini unahitaji njia ya faili. Hii imefanya iwe rahisi kudhibiti kile kinachokimbia kwenye mfumo wa Linux, inafanya iwe rahisi kugundua vitisho na zana za mshambuliaji au kuzuia wao kujaribu kutekeleza chochote chao kabisa (_e. g._ kutoruhusu watumiaji wasio na mamlaka kuweka faili zinazoweza kutekelezwa mahali popote).

Lakini mbinu hii iko hapa kubadilisha yote haya. Ikiwa huwezi kuanzisha mchakato unayotaka... **basi unachukua nyara mchakato mmoja uliopo tayari**.

Mbinu hii inakuwezesha **kuzidi mbinu za kawaida za ulinzi kama vile kusoma tu, noexec, orodha ya majina ya faili, orodha ya hash...**

## Mahitaji

Script ya mwisho inategemea zana zifuatazo kufanya kazi, zinahitaji kupatikana katika mfumo unaoshambulia (kwa kawaida utazipata kila mahali):
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## Mbinu

Ikiwa unaweza kubadilisha kumbukumbu ya mchakato kwa njia isiyo na mipaka, basi unaweza kuichukua. Hii inaweza kutumika kuhamasisha mchakato uliopo na kuubadilisha na programu nyingine. Tunaweza kufikia hili ama kwa kutumia syscall ya `ptrace()` (ambayo inahitaji uwe na uwezo wa kutekeleza syscalls au kuwa na gdb inapatikana kwenye mfumo) au, kwa njia ya kuvutia zaidi, kuandika kwenye `/proc/$pid/mem`.

Faili `/proc/$pid/mem` ni ramani moja kwa moja ya nafasi yote ya anwani ya mchakato (_e. g._ kutoka `0x0000000000000000` hadi `0x7ffffffffffff000` katika x86-64). Hii ina maana kwamba kusoma au kuandika kwenye faili hili kwa offset `x` ni sawa na kusoma kutoka au kubadilisha maudhui kwenye anwani ya virtual `x`.

Sasa, tuna matatizo manne ya msingi ya kukabiliana nayo:

* Kwa ujumla, ni root tu na mmiliki wa programu ya faili wanaweza kuibadilisha.
* ASLR.
* Ikiwa tutajaribu kusoma au kuandika kwenye anwani ambayo haijapangwa katika nafasi ya anwani ya programu, tutapata kosa la I/O.

Matatizo haya yana suluhisho ambayo, ingawa si kamilifu, ni mazuri:

* Watafsiri wengi wa shell wanaruhusu uundaji wa vigezo vya faili ambavyo vitarithiwa na michakato ya watoto. Tunaweza kuunda fd inayopiga faili ya `mem` ya shell yenye ruhusa za kuandika... hivyo michakato ya watoto inayotumia fd hiyo itakuwa na uwezo wa kubadilisha kumbukumbu ya shell.
* ASLR si tatizo hata kidogo, tunaweza kuangalia faili ya `maps` ya shell au nyingine yoyote kutoka procfs ili kupata taarifa kuhusu nafasi ya anwani ya mchakato.
* Hivyo tunahitaji `lseek()` juu ya faili. Kutoka kwa shell hili haliwezi kufanywa isipokuwa kwa kutumia `dd` maarufu.

### Kwa maelezo zaidi

Hatua hizo ni rahisi na hazihitaji aina yoyote ya utaalamu ili kuzielewa:

* Changanua binary tunayotaka kuendesha na loader ili kugundua ni ramani zipi wanahitaji. Kisha tengeneza "shell"code itakayofanya, kwa ujumla, hatua sawa na zile ambazo kernel inafanya kila wakati inapoita `execve()`:
* Unda ramani hizo.
* Soma binaries ndani yao.
* Weka ruhusa.
* Hatimaye anza stack na hoja za programu na weka vector ya ziada (inayohitajika na loader).
* Ruka kwenye loader na uache ifanye yaliyobaki (pakia maktaba zinazohitajika na programu).
* Pata kutoka kwa faili ya `syscall` anwani ambayo mchakato utarudi baada ya syscall inayotekelezwa.
* Badilisha mahali hapo, ambalo litakuwa executable, na shellcode yetu (kupitia `mem` tunaweza kubadilisha kurasa zisizoweza kuandikwa).
* Pass programu tunayotaka kuendesha kwa stdin ya mchakato (itakuwa `read()` na "shell"code hiyo).
* Katika hatua hii ni juu ya loader kupakia maktaba zinazohitajika kwa programu yetu na kuruka ndani yake.

**Angalia chombo katika** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

Kuna chaguzi kadhaa za `dd`, moja ya hizo, `tail`, kwa sasa ndiyo programu ya default inayotumika ku `lseek()` kupitia faili ya `mem` (ambayo ilikuwa sababu pekee ya kutumia `dd`). Chaguzi hizo ni:
```bash
tail
hexdump
cmp
xxd
```
Kuweka mabadiliko ya `SEEKER` unaweza kubadilisha mtafutaji anayetumika, _e. g._:
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Ikiwa unapata mtafutaji mwingine halali ambao haujatekelezwa katika skripti, bado unaweza kuutumia kwa kuweka kiambishi cha `SEEKER_ARGS`:
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Block hii, EDRs.

## References
* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

{% hint style="success" %}
Jifunze & fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze & fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **fuata** sisi kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
