# CGroups

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

## Grundinformationen

**Linux Control Groups**, oder **cgroups**, sind ein Feature des Linux-Kernels, das die Zuweisung, Begrenzung und Priorisierung von Systemressourcen wie CPU, Speicher und Festplatten-I/O zwischen Prozessgruppen ermöglicht. Sie bieten einen Mechanismus zur **Verwaltung und Isolierung der Ressourcennutzung** von Prozesssammlungen, was für Zwecke wie Ressourcenbegrenzung, Arbeitslastisolierung und Ressourcenpriorisierung zwischen verschiedenen Prozessgruppen von Vorteil ist.

Es gibt **zwei Versionen von cgroups**: Version 1 und Version 2. Beide können gleichzeitig auf einem System verwendet werden. Der Hauptunterschied besteht darin, dass **cgroups Version 2** eine **hierarchische, baumartige Struktur** einführt, die eine nuanciertere und detailliertere Ressourcenzuteilung zwischen Prozessgruppen ermöglicht. Darüber hinaus bringt Version 2 verschiedene Verbesserungen mit sich, darunter:

Neben der neuen hierarchischen Organisation führte cgroups Version 2 auch **mehrere andere Änderungen und Verbesserungen** ein, wie die Unterstützung für **neue Ressourcencontroller**, bessere Unterstützung für Legacy-Anwendungen und verbesserte Leistung.

Insgesamt bietet cgroups **Version 2 mehr Funktionen und eine bessere Leistung** als Version 1, aber letztere kann in bestimmten Szenarien, in denen die Kompatibilität mit älteren Systemen ein Anliegen ist, weiterhin verwendet werden.

Sie können die v1- und v2-cgroups für jeden Prozess auflisten, indem Sie die cgroup-Datei in /proc/\<pid> ansehen. Sie können damit beginnen, die cgroups Ihrer Shell mit diesem Befehl anzusehen:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
Die Ausgabestruktur ist wie folgt:

* **Zahlen 2–12**: cgroups v1, wobei jede Zeile ein anderes cgroup darstellt. Die Controller dafür sind neben der Zahl angegeben.
* **Zahl 1**: Ebenfalls cgroups v1, jedoch ausschließlich für Verwaltungszwecke (gesetzt von z.B. systemd) und ohne einen Controller.
* **Zahl 0**: Stellt cgroups v2 dar. Es sind keine Controller aufgeführt, und diese Zeile ist ausschließlich auf Systemen, die nur cgroups v2 ausführen, vorhanden.
* Die **Namen sind hierarchisch**, ähnlich wie Dateipfade, und zeigen die Struktur und Beziehung zwischen verschiedenen cgroups an.
* **Namen wie /user.slice oder /system.slice** spezifizieren die Kategorisierung von cgroups, wobei user.slice typischerweise für Anmeldesitzungen, die von systemd verwaltet werden, und system.slice für Systemdienste verwendet wird.

### Anzeigen von cgroups

Das Dateisystem wird typischerweise für den Zugriff auf **cgroups** verwendet, abweichend von der Unix-Systemaufrufschnittstelle, die traditionell für Kernel-Interaktionen genutzt wird. Um die cgroup-Konfiguration einer Shell zu untersuchen, sollte die **/proc/self/cgroup**-Datei überprüft werden, die die cgroup der Shell offenbart. Anschließend kann man im Verzeichnis **/sys/fs/cgroup** (oder **`/sys/fs/cgroup/unified`**) navigieren und ein Verzeichnis finden, das den Namen der cgroup trägt, um verschiedene Einstellungen und Informationen zur Ressourcennutzung der cgroup zu beobachten.

![Cgroup-Dateisystem](<../../../.gitbook/assets/image (1128).png>)

Die wichtigsten Schnittstellendateien für cgroups sind mit **cgroup** vorangestellt. Die **cgroup.procs**-Datei, die mit Standardbefehlen wie cat angezeigt werden kann, listet die Prozesse innerhalb der cgroup auf. Eine andere Datei, **cgroup.threads**, enthält Thread-Informationen.

![Cgroup-Prozesse](<../../../.gitbook/assets/image (281).png>)

Cgroups, die Shells verwalten, umfassen typischerweise zwei Controller, die den Speicherverbrauch und die Prozessanzahl regulieren. Um mit einem Controller zu interagieren, sollten Dateien mit dem Präfix des Controllers konsultiert werden. Zum Beispiel würde **pids.current** herangezogen, um die Anzahl der Threads in der cgroup zu ermitteln.

![Cgroup-Speicher](<../../../.gitbook/assets/image (677).png>)

Die Angabe von **max** in einem Wert deutet auf das Fehlen eines spezifischen Limits für die cgroup hin. Aufgrund der hierarchischen Natur von cgroups können jedoch Limits von einer cgroup auf einer niedrigeren Ebene in der Verzeichnisstruktur auferlegt werden.

### Manipulieren und Erstellen von cgroups

Prozesse werden cgroups zugewiesen, indem **ihre Prozess-ID (PID) in die `cgroup.procs`-Datei geschrieben wird**. Dies erfordert Root-Rechte. Um beispielsweise einen Prozess hinzuzufügen:
```bash
echo [pid] > cgroup.procs
```
Ebenso wird **das Ändern von cgroup-Attributen, wie das Festlegen eines PID-Limits**, durchgeführt, indem der gewünschte Wert in die entsprechende Datei geschrieben wird. Um ein Maximum von 3.000 PIDs für eine cgroup festzulegen:
```bash
echo 3000 > pids.max
```
**Das Erstellen neuer cgroups** beinhaltet das Anlegen eines neuen Unterverzeichnisses innerhalb der cgroup-Hierarchie, was den Kernel dazu veranlasst, automatisch die erforderlichen Schnittstellendateien zu generieren. Obwohl cgroups ohne aktive Prozesse mit `rmdir` entfernt werden können, sollten Sie sich bestimmter Einschränkungen bewusst sein:

* **Prozesse können nur in Blatt-cgroups platziert werden** (d.h. in den am tiefsten geschachtelten in einer Hierarchie).
* **Eine cgroup kann keinen Controller besitzen, der in ihrem Elternteil fehlt**.
* **Controller für untergeordnete cgroups müssen ausdrücklich** in der Datei `cgroup.subtree_control` **deklariert werden**. Zum Beispiel, um CPU- und PID-Controller in einer untergeordneten cgroup zu aktivieren:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
Der **root cgroup** ist eine Ausnahme von diesen Regeln und ermöglicht die direkte Platzierung von Prozessen. Dies kann verwendet werden, um Prozesse aus der Verwaltung von systemd zu entfernen.

**Die Überwachung der CPU-Nutzung** innerhalb eines cgroups ist über die Datei `cpu.stat` möglich, die die insgesamt verbrachte CPU-Zeit anzeigt, was hilfreich ist, um die Nutzung über die Unterprozesse eines Dienstes zu verfolgen:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>CPU-Nutzungsstatistiken, wie sie in der Datei cpu.stat angezeigt werden</p></figcaption></figure>

## Referenzen

* **Buch: Wie Linux funktioniert, 3. Auflage: Was jeder Superuser wissen sollte von Brian Ward**

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
