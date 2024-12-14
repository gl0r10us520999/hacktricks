# Diamond Ticket

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

## Diamond Ticket

**Wie ein goldenes Ticket** ist ein Diamantticket ein TGT, das verwendet werden kann, um **auf jeden Dienst als jeder Benutzer zuzugreifen**. Ein goldenes Ticket wird vollständig offline gefälscht, mit dem krbtgt-Hash dieser Domäne verschlüsselt und dann in eine Anmeldesitzung übergeben. Da Domänencontroller TGTs, die sie (oder sie) legitim ausgestellt haben, nicht verfolgen, akzeptieren sie gerne TGTs, die mit ihrem eigenen krbtgt-Hash verschlüsselt sind.

Es gibt zwei gängige Techniken, um die Verwendung von goldenen Tickets zu erkennen:

* Suchen Sie nach TGS-REQs, die kein entsprechendes AS-REQ haben.
* Suchen Sie nach TGTs, die alberne Werte haben, wie die Standardlebensdauer von 10 Jahren von Mimikatz.

Ein **Diamantticket** wird erstellt, indem **die Felder eines legitimen TGTs, das von einem DC ausgestellt wurde, modifiziert werden**. Dies wird erreicht, indem ein **TGT angefordert**, es mit dem krbtgt-Hash der Domäne **entschlüsselt**, die gewünschten Felder des Tickets **modifiziert** und dann **wieder verschlüsselt** wird. Dies **überwindet die beiden oben genannten Mängel** eines goldenen Tickets, weil:

* TGS-REQs werden ein vorhergehendes AS-REQ haben.
* Das TGT wurde von einem DC ausgestellt, was bedeutet, dass es alle korrekten Details aus der Kerberos-Richtlinie der Domäne haben wird. Obwohl diese in einem goldenen Ticket genau gefälscht werden können, ist es komplexer und anfälliger für Fehler.
```bash
# Get user RID
powershell Get-DomainUser -Identity <username> -Properties objectsid

.\Rubeus.exe diamond /tgtdeleg /ticketuser:<username> /ticketuserid:<RID of username> /groups:512

# /tgtdeleg uses the Kerberos GSS-API to obtain a useable TGT for the user without needing to know their password, NTLM/AES hash, or elevation on the host.
# /ticketuser is the username of the principal to impersonate.
# /ticketuserid is the domain RID of that principal.
# /groups are the desired group RIDs (512 being Domain Admins).
# /krbkey is the krbtgt AES256 hash.
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
