# Vitu vya Windows

## Vitu vya Windows

{% hint style="success" %}
Jifunze na zoezi la AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Mafunzo ya HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na zoezi la GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Mafunzo ya HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**Kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au kikundi cha [**telegram**](https://t.me/peass) au **tufuate** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za udukuzi kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Vitu vya Kizazi cha Windows

### Taarifa za Windows 10

Katika njia `\Users\<jina la mtumiaji>\AppData\Local\Microsoft\Windows\Notifications` unaweza kupata database `appdb.dat` (kabla ya Windows anniversary) au `wpndatabase.db` (baada ya Windows Anniversary).

Ndani ya database hii ya SQLite, unaweza kupata meza ya `Notification` na taarifa zote za arifa (kwa muundo wa XML) ambazo zinaweza kuwa na data muhimu.

### Muda

Muda ni sifa ya Windows ambayo hutoa **historia ya mfululizo** ya kurasa za wavuti zilizotembelewa, nyaraka zilizohaririwa, na programu zilizotekelezwa.

Database iko katika njia `\Users\<jina la mtumiaji>\AppData\Local\ConnectedDevicesPlatform\<id>\ActivitiesCache.db`. Database hii inaweza kufunguliwa na chombo cha SQLite au na chombo [**WxTCmd**](https://github.com/EricZimmerman/WxTCmd) **ambacho huzalisha faili 2 ambazo zinaweza kufunguliwa na chombo** [**TimeLine Explorer**](https://ericzimmerman.github.io/#!index.md).

### ADS (Vijia vya Data Badala)

Faili zilizopakuliwa zinaweza kuwa na **ADS Zone.Identifier** inayoonyesha **jinsi** ilivyopakuliwa kutoka kwenye mtandao wa ndani, mtandao, n.k. Baadhi ya programu (kama vivinjari) kawaida huingiza **habari zaidi** kama **URL** ambapo faili ilipakuliwa.

## **Nakala za Faili**

### Bakuli la Takataka

Katika Vista/Win7/Win8/Win10 **Bakuli la Takataka** linaweza kupatikana katika folda **`$Recycle.bin`** kwenye mizizi ya diski (`C:\$Recycle.bin`).\
Wakati faili inapofutwa katika folda hii, faili 2 maalum huzalishwa:

* `$I{id}`: Taarifa za faili (tarehe ya kufutwa}
* `$R{id}`: Yaliyomo ya faili

![](<../../../.gitbook/assets/image (486).png>)

Ukiwa na faili hizi, unaweza kutumia chombo [**Rifiuti**](https://github.com/abelcheung/rifiuti2) kupata anwani ya asili ya faili zilizofutwa na tarehe iliyofutwa (tumia `rifiuti-vista.exe` kwa Vista – Win10).
```
.\rifiuti-vista.exe C:\Users\student\Desktop\Recycle
```
![](<../../../.gitbook/assets/image (495) (1) (1) (1).png>)

### Nakala za Kivuli za Vifaa

Kivuli cha Nakala ni teknolojia iliyomo katika Microsoft Windows inayoweza kuunda **nakala za nakala rudufu** au picha ndogo za faili au sehemu za kompyuta, hata wakati zinatumika.

Nakala hizi za rudufu kawaida zinapatikana katika `\System Volume Information` kutoka kwa mzizi wa mfumo wa faili na jina linajumuisha **UIDs** zilizoonyeshwa katika picha ifuatayo:

![](<../../../.gitbook/assets/image (520).png>)

Kwa kufunga picha ya uchunguzi kwa kutumia **ArsenalImageMounter**, zana [**ShadowCopyView**](https://www.nirsoft.net/utils/shadow\_copy\_view.html) inaweza kutumika kuangalia nakala ya kivuli na hata **kutoa faili** kutoka kwa nakala za rudufu za kivuli.

![](<../../../.gitbook/assets/image (521).png>)

Kuingia kwa usajili `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\BackupRestore` ina faili na funguo **za kutofanya rudufu**:

![](<../../../.gitbook/assets/image (522).png>)

Usajili `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\VSS` pia una habari ya usanidi kuhusu `Nakala za Kivuli za Vifaa`.

### Faili za Kiotomatiki za Kuhifadhiwa kwa Ofisi

Unaweza kupata faili za kuhifadhiwa kwa ofisi katika: `C:\Usuarios\\AppData\Roaming\Microsoft{Excel|Word|Powerpoint}\`

## Vipengele vya Shell

Kipengele cha shell ni kipengele kinachojumuisha habari juu ya jinsi ya kupata faili nyingine.

### Nyaraka za Hivi Karibuni (LNK)

Windows **kiotomatiki** **huunda** viungo hivi **vikorofi** wakati mtumiaji **anapofungua, kutumia au kuunda faili** katika:

* Win7-Win10: `C:\Users\\AppData\Roaming\Microsoft\Windows\Recent\`
* Ofisi: `C:\Users\\AppData\Roaming\Microsoft\Office\Recent\`

Unapounda folda, kiungo kwa folda, kwa folda mama, na kwa folda ya babu pia hujengwa.

Faili hizi za viungo zilizoundwa kiotomatiki **zina habari kuhusu asili** kama ikiwa ni **faili** **au** **folda**, **nyakati za MAC** za faili hiyo, **habari ya kiasi** ambapo faili imewekwa na **folda ya faili ya lengo**. Habari hii inaweza kuwa muhimu kwa kupona faili hizo ikiwa zimeondolewa.

Pia, **tarehe ya kuundwa kwa faili ya kiungo** ni **wakati wa kwanza** faili ya asili ilikuwa **imetumika kwanza** na **tarehe** **iliyobadilishwa** ya faili ya kiungo ni **wakati wa mwisho** faili ya asili iliotumika.

Kuangalia faili hizi unaweza kutumia [**LinkParser**](http://4discovery.com/our-tools/).

Katika zana hizi utapata **seti 2** za alama za muda:

* **Seti ya Kwanza:**
1. Tarehe ya Kubadilishwa kwa Faili
2. Tarehe ya Kufikia Faili
3. Tarehe ya Kuundwa kwa Faili
* **Seti ya Pili:**
1. Tarehe ya Kubadilishwa kwa Kiungo
2. Tarehe ya Kufikia Kiungo
3. Tarehe ya Kuundwa kwa Kiungo.

Seti ya kwanza ya alama za muda inahusiana na **alama za muda za faili yenyewe**. Seti ya pili inahusiana na **alama za muda za faili iliyounganishwa**.

Unaweza kupata habari sawa kwa kutumia zana ya Windows CLI: [**LECmd.exe**](https://github.com/EricZimmerman/LECmd)
```
LECmd.exe -d C:\Users\student\Desktop\LNKs --csv C:\Users\student\Desktop\LNKs
```
### Jumplists

Hizi ni faili za hivi karibuni zilizoonyeshwa kwa kila programu. Ni orodha ya **faili za hivi karibuni zilizotumiwa na programu** ambazo unaweza kufikia kwenye kila programu. Zinaweza kuundwa **kiotomatiki au kuwa za kawaida**.

**Jumplists** zilizoundwa kiotomatiki zimehifadhiwa katika `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\`. Jumplists hizo zinaitwa kufuatia muundo `{id}.autmaticDestinations-ms` ambapo ID ya awali ni ID ya programu.

Jumplists za kawaida zimehifadhiwa katika `C:\Users\{username}\AppData\Roaming\Microsoft\Windows\Recent\CustomDestination\` na huundwa na programu kawaida kwa sababu kitu **muhimu** kimetokea na faili (labda imepewa alama ya kupendwa).

**Muda wa kuundwa** wa jumplist yoyote unaonyesha **wakati wa kwanza faili ilipofikiwa** na **muda wa kuhaririwa mara ya mwisho**.

Unaweza kukagua jumplists kwa kutumia [**JumplistExplorer**](https://ericzimmerman.github.io/#!index.md).

![](<../../../.gitbook/assets/image (474).png>)

(_Tafadhali kumbuka kuwa alama za wakati zinazotolewa na JumplistExplorer zinahusiana na faili ya jumplist yenyewe_)

### Shellbags

[Tafadhali fuata kiungo hiki kujifunza ni nini shellbags.](interesting-windows-registry-keys.md#shellbags)

## Matumizi ya USB za Windows

Inawezekana kutambua kuwa kifaa cha USB kilichotumiwa kutokana na uundaji wa:

* Folda za Hivi Karibuni za Windows
* Folda za Hivi Karibuni za Microsoft Office
* Jumplists

Tafadhali kumbuka kuwa baadhi ya faili za LNK badala ya kuashiria njia ya asili, zinaashiria kwenye folda ya WPDNSE:

![](<../../../.gitbook/assets/image (476).png>)

Faili katika folda ya WPDNSE ni nakala ya zile za asili, hivyo hazitadumu baada ya kuanza upya kwa PC na GUID inachukuliwa kutoka kwa shellbag.

### Taarifa za Usajili

[Angalia ukurasa huu kujifunza](interesting-windows-registry-keys.md#usb-information) ni funguo zipi za usajili zina taarifa muhimu kuhusu vifaa vilivyounganishwa vya USB.

### setupapi

Angalia faili `C:\Windows\inf\setupapi.dev.log` kupata alama za wakati kuhusu wakati uunganishaji wa USB ulifanyika (tafuta `Section start`).

![](<../../../.gitbook/assets/image (477) (2) (2) (2) (2) (2) (2) (2) (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (14).png>)

### USB Detective

[**USBDetective**](https://usbdetective.com) inaweza kutumika kupata taarifa kuhusu vifaa vya USB vilivyokuwa vimeunganishwa kwenye picha.

![](<../../../.gitbook/assets/image (483).png>)

### Usafi wa Plug and Play

Kazi iliyopangwa inayojulikana kama 'Usafi wa Plug and Play' imeundwa kimsingi kwa ajili ya kuondoa toleo za zamani za madereva. Tofauti na lengo lake lililoelezwa la kuhifadhi toleo la karibuni la pakiti ya dereva, vyanzo vya mtandaoni vinapendekeza pia inalenga madereva ambayo hayajatumika kwa siku 30. Kwa hivyo, madereva kwa vifaa vinavyoweza kuondolewa ambavyo havijakuwa vimeunganishwa katika siku 30 zilizopita zinaweza kuwa chini ya kufutwa.

Kazi hiyo iko katika njia ifuatayo:
`C:\Windows\System32\Tasks\Microsoft\Windows\Plug and Play\Plug and Play Cleanup`.

Picha inayoonyesha maudhui ya kazi hiyo imepatikana:
![](https://2.bp.blogspot.com/-wqYubtuR_W8/W19bV5S9XyI/AAAAAAAANhU/OHsBDEvjqmg9ayzdNwJ4y2DKZnhCdwSMgCLcBGAs/s1600/xml.png)

**Vipengele muhimu na Mipangilio ya Kazi:**
- **pnpclean.dll**: DLL hii inahusika na mchakato halisi wa usafi.
- **UseUnifiedSchedulingEngine**: Imewekwa kuwa `TRUE`, ikionyesha matumizi ya injini ya kawaida ya kupangilia kazi.
- **MaintenanceSettings**:
- **Kipindi ('P1M')**: Inaelekeza Meneja wa Kazi kuanzisha kazi ya usafi kila mwezi wakati wa matengenezo ya kiotomatiki ya kawaida.
- **Mwisho wa Muda ('P2M')**: Inaagiza Meneja wa Kazi, ikiwa kazi itashindwa kwa miezi miwili mfululizo, kutekeleza kazi wakati wa matengenezo ya dharura ya kiotomatiki.

Usanidi huu unahakikisha matengenezo ya kawaida na usafi wa madereva, na utoaji wa kujaribu tena kazi katika kesi ya kushindwa mfululizo.

**Kwa habari zaidi angalia:** [**https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html**](https://blog.1234n6.com/2018/07/windows-plug-and-play-cleanup.html)

## Barua pepe

Barua pepe zina sehemu **2 za kuvutia: Vichwa vya habari na maudhui** ya barua pepe. Katika **vichwa vya habari** unaweza kupata habari kama:

* **Nani** alituma barua pepe (anwani ya barua pepe, IP, seva za barua pepe ambazo zimeelekeza barua pepe)
* **Lini** barua pepe ilitumwa

Pia, ndani ya vichwa vya habari vya `References` na `In-Reply-To` unaweza kupata kitambulisho cha ujumbe:

![](<../../../.gitbook/assets/image (484).png>)

### Programu ya Barua pepe ya Windows

Programu hii huihifadhi barua pepe kwa HTML au maandishi. Unaweza kupata barua pepe ndani ya vijisehemu ndani ya `\Users\<username>\AppData\Local\Comms\Unistore\data\3\`. Barua pepe zimehifadhiwa kwa kipengee cha `.dat`.

**Metadata** ya barua pepe na **mawasiliano** yanaweza kupatikana ndani ya **database ya EDB**: `\Users\<username>\AppData\Local\Comms\UnistoreDB\store.vol`

**Badilisha kipengee** cha faili kutoka `.vol` hadi `.edb` na unaweza kutumia zana [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) kuifungua. Ndani ya jedwali la `Message` unaweza kuona barua pepe.

### Microsoft Outlook

Wakati seva za Exchange au wateja wa Outlook wanapotumika kutakuwa na vichwa vya MAPI:

* `Mapi-Client-Submit-Time`: Wakati wa mfumo wakati barua pepe iliotumwa
* `Mapi-Conversation-Index`: Idadi ya ujumbe wa watoto wa mjadala na muda wa kila ujumbe wa mjadala
* `Mapi-Entry-ID`: Kitambulisho cha ujumbe.
* `Mappi-Message-Flags` na `Pr_last_Verb-Executed`: Taarifa kuhusu mteja wa MAPI (ujumbe umesomwa? haujasomwa? umejibiwa? umepelekwa upya? nje ya ofisi?)

Katika mteja wa Microsoft Outlook, ujumbe uliotumwa/kupokelewa, data za mawasiliano, na data ya kalenda zimehifadhiwa kwenye faili ya PST katika:

* `%USERPROFILE%\Local Settings\Application Data\Microsoft\Outlook` (WinXP)
* `%USERPROFILE%\AppData\Local\Microsoft\Outlook`

Njia ya usajili `HKEY_CURRENT_USER\Software\Microsoft\WindowsNT\CurrentVersion\Windows Messaging Subsystem\Profiles\Outlook` inaonyesha faili inayotumiwa.

Unaweza kufungua faili ya PST kwa kutumia zana [**Kernel PST Viewer**](https://www.nucleustechnologies.com/es/visor-de-pst.html).

![](<../../../.gitbook/assets/image (485).png>)
### Faili za Microsoft Outlook OST

**Faili la OST** huzalishwa na Microsoft Outlook wakati imepangwa na **IMAP** au seva ya **Exchange**, ikihifadhi habari sawa na faili ya PST. Faili hii inasawazishwa na seva, ikihifadhi data kwa **miezi 12 iliyopita** hadi **ukubwa wa juu wa 50GB**, na iko katika saraka ile ile na faili ya PST. Ili kuona faili ya OST, [**Mwangalizi wa OST wa Kernel**](https://www.nucleustechnologies.com/ost-viewer.html) inaweza kutumika.

### Kupata Viambatisho

Viambatisho vilivyopotea vinaweza kupatikana kutoka:

- Kwa **IE10**: `%APPDATA%\Local\Microsoft\Windows\Temporary Internet Files\Content.Outlook`
- Kwa **IE11 na zaidi**: `%APPDATA%\Local\Microsoft\InetCache\Content.Outlook`

### Faili za Thunderbird MBOX

**Thunderbird** hutumia **faili za MBOX** kuhifadhi data, zilizoko kwenye `\Users\%USERNAME%\AppData\Roaming\Thunderbird\Profiles`.

### Vielelezo vya Picha

- **Windows XP na 8-8.1**: Kufikia saraka na vielelezo huzalisha faili ya `thumbs.db` ikihifadhi hakikisho za picha, hata baada ya kufutwa.
- **Windows 7/10**: `thumbs.db` huzalishwa unapofikia kwa njia ya mtandao kupitia njia ya UNC.
- **Windows Vista na mpya**: Hakikisho za vielelezo zimejumuishwa katika `%userprofile%\AppData\Local\Microsoft\Windows\Explorer` na faili zinaitwa **thumbcache\_xxx.db**. [**Thumbsviewer**](https://thumbsviewer.github.io) na [**ThumbCache Viewer**](https://thumbcacheviewer.github.io) ni zana za kuona faili hizi.

### Taarifa za Usajili wa Windows

Usajili wa Windows, ukihifadhi data nyingi za shughuli za mfumo na mtumiaji, imejumuishwa katika faili zifuatazo:

- `%windir%\System32\Config` kwa funguo za chini za `HKEY_LOCAL_MACHINE` mbalimbali.
- `%UserProfile%{User}\NTUSER.DAT` kwa `HKEY_CURRENT_USER`.
- Windows Vista na toleo jipya hufanya nakala za usalama za faili za usajili za `HKEY_LOCAL_MACHINE` katika `%Windir%\System32\Config\RegBack\`.
- Aidha, taarifa za utekelezaji wa programu zimehifadhiwa katika `%UserProfile%\{User}\AppData\Local\Microsoft\Windows\USERCLASS.DAT` kutoka Windows Vista na Windows 2008 Server kuendelea.

### Zana

Baadhi ya zana ni muhimu kuchambua faili za usajili:

* **Mhariri wa Usajili**: Imewekwa kwenye Windows. Ni GUI ya kutembea kupitia usajili wa Windows wa kikao cha sasa.
* [**Mchunguzi wa Usajili**](https://ericzimmerman.github.io/#!index.md): Inakuruhusu kupakia faili ya usajili na kutembea kupitia kwa GUI. Pia ina Alama za Vitabu zinazoonyesha funguo zenye habari muhimu.
* [**RegRipper**](https://github.com/keydet89/RegRipper3.0): Tena, ina GUI inayoruhusu kutembea kupitia usajili uliopakiwa na pia ina programu-jalizi zinazoonyesha habari muhimu ndani ya usajili uliopakiwa.
* [**Windows Registry Recovery**](https://www.mitec.cz/wrr.html): Programu nyingine ya GUI inayoweza kutoa habari muhimu kutoka kwa usajili uliopakiwa.

### Kurejesha Elementi Iliyofutwa

Wakati funguo inapofutwa, inaashiria hivyo, lakini mpaka nafasi inayochukua inahitajika, haitaondolewa. Kwa hivyo, kutumia zana kama **Mchunguzi wa Usajili** inawezekana kurejesha funguo hizi zilizofutwa.

### Muda wa Mwisho wa Kuandika

Kila Funguo-Kitu kina **muda** unaonyesha wakati wa mwisho uliobadilishwa.

### SAM

Faili/hive ya **SAM** ina **watumiaji, vikundi na nywila za watumiaji** za mfumo.

Katika `SAM\Domains\Account\Users` unaweza kupata jina la mtumiaji, RID, kuingia mwisho, kuingia kushindwa mwisho, kuhesabu kuingia, sera ya nywila na wakati akaunti ilianzishwa. Ili kupata **nywila** unahitaji pia faili/hive ya **SYSTEM**.

### Viingilio Vyenye Kuvutia katika Usajili wa Windows

{% content-ref url="interesting-windows-registry-keys.md" %}
[interesting-windows-registry-keys.md](interesting-windows-registry-keys.md)
{% endcontent-ref %}

## Programu Zilizotekelezwa

### Mchakato wa Msingi wa Windows

Katika [makala hii](https://jonahacks.medium.com/investigating-common-windows-processes-18dee5f97c1d) unaweza kujifunza kuhusu mchakato wa kawaida wa Windows ili kugundua tabia za shaka.

### Programu za Hivi Karibuni za Windows

Ndani ya usajili wa `NTUSER.DAT` katika njia `Software\Microsoft\Current Version\Search\RecentApps` unaweza kupata funguo za ziada zenye habari kuhusu **programu iliyotekelezwa**, **wakati wa mwisho** ilitekelezwa, na **idadi ya mara** iliyozinduliwa.

### BAM (Msimamizi wa Shughuli za Nyuma)

Unaweza kufungua faili ya `SYSTEM` na mhariri wa usajili na ndani ya njia `SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID}` unaweza kupata habari kuhusu **programu zilizotekelezwa na kila mtumiaji** (kumbuka `{SID}` katika njia) na **wakati** walitekelezwa (wakati uko ndani ya thamani ya Data ya usajili).

### Windows Prefetch

Prefetching ni mbinu inayoruhusu kompyuta kupakua kimya-kimya **rasilimali zinazohitajika** ili kuonyesha yaliyomo ambayo mtumiaji **anaweza kufikia hivi karibuni** ili rasilimali ziweze kupatikana haraka.

Windows prefetch inajumuisha kuunda **hifadhi za programu zilizotekelezwa** ili ziweze kupakia haraka. Hifadhi hizi zinaundwa kama faili za `.pf` ndani ya njia: `C:\Windows\Prefetch`. Kuna kikomo cha faili 128 katika XP/VISTA/WIN7 na faili 1024 katika Win8/Win10.

Jina la faili linaundwa kama `{jina_la_programu}-{hash}.pf` (hash inategemea njia na hoja za programu). Katika W10 faili hizi zimehifadhiwa. Tafadhali kumbuka kwamba uwepo wa faili pekee unaonyesha kwamba **programu ilitekelezwa** wakati fulani.

Faili ya `C:\Windows\Prefetch\Layout.ini` ina **majina ya saraka za faili zilizoprefetch**. Faili hii ina **habari kuhusu idadi ya utekelezaji**, **tarehe** za utekelezaji na **faili** **zilizofunguliwa** na programu.

Kutazama faili hizi unaweza kutumia zana [**PEcmd.exe**](https://github.com/EricZimmerman/PECmd):
```bash
.\PECmd.exe -d C:\Users\student\Desktop\Prefetch --html "C:\Users\student\Desktop\out_folder"
```
![](<../../../.gitbook/assets/image (487).png>)

### Superprefetch

**Superprefetch** ina lengo sawa na prefetch, **kuwezesha programu kupakia haraka** kwa kutabiri ni nini kitakachopakiwa next. Hata hivyo, haisubiri huduma ya prefetch.\
Huduma hii itazalisha faili za database katika `C:\Windows\Prefetch\Ag*.db`.

Katika hizi database unaweza kupata **jina** la **programu**, **idadi** ya **utekelezaji**, **faili** **zilizofunguliwa**, **upatikanaji** wa **kiasi**, **njia kamili**, **muda** na **muda wa alama**.

Unaweza kupata habari hii kwa kutumia zana [**CrowdResponse**](https://www.crowdstrike.com/resources/community-tools/crowdresponse/).

### SRUM

**System Resource Usage Monitor** (SRUM) **inachunguza** **rasilimali** **zilizotumiwa** **na mchakato**. Ilianza katika W8 na hifadhi data katika database ya ESE iliyoko `C:\Windows\System32\sru\SRUDB.dat`.

Inatoa habari ifuatayo:

* AppID na Njia
* Mtumiaji aliyetekeleza mchakato
* Bytes Zilizotumwa
* Bytes Zilizopokelewa
* Interface ya Mtandao
* Muda wa Uunganisho
* Muda wa Mchakato

Habari hii huzalishwa upya kila baada ya dakika 60.

Unaweza kupata tarehe kutoka kwa faili hii kwa kutumia zana [**srum\_dump**](https://github.com/MarkBaggett/srum-dump).
```bash
.\srum_dump.exe -i C:\Users\student\Desktop\SRUDB.dat -t SRUM_TEMPLATE.xlsx -o C:\Users\student\Desktop\srum
```
### AppCompatCache (ShimCache)

**AppCompatCache**, inayojulikana pia kama **ShimCache**, ni sehemu ya **Application Compatibility Database** iliyoendelezwa na **Microsoft** kushughulikia matatizo ya utangamano wa programu. Kipengele hiki cha mfumo hurekodi vipande mbalimbali vya metadata ya faili, ambavyo ni pamoja na:

- Njia kamili ya faili
- Ukubwa wa faili
- Muda wa Mwisho wa Kubadilishwa chini ya **$Standard\_Information** (SI)
- Muda wa Mwisho wa Kusasishwa wa ShimCache
- Bendera ya Utekelezaji wa Mchakato

Data kama hiyo hifadhiwa ndani ya usajili katika maeneo maalum kulingana na toleo la mfumo wa uendeshaji:

- Kwa XP, data hifadhiwa chini ya `SYSTEM\CurrentControlSet\Control\SessionManager\Appcompatibility\AppcompatCache` na uwezo wa kuingiza vipengele 96.
- Kwa Server 2003, pamoja na toleo za Windows 2008, 2012, 2016, 7, 8, na 10, njia ya kuhifadhi ni `SYSTEM\CurrentControlSet\Control\SessionManager\AppcompatCache\AppCompatCache`, ikiruhusu kuingiza vipengele 512 na 1024, mtawalia.

Kutafsiri habari iliyohifadhiwa, zana ya [**AppCompatCacheParser**](https://github.com/EricZimmerman/AppCompatCacheParser) inapendekezwa kutumika.

![](<../../../.gitbook/assets/image (488).png>)

### Amcache

Faili ya **Amcache.hve** ni msingi wa usajili unaorekodi maelezo kuhusu programu zilizotekelezwa kwenye mfumo. Kawaida hupatikana kwenye `C:\Windows\AppCompat\Programas\Amcache.hve`.

Faili hii ni muhimu kwa kuhifadhi rekodi za michakato iliyotekelezwa hivi karibuni, ikiwa ni pamoja na njia za faili za utekelezaji na hash zao za SHA1. Taarifa hii ni ya thamani kwa kufuatilia shughuli za programu kwenye mfumo.

Kutolea nje na kuchambua data kutoka kwa **Amcache.hve**, zana ya [**AmcacheParser**](https://github.com/EricZimmerman/AmcacheParser) inaweza kutumika. Amri ifuatayo ni mfano wa jinsi ya kutumia AmcacheParser kutafsiri maudhui ya faili ya **Amcache.hve** na kutoa matokeo kwa muundo wa CSV:
```bash
AmcacheParser.exe -f C:\Users\genericUser\Desktop\Amcache.hve --csv C:\Users\genericUser\Desktop\outputFolder
```
Miongoni mwa faili za CSV zilizozalishwa, `Amcache_Unassociated file entries` ni muhimu sana kutokana na habari tajiri inayotoa kuhusu viingilio vya faili visivyo husishwa.

Faili ya CVS yenye kuvutia zaidi iliyozalishwa ni `Amcache_Unassociated file entries`.

### RecentFileCache

Kielelezo hiki kinaweza kupatikana tu katika W7 katika `C:\Windows\AppCompat\Programs\RecentFileCache.bcf` na ina habari kuhusu utekelezaji wa hivi karibuni wa baadhi ya binaries.

Unaweza kutumia zana [**RecentFileCacheParse**](https://github.com/EricZimmerman/RecentFileCacheParser) kuchambua faili.

### Kazi zilizopangwa

Unaweza kuzitoa kutoka `C:\Windows\Tasks` au `C:\Windows\System32\Tasks` na kusoma kama XML.

### Huduma

Unaweza kuzipata katika usajili chini ya `SYSTEM\ControlSet001\Services`. Unaweza kuona nini kitatekelezwa na lini.

### **Duka la Windows**

Programu zilizosakinishwa zinaweza kupatikana katika `\ProgramData\Microsoft\Windows\AppRepository\`\
Hifadhi hii ina **logi** na **kila programu iliyosakinishwa** kwenye mfumo ndani ya **database** **`StateRepository-Machine.srd`**.

Ndani ya jedwali la Programu katika hii database, ni pamoja na vitengo: "Kitambulisho cha Programu", "Nambari ya Pakiti", na "Jina la Kuonyesha". Vitengo hivi vina habari kuhusu programu zilizosakinishwa awali na zilizosakinishwa na inaweza kupatikana ikiwa baadhi ya programu zilisakinishwa kwa sababu vitambulisho vya programu zilizosakinishwa vinapaswa kuwa vya mfululizo.

Pia ni **pamoja na programu iliyosakinishwa** katika njia ya usajili: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Applications\`\
Na **programu zilizofutwa** katika: `Software\Microsoft\Windows\CurrentVersion\Appx\AppxAllUserStore\Deleted\`

## Matukio ya Windows

Habari inayoonekana ndani ya matukio ya Windows ni:

* Kilichotokea
* Muda (UTC + 0)
* Watumiaji waliohusika
* Wenyeji waliohusika (jina la mwenyeji, IP)
* Mali zilizopatikana (faili, folda, printa, huduma)

Vipakuli viko katika `C:\Windows\System32\config` kabla ya Windows Vista na katika `C:\Windows\System32\winevt\Logs` baada ya Windows Vista. Kabla ya Windows Vista, vipakuli vya matukio vilikuwa katika muundo wa binary na baada yake, wako katika **muundo wa XML** na hutumia kifaa cha **.evtx**.

Mahali pa faili za matukio zinaweza kupatikana katika usajili wa SYSTEM katika **`HKLM\SYSTEM\CurrentControlSet\services\EventLog\{Application|System|Security}`**

Zinaweza kuonekana kutoka kwa Mwangalizi wa Matukio ya Windows (**`eventvwr.msc`**) au kwa zana nyingine kama [**Mwangalizi wa Matukio**](https://eventlogxp.com) **au** [**Mwangalizi wa Evtx/EvtxECmd**](https://ericzimmerman.github.io/#!index.md)**.**

## Kuelewa Kuingiza Matukio ya Usalama ya Windows

Matukio ya ufikiaji hurekodiwa katika faili ya usanidi wa usalama iliyoko katika `C:\Windows\System32\winevt\Security.evtx`. Ukubwa wa faili hii unaweza kurekebishwa, na unapofikia uwezo wake, matukio ya zamani hufutwa. Matukio yaliyorekodiwa ni pamoja na kuingia na kutoka kwa watumiaji, hatua za watumiaji, na mabadiliko kwa mipangilio ya usalama, pamoja na ufikiaji wa mali kama faili, folda, na mali zilizoshirikiwa.

### Vitambulisho muhimu vya Matukio ya Usajili wa Watumiaji:

- **Tukio la 4624**: Inaonyesha mafanikio ya kuingia kwa mtumiaji.
- **Tukio la 4625**: Inaashiria kushindwa kwa kuingia.
- **Matukio ya 4634/4647**: Yanawakilisha matukio ya kutoka kwa mtumiaji.
- **Tukio la 4672**: Linabainisha kuingia na mamlaka ya usimamizi.

#### Aina za Ndani ndani ya Tukio la 4634/4647:

- **Mwingiliano (2)**: Kuingia moja kwa moja kwa mtumiaji.
- **Mtandao (3)**: Kufikia folda zilizoshirikiwa.
- **Batch (4)**: Utekelezaji wa michakato ya batch.
- **Huduma (5)**: Kuzindua huduma.
- **Mwakilishi (6)**: Uthibitishaji wa mwakilishi.
- **Kufungua (7)**: Skrini iliyofunguliwa kwa nenosiri.
- **Mtandao wa Wazi (8)**: Uhamishaji wa nenosiri wazi, mara nyingi kutoka kwa IIS.
- **Vibali Vipya (9)**: Matumizi ya vibali tofauti kwa ufikiaji.
- **Mwingiliano wa Mbali (10)**: Kuingia kwa mbali kwenye dawati au huduma za terminali.
- **Mwingiliano wa Akiba (11)**: Kuingia na vibali vilivyohifadhiwa bila mawasiliano na kituo cha upelekaji wa kikoa.
- **Mwingiliano wa Mbali wa Akiba (12)**: Kuingia kwa mbali na vibali vilivyohifadhiwa.
- **Kufungua wa Akiba (13)**: Kufungua kwa vibali vilivyohifadhiwa.

#### Vigezo vya Hali na Hali za Ndani kwa Tukio la 4625:

- **0xC0000064**: Jina la mtumiaji halipo - Inaweza kuashiria shambulio la uchunguzi wa majina ya watumiaji.
- **0xC000006A**: Jina sahihi la mtumiaji lakini nenosiri sio sahihi - Jaribio la kudhanisha au kujaribu kuvunja nenosiri.
- **0xC0000234**: Akaunti ya mtumiaji imefungwa - Inaweza kufuata shambulio la kuvunja nenosiri linalosababisha kuingia kwa mara nyingi.
- **0xC0000072**: Akaunti imelemazwa - Jaribio lisiloruhusiwa la kupata akaunti zilizolemazwa.
- **0xC000006F**: Kuingia nje ya muda ulioruhusiwa - Inaonyesha jaribio la kupata nje ya masaa ya kuingia yaliyowekwa, ishara inayowezekana ya ufikiaji usioruhusiwa.
- **0xC0000070**: Ukiukaji wa vikwazo vya kituo cha kazi - Inaweza kuwa jaribio la kuingia kutoka eneo lisiloruhusiwa.
- **0xC0000193**: Akaunti imeisha muda wake - Jaribio la kupata na akaunti za watumiaji zilizomaliza muda.
- **0xC0000071**: Nenosiri limeisha muda wake - Jaribio la kuingia na nywila zilizopitwa na wakati.
- **0xC0000133**: Matatizo ya usawazishaji wa muda - Tofauti kubwa za muda kati ya mteja na seva zinaweza kuwa ishara ya mashambulizi ya hali ya juu kama vile pass-the-ticket.
- **0xC0000224**: Inahusu hitilafu ya mfumo badala ya suala la usalama.
- **0xC000015b**: Aina iliyokataliwa ya kuingia - Jaribio la kupata na aina isiyoruhusiwa ya kuingia, kama mtumiaji anayejaribu kutekeleza kuingia kwa huduma.

#### Tukio la 4616:
- **Mabadiliko ya Muda**: Kubadilisha muda wa mfumo, inaweza kuficha mstari wa matukio.

#### Tukio la 6005 na 6006:
- **Kuanza na Kuzima kwa Mfumo**: Tukio la 6005 linaonyesha kuanza kwa mfumo, wakati Tukio la 6006 linamaanisha kuzima kwake.

#### Tukio la 1102:
- **Kufuta Kumbukumbu**: Kufuta kwa kumbukumbu za usalama, ambayo mara nyingi ni ishara ya kuficha shughuli haramu.

#### Matukio kwa Kufuatilia Kifaa cha USB:
- **20001 / 20003 / 10000**: Uunganisho wa kifaa cha USB mara ya kwanza.
- **10100**: Sasisho la dereva wa USB.
- **Tukio la 112**: Wakati wa kuingiza kifaa cha USB.

Kwa mifano halisi ya kusimuliza aina hizi za kuingia na fursa za kudondosha vibali, tazama [mwongozo kamili wa Altered Security](https://www.alteredsecurity.com/post/fantastic-windows-logon-types-and-where-to-find-credentials-in-them).

Maelezo ya matukio, ikiwa ni pamoja na vigezo vya hali na hali za ndani, hutoa ufahamu zaidi katika sababu za matukio, hasa muhimu katika Tukio la 4625.

### Kurejesha Matukio ya Windows

Ili kuongeza nafasi za kurejesha Matukio ya Windows yaliyofutwa, ni vyema kuzima kompyuta ya washukiwa moja kwa moja kwa kuitoa umeme. **Bulk_extractor**, zana ya kurejesha ikilenga kifaa cha `.evtx`, inapendekezwa kujaribu kurejesha matukio kama hayo.

### Kutambua Mashambulizi ya Kawaida kupitia Matukio ya Windows

Kwa mwongozo kamili wa kutumia Vitambulisho vya Matukio ya Windows kutambua mashambulizi ya mtandao ya kawaida, tembelea [Red Team Recipe](https://redteamrecipe.com/event-codes/).

#### Mashambulizi ya Kuvunja Nguvu

Yanaweza kutambulika na rekodi nyingi za Tukio la 4625, ikifuatiwa na Tukio la 4624 ikiwa shambulio linafanikiwa.

#### Mabadiliko ya Muda

Yaliyorekodiwa na Tukio la 4616, mabadiliko ya muda wa mfumo yanaweza kufanya uchambuzi wa kiforensiki kuwa mgumu.

#### Kufuatilia Kifaa cha USB

Matukio muhimu ya Mfumo kwa kufuatilia kifaa cha USB ni pamoja na 20001/20003/10000 kwa matumizi ya kwanza, 10100 kwa sasisho za dereva, na Tukio la 112 kutoka kwa DeviceSetupManager kwa alama za wakati wa kuingiza.
#### Matukio ya Nguvu ya Mfumo

Tukio la 6005 linaashiria kuanza kwa mfumo, wakati Tukio la 6006 linabainisha kuzimwa.

#### Kufuta Kumbukumbu

Usalama wa Tukio la 1102 unamaanisha kufutwa kwa kumbukumbu, tukio muhimu kwa uchambuzi wa kiforensiki.

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}
