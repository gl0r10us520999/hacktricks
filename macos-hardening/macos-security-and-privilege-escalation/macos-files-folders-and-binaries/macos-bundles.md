# macOS Bundles

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

## Grundlegende Informationen

Bundles in macOS dienen als Container für eine Vielzahl von Ressourcen, einschließlich Anwendungen, Bibliotheken und anderen notwendigen Dateien, wodurch sie im Finder als einzelne Objekte erscheinen, wie die vertrauten `*.app`-Dateien. Das am häufigsten vorkommende Bundle ist das `.app`-Bundle, obwohl auch andere Typen wie `.framework`, `.systemextension` und `.kext` verbreitet sind.

### Wesentliche Komponenten eines Bundles

Innerhalb eines Bundles, insbesondere im `<application>.app/Contents/`-Verzeichnis, befinden sich eine Vielzahl wichtiger Ressourcen:

* **\_CodeSignature**: Dieses Verzeichnis speichert die Code-Signierungsdetails, die für die Überprüfung der Integrität der Anwendung entscheidend sind. Du kannst die Code-Signierungsinformationen mit Befehlen wie: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Enthält die ausführbare Binärdatei der Anwendung, die bei Benutzerinteraktion ausgeführt wird.
* **Resources**: Ein Repository für die Benutzeroberflächenkomponenten der Anwendung, einschließlich Bilder, Dokumente und Schnittstellenbeschreibungen (nib/xib-Dateien).
* **Info.plist**: Dient als Hauptkonfigurationsdatei der Anwendung, die für das System entscheidend ist, um die Anwendung korrekt zu erkennen und mit ihr zu interagieren.

#### Wichtige Schlüssel in Info.plist

Die `Info.plist`-Datei ist ein Grundpfeiler für die Anwendungs-Konfiguration und enthält Schlüssel wie:

* **CFBundleExecutable**: Gibt den Namen der Hauptausführungsdatei im Verzeichnis `Contents/MacOS` an.
* **CFBundleIdentifier**: Stellt einen globalen Identifikator für die Anwendung bereit, der von macOS umfassend für das Anwendungsmanagement verwendet wird.
* **LSMinimumSystemVersion**: Gibt die minimale Version von macOS an, die erforderlich ist, damit die Anwendung ausgeführt werden kann.

### Erforschen von Bundles

Um den Inhalt eines Bundles, wie `Safari.app`, zu erkunden, kann der folgende Befehl verwendet werden: `bash ls -lR /Applications/Safari.app/Contents`

Diese Erkundung zeigt Verzeichnisse wie `_CodeSignature`, `MacOS`, `Resources` und Dateien wie `Info.plist`, die jeweils einen einzigartigen Zweck erfüllen, von der Sicherung der Anwendung bis zur Definition ihrer Benutzeroberfläche und Betriebsparameter.

#### Zusätzliche Bundle-Verzeichnisse

Über die gängigen Verzeichnisse hinaus können Bundles auch Folgendes enthalten:

* **Frameworks**: Enthält gebündelte Frameworks, die von der Anwendung verwendet werden. Frameworks sind wie dylibs mit zusätzlichen Ressourcen.
* **PlugIns**: Ein Verzeichnis für Plug-ins und Erweiterungen, die die Fähigkeiten der Anwendung erweitern.
* **XPCServices**: Enthält XPC-Dienste, die von der Anwendung für die Kommunikation außerhalb des Prozesses verwendet werden.

Diese Struktur stellt sicher, dass alle notwendigen Komponenten innerhalb des Bundles gekapselt sind, was eine modulare und sichere Anwendungsumgebung ermöglicht.

Für detailliertere Informationen zu `Info.plist`-Schlüsseln und deren Bedeutungen bietet die Apple-Entwicklerdokumentation umfangreiche Ressourcen: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
