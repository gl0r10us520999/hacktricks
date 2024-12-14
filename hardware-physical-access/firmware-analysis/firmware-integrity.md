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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Firmware-Integrität

Die **benutzerdefinierte Firmware und/oder kompilierten Binärdateien können hochgeladen werden, um Integritäts- oder Signaturüberprüfungsfehler auszunutzen**. Die folgenden Schritte können für die Kompilierung eines Backdoor-Bind-Shells befolgt werden:

1. Die Firmware kann mit firmware-mod-kit (FMK) extrahiert werden.
2. Die Ziel-Firmware-Architektur und Endianness sollten identifiziert werden.
3. Ein Cross-Compiler kann mit Buildroot oder anderen geeigneten Methoden für die Umgebung erstellt werden.
4. Die Backdoor kann mit dem Cross-Compiler erstellt werden.
5. Die Backdoor kann in das extrahierte Firmware-Verzeichnis /usr/bin kopiert werden.
6. Die entsprechende QEMU-Binärdatei kann in das extrahierte Firmware-Rootfs kopiert werden.
7. Die Backdoor kann mit chroot und QEMU emuliert werden.
8. Die Backdoor kann über netcat zugegriffen werden.
9. Die QEMU-Binärdatei sollte aus dem extrahierten Firmware-Rootfs entfernt werden.
10. Die modifizierte Firmware kann mit FMK neu verpackt werden.
11. Die mit einer Backdoor versehene Firmware kann getestet werden, indem sie mit dem Firmware-Analyse-Toolkit (FAT) emuliert und eine Verbindung zur Ziel-Backdoor-IP und dem Port über netcat hergestellt wird.

Wenn bereits eine Root-Shell durch dynamische Analyse, Bootloader-Manipulation oder Hardware-Sicherheitstests erlangt wurde, können vorkompilierte bösartige Binärdateien wie Implantate oder Reverse-Shells ausgeführt werden. Automatisierte Payload/Implantat-Tools wie das Metasploit-Framework und 'msfvenom' können mit den folgenden Schritten genutzt werden:

1. Die Ziel-Firmware-Architektur und Endianness sollten identifiziert werden.
2. Msfvenom kann verwendet werden, um die Ziel-Payload, die IP des Angreifers, die hörende Portnummer, den Dateityp, die Architektur, die Plattform und die Ausgabedatei anzugeben.
3. Die Payload kann auf das kompromittierte Gerät übertragen werden, und es sollte sichergestellt werden, dass sie Ausführungsberechtigungen hat.
4. Metasploit kann vorbereitet werden, um eingehende Anfragen zu bearbeiten, indem msfconsole gestartet und die Einstellungen gemäß der Payload konfiguriert werden.
5. Die Meterpreter-Reverse-Shell kann auf dem kompromittierten Gerät ausgeführt werden.
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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
