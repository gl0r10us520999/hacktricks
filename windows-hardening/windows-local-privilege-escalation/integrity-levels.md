# Integriteitsvlakke

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsieplanne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Integriteitsvlakke

In Windows Vista en later weergawes, kom alle beskermde items met 'n **integriteitsvlak** etiket. Hierdie opstelling ken meestal 'n "medium" integriteitsvlak toe aan lêers en registersleutels, behalwe vir sekere vouers en lêers waartoe Internet Explorer 7 op 'n lae integriteitsvlak kan skryf. Die standaardgedrag is dat prosesse wat deur standaardgebruikers geïnisieer word, 'n medium integriteitsvlak het, terwyl dienste tipies op 'n stelselintegriteitsvlak werk. 'n Hoë-integriteitsetiket beskerm die wortelgids.

'n Sleutelreël is dat voorwerpe nie gewysig kan word deur prosesse met 'n laer integriteitsvlak as die voorwerp se vlak nie. Die integriteitsvlakke is:

* **Onbetroubaar**: Hierdie vlak is vir prosesse met anonieme aanmeldings. %%%Voorbeeld: Chrome%%%
* **Laag**: Hoofsaaklik vir internetinteraksies, veral in Internet Explorer se Beskermde Modus, wat geassosieerde lêers en prosesse beïnvloed, en sekere vouers soos die **Tydelike Internet-gids**. Lae integriteitsprosesse ondervind beduidende beperkings, insluitend geen register skrywe toegang en beperkte gebruiker profiel skrywe toegang nie.
* **Medium**: Die standaardvlak vir die meeste aktiwiteite, toegeken aan standaardgebruikers en voorwerpe sonder spesifieke integriteitsvlakke. Selfs lede van die Administrators-groep werk standaard op hierdie vlak.
* **Hoog**: Gereserveer vir administrateurs, wat hulle toelaat om voorwerpe op laer integriteitsvlakke te wysig, insluitend dié op die hoë vlak self.
* **Stelsel**: Die hoogste operasionele vlak vir die Windows-kern en kern dienste, buite bereik selfs vir administrateurs, wat beskerming van noodsaaklike stelselfunksies verseker.
* **Installeerder**: 'n Unieke vlak wat bo alle ander staan, wat voorwerpe op hierdie vlak in staat stel om enige ander voorwerp te deïnstalleer.

Jy kan die integriteitsvlak van 'n proses verkry met **Process Explorer** van **Sysinternals**, deur die **eienskappe** van die proses te benader en die "**Sekuriteit**" oortjie te besigtig:

![](<../../.gitbook/assets/image (824).png>)

Jy kan ook jou **huidige integriteitsvlak** verkry met `whoami /groups`

![](<../../.gitbook/assets/image (325).png>)

### Integriteitsvlakke in die lêerstelsel

'n Voorwerp binne die lêerstelsel mag 'n **minimum integriteitsvlak vereiste** benodig en as 'n proses nie hierdie integriteitsvlak het nie, sal dit nie in staat wees om daarmee te kommunikeer.\
Byvoorbeeld, kom ons **skep 'n gewone lêer vanaf 'n gewone gebruiker se konsole en kyk na die toestemmings**:
```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```
Nou, kom ons ken 'n minimum integriteitsvlak van **Hoog** aan die lêer toe. Dit **moet gedoen word vanaf 'n konsole** wat as **administrateur** loop, aangesien 'n **gewone konsole** in Medium Integriteitsvlak sal loop en **nie toegelaat sal word** om 'n Hoë Integriteitsvlak aan 'n objek toe te ken:
```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```
Dit is waar dinge interessant raak. Jy kan sien dat die gebruiker `DESKTOP-IDJHTKP\user` **VOLLE regte** oor die lêer het (werklik, dit was die gebruiker wat die lêer geskep het), egter, as gevolg van die minimum integriteitsvlak wat geïmplementeer is, sal hy nie in staat wees om die lêer weer te wysig nie, tensy hy binne 'n Hoë Integriteitsvlak loop (let op dat hy dit sal kan lees):
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**Daarom, wanneer 'n lêer 'n minimum integriteitsvlak het, moet jy ten minste op daardie integriteitsvlak loop om dit te kan wysig.**
{% endhint %}

### Integriteitsvlakke in Binaries

Ek het 'n kopie van `cmd.exe` gemaak in `C:\Windows\System32\cmd-low.exe` en dit 'n **integriteitsvlak van laag vanaf 'n administrateurkonsol gestel:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
Now, when I run `cmd-low.exe` it will **run under a low-integrity level** instead of a medium one:

![](<../../.gitbook/assets/image (313).png>)

For curious people, if you assign high integrity level to a binary (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`) it won't run with high integrity level automatically (if you invoke it from a medium integrity level --by default-- it will run under a medium integrity level).

### Integrity Levels in Processes

Nie alle lêers en vouers het 'n minimum integriteitsvlak nie, **maar alle prosesse loop onder 'n integriteitsvlak**. En soortgelyk aan wat met die lêerstelsel gebeur het, **as 'n proses binne 'n ander proses wil skryf, moet dit ten minste dieselfde integriteitsvlak hê**. Dit beteken dat 'n proses met 'n lae integriteitsvlak nie 'n handvatsel met volle toegang tot 'n proses met 'n medium integriteitsvlak kan oopmaak nie.

As gevolg van die beperkings wat in hierdie en die vorige afdeling bespreek is, is dit altyd **aanbeveel om 'n proses in die laagste moontlike integriteitsvlak te laat loop**.


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
