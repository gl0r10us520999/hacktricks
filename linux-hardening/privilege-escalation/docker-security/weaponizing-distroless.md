# Waffentechniken für Distroless

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

## Was ist Distroless

Ein distroless Container ist eine Art von Container, der **nur die notwendigen Abhängigkeiten enthält, um eine bestimmte Anwendung auszuführen**, ohne zusätzliche Software oder Tools, die nicht erforderlich sind. Diese Container sind darauf ausgelegt, so **leichtgewichtig** und **sicher** wie möglich zu sein, und sie zielen darauf ab, die **Angriffsfläche zu minimieren**, indem sie unnötige Komponenten entfernen.

Distroless-Container werden häufig in **Produktionsumgebungen eingesetzt, in denen Sicherheit und Zuverlässigkeit von größter Bedeutung sind**.

Einige **Beispiele** für **distroless Container** sind:

* Bereitgestellt von **Google**: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* Bereitgestellt von **Chainguard**: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Waffentechniken für Distroless

Das Ziel, einen distroless Container zu waffen, besteht darin, **willkürliche Binärdateien und Payloads auszuführen, selbst mit den Einschränkungen**, die durch **distroless** (Fehlen gängiger Binärdateien im System) und auch durch Schutzmaßnahmen, die häufig in Containern zu finden sind, wie **schreibgeschützt** oder **nicht ausführbar** in `/dev/shm`, impliziert werden.

### Durch den Speicher

Kommt irgendwann im Jahr 2023...

### Über vorhandene Binärdateien

#### openssl

****[**In diesem Beitrag,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) wird erklärt, dass die Binärdatei **`openssl`** häufig in diesen Containern zu finden ist, möglicherweise weil sie **benötigt** wird von der Software, die im Container ausgeführt werden soll.


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
