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


## smss.exe

**Session Manager**.\
Session 0 startet **csrss.exe** und **wininit.exe** (**OS** **Dienste**), während Session 1 **csrss.exe** und **winlogon.exe** (**Benutzer** **session**) startet. Du solltest jedoch **nur einen Prozess** dieses **Binaries** ohne Kinder im Prozessbaum sehen.

Außerdem können Sitzungen außer 0 und 1 bedeuten, dass RDP-Sitzungen stattfinden.


## csrss.exe

**Client/Server Run Subsystem Process**.\
Es verwaltet **Prozesse** und **Threads**, stellt die **Windows** **API** für andere Prozesse zur Verfügung und **ordnet Laufwerksbuchstaben zu**, erstellt **Temp-Dateien** und verwaltet den **Herunterfahr** **prozess**.

Es gibt einen **der in Session 0 läuft und einen weiteren in Session 1** (also **2 Prozesse** im Prozessbaum). Ein weiterer wird **pro neuer Sitzung** erstellt.


## winlogon.exe

**Windows Logon Process**.\
Es ist verantwortlich für die Benutzer-**Anmeldung**/**Abmeldung**. Es startet **logonui.exe**, um nach Benutzername und Passwort zu fragen, und ruft dann **lsass.exe** auf, um diese zu überprüfen.

Dann startet es **userinit.exe**, das in **`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`** mit dem Schlüssel **Userinit** angegeben ist.

Darüber hinaus sollte der vorherige Registrierungseintrag **explorer.exe** im **Shell-Schlüssel** haben oder könnte als **Malware-Persistenzmethode** missbraucht werden.


## wininit.exe

**Windows Initialization Process**. \
Es startet **services.exe**, **lsass.exe** und **lsm.exe** in Session 0. Es sollte nur 1 Prozess geben.


## userinit.exe

**Userinit Logon Application**.\
Lädt die **ntduser.dat in HKCU** und initialisiert die **Benutzer** **Umgebung** und führt **Anmeldeskripte** und **GPO** aus.

Es startet **explorer.exe**.


## lsm.exe

**Local Session Manager**.\
Es arbeitet mit smss.exe zusammen, um Benutzersitzungen zu manipulieren: Anmeldung/Abmeldung, Shell-Start, Desktop sperren/entsperren usw.

Nach W7 wurde lsm.exe in einen Dienst (lsm.dll) umgewandelt.

Es sollte nur 1 Prozess in W7 geben und davon einen Dienst, der die DLL ausführt.


## services.exe

**Service Control Manager**.\
Es **lädt** **Dienste**, die als **Auto-Start** konfiguriert sind, und **Treiber**.

Es ist der übergeordnete Prozess von **svchost.exe**, **dllhost.exe**, **taskhost.exe**, **spoolsv.exe** und vielen mehr.

Dienste sind in `HKLM\SYSTEM\CurrentControlSet\Services` definiert, und dieser Prozess hält eine DB im Speicher mit Dienstinformationen, die von sc.exe abgefragt werden können.

Beachte, wie **einige** **Dienste** in einem **eigenen Prozess** laufen und andere einen **svchost.exe-Prozess** **teilen**.

Es sollte nur 1 Prozess geben.


## lsass.exe

**Local Security Authority Subsystem**.\
Es ist verantwortlich für die Benutzer-**Authentifizierung** und erstellt die **Sicherheits** **Tokens**. Es verwendet Authentifizierungspakete, die in `HKLM\System\CurrentControlSet\Control\Lsa` gespeichert sind.

Es schreibt in das **Sicherheits** **ereignis** **protokoll** und es sollte nur 1 Prozess geben.

Beachte, dass dieser Prozess stark angegriffen wird, um Passwörter zu dumpen.


## svchost.exe

**Generic Service Host Process**.\
Es hostet mehrere DLL-Dienste in einem gemeinsamen Prozess.

Normalerweise wirst du feststellen, dass **svchost.exe** mit dem `-k`-Flag gestartet wird. Dies wird eine Abfrage an die Registrierung **HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Svchost** auslösen, wo es einen Schlüssel mit dem Argument gibt, das in -k erwähnt wird, das die zu startenden Dienste im selben Prozess enthält.

Zum Beispiel: `-k UnistackSvcGroup` wird starten: `PimIndexMaintenanceSvc MessagingService WpnUserService CDPUserSvc UnistoreSvc UserDataSvc OneSyncSvc`

Wenn das **Flag `-s`** ebenfalls mit einem Argument verwendet wird, wird svchost aufgefordert, **nur den angegebenen Dienst** in diesem Argument zu starten.

Es wird mehrere Prozesse von `svchost.exe` geben. Wenn einer von ihnen **das `-k`-Flag nicht verwendet**, ist das sehr verdächtig. Wenn du feststellst, dass **services.exe nicht der übergeordnete Prozess** ist, ist das ebenfalls sehr verdächtig.


## taskhost.exe

Dieser Prozess fungiert als Host für Prozesse, die von DLLs ausgeführt werden. Er lädt auch die Dienste, die von DLLs ausgeführt werden.

In W8 wird dies taskhostex.exe genannt und in W10 taskhostw.exe.


## explorer.exe

Dies ist der Prozess, der für den **Desktop des Benutzers** und das Starten von Dateien über Dateierweiterungen verantwortlich ist.

**Es sollte nur 1** Prozess **pro angemeldetem Benutzer** gestartet werden.

Dies wird von **userinit.exe** ausgeführt, die beendet werden sollte, sodass **kein übergeordneter** Prozess für diesen Prozess erscheinen sollte.


# Auffinden von bösartigen Prozessen

* Läuft es vom erwarteten Pfad? (Keine Windows-Binärdateien laufen von temporären Orten)
* Kommuniziert es mit seltsamen IPs?
* Überprüfe digitale Signaturen (Microsoft-Artefakte sollten signiert sein)
* Ist es korrekt geschrieben?
* Läuft es unter dem erwarteten SID?
* Ist der übergeordnete Prozess der erwartete (falls vorhanden)?
* Sind die Kinderprozesse die erwarteten? (keine cmd.exe, wscript.exe, powershell.exe..?)
