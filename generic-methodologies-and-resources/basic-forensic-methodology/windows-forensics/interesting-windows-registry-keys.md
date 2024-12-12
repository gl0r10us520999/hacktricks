# Interesantni Windows registarski ključevi

### Interesantni Windows registarski ključevi

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks obuka AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks obuka GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Delite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

### **Informacije o verziji Windowsa i vlasniku**
- Na lokaciji **`Software\Microsoft\Windows NT\CurrentVersion`**, možete pronaći verziju Windowsa, Service Pack, vreme instalacije i ime registrovanog vlasnika na jednostavan način.

### **Ime računara**
- Ime računara se nalazi pod **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Podešavanje vremenske zone**
- Vremenska zona sistema se čuva u **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Pracenje vremena pristupa**
- Podrazumevano, praćenje vremena poslednjeg pristupa je isključeno (**`NtfsDisableLastAccessUpdate=1`**). Da biste ga omogućili, koristite:
`fsutil behavior set disablelastaccess 0`

### Verzije Windowsa i Service Pack-ovi
- **Verzija Windowsa** označava izdanje (npr. Home, Pro) i njegovo izdanje (npr. Windows 10, Windows 11), dok **Service Pack-ovi** predstavljaju ažuriranja koja uključuju ispravke i ponekad nove funkcije.

### Omogućavanje vremena poslednjeg pristupa
- Omogućavanje praćenja vremena poslednjeg pristupa omogućava vam da vidite kada su datoteke poslednji put otvorene, što može biti ključno za forenzičku analizu ili praćenje sistema.

### Detalji o mrežnim informacijama
- Registar čuva obimne podatke o mrežnim konfiguracijama, uključujući **tipove mreža (bežične, kablove, 3G)** i **kategorije mreže (Javna, Privatna/Domaća, Domen/Radna)**, što je od vitalnog značaja za razumevanje postavki bezbednosti mreže i dozvola.

### Keširanje klijentske strane (CSC)
- **CSC** poboljšava pristup datotekama van mreže keširanjem kopija deljenih datoteka. Različite postavke **CSCFlags** kontrolišu kako i koje datoteke su keširane, utičući na performanse i korisničko iskustvo, posebno u okruženjima sa povremenom konekcijom.

### Programi koji se automatski pokreću
- Programi navedeni u različitim `Run` i `RunOnce` registarskim ključevima automatski se pokreću prilikom pokretanja sistema, utičući na vreme pokretanja sistema i potencijalno predstavljajući tačke interesa za identifikaciju malvera ili neželjenog softvera.

### Shellbags
- **Shellbags** ne samo što čuvaju preferencije za prikaze fascikli, već pružaju i forenzičke dokaze o pristupu fasciklama čak i ako fascikla više ne postoji. Oni su neprocenjivi za istrage, otkrivajući aktivnosti korisnika koje nisu očigledne na druge načine.

### USB informacije i forenzika
- Detalji čuvani u registru o USB uređajima mogu pomoći u praćenju koji su uređaji bili povezani sa računarom, potencijalno povezujući uređaj sa prenosom osetljivih datoteka ili incidentima neovlašćenog pristupa.

### Serijski broj zapremine
- **Serijski broj zapremine** može biti ključan za praćenje specifične instance sistema datoteka, koristan u forenzičkim scenarijima gde je potrebno utvrditi poreklo datoteke na različitim uređajima.

### **Detalji isključivanja**
- Vreme isključivanja i broj (samo za XP) čuvaju se u **`System\ControlSet001\Control\Windows`** i **`System\ControlSet001\Control\Watchdog\Display`**.

### **Konfiguracija mreže**
- Za detaljne informacije o mrežnim interfejsima, pogledajte **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Prva i poslednja vremena povezivanja na mrežu, uključujući VPN veze, beleže se pod različitim putanjama u **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Deljeni folderi**
- Deljeni folderi i podešavanja nalaze se pod **`System\ControlSet001\Services\lanmanserver\Shares`**. Postavke keširanja klijentske strane (CSC) određuju dostupnost datoteka van mreže.

### **Programi koji se automatski pokreću**
- Putanje poput **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** i slični unosi pod `Software\Microsoft\Windows\CurrentVersion` detaljno opisuju programe postavljene da se pokreću prilikom pokretanja sistema.

### **Pretrage i otkucane putanje**
- Pretrage i otkucane putanje u Exploreru prate se u registru pod **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** za WordwheelQuery i TypedPaths, redom.

### **Nedavni dokumenti i Office datoteke**
- Nedavni dokumenti i Office datoteke na koje je pristupano beleže se u `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` i specifičnim putanjama za verzije Office-a.

### **Nedavno korišćeni (MRU) elementi**
- Liste MRU, koje pokazuju nedavne putanje do datoteka i komande, čuvaju se u različitim podključevima `ComDlg32` i `Explorer` pod `NTUSER.DAT`.

### **Pracenje aktivnosti korisnika**
- Funkcija User Assist beleži detaljnu statistiku korišćenja aplikacija, uključujući broj pokretanja i vreme poslednjeg pokretanja, na lokaciji **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Analiza Shellbags-a**
- Shellbags, koji otkrivaju detalje pristupa fasciklama, čuvaju se u `USRCLASS.DAT` i `NTUSER.DAT` pod `Software\Microsoft\Windows\Shell`. Koristite **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** za analizu.

### **Istorija USB uređaja**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** i **`HKLM\SYSTEM\ControlSet001\Enum\USB`** sadrže bogate detalje o povezanim USB uređajima, uključujući proizvođača, naziv proizvoda i vremenske oznake povezivanja.
- Korisnik povezan sa određenim USB uređajem može se locirati pretragom `NTUSER.DAT` košnica za **{GUID}** uređaja.
- Poslednji montirani uređaj i njegov serijski broj zapremine mogu se pratiti kroz `System\MountedDevices` i `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, redom.

Ovaj vodič sažima ključne putanje i metode za pristup detaljnim informacijama o sistemu, mreži i aktivnostima korisnika na Windows sistemima, teži ka jasnoći i upotrebljivosti.

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks obuka AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks obuka GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Delite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
