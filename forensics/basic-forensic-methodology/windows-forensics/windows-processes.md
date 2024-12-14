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


## smss.exe

**Meneja wa Kikao**.\
Kikao 0 kinaanzisha **csrss.exe** na **wininit.exe** (**huduma za OS**) wakati Kikao 1 kinaanzisha **csrss.exe** na **winlogon.exe** (**kikao cha Mtumiaji**). Hata hivyo, unapaswa kuona **mchakato mmoja tu** wa hiyo **binary** bila watoto katika mti wa michakato.

Pia, vikao mbali na 0 na 1 vinaweza kumaanisha kuwa vikao vya RDP vinafanyika.


## csrss.exe

**Mchakato wa Mfumo wa Mteja/Server**.\
Inasimamia **michakato** na **nyuzi**, inafanya **API ya Windows** ipatikane kwa michakato mingine na pia **inaandika herufi za diski**, kuunda **faili za muda**, na kushughulikia **mchakato wa kuzima**.

Kuna moja **inayoendesha katika Kikao 0 na nyingine katika Kikao 1** (hivyo **michakato 2** katika mti wa michakato). Nyingine inaundwa **kwa kila Kikao kipya**.


## winlogon.exe

**Mchakato wa Kuingia wa Windows**.\
Inawajibika kwa **kuingia**/**kutoka** kwa mtumiaji. Inaanzisha **logonui.exe** ili kuuliza jina la mtumiaji na nenosiri kisha inaita **lsass.exe** ili kuyathibitisha.

Kisha inaanzisha **userinit.exe** ambayo imeainishwa katika **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** na ufunguo **Userinit**.

Zaidi ya hayo, rejista ya awali inapaswa kuwa na **explorer.exe** katika ufunguo wa **Shell** au inaweza kutumika vibaya kama **njia ya kudumu ya malware**.


## wininit.exe

**Mchakato wa Kuanza wa Windows**. \
Inaanzisha **services.exe**, **lsass.exe**, na **lsm.exe** katika Kikao 0. Inapaswa kuwa na mchakato 1 tu.


## userinit.exe

**Programu ya Kuingia ya Userinit**.\
Inapakia **ntduser.dat katika HKCU** na kuanzisha **mazingira ya mtumiaji** na kuendesha **scripts za kuingia** na **GPO**.

Inaanzisha **explorer.exe**.


## lsm.exe

**Meneja wa Kikao cha Mitaa**.\
Inafanya kazi na smss.exe ili kudhibiti vikao vya watumiaji: Kuingia/kutoka, kuanzisha shell, kufunga/kufungua desktop, nk.

Baada ya W7 lsm.exe ilibadilishwa kuwa huduma (lsm.dll).

Inapaswa kuwa na mchakato 1 tu katika W7 na kutoka kwao huduma inayokimbia DLL.


## services.exe

**Meneja wa Kudhibiti Huduma**.\
In **pakia** **huduma** zilizowekwa kama **kuanzisha kiotomatiki** na **madereva**.

Ni mchakato mzazi wa **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** na mengi zaidi.

Huduma zinafafanuliwa katika `HKLM\SYSTEM\CurrentControlSet\Services` na mchakato huu unahifadhi DB katika kumbukumbu ya taarifa za huduma ambazo zinaweza kuulizwa na sc.exe.

Kumbuka jinsi **huduma** **zingine** zitakuwa zinaendesha katika **mchakato wao wenyewe** na nyingine zitakuwa **zinashiriki mchakato wa svchost.exe**.

Inapaswa kuwa na mchakato 1 tu.


## lsass.exe

**Mifumo ya Mitaa ya Usalama**.\
Inawajibika kwa **uthibitishaji** wa mtumiaji na kuunda **tokens** za **usalama**. Inatumia pakiti za uthibitishaji zilizoko katika `HKLM\System\CurrentControlSet\Control\Lsa`.

Inandika kwenye **kumbukumbu ya tukio la Usalama** na inapaswa kuwa na mchakato 1 tu.

Kumbuka kuwa mchakato huu unashambuliwa sana ili kutolewa nenosiri.


## svchost.exe

**Mchakato wa Kihuduma wa Kawaida**.\
Inahifadhi huduma nyingi za DLL katika mchakato mmoja wa pamoja.

Kwa kawaida, utapata kuwa **svchost.exe** inaanzishwa na bendera `-k`. Hii itaanzisha uchunguzi wa rejista **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** ambapo kutakuwa na ufunguo wenye hoja iliyotajwa katika -k ambayo itakuwa na huduma za kuanzisha katika mchakato sawa.

Kwa mfano: `-k UnistackSvcGroup` itaanzisha: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Ikiwa **bendera `-s`** pia inatumika na hoja, basi svchost inaombwa **kuanzisha huduma iliyotajwa tu** katika hoja hii.

Kutakuwa na michakato kadhaa ya `svchost.exe`. Ikiwa yoyote kati yao **haitumii bendera `-k`**, basi hiyo ni ya kushangaza sana. Ikiwa unapata kuwa **services.exe si mzazi**, hiyo pia ni ya kushangaza.


## taskhost.exe

Mchakato huu unafanya kazi kama mwenyeji wa michakato inayokimbia kutoka kwa DLLs. Pia inapakia huduma zinazokimbia kutoka kwa DLLs.

Katika W8 hii inaitwa taskhostex.exe na katika W10 taskhostw.exe.


## explorer.exe

Huu ni mchakato unaohusika na **desktop ya mtumiaji** na kuanzisha faili kupitia nyongeza za faili.

**Mchakato 1 tu** unapaswa kuanzishwa **kwa kila mtumiaji aliyeingia.**

Hii inakimbia kutoka **userinit.exe** ambayo inapaswa kumalizika, hivyo **hakuna mzazi** anapaswa kuonekana kwa mchakato huu.


# Kukamata Michakato ya Uhalifu

* Je, inakimbia kutoka kwenye njia inayotarajiwa? (Hakuna binaries za Windows zinazoendesha kutoka eneo la muda)
* Je, inawasiliana na IP za ajabu?
* Angalia saini za kidijitali (vitu vya Microsoft vinapaswa kusainiwa)
* Je, imeandikwa vizuri?
* Je, inakimbia chini ya SID inayotarajiwa?
* Je, mchakato mzazi ni ule unaotarajiwa (ikiwa upo)?
* Je, michakato ya watoto ni zile zinazotarajiwa? (hakuna cmd.exe, wscript.exe, powershell.exe..?)
