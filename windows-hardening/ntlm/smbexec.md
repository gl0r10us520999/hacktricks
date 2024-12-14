# SmbExec/ScExec

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

## Wie es funktioniert

**Smbexec** ist ein Tool, das für die Ausführung von Remote-Befehlen auf Windows-Systemen verwendet wird, ähnlich wie **Psexec**, aber es vermeidet es, schädliche Dateien auf dem Zielsystem abzulegen.

### Wichtige Punkte zu **SMBExec**

- Es funktioniert, indem es einen temporären Dienst (zum Beispiel "BTOBTO") auf der Zielmaschine erstellt, um Befehle über cmd.exe (%COMSPEC%) auszuführen, ohne Binärdateien abzulegen.
- Trotz seines stealthy Ansatzes generiert es Ereignisprotokolle für jeden ausgeführten Befehl und bietet eine Form von nicht-interaktivem "Shell".
- Der Befehl zur Verbindung mit **Smbexec** sieht folgendermaßen aus:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Befehle ohne Binärdateien ausführen

- **Smbexec** ermöglicht die direkte Ausführung von Befehlen über Dienstbinärpfade, wodurch die Notwendigkeit physischer Binärdateien auf dem Ziel entfällt.
- Diese Methode ist nützlich, um einmalige Befehle auf einem Windows-Ziel auszuführen. Zum Beispiel ermöglicht die Kombination mit dem `web_delivery`-Modul von Metasploit die Ausführung eines auf PowerShell ausgerichteten Reverse-Meterpreter-Payloads.
- Durch das Erstellen eines Remote-Dienstes auf dem Rechner des Angreifers mit binPath, der so eingestellt ist, dass der bereitgestellte Befehl über cmd.exe ausgeführt wird, ist es möglich, das Payload erfolgreich auszuführen, einen Callback und die Payload-Ausführung mit dem Metasploit-Listener zu erreichen, selbst wenn Dienstantwortfehler auftreten.

### Beispielbefehle

Das Erstellen und Starten des Dienstes kann mit den folgenden Befehlen durchgeführt werden:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
FOr further details check [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## References
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

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
