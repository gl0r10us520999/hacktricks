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

**Menadžer sesija**.\
Sesija 0 pokreće **csrss.exe** i **wininit.exe** (**OS** **usluge**) dok Sesija 1 pokreće **csrss.exe** i **winlogon.exe** (**Korisnička** **sesija**). Međutim, trebali biste videti **samo jedan proces** tog **binarija** bez dece u stablu procesa.

Takođe, sesije osim 0 i 1 mogu značiti da se odvijaju RDP sesije.


## csrss.exe

**Klijent/Server Run Subsystem Process**.\
Upravlja **procesima** i **nitima**, omogućava **Windows** **API** za druge procese i takođe **mapira slova drajvera**, kreira **temp fajlove** i upravlja **shutdown** **procesom**.

Postoji jedan **koji se izvršava u Sesiji 0 i još jedan u Sesiji 1** (tako da **2 procesa** u stablu procesa). Još jedan se kreira **po novoj Sesiji**.


## winlogon.exe

**Windows Logon Process**.\
Odgovoran je za korisnički **logon**/**logoff**. Pokreće **logonui.exe** da zatraži korisničko ime i lozinku, a zatim poziva **lsass.exe** da ih verifikuje.

Zatim pokreće **userinit.exe** koji je specificiran u **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** sa ključem **Userinit**.

Pored toga, prethodni registar bi trebao imati **explorer.exe** u **Shell ključu** ili bi mogao biti zloupotrebljen kao **metoda postojanosti malvera**.


## wininit.exe

**Windows Initialization Process**. \
Pokreće **services.exe**, **lsass.exe**, i **lsm.exe** u Sesiji 0. Trebao bi postojati samo 1 proces.


## userinit.exe

**Userinit Logon Application**.\
Učitava **ntduser.dat u HKCU** i inicijalizuje **korisničko** **okruženje** i pokreće **logon** **skripte** i **GPO**.

Pokreće **explorer.exe**.


## lsm.exe

**Menadžer lokalnih sesija**.\
Radi sa smss.exe da manipuliše korisničkim sesijama: Logon/logoff, pokretanje shell-a, zaključavanje/otključavanje desktop-a, itd.

Nakon W7, lsm.exe je transformisan u uslugu (lsm.dll).

Trebalo bi da postoji samo 1 proces u W7 i od njih usluga koja pokreće DLL.


## services.exe

**Menadžer kontrole usluga**.\
**Učitava** **usluge** konfigurirane kao **auto-start** i **draivere**.

To je roditeljski proces **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** i mnoge druge.

Usluge su definisane u `HKLM\SYSTEM\CurrentControlSet\Services` i ovaj proces održava bazu podataka u memoriji o informacijama o uslugama koje se mogu upititi putem sc.exe.

Obratite pažnju kako će **neke** **usluge** raditi u **svojim procesima** dok će druge **deliti svchost.exe proces**.

Trebalo bi da postoji samo 1 proces.


## lsass.exe

**Podsystem lokalne bezbednosti**.\
Odgovoran je za **autentifikaciju** korisnika i kreira **bezbednosne** **tokene**. Koristi pakete autentifikacije smeštene u `HKLM\System\CurrentControlSet\Control\Lsa`.

Piše u **log** **događaja** **bezbednosti** i trebalo bi da postoji samo 1 proces.

Imajte na umu da je ovaj proces često napadnut da bi se iskopirali lozinke.


## svchost.exe

**Generički proces hosta usluga**.\
Hostuje više DLL usluga u jednom zajedničkom procesu.

Obično ćete naći da je **svchost.exe** pokrenut sa `-k` flagom. Ovo će pokrenuti upit u registru **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** gde će postojati ključ sa argumentom pomenutim u -k koji će sadržati usluge koje treba pokrenuti u istom procesu.

Na primer: `-k UnistackSvcGroup` će pokrenuti: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Ako se **flag `-s`** takođe koristi sa argumentom, tada se od svchost-a traži da **pokrene samo specificiranu uslugu** u ovom argumentu.

Biće nekoliko procesa `svchost.exe`. Ako nijedan od njih **ne koristi `-k` flag**, to je takođe veoma sumnjivo. Ako otkrijete da **services.exe nije roditelj**, to je takođe veoma sumnjivo.


## taskhost.exe

Ovaj proces deluje kao host za procese koji se izvršavaju iz DLL-ova. Takođe učitava usluge koje se izvršavaju iz DLL-ova.

U W8 ovo se zove taskhostex.exe, a u W10 taskhostw.exe.


## explorer.exe

Ovo je proces odgovoran za **desktop korisnika** i pokretanje fajlova putem ekstenzija fajlova.

**Samo 1** proces bi trebao biti pokrenut **po prijavljenom korisniku.**

Ovo se pokreće iz **userinit.exe** koji bi trebao biti prekinut, tako da **nema roditelja** za ovaj proces.


# Otkrivanje zlonamernih procesa

* Da li se pokreće iz očekivane putanje? (Nijedna Windows binarna datoteka se ne pokreće iz temp lokacije)
* Da li komunicira sa čudnim IP-ovima?
* Proverite digitalne potpise (Microsoft artefakti bi trebali biti potpisani)
* Da li je pravilno napisano?
* Da li se izvršava pod očekivanim SID-om?
* Da li je roditeljski proces očekivani (ako postoji)?
* Da li su procesi dece oni koje očekujete? (nema cmd.exe, wscript.exe, powershell.exe..?)
