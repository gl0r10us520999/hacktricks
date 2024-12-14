{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Opleiding AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Opleiding GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# Tydstempels

'n Aanvaller mag belangstel om **die tydstempels van lêers te verander** om nie opgespoor te word nie.\
Dit is moontlik om die tydstempels binne die MFT in eienskappe `$STANDARD_INFORMATION` __ en __ `$FILE_NAME` te vind.

Albei eienskappe het 4 tydstempels: **Wysiging**, **toegang**, **skepping**, en **MFT registrasie wysiging** (MACE of MACB).

**Windows verkenner** en ander gereedskap wys die inligting van **`$STANDARD_INFORMATION`**.

## TimeStomp - Anti-forensiese Gereedskap

Hierdie gereedskap **wysig** die tydstempel inligting binne **`$STANDARD_INFORMATION`** **maar** **nie** die inligting binne **`$FILE_NAME`** nie. Daarom is dit moontlik om **verdagte** **aktiwiteit** te **identifiseer**.

## Usnjrnl

Die **USN Journal** (Update Sequence Number Journal) is 'n kenmerk van die NTFS (Windows NT lêerstelsel) wat volume veranderinge opneem. Die [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) gereedskap maak dit moontlik om hierdie veranderinge te ondersoek.

![](<../../.gitbook/assets/image (449).png>)

Die vorige beeld is die **uitset** wat deur die **gereedskap** gewys word waar dit waargeneem kan word dat sommige **veranderinge gemaak is** aan die lêer.

## $LogFile

**Alle metadata veranderinge aan 'n lêerstelsel word gelog** in 'n proses bekend as [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead_logging). Die gelogde metadata word in 'n lêer genaamd `**$LogFile**`, geleë in die wortelgids van 'n NTFS lêerstelsel, gehou. Gereedskap soos [LogFileParser](https://github.com/jschicht/LogFileParser) kan gebruik word om hierdie lêer te ontleed en veranderinge te identifiseer.

![](<../../.gitbook/assets/image (450).png>)

Weer eens, in die uitset van die gereedskap is dit moontlik om te sien dat **sommige veranderinge gemaak is**.

Met dieselfde gereedskap is dit moontlik om te identifiseer **tot watter tyd die tydstempels gewysig is**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Lêer se skeppingstyd
* ATIME: Lêer se wysigingstyd
* MTIME: Lêer se MFT registrasie wysiging
* RTIME: Lêer se toegangstyd

## `$STANDARD_INFORMATION` en `$FILE_NAME` vergelyking

'n Ander manier om verdagte gewysigde lêers te identifiseer, sou wees om die tyd op albei eienskappe te vergelyk en te soek na **ongelykhede**.

## Nanosekondes

**NTFS** tydstempels het 'n **presisie** van **100 nanosekondes**. Dan, om lêers met tydstempels soos 2010-10-10 10:10:**00.000:0000 te vind is baie verdag**.

## SetMace - Anti-forensiese Gereedskap

Hierdie gereedskap kan albei eienskappe `$STARNDAR_INFORMATION` en `$FILE_NAME` wysig. Dit is egter nodig vir 'n lewende OS om hierdie inligting te wysig vanaf Windows Vista.

# Data Versteek

NFTS gebruik 'n kluster en die minimum inligting grootte. Dit beteken dat as 'n lêer 'n kluster en 'n half gebruik, die **oorblywende half nooit gebruik gaan word** totdat die lêer verwyder word. Dan is dit moontlik om **data in hierdie slack ruimte te versteek**.

Daar is gereedskap soos slacker wat toelaat om data in hierdie "versteekte" ruimte te versteek. 'n Analise van die `$logfile` en `$usnjrnl` kan egter wys dat sommige data bygevoeg is:

![](<../../.gitbook/assets/image (452).png>)

Dan is dit moontlik om die slack ruimte te herwin met gereedskap soos FTK Imager. Let daarop dat hierdie soort gereedskap die inhoud obfuskeer of selfs versleuteld kan stoor.

# UsbKill

Dit is 'n gereedskap wat die **rekenaar sal afskakel as enige verandering in die USB** poorte opgespoor word.\
'n Manier om dit te ontdek, sou wees om die lopende prosesse te inspekteer en **elke python skrip wat loop te hersien**.

# Lewende Linux Verspreidings

Hierdie distros word **binne die RAM** geheue uitgevoer. Die enigste manier om hulle te ontdek is **in die geval dat die NTFS lêerstelsel met skryf toestemmings gemonteer is**. As dit net met lees toestemmings gemonteer is, sal dit nie moontlik wees om die indringing te ontdek nie.

# Veilige Verwydering

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Windows Konfigurasie

Dit is moontlik om verskeie Windows logging metodes te deaktiveer om die forensiese ondersoek baie moeiliker te maak.

## Deaktiveer Tydstempels - UserAssist

Dit is 'n registriesleutel wat datums en ure behou wanneer elke uitvoerbare lêer deur die gebruiker uitgevoer is.

Om UserAssist te deaktiveer, is twee stappe nodig:

1. Stel twee registriesleutels in, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` en `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, albei op nul om aan te dui dat ons wil hê UserAssist moet gedeaktiveer word.
2. Maak jou registriesubbome skoon wat lyk soos `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

## Deaktiveer Tydstempels - Prefetch

Dit sal inligting oor die toepassings wat uitgevoer is, stoor met die doel om die prestasie van die Windows stelsel te verbeter. Dit kan egter ook nuttig wees vir forensiese praktyke.

* Voer `regedit` uit
* Kies die lêer pad `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Regsklik op beide `EnablePrefetcher` en `EnableSuperfetch`
* Kies Wysig op elkeen van hierdie om die waarde van 1 (of 3) na 0 te verander
* Herbegin

## Deaktiveer Tydstempels - Laaste Toegangstyd

Wanneer 'n gids vanaf 'n NTFS volume op 'n Windows NT bediener geopen word, neem die stelsel die tyd om **'n tydstempel veld op elke gelysde gids op te dateer**, wat die laaste toegangstyd genoem word. Op 'n intensief gebruikte NTFS volume kan dit die prestasie beïnvloed.

1. Maak die Registrie Redigeerder (Regedit.exe) oop.
2. Blaai na `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Soek na `NtfsDisableLastAccessUpdate`. As dit nie bestaan nie, voeg hierdie DWORD by en stel die waarde op 1, wat die proses sal deaktiveer.
4. Sluit die Registrie Redigeerder, en herbegin die bediener.

## Verwyder USB Geskiedenis

Alle **USB Toestel Inskrywings** word in die Windows Registrie onder die **USBSTOR** registriesleutel gestoor wat sub sleutels bevat wat geskep word wanneer jy 'n USB Toestel in jou PC of Laptop inprop. Jy kan hierdie sleutel hier vind H`KEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Deur dit te verwyder** sal jy die USB geskiedenis verwyder.\
Jy kan ook die gereedskap [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) gebruik om seker te maak jy het dit verwyder (en om dit te verwyder).

'n Ander lêer wat inligting oor die USB's stoor, is die lêer `setupapi.dev.log` binne `C:\Windows\INF`. Dit moet ook verwyder word.

## Deaktiveer Skadu Kopieë

**Lys** skadu kopieë met `vssadmin list shadowstorage`\
**Verwyder** hulle deur `vssadmin delete shadow` te loop

Jy kan hulle ook via GUI verwyder deur die stappe voor te stel in [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Om skadu kopieë te deaktiveer [stappe van hier](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows):

1. Maak die Dienste program oop deur "dienste" in die teks soekboks te tik nadat jy op die Windows begin knoppie geklik het.
2. Vind "Volume Shadow Copy" in die lys, kies dit, en toegang eienskappe deur regsklik.
3. Kies Deaktiveer van die "Opstart tipe" keuselys, en bevestig die verandering deur Toepas en OK te klik.

Dit is ook moontlik om die konfigurasie van watter lêers in die skadu kopie gaan gekopieer word in die registrie `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot` te wysig.

## Oorskryf verwyderde lêers

* Jy kan 'n **Windows gereedskap** gebruik: `cipher /w:C` Dit sal cipher aanwys om enige data uit die beskikbare ongebruikte skyfspasie binne die C skyf te verwyder.
* Jy kan ook gereedskap soos [**Eraser**](https://eraser.heidi.ie) gebruik.

## Verwyder Windows gebeurtenis logs

* Windows + R --> eventvwr.msc --> Brei "Windows Logs" uit --> Regsklik op elke kategorie en kies "Skoon Log"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Deaktiveer Windows gebeurtenis logs

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Binne die dienste afdeling deaktiveer die diens "Windows Event Log"
* `WEvtUtil.exec clear-log` of `WEvtUtil.exe cl`

## Deaktiveer $UsnJrnl

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Opleiding AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Opleiding GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
