# macOS Prozessmissbrauch

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

## Grundlegende Informationen zu Prozessen

Ein Prozess ist eine Instanz eines laufenden ausführbaren Programms, jedoch führen Prozesse keinen Code aus, das sind Threads. Daher **sind Prozesse nur Container für laufende Threads**, die den Speicher, Deskriptoren, Ports, Berechtigungen... bereitstellen.

Traditionell wurden Prozesse innerhalb anderer Prozesse (außer PID 1) durch den Aufruf von **`fork`** gestartet, was eine exakte Kopie des aktuellen Prozesses erstellt, und dann würde der **Kindprozess** normalerweise **`execve`** aufrufen, um die neue ausführbare Datei zu laden und auszuführen. Dann wurde **`vfork`** eingeführt, um diesen Prozess schneller zu machen, ohne Speicher zu kopieren.\
Dann wurde **`posix_spawn`** eingeführt, das **`vfork`** und **`execve`** in einem Aufruf kombiniert und Flags akzeptiert:

* `POSIX_SPAWN_RESETIDS`: Setze effektive IDs auf reale IDs zurück
* `POSIX_SPAWN_SETPGROUP`: Setze die Prozessgruppen-Zugehörigkeit
* `POSUX_SPAWN_SETSIGDEF`: Setze das Standardverhalten von Signalen
* `POSIX_SPAWN_SETSIGMASK`: Setze die Signalmaske
* `POSIX_SPAWN_SETEXEC`: Exec im selben Prozess (wie `execve` mit mehr Optionen)
* `POSIX_SPAWN_START_SUSPENDED`: Starten im angehaltenen Zustand
* `_POSIX_SPAWN_DISABLE_ASLR`: Starten ohne ASLR
* `_POSIX_SPAWN_NANO_ALLOCATOR:` Verwende den Nano-Allocator von libmalloc
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Erlaube `rwx` auf Datensegmenten
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: Schließe alle Dateibeschreibungen bei exec(2) standardmäßig
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` Randomisiere die hohen Bits des ASLR-Slides

Darüber hinaus erlaubt `posix_spawn`, ein Array von **`posix_spawnattr`** anzugeben, das einige Aspekte des gestarteten Prozesses steuert, und **`posix_spawn_file_actions`**, um den Zustand der Deskriptoren zu ändern.

Wenn ein Prozess stirbt, sendet er den **Rückgabewert an den übergeordneten Prozess** (wenn der übergeordnete Prozess gestorben ist, ist der neue übergeordnete Prozess PID 1) mit dem Signal `SIGCHLD`. Der übergeordnete Prozess muss diesen Wert abrufen, indem er `wait4()` oder `waitid()` aufruft, und bis das geschieht, bleibt das Kind in einem Zombie-Zustand, in dem es weiterhin aufgeführt ist, aber keine Ressourcen verbraucht.

### PIDs

PIDs, Prozessidentifikatoren, identifizieren einen einzigartigen Prozess. In XNU sind die **PIDs** **64 Bit** und steigen monoton an und **wickeln sich niemals** (um Missbrauch zu vermeiden).

### Prozessgruppen, Sitzungen & Koalitionen

**Prozesse** können in **Gruppen** eingefügt werden, um die Handhabung zu erleichtern. Zum Beispiel werden Befehle in einem Shell-Skript in derselben Prozessgruppe sein, sodass es möglich ist, **sie zusammen zu signalisieren**, indem man beispielsweise kill verwendet.\
Es ist auch möglich, **Prozesse in Sitzungen zu gruppieren**. Wenn ein Prozess eine Sitzung startet (`setsid(2)`), werden die Kindprozesse in die Sitzung gesetzt, es sei denn, sie starten ihre eigene Sitzung.

Koalition ist eine weitere Möglichkeit, Prozesse in Darwin zu gruppieren. Ein Prozess, der einer Koalition beitritt, ermöglicht den Zugriff auf Poolressourcen, teilt ein Hauptbuch oder steht Jetsam gegenüber. Koalitionen haben unterschiedliche Rollen: Führer, XPC-Dienst, Erweiterung.

### Berechtigungen & Personae

Jeder Prozess hält **Berechtigungen**, die **seine Privilegien** im System identifizieren. Jeder Prozess hat eine primäre `uid` und eine primäre `gid` (obwohl er mehreren Gruppen angehören kann).\
Es ist auch möglich, die Benutzer- und Gruppen-ID zu ändern, wenn die Binärdatei das `setuid/setgid`-Bit hat.\
Es gibt mehrere Funktionen, um **neue uids/gids festzulegen**.

Der Systemaufruf **`persona`** bietet ein **alternatives** Set von **Berechtigungen**. Die Annahme einer Persona übernimmt ihre uid, gid und Gruppenmitgliedschaften **auf einmal**. Im [**Quellcode**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h) ist es möglich, die Struktur zu finden:
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## Threads Grundinformationen

1. **POSIX-Threads (pthreads):** macOS unterstützt POSIX-Threads (`pthreads`), die Teil einer standardisierten Thread-API für C/C++ sind. Die Implementierung von pthreads in macOS befindet sich in `/usr/lib/system/libsystem_pthread.dylib`, die aus dem öffentlich verfügbaren `libpthread`-Projekt stammt. Diese Bibliothek bietet die notwendigen Funktionen zur Erstellung und Verwaltung von Threads.
2. **Threads erstellen:** Die Funktion `pthread_create()` wird verwendet, um neue Threads zu erstellen. Intern ruft diese Funktion `bsdthread_create()` auf, einen niedrigeren Systemaufruf, der spezifisch für den XNU-Kernel (den Kernel, auf dem macOS basiert) ist. Dieser Systemaufruf nimmt verschiedene Flags entgegen, die aus `pthread_attr` (Attributen) abgeleitet sind und das Verhalten des Threads spezifizieren, einschließlich der Planungsrichtlinien und der Stackgröße.
* **Standard-Stackgröße:** Die Standard-Stackgröße für neue Threads beträgt 512 KB, was für typische Operationen ausreichend ist, aber über Thread-Attribute angepasst werden kann, wenn mehr oder weniger Platz benötigt wird.
3. **Thread-Initialisierung:** Die Funktion `__pthread_init()` ist während der Thread-Einrichtung entscheidend und nutzt das Argument `env[]`, um Umgebungsvariablen zu parsen, die Details über den Standort und die Größe des Stacks enthalten können.

#### Thread-Beendigung in macOS

1. **Threads beenden:** Threads werden typischerweise durch den Aufruf von `pthread_exit()` beendet. Diese Funktion ermöglicht es einem Thread, sauber zu beenden, notwendige Aufräumarbeiten durchzuführen und dem Thread zu erlauben, einen Rückgabewert an alle Joiner zu senden.
2. **Thread-Aufräumung:** Nach dem Aufruf von `pthread_exit()` wird die Funktion `pthread_terminate()` aufgerufen, die die Entfernung aller zugehörigen Thread-Strukturen behandelt. Sie gibt Mach-Thread-Ports frei (Mach ist das Kommunikationssubsystem im XNU-Kernel) und ruft `bsdthread_terminate` auf, einen Systemaufruf, der die kernelseitigen Strukturen, die mit dem Thread verbunden sind, entfernt.

#### Synchronisationsmechanismen

Um den Zugriff auf gemeinsame Ressourcen zu verwalten und Rennbedingungen zu vermeiden, bietet macOS mehrere Synchronisationsprimitive. Diese sind entscheidend in Multi-Threading-Umgebungen, um die Datenintegrität und Systemstabilität zu gewährleisten:

1. **Mutexe:**
* **Regulärer Mutex (Signatur: 0x4D555458):** Standard-Mutex mit einem Speicherbedarf von 60 Bytes (56 Bytes für den Mutex und 4 Bytes für die Signatur).
* **Schneller Mutex (Signatur: 0x4d55545A):** Ähnlich wie ein regulärer Mutex, aber für schnellere Operationen optimiert, ebenfalls 60 Bytes groß.
2. **Bedingungsvariablen:**
* Wird verwendet, um auf das Eintreten bestimmter Bedingungen zu warten, mit einer Größe von 44 Bytes (40 Bytes plus eine 4-Byte-Signatur).
* **Bedingungsvariablenattribute (Signatur: 0x434e4441):** Konfigurationsattribute für Bedingungsvariablen, mit einer Größe von 12 Bytes.
3. **Once-Variable (Signatur: 0x4f4e4345):**
* Stellt sicher, dass ein Stück Initialisierungscode nur einmal ausgeführt wird. Ihre Größe beträgt 12 Bytes.
4. **Lese-Schreib-Sperren:**
* Ermöglicht mehreren Lesern oder einem Schreiber gleichzeitig den Zugriff und erleichtert den effizienten Zugriff auf gemeinsame Daten.
* **Lese-Schreib-Sperre (Signatur: 0x52574c4b):** Größe von 196 Bytes.
* **Lese-Schreib-Sperrenattribute (Signatur: 0x52574c41):** Attribute für Lese-Schreib-Sperren, 20 Bytes groß.

{% hint style="success" %}
Die letzten 4 Bytes dieser Objekte werden verwendet, um Überläufe zu erkennen.
{% endhint %}

### Thread-lokale Variablen (TLV)

**Thread-lokale Variablen (TLV)** im Kontext von Mach-O-Dateien (dem Format für ausführbare Dateien in macOS) werden verwendet, um Variablen zu deklarieren, die spezifisch für **jeden Thread** in einer Multi-Thread-Anwendung sind. Dies stellt sicher, dass jeder Thread seine eigene separate Instanz einer Variablen hat, was eine Möglichkeit bietet, Konflikte zu vermeiden und die Datenintegrität aufrechtzuerhalten, ohne explizite Synchronisationsmechanismen wie Mutexe zu benötigen.

In C und verwandten Sprachen können Sie eine thread-lokale Variable mit dem **`__thread`**-Schlüsselwort deklarieren. So funktioniert es in Ihrem Beispiel:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
Dieses Snippet definiert `tlv_var` als eine thread-lokale Variable. Jeder Thread, der diesen Code ausführt, hat sein eigenes `tlv_var`, und Änderungen, die ein Thread an `tlv_var` vornimmt, wirken sich nicht auf `tlv_var` in einem anderen Thread aus.

Im Mach-O-Binary sind die Daten, die mit thread-lokalen Variablen verbunden sind, in spezifischen Abschnitten organisiert:

* **`__DATA.__thread_vars`**: Dieser Abschnitt enthält die Metadaten über die thread-lokalen Variablen, wie ihre Typen und den Initialisierungsstatus.
* **`__DATA.__thread_bss`**: Dieser Abschnitt wird für thread-lokale Variablen verwendet, die nicht explizit initialisiert sind. Es ist ein Teil des Speichers, der für null-initialisierte Daten reserviert ist.

Mach-O bietet auch eine spezifische API namens **`tlv_atexit`**, um thread-lokale Variablen zu verwalten, wenn ein Thread beendet wird. Diese API ermöglicht es Ihnen, **Destruktoren** zu registrieren – spezielle Funktionen, die thread-lokale Daten bereinigen, wenn ein Thread endet.

### Thread-Prioritäten

Das Verständnis von Thread-Prioritäten beinhaltet, wie das Betriebssystem entscheidet, welche Threads ausgeführt werden und wann. Diese Entscheidung wird durch das Prioritätsniveau beeinflusst, das jedem Thread zugewiesen ist. In macOS und Unix-ähnlichen Systemen wird dies mit Konzepten wie `nice`, `renice` und Quality of Service (QoS) Klassen gehandhabt.

#### Nice und Renice

1. **Nice:**
* Der `nice`-Wert eines Prozesses ist eine Zahl, die seine Priorität beeinflusst. Jeder Prozess hat einen nice-Wert, der von -20 (höchste Priorität) bis 19 (niedrigste Priorität) reicht. Der Standard-nice-Wert, wenn ein Prozess erstellt wird, ist typischerweise 0.
* Ein niedrigerer nice-Wert (näher an -20) macht einen Prozess "egoistischer" und gibt ihm mehr CPU-Zeit im Vergleich zu anderen Prozessen mit höheren nice-Werten.
2. **Renice:**
* `renice` ist ein Befehl, der verwendet wird, um den nice-Wert eines bereits laufenden Prozesses zu ändern. Dies kann verwendet werden, um die Priorität von Prozessen dynamisch anzupassen, indem entweder ihre CPU-Zeit-Zuweisung basierend auf neuen nice-Werten erhöht oder verringert wird.
* Zum Beispiel, wenn ein Prozess vorübergehend mehr CPU-Ressourcen benötigt, könnten Sie seinen nice-Wert mit `renice` senken.

#### Quality of Service (QoS) Klassen

QoS-Klassen sind ein modernerer Ansatz zur Handhabung von Thread-Prioritäten, insbesondere in Systemen wie macOS, die **Grand Central Dispatch (GCD)** unterstützen. QoS-Klassen ermöglichen es Entwicklern, Arbeiten in verschiedene Ebenen basierend auf ihrer Wichtigkeit oder Dringlichkeit zu **kategorisieren**. macOS verwaltet die Thread-Priorisierung automatisch basierend auf diesen QoS-Klassen:

1. **Benutzerinteraktiv:**
* Diese Klasse ist für Aufgaben, die derzeit mit dem Benutzer interagieren oder sofortige Ergebnisse erfordern, um eine gute Benutzererfahrung zu bieten. Diese Aufgaben erhalten die höchste Priorität, um die Benutzeroberfläche reaktionsschnell zu halten (z. B. Animationen oder Ereignisbehandlung).
2. **Benutzerinitiiert:**
* Aufgaben, die der Benutzer initiiert und sofortige Ergebnisse erwartet, wie das Öffnen eines Dokuments oder das Klicken auf eine Schaltfläche, die Berechnungen erfordert. Diese haben eine hohe Priorität, liegen jedoch unter benutzerinteraktiven Aufgaben.
3. **Dienstprogramm:**
* Diese Aufgaben sind langlaufend und zeigen typischerweise einen Fortschrittsindikator an (z. B. Dateien herunterladen, Daten importieren). Sie haben eine niedrigere Priorität als benutzerinitiierte Aufgaben und müssen nicht sofort abgeschlossen werden.
4. **Hintergrund:**
* Diese Klasse ist für Aufgaben, die im Hintergrund arbeiten und für den Benutzer nicht sichtbar sind. Dazu können Aufgaben wie Indizierung, Synchronisierung oder Backups gehören. Sie haben die niedrigste Priorität und einen minimalen Einfluss auf die Systemleistung.

Durch die Verwendung von QoS-Klassen müssen Entwickler die genauen Prioritätszahlen nicht verwalten, sondern sich auf die Art der Aufgabe konzentrieren, und das System optimiert die CPU-Ressourcen entsprechend.

Darüber hinaus gibt es verschiedene **Thread-Planungspolitiken**, die eine Reihe von Planungsparametern spezifizieren, die der Scheduler berücksichtigen wird. Dies kann mit `thread_policy_[set/get]` durchgeführt werden. Dies könnte in Race-Condition-Angriffen nützlich sein.

## MacOS Prozessmissbrauch

MacOS bietet, wie jedes andere Betriebssystem, eine Vielzahl von Methoden und Mechanismen, damit **Prozesse interagieren, kommunizieren und Daten teilen**. Während diese Techniken für das effiziente Funktionieren des Systems unerlässlich sind, können sie auch von Bedrohungsakteuren missbraucht werden, um **böswillige Aktivitäten** durchzuführen.

### Bibliotheksinjektion

Bibliotheksinjektion ist eine Technik, bei der ein Angreifer **einen Prozess zwingt, eine bösartige Bibliothek zu laden**. Nach der Injektion läuft die Bibliothek im Kontext des Zielprozesses und gibt dem Angreifer die gleichen Berechtigungen und den gleichen Zugriff wie der Prozess.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Funktionshooking

Funktionshooking beinhaltet das **Abfangen von Funktionsaufrufen** oder Nachrichten innerhalb eines Softwarecodes. Durch das Hooken von Funktionen kann ein Angreifer das **Verhalten** eines Prozesses ändern, sensible Daten beobachten oder sogar die Ausführungsreihenfolge kontrollieren.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Interprozesskommunikation

Interprozesskommunikation (IPC) bezieht sich auf verschiedene Methoden, durch die separate Prozesse **Daten teilen und austauschen**. Während IPC für viele legitime Anwendungen grundlegend ist, kann es auch missbraucht werden, um die Prozessisolierung zu untergraben, sensible Informationen zu leaken oder unbefugte Aktionen durchzuführen.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Electron-Anwendungsinjektion

Electron-Anwendungen, die mit bestimmten Umgebungsvariablen ausgeführt werden, könnten anfällig für Prozessinjektionen sein:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Chromium-Injektion

Es ist möglich, die Flags `--load-extension` und `--use-fake-ui-for-media-stream` zu verwenden, um einen **Man-in-the-Browser-Angriff** durchzuführen, der es ermöglicht, Tastenanschläge, Datenverkehr, Cookies zu stehlen und Skripte in Seiten einzufügen...:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Dirty NIB

NIB-Dateien **definieren Benutzeroberflächenelemente (UI)** und deren Interaktionen innerhalb einer Anwendung. Sie können jedoch **willkürliche Befehle ausführen**, und **Gatekeeper stoppt nicht**, dass eine bereits ausgeführte Anwendung ausgeführt wird, wenn eine **NIB-Datei geändert wird**. Daher könnten sie verwendet werden, um beliebige Programme willkürliche Befehle ausführen zu lassen:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Java-Anwendungsinjektion

Es ist möglich, bestimmte Java-Funktionen (wie die **`_JAVA_OPTS`**-Umgebungsvariable) zu missbrauchen, um eine Java-Anwendung **willkürlichen Code/Befehle** ausführen zu lassen.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Net-Anwendungsinjektion

Es ist möglich, Code in .Net-Anwendungen zu injizieren, indem man **die .Net-Debugging-Funktionalität missbraucht** (nicht durch macOS-Schutzmaßnahmen wie Runtime-Härtung geschützt).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Perl-Injektion

Überprüfen Sie verschiedene Optionen, um ein Perl-Skript willkürlichen Code ausführen zu lassen in:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Ruby-Injektion

Es ist auch möglich, Ruby-Umgebungsvariablen zu missbrauchen, um willkürliche Skripte willkürlichen Code ausführen zu lassen:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Python-Injektion

Wenn die Umgebungsvariable **`PYTHONINSPECT`** gesetzt ist, wird der Python-Prozess in eine Python-CLI wechseln, sobald er beendet ist. Es ist auch möglich, **`PYTHONSTARTUP`** zu verwenden, um ein Python-Skript anzugeben, das zu Beginn einer interaktiven Sitzung ausgeführt werden soll.\
Beachten Sie jedoch, dass das **`PYTHONSTARTUP`**-Skript nicht ausgeführt wird, wenn **`PYTHONINSPECT`** die interaktive Sitzung erstellt.

Andere Umgebungsvariablen wie **`PYTHONPATH`** und **`PYTHONHOME`** könnten ebenfalls nützlich sein, um einen Python-Befehl willkürlichen Code ausführen zu lassen.

Beachten Sie, dass ausführbare Dateien, die mit **`pyinstaller`** kompiliert wurden, diese Umgebungsvariablen nicht verwenden, auch wenn sie mit einem eingebetteten Python ausgeführt werden.

{% hint style="danger" %}
Insgesamt konnte ich keinen Weg finden, um Python willkürlichen Code durch den Missbrauch von Umgebungsvariablen auszuführen.\
Die meisten Leute installieren jedoch Python mit **Homebrew**, was Python an einem **beschreibbaren Ort** für den Standard-Admin-Benutzer installiert. Sie können es mit etwas wie:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Even **root** wird diesen Code ausführen, wenn Python ausgeführt wird.
{% endhint %}

## Erkennung

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) ist eine Open-Source-Anwendung, die **Prozessinjektions**-Aktionen **erkennen und blockieren** kann:

* Verwendung von **Umgebungsvariablen**: Es wird die Anwesenheit einer der folgenden Umgebungsvariablen überwachen: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** und **`ELECTRON_RUN_AS_NODE`**
* Verwendung von **`task_for_pid`**-Aufrufen: Um zu finden, wann ein Prozess den **Task-Port eines anderen** abrufen möchte, was das Injizieren von Code in den Prozess ermöglicht.
* **Electron-App-Parameter**: Jemand kann die Befehlszeilenargumente **`--inspect`**, **`--inspect-brk`** und **`--remote-debugging-port`** verwenden, um eine Electron-App im Debugging-Modus zu starten und somit Code in sie einzufügen.
* Verwendung von **Symlinks** oder **Hardlinks**: Typischerweise ist der häufigste Missbrauch, einen Link mit unseren Benutzerprivilegien zu **platzieren** und ihn auf einen Ort mit höheren Privilegien zu **verweisen**. Die Erkennung ist für sowohl Hardlinks als auch Symlinks sehr einfach. Wenn der Prozess, der den Link erstellt, ein **anderes Privilegieniveau** als die Zieldatei hat, erstellen wir einen **Alarm**. Leider ist im Fall von Symlinks eine Blockierung nicht möglich, da wir keine Informationen über das Ziel des Links vor der Erstellung haben. Dies ist eine Einschränkung des EndpointSecurity-Frameworks von Apple.

### Aufrufe von anderen Prozessen

In [**diesem Blogbeitrag**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) finden Sie, wie es möglich ist, die Funktion **`task_name_for_pid`** zu verwenden, um Informationen über andere **Prozesse, die Code in einen Prozess injizieren**, zu erhalten und dann Informationen über diesen anderen Prozess zu erhalten.

Beachten Sie, dass Sie, um diese Funktion aufzurufen, **die gleiche UID** wie die des Prozesses haben müssen, der ausgeführt wird, oder **root** sein müssen (und es gibt Informationen über den Prozess zurück, nicht einen Weg, um Code zu injizieren).

## Referenzen

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

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
