# Interessante Windows Registriesleutels

### Interessante Windows Registriesleutels

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Ondersteun HackTricks</summary>

* Kyk na die [**subskripsieplanne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}


### **Windows Weergawe en Eienaar Inligting**
- Geleë by **`Software\Microsoft\Windows NT\CurrentVersion`**, sal jy die Windows weergawe, dienspakket, installasietyd en die geregistreerde eienaar se naam op 'n eenvoudige manier vind.

### **Rekenaarnaam**
- Die gasheernaam is onder **`System\ControlSet001\Control\ComputerName\ComputerName`** te vind.

### **Tydsone Instelling**
- Die stelseltijdsones is gestoor in **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Toegangstyd Opvolging**
- Standaard is die laaste toegangstyd opvolging afgeskakel (**`NtfsDisableLastAccessUpdate=1`**). Om dit in te skakel, gebruik:
`fsutil behavior set disablelastaccess 0`

### Windows Weergawes en Dienspakkette
- Die **Windows weergawe** dui die uitgawe aan (bv. Tuis, Pro) en sy vrystelling (bv. Windows 10, Windows 11), terwyl **dienspakkette** opdaterings is wat regstellings en soms nuwe funksies insluit.

### Laaste Toegangstyd Inskakeling
- Die inskakeling van laaste toegangstyd opvolging laat jou toe om te sien wanneer lêers laas geopen is, wat krities kan wees vir forensiese analise of stelseltelling.

### Netwerk Inligting Besonderhede
- Die register hou uitgebreide data oor netwerk konfigurasies, insluitend **tipes netwerke (draadloos, kabel, 3G)** en **netwerk kategorieë (Publiek, Privaat/Huis, Domein/ Werk)**, wat noodsaaklik is om netwerk sekuriteitsinstellings en toestemmings te verstaan.

### Kliëntkant Kaping (CSC)
- **CSC** verbeter offline lêer toegang deur kopieë van gedeelde lêers te kas. Verskillende **CSCFlags** instellings beheer hoe en watter lêers gekas word, wat prestasie en gebruikerservaring beïnvloed, veral in omgewings met intermitterende konnektiwiteit.

### AutoStart Programme
- Programme wat in verskeie `Run` en `RunOnce` registriesleutels gelys is, word outomaties by opstart gelaai, wat die stelselaanlooptyd beïnvloed en moontlik punte van belang kan wees om malware of ongewenste sagteware te identifiseer.

### Shellbags
- **Shellbags** stoor nie net voorkeure vir vouer aansigte nie, maar bied ook forensiese bewyse van vouer toegang selfs al bestaan die vouer nie meer nie. Hulle is van onskatbare waarde vir ondersoeke, wat gebruikersaktiwiteit onthul wat nie duidelik is deur ander middele nie.

### USB Inligting en Forensika
- Die besonderhede wat in die register oor USB toestelle gestoor is, kan help om te spoor watter toestelle aan 'n rekenaar gekoppel was, wat moontlik 'n toestel aan sensitiewe lêeroordrag of ongeoorloofde toegang insidente kan koppel.

### Volume Serienommer
- Die **Volume Serienommer** kan noodsaaklik wees om die spesifieke geval van 'n lêerstelsel te spoor, nuttig in forensiese scenario's waar lêer oorsprong oor verskillende toestelle gevestig moet word.

### **Afsluit Besonderhede**
- Afsluit tyd en telling (laasgenoemde slegs vir XP) word in **`System\ControlSet001\Control\Windows`** en **`System\ControlSet001\Control\Watchdog\Display`** gehou.

### **Netwerk Konfigurasie**
- Vir gedetailleerde netwerk koppelvlak inligting, verwys na **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Eerste en laaste netwerkverbinding tye, insluitend VPN verbindings, word onder verskeie paaie in **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`** gelog.

### **Gedeelde Vouers**
- Gedeelde vouers en instellings is onder **`System\ControlSet001\Services\lanmanserver\Shares`**. Die Kliëntkant Kaping (CSC) instellings bepaal offline lêer beskikbaarheid.

### **Programme wat Outomaties Begin**
- Paaie soos **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** en soortgelyke inskrywings onder `Software\Microsoft\Windows\CurrentVersion` detail programme wat ingestel is om by opstart te loop.

### **Soek en Getypte Paaie**
- Explorer soeke en getypte paaie word in die register onder **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** vir WordwheelQuery en GetyptePaaie, onderskeidelik, opgeteken.

### **Onlangse Dokumente en Office Lêers**
- Onlangse dokumente en Office lêers wat toegang verkry is, word opgemerk in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` en spesifieke Office weergawe paaie.

### **Mees Onlangs Gebruikte (MRU) Items**
- MRU lyste, wat onlangse lêer paaie en opdragte aandui, word in verskeie `ComDlg32` en `Explorer` subsleutels onder `NTUSER.DAT` gestoor.

### **Gebruikersaktiwiteit Opvolging**
- Die User Assist funksie log gedetailleerde toepassingsgebruik statistieke, insluitend loop telling en laaste loop tyd, by **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Shellbags Analise**
- Shellbags, wat vouer toegang besonderhede onthul, word in `USRCLASS.DAT` en `NTUSER.DAT` onder `Software\Microsoft\Windows\Shell` gestoor. Gebruik **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** vir analise.

### **USB Toestel Geskiedenis**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** en **`HKLM\SYSTEM\ControlSet001\Enum\USB`** bevat ryk besonderhede oor gekoppelde USB toestelle, insluitend vervaardiger, produknaam, en verbindingstydstempels.
- Die gebruiker wat met 'n spesifieke USB toestel geassosieer word, kan bepaal word deur `NTUSER.DAT` hives vir die toestel se **{GUID}** te soek.
- Die laaste gemonteerde toestel en sy volume serienommer kan opgespoor word deur `System\MountedDevices` en `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, onderskeidelik.

Hierdie gids konsentreer die belangrike paaie en metodes om toegang te verkry tot gedetailleerde stelsel-, netwerk- en gebruikersaktiwiteit inligting op Windows stelsels, met die doel om duidelikheid en bruikbaarheid te bied.
