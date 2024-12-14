# macOS MDM

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}

**Um mehr über macOS MDMs zu erfahren, siehe:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Grundlagen

### **MDM (Mobile Device Management) Übersicht**

[Mobile Device Management](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) wird verwendet, um verschiedene Endbenutzergeräte wie Smartphones, Laptops und Tablets zu überwachen. Insbesondere für Apples Plattformen (iOS, macOS, tvOS) umfasst es eine Reihe spezialisierter Funktionen, APIs und Praktiken. Der Betrieb von MDM hängt von einem kompatiblen MDM-Server ab, der entweder kommerziell erhältlich oder Open Source ist und das [MDM-Protokoll](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf) unterstützen muss. Wichtige Punkte sind:

* Zentralisierte Kontrolle über Geräte.
* Abhängigkeit von einem MDM-Server, der dem MDM-Protokoll entspricht.
* Fähigkeit des MDM-Servers, verschiedene Befehle an Geräte zu senden, z. B. remote Datenlöschung oder Konfigurationsinstallation.

### **Grundlagen des DEP (Device Enrollment Program)**

Das [Device Enrollment Program](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP), das von Apple angeboten wird, vereinfacht die Integration von Mobile Device Management (MDM), indem es eine Zero-Touch-Konfiguration für iOS-, macOS- und tvOS-Geräte ermöglicht. DEP automatisiert den Registrierungsprozess, sodass Geräte sofort einsatzbereit sind, mit minimalem Benutzer- oder Verwaltungsaufwand. Wesentliche Aspekte sind:

* Ermöglicht es Geräten, sich autonom bei einem vordefinierten MDM-Server bei der ersten Aktivierung zu registrieren.
* Primär vorteilhaft für brandneue Geräte, aber auch anwendbar für Geräte, die neu konfiguriert werden.
* Erleichtert eine unkomplizierte Einrichtung, sodass Geräte schnell für die organisatorische Nutzung bereit sind.

### **Sicherheitsüberlegung**

Es ist wichtig zu beachten, dass die durch DEP gebotene einfache Registrierung, obwohl vorteilhaft, auch Sicherheitsrisiken darstellen kann. Wenn Schutzmaßnahmen für die MDM-Registrierung nicht ausreichend durchgesetzt werden, könnten Angreifer diesen vereinfachten Prozess ausnutzen, um ihr Gerät auf dem MDM-Server der Organisation zu registrieren und sich als Unternehmensgerät auszugeben.

{% hint style="danger" %}
**Sicherheitswarnung**: Die vereinfachte DEP-Registrierung könnte potenziell die unbefugte Registrierung von Geräten auf dem MDM-Server der Organisation ermöglichen, wenn keine angemessenen Schutzmaßnahmen vorhanden sind.
{% endhint %}

### Grundlagen Was ist SCEP (Simple Certificate Enrollment Protocol)?

* Ein relativ altes Protokoll, das vor der weit verbreiteten Nutzung von TLS und HTTPS erstellt wurde.
* Bietet Clients eine standardisierte Möglichkeit, eine **Zertifikatsanforderung** (CSR) zu senden, um ein Zertifikat zu erhalten. Der Client wird den Server bitten, ihm ein signiertes Zertifikat zu geben.

### Was sind Konfigurationsprofile (auch mobileconfigs)?

* Apples offizielle Methode zur **Festlegung/Durchsetzung von Systemkonfigurationen.**
* Dateiformat, das mehrere Payloads enthalten kann.
* Basierend auf Property-Listen (der XML-Art).
* „kann signiert und verschlüsselt werden, um ihre Herkunft zu validieren, ihre Integrität sicherzustellen und ihren Inhalt zu schützen.“ Grundlagen — Seite 70, iOS Security Guide, Januar 2018.

## Protokolle

### MDM

* Kombination aus APNs (**Apple-Servern**) + RESTful API (**MDM** **Anbieter**-Servern)
* **Kommunikation** erfolgt zwischen einem **Gerät** und einem Server, der mit einem **Geräteverwaltungsprodukt** verbunden ist
* **Befehle** werden vom MDM an das Gerät in **plist-kodierten Dictionaries** übermittelt
* Überall über **HTTPS**. MDM-Server können (und sind normalerweise) gepinnt.
* Apple gewährt dem MDM-Anbieter ein **APNs-Zertifikat** zur Authentifizierung

### DEP

* **3 APIs**: 1 für Wiederverkäufer, 1 für MDM-Anbieter, 1 für Geräteidentität (nicht dokumentiert):
* Die sogenannte [DEP "Cloud-Service"-API](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Diese wird von MDM-Servern verwendet, um DEP-Profile mit bestimmten Geräten zu verknüpfen.
* Die [DEP-API, die von autorisierten Apple-Wiederverkäufern verwendet wird](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html), um Geräte zu registrieren, den Registrierungsstatus zu überprüfen und den Transaktionsstatus zu überprüfen.
* Die nicht dokumentierte private DEP-API. Diese wird von Apple-Geräten verwendet, um ihr DEP-Profil anzufordern. Unter macOS ist das `cloudconfigurationd`-Binary für die Kommunikation über diese API verantwortlich.
* Moderner und **JSON**-basiert (im Vergleich zu plist)
* Apple gewährt dem MDM-Anbieter ein **OAuth-Token**

**DEP "Cloud-Service"-API**

* RESTful
* synchronisiert Geräteaufzeichnungen von Apple zum MDM-Server
* synchronisiert „DEP-Profile“ von MDM-Server zu Apple (später an das Gerät geliefert)
* Ein DEP „Profil“ enthält:
* MDM-Anbieter-Server-URL
* Zusätzliche vertrauenswürdige Zertifikate für die Server-URL (optionales Pinning)
* Zusätzliche Einstellungen (z. B. welche Bildschirme im Setup-Assistenten übersprungen werden sollen)

## Seriennummer

Apple-Geräte, die nach 2010 hergestellt wurden, haben in der Regel **12-stellige alphanumerische** Seriennummern, wobei die **ersten drei Ziffern den Herstellungsort** darstellen, die folgenden **zwei** das **Jahr** und die **Woche** der Herstellung angeben, die nächsten **drei** Ziffern eine **eindeutige** **Kennung** bereitstellen und die **letzten** **vier** Ziffern die **Modellnummer** darstellen.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## Schritte zur Registrierung und Verwaltung

1. Erstellung des Geräteaufzeichnisses (Wiederverkäufer, Apple): Der Datensatz für das neue Gerät wird erstellt
2. Zuweisung des Geräteaufzeichnisses (Kunde): Das Gerät wird einem MDM-Server zugewiesen
3. Synchronisierung des Geräteaufzeichnisses (MDM-Anbieter): MDM synchronisiert die Geräteaufzeichnungen und schiebt die DEP-Profile zu Apple
4. DEP-Check-in (Gerät): Gerät erhält sein DEP-Profil
5. Abruf des Profils (Gerät)
6. Installation des Profils (Gerät) a. inkl. MDM-, SCEP- und Root-CA-Payloads
7. Ausgabe des MDM-Befehls (Gerät)

![](<../../../.gitbook/assets/image (694).png>)

Die Datei `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` exportiert Funktionen, die als **hochgradige "Schritte"** des Registrierungsprozesses betrachtet werden können.

### Schritt 4: DEP-Check-in - Abrufen des Aktivierungsdatensatzes

Dieser Teil des Prozesses tritt auf, wenn ein **Benutzer einen Mac zum ersten Mal bootet** (oder nach einer vollständigen Löschung)

![](<../../../.gitbook/assets/image (1044).png>)

oder beim Ausführen von `sudo profiles show -type enrollment`

* Bestimmen **ob das Gerät DEP-fähig ist**
* Aktivierungsdatensatz ist der interne Name für **DEP „Profil“**
* Beginnt, sobald das Gerät mit dem Internet verbunden ist
* Angetrieben von **`CPFetchActivationRecord`**
* Implementiert durch **`cloudconfigurationd`** über XPC. Der **"Setup-Assistent"** (wenn das Gerät zum ersten Mal gebootet wird) oder der **`profiles`**-Befehl wird **diesen Daemon kontaktieren**, um den Aktivierungsdatensatz abzurufen.
* LaunchDaemon (läuft immer als root)

Es folgen einige Schritte, um den Aktivierungsdatensatz durch **`MCTeslaConfigurationFetcher`** abzurufen. Dieser Prozess verwendet eine Verschlüsselung namens **Absinthe**

1. Abrufen des **Zertifikats**
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. **Initialisiere** den Zustand aus dem Zertifikat (**`NACInit`**)
1. Verwendet verschiedene gerätespezifische Daten (d. h. **Seriennummer über `IOKit`**)
3. Abrufen des **Sitzungsschlüssels**
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. Sitzung einrichten (**`NACKeyEstablishment`**)
5. Die Anfrage stellen
1. POST an [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) und die Daten `{ "action": "RequestProfileConfiguration", "sn": "" }` senden
2. Die JSON-Payload ist mit Absinthe (**`NACSign`**) verschlüsselt
3. Alle Anfragen über HTTPs, integrierte Root-Zertifikate werden verwendet

![](<../../../.gitbook/assets/image (566) (1).png>)

Die Antwort ist ein JSON-Dictionary mit einigen wichtigen Daten wie:

* **url**: URL des MDM-Anbieter-Hosts für das Aktivierungsprofil
* **anchor-certs**: Array von DER-Zertifikaten, die als vertrauenswürdige Anker verwendet werden

### **Schritt 5: Abruf des Profils**

![](<../../../.gitbook/assets/image (444).png>)

* Anfrage wird an **URL gesendet, die im DEP-Profil angegeben ist**.
* **Ankerzertifikate** werden verwendet, um **Vertrauen zu bewerten**, wenn sie bereitgestellt werden.
* Erinnerung: die **anchor\_certs**-Eigenschaft des DEP-Profils
* **Anfrage ist ein einfaches .plist** mit Geräteidentifikation
* Beispiele: **UDID, OS-Version**.
* CMS-signiert, DER-kodiert
* Signiert mit dem **Geräteidentitätszertifikat (von APNS)**
* **Zertifikatskette** umfasst abgelaufene **Apple iPhone Device CA**

![](<../../../.gitbook/assets/image (567) (1) (2) (2) (2) (2) (2) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (2).png>)

### Schritt 6: Installation des Profils

* Nach dem Abruf wird das **Profil im System gespeichert**
* Dieser Schritt beginnt automatisch (wenn im **Setup-Assistenten**)
* Angetrieben von **`CPInstallActivationProfile`**
* Implementiert von mdmclient über XPC
* LaunchDaemon (als root) oder LaunchAgent (als Benutzer), je nach Kontext
* Konfigurationsprofile haben mehrere Payloads zur Installation
* Das Framework hat eine pluginbasierte Architektur zur Installation von Profilen
* Jeder Payload-Typ ist mit einem Plugin verbunden
* Kann XPC (im Framework) oder klassisches Cocoa (in ManagedClient.app) sein
* Beispiel:
* Zertifikat-Payloads verwenden CertificateService.xpc

Typischerweise wird das **Aktivierungsprofil**, das von einem MDM-Anbieter bereitgestellt wird, **die folgenden Payloads enthalten**:

* `com.apple.mdm`: um das Gerät in MDM zu **registrieren**
* `com.apple.security.scep`: um dem Gerät ein **Client-Zertifikat** sicher bereitzustellen.
* `com.apple.security.pem`: um **vertrauenswürdige CA-Zertifikate** im System-Schlüsselbund des Geräts zu **installieren**.
* Die Installation der MDM-Payload entspricht dem **MDM-Check-in in der Dokumentation**
* Payload **enthält wichtige Eigenschaften**:
*
* MDM-Check-In-URL (**`CheckInURL`**)
* MDM-Befehlsabfrage-URL (**`ServerURL`**) + APNs-Thema, um es auszulösen
* Um die MDM-Payload zu installieren, wird eine Anfrage an **`CheckInURL`** gesendet
* Implementiert in **`mdmclient`**
* MDM-Payload kann von anderen Payloads abhängen
* Ermöglicht **Anfragen, die an bestimmte Zertifikate gebunden sind**:
* Eigenschaft: **`CheckInURLPinningCertificateUUIDs`**
* Eigenschaft: **`ServerURLPinningCertificateUUIDs`**
* Wird über PEM-Payload bereitgestellt
* Ermöglicht es dem Gerät, mit einem Identitätszertifikat attribuiert zu werden:
* Eigenschaft: IdentityCertificateUUID
* Wird über SCEP-Payload bereitgestellt

### **Schritt 7: Warten auf MDM-Befehle**

* Nach Abschluss des MDM-Check-ins kann der Anbieter **Push-Benachrichtigungen über APNs ausgeben**
* Bei Empfang wird dies von **`mdmclient`** verarbeitet
* Um nach MDM-Befehlen zu fragen, wird eine Anfrage an ServerURL gesendet
* Nutzt die zuvor installierte MDM-Payload:
* **`ServerURLPinningCertificateUUIDs`** für die Pinning-Anfrage
* **`IdentityCertificateUUID`** für das TLS-Client-Zertifikat

## Angriffe

### Geräte in anderen Organisationen registrieren

Wie bereits erwähnt, um ein Gerät in eine Organisation zu registrieren, **wird nur eine Seriennummer benötigt, die zu dieser Organisation gehört**. Sobald das Gerät registriert ist, werden mehrere Organisationen sensible Daten auf dem neuen Gerät installieren: Zertifikate, Anwendungen, WLAN-Passwörter, VPN-Konfigurationen [und so weiter](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Daher könnte dies ein gefährlicher Einstiegspunkt für Angreifer sein, wenn der Registrierungsprozess nicht korrekt geschützt ist:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
