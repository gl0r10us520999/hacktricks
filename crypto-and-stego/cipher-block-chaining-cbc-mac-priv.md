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


# CBC

Wenn das **Cookie** **nur** der **Benutzername** ist (oder der erste Teil des Cookies der Benutzername ist) und du den Benutzernamen "**admin**" nachahmen möchtest. Dann kannst du den Benutzernamen **"bdmin"** erstellen und das **erste Byte** des Cookies **bruteforcen**.

# CBC-MAC

**Cipher Block Chaining Message Authentication Code** (**CBC-MAC**) ist eine Methode, die in der Kryptographie verwendet wird. Sie funktioniert, indem sie eine Nachricht blockweise verschlüsselt, wobei die Verschlüsselung jedes Blocks mit dem vorherigen verknüpft ist. Dieser Prozess erstellt eine **Kette von Blöcken**, die sicherstellt, dass die Änderung auch nur eines einzelnen Bits der ursprünglichen Nachricht zu einer unvorhersehbaren Änderung im letzten Block der verschlüsselten Daten führt. Um eine solche Änderung vorzunehmen oder rückgängig zu machen, ist der Verschlüsselungsschlüssel erforderlich, was die Sicherheit gewährleistet.

Um den CBC-MAC der Nachricht m zu berechnen, verschlüsselt man m im CBC-Modus mit einem Null-Initialisierungsvektor und behält den letzten Block. Die folgende Abbildung skizziert die Berechnung des CBC-MAC einer Nachricht, die aus Blöcken besteht![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) unter Verwendung eines geheimen Schlüssels k und eines Blockchiffrierverfahrens E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Verwundbarkeit

Mit CBC-MAC ist normalerweise der **IV 0**.\
Das ist ein Problem, weil 2 bekannte Nachrichten (`m1` und `m2`) unabhängig 2 Signaturen (`s1` und `s2`) erzeugen. Also:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Dann wird eine Nachricht, die aus m1 und m2 concatenated (m3) besteht, 2 Signaturen (s31 und s32) erzeugen:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Was möglich ist, ohne den Schlüssel der Verschlüsselung zu kennen.**

Stell dir vor, du verschlüsselst den Namen **Administrator** in **8-Byte**-Blöcken:

* `Administ`
* `rator\00\00\00`

Du kannst einen Benutzernamen namens **Administ** (m1) erstellen und die Signatur (s1) abrufen.\
Dann kannst du einen Benutzernamen erstellen, der das Ergebnis von `rator\00\00\00 XOR s1` ist. Dies wird `E(m2 XOR s1 XOR 0)` erzeugen, was s32 ist.\
Jetzt kannst du s32 als die Signatur des vollständigen Namens **Administrator** verwenden.

### Zusammenfassung

1. Hole die Signatur des Benutzernamens **Administ** (m1), die s1 ist
2. Hole die Signatur des Benutzernamens **rator\x00\x00\x00 XOR s1 XOR 0**, die s32 ist**.**
3. Setze das Cookie auf s32 und es wird ein gültiges Cookie für den Benutzer **Administrator** sein.

# Angriff auf die Kontrolle des IV

Wenn du den verwendeten IV kontrollieren kannst, könnte der Angriff sehr einfach sein.\
Wenn das Cookie nur der verschlüsselte Benutzername ist, kannst du, um den Benutzer "**administrator**" nachzuahmen, den Benutzer "**Administrator**" erstellen und du erhältst sein Cookie.\
Jetzt, wenn du den IV kontrollieren kannst, kannst du das erste Byte des IV ändern, sodass **IV\[0] XOR "A" == IV'\[0] XOR "a"** und das Cookie für den Benutzer **Administrator** regenerieren. Dieses Cookie wird gültig sein, um den Benutzer **administrator** mit dem ursprünglichen **IV** nachzuahmen.

## Referenzen

Weitere Informationen unter [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
