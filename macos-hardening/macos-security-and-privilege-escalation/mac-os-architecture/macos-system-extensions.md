# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

Im Gegensatz zu Kernel Extensions **laufen System Extensions im Benutzermodus** anstelle des Kernelmodus, was das Risiko eines Systemabsturzes aufgrund von Erweiterungsfehlern verringert.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Es gibt drei Arten von Systemerweiterungen: **DriverKit**-Erweiterungen, **Netzwerk**-Erweiterungen und **Endpoint Security**-Erweiterungen.

### **DriverKit-Erweiterungen**

DriverKit ist ein Ersatz für Kernel-Erweiterungen, die **Hardwareunterstützung** bieten. Es ermöglicht Gerätetreibern (wie USB-, Serial-, NIC- und HID-Treibern), im Benutzermodus anstelle des Kernelmodus zu laufen. Das DriverKit-Framework umfasst **Benutzermodusversionen bestimmter I/O Kit-Klassen**, und der Kernel leitet normale I/O Kit-Ereignisse an den Benutzermodus weiter, was eine sicherere Umgebung für diese Treiber bietet.

### **Netzwerk-Erweiterungen**

Netzwerk-Erweiterungen bieten die Möglichkeit, Netzwerkverhalten anzupassen. Es gibt mehrere Arten von Netzwerk-Erweiterungen:

* **App Proxy**: Dies wird verwendet, um einen VPN-Client zu erstellen, der ein flow-orientiertes, benutzerdefiniertes VPN-Protokoll implementiert. Das bedeutet, dass er den Netzwerkverkehr basierend auf Verbindungen (oder Flows) anstelle einzelner Pakete verarbeitet.
* **Packet Tunnel**: Dies wird verwendet, um einen VPN-Client zu erstellen, der ein packet-orientiertes, benutzerdefiniertes VPN-Protokoll implementiert. Das bedeutet, dass er den Netzwerkverkehr basierend auf einzelnen Paketen verarbeitet.
* **Filter Data**: Dies wird verwendet, um Netzwerk-"Flows" zu filtern. Es kann Netzwerkdaten auf Flussebene überwachen oder ändern.
* **Filter Packet**: Dies wird verwendet, um einzelne Netzwerkpakete zu filtern. Es kann Netzwerkdaten auf Paketebene überwachen oder ändern.
* **DNS Proxy**: Dies wird verwendet, um einen benutzerdefinierten DNS-Anbieter zu erstellen. Es kann verwendet werden, um DNS-Anfragen und -Antworten zu überwachen oder zu ändern.

## Endpoint Security Framework

Endpoint Security ist ein von Apple in macOS bereitgestelltes Framework, das eine Reihe von APIs für die Systemsicherheit bietet. Es ist für die Verwendung durch **Sicherheitsanbieter und Entwickler gedacht, um Produkte zu erstellen, die Systemaktivitäten überwachen und steuern können**, um böswillige Aktivitäten zu identifizieren und zu schützen.

Dieses Framework bietet eine **Sammlung von APIs zur Überwachung und Steuerung von Systemaktivitäten**, wie z.B. Prozessausführungen, Dateisystemereignisse, Netzwerk- und Kernelereignisse.

Der Kern dieses Frameworks ist im Kernel implementiert, als Kernel Extension (KEXT) unter **`/System/Library/Extensions/EndpointSecurity.kext`**. Diese KEXT besteht aus mehreren wichtigen Komponenten:

* **EndpointSecurityDriver**: Dies fungiert als "Einstiegspunkt" für die Kernel-Erweiterung. Es ist der Hauptinteraktionspunkt zwischen dem OS und dem Endpoint Security-Framework.
* **EndpointSecurityEventManager**: Diese Komponente ist verantwortlich für die Implementierung von Kernel-Hooks. Kernel-Hooks ermöglichen es dem Framework, Systemereignisse zu überwachen, indem Systemaufrufe abgefangen werden.
* **EndpointSecurityClientManager**: Dies verwaltet die Kommunikation mit Benutzermodus-Clients und verfolgt, welche Clients verbunden sind und Ereignisbenachrichtigungen erhalten müssen.
* **EndpointSecurityMessageManager**: Dies sendet Nachrichten und Ereignisbenachrichtigungen an Benutzermodus-Clients.

Die Ereignisse, die das Endpoint Security-Framework überwachen kann, sind in folgende Kategorien unterteilt:

* Dateiereignisse
* Prozessereignisse
* Socketereignisse
* Kernelereignisse (wie das Laden/Entladen einer Kernel-Erweiterung oder das Öffnen eines I/O Kit-Geräts)

### Architektur des Endpoint Security Frameworks

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Die Kommunikation im Benutzermodus** mit dem Endpoint Security-Framework erfolgt über die IOUserClient-Klasse. Es werden zwei verschiedene Unterklassen verwendet, abhängig von der Art des Aufrufers:

* **EndpointSecurityDriverClient**: Dies erfordert die Berechtigung `com.apple.private.endpoint-security.manager`, die nur vom Systemprozess `endpointsecurityd` gehalten wird.
* **EndpointSecurityExternalClient**: Dies erfordert die Berechtigung `com.apple.developer.endpoint-security.client`. Dies würde typischerweise von drittanbieter Sicherheitssoftware verwendet, die mit dem Endpoint Security-Framework interagieren muss.

Die Endpoint Security Extensions:**`libEndpointSecurity.dylib`** ist die C-Bibliothek, die Systemerweiterungen verwenden, um mit dem Kernel zu kommunizieren. Diese Bibliothek verwendet das I/O Kit (`IOKit`), um mit der Endpoint Security KEXT zu kommunizieren.

**`endpointsecurityd`** ist ein wichtiger Systemdaemon, der an der Verwaltung und dem Starten von Endpoint Security-Systemerweiterungen beteiligt ist, insbesondere während des frühen Bootprozesses. **Nur Systemerweiterungen**, die in ihrer `Info.plist`-Datei mit **`NSEndpointSecurityEarlyBoot`** gekennzeichnet sind, erhalten diese frühe Bootbehandlung.

Ein weiterer Systemdaemon, **`sysextd`**, **validiert Systemerweiterungen** und verschiebt sie an die richtigen Systemstandorte. Er fordert dann den relevanten Daemon auf, die Erweiterung zu laden. Das **`SystemExtensions.framework`** ist verantwortlich für das Aktivieren und Deaktivieren von Systemerweiterungen.

## Umgehung des ESF

ESF wird von Sicherheitstools verwendet, die versuchen, einen Red Teamer zu erkennen, daher klingt jede Information darüber, wie dies vermieden werden könnte, interessant.

### CVE-2021-30965

Das Problem ist, dass die Sicherheitsanwendung **Vollzugriff auf die Festplatte** benötigt. Wenn ein Angreifer dies entfernen könnte, könnte er verhindern, dass die Software ausgeführt wird:
```bash
tccutil reset All
```
Für **weitere Informationen** zu diesem Bypass und verwandten Themen siehe den Vortrag [#OBTS v5.0: "Die Achillesferse von EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Am Ende wurde dies behoben, indem die neue Berechtigung **`kTCCServiceEndpointSecurityClient`** der Sicherheitsanwendung, die von **`tccd`** verwaltet wird, gegeben wurde, sodass `tccutil` ihre Berechtigungen nicht löscht und sie weiterhin ausgeführt werden kann.

## Referenzen

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
