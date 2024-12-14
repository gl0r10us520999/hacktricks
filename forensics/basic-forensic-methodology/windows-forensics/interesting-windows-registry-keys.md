# Interessante Windows-Registrierungsschlüssel

### Interessante Windows-Registrierungsschlüssel

{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}


### **Windows-Version und Besitzerinformationen**
- Unter **`Software\Microsoft\Windows NT\CurrentVersion`** finden Sie die Windows-Version, das Service Pack, die Installationszeit und den Namen des registrierten Eigentümers auf einfache Weise.

### **Computername**
- Der Hostname befindet sich unter **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Zeitzoneneinstellung**
- Die Zeitzone des Systems wird in **`System\ControlSet001\Control\TimeZoneInformation`** gespeichert.

### **Zugriffszeitverfolgung**
- Standardmäßig ist die Verfolgung der letzten Zugriffszeit deaktiviert (**`NtfsDisableLastAccessUpdate=1`**). Um sie zu aktivieren, verwenden Sie:
`fsutil behavior set disablelastaccess 0`

### Windows-Versionen und Service Packs
- Die **Windows-Version** gibt die Edition an (z. B. Home, Pro) und deren Veröffentlichung (z. B. Windows 10, Windows 11), während **Service Packs** Updates sind, die Fehlerbehebungen und manchmal neue Funktionen enthalten.

### Aktivierung der letzten Zugriffszeit
- Die Aktivierung der Verfolgung der letzten Zugriffszeit ermöglicht es Ihnen zu sehen, wann Dateien zuletzt geöffnet wurden, was für die forensische Analyse oder Systemüberwachung entscheidend sein kann.

### Netzwerkdetails
- Die Registrierung enthält umfangreiche Daten zu Netzwerkkonfigurationen, einschließlich **Netzwerktypen (drahtlos, kabelgebunden, 3G)** und **Netzwerkkategorien (Öffentlich, Privat/Zuhause, Domäne/Arbeit)**, die für das Verständnis von Netzwerksicherheitseinstellungen und Berechtigungen von entscheidender Bedeutung sind.

### Clientseitiger Cache (CSC)
- **CSC** verbessert den Offline-Dateizugriff, indem Kopien von freigegebenen Dateien zwischengespeichert werden. Verschiedene **CSCFlags**-Einstellungen steuern, wie und welche Dateien zwischengespeichert werden, was die Leistung und Benutzererfahrung beeinflusst, insbesondere in Umgebungen mit intermittierender Konnektivität.

### Autostartprogramme
- Programme, die in verschiedenen `Run`- und `RunOnce`-Registrierungsschlüsseln aufgeführt sind, werden beim Start automatisch gestartet, was die Bootzeit des Systems beeinflusst und potenzielle Punkte zur Identifizierung von Malware oder unerwünschter Software sein kann.

### Shellbags
- **Shellbags** speichern nicht nur Präferenzen für Ordnersichten, sondern liefern auch forensische Beweise für den Ordnerzugriff, selbst wenn der Ordner nicht mehr existiert. Sie sind für Ermittlungen von unschätzbarem Wert, da sie Benutzeraktivitäten offenbaren, die durch andere Mittel nicht offensichtlich sind.

### USB-Informationen und Forensik
- Die in der Registrierung gespeicherten Details zu USB-Geräten können helfen, nachzuvollziehen, welche Geräte mit einem Computer verbunden waren, was möglicherweise ein Gerät mit sensiblen Dateiübertragungen oder Vorfällen unbefugten Zugriffs verknüpfen kann.

### Volumenseriennummer
- Die **Volumenseriennummer** kann entscheidend sein, um die spezifische Instanz eines Dateisystems zu verfolgen, was in forensischen Szenarien nützlich ist, in denen der Ursprung von Dateien über verschiedene Geräte hinweg festgestellt werden muss.

### **Herunterfahrdetails**
- Die Herunterfahrzeit und die Anzahl (letzteres nur für XP) werden in **`System\ControlSet001\Control\Windows`** und **`System\ControlSet001\Control\Watchdog\Display`** gespeichert.

### **Netzwerkkonfiguration**
- Für detaillierte Informationen zu Netzwerkinterfaces verweisen Sie auf **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Erste und letzte Netzwerkverbindungszeiten, einschließlich VPN-Verbindungen, werden unter verschiedenen Pfaden in **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`** protokolliert.

### **Freigegebene Ordner**
- Freigegebene Ordner und Einstellungen befinden sich unter **`System\ControlSet001\Services\lanmanserver\Shares`**. Die Einstellungen des Clientseitigen Caches (CSC) bestimmen die Verfügbarkeit von Offline-Dateien.

### **Programme, die automatisch starten**
- Pfade wie **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** und ähnliche Einträge unter `Software\Microsoft\Windows\CurrentVersion` geben Programme an, die beim Start ausgeführt werden sollen.

### **Suchanfragen und eingegebene Pfade**
- Explorer-Suchanfragen und eingegebene Pfade werden in der Registrierung unter **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** für WordwheelQuery und TypedPaths verfolgt.

### **Kürzlich verwendete Dokumente und Office-Dateien**
- Kürzlich aufgerufene Dokumente und Office-Dateien werden in `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` und spezifischen Office-Version-Pfaden vermerkt.

### **Zuletzt verwendete (MRU) Elemente**
- MRU-Listen, die kürzliche Dateipfade und Befehle anzeigen, werden in verschiedenen `ComDlg32`- und `Explorer`-Unterklassen unter `NTUSER.DAT` gespeichert.

### **Benutzeraktivitätsverfolgung**
- Die Benutzerassistenzfunktion protokolliert detaillierte Anwendungsnutzungsstatistiken, einschließlich der Anzahl der Ausführungen und der letzten Ausführungszeit, unter **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Shellbags-Analyse**
- Shellbags, die Details zum Ordnerzugriff offenbaren, werden in `USRCLASS.DAT` und `NTUSER.DAT` unter `Software\Microsoft\Windows\Shell` gespeichert. Verwenden Sie **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** zur Analyse.

### **USB-Gerätehistorie**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** und **`HKLM\SYSTEM\ControlSet001\Enum\USB`** enthalten umfangreiche Details zu angeschlossenen USB-Geräten, einschließlich Hersteller, Produktname und Verbindungszeitstempel.
- Der Benutzer, der mit einem bestimmten USB-Gerät verbunden ist, kann ermittelt werden, indem die `NTUSER.DAT`-Hives nach der **{GUID}** des Geräts durchsucht werden.
- Das zuletzt montierte Gerät und seine Volumenseriennummer können über `System\MountedDevices` und `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt` zurückverfolgt werden.

Dieser Leitfaden fasst die entscheidenden Pfade und Methoden zum Zugriff auf detaillierte System-, Netzwerk- und Benutzeraktivitätsinformationen auf Windows-Systemen zusammen, mit dem Ziel, Klarheit und Benutzerfreundlichkeit zu gewährleisten.



{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}
