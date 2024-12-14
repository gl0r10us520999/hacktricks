# Kryptografische/Kompressionsalgorithmen

## Kryptografische/Kompressionsalgorithmen

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

## Identifizierung von Algorithmen

Wenn du in einem Code **Rechts- und Linksverschiebungen, XORs und mehrere arithmetische Operationen** verwendest, ist es sehr wahrscheinlich, dass es sich um die Implementierung eines **kryptografischen Algorithmus** handelt. Hier werden einige Möglichkeiten gezeigt, um **den verwendeten Algorithmus zu identifizieren, ohne jeden Schritt umkehren zu müssen**.

### API-Funktionen

**CryptDeriveKey**

Wenn diese Funktion verwendet wird, kannst du herausfinden, welcher **Algorithmus verwendet wird**, indem du den Wert des zweiten Parameters überprüfst:

![](<../../.gitbook/assets/image (156).png>)

Überprüfe hier die Tabelle möglicher Algorithmen und deren zugewiesene Werte: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

Komprimiert und dekomprimiert einen gegebenen Datenpuffer.

**CryptAcquireContext**

Aus [den Dokumenten](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta): Die **CryptAcquireContext**-Funktion wird verwendet, um einen Handle für einen bestimmten Schlüsselcontainer innerhalb eines bestimmten kryptografischen Dienstanbieters (CSP) zu erwerben. **Dieser zurückgegebene Handle wird in Aufrufen von CryptoAPI**-Funktionen verwendet, die den ausgewählten CSP nutzen.

**CryptCreateHash**

Initiert das Hashing eines Datenstroms. Wenn diese Funktion verwendet wird, kannst du herausfinden, welcher **Algorithmus verwendet wird**, indem du den Wert des zweiten Parameters überprüfst:

![](<../../.gitbook/assets/image (549).png>)

\
Überprüfe hier die Tabelle möglicher Algorithmen und deren zugewiesene Werte: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### Code-Konstanten

Manchmal ist es wirklich einfach, einen Algorithmus zu identifizieren, da er einen speziellen und einzigartigen Wert verwenden muss.

![](<../../.gitbook/assets/image (833).png>)

Wenn du nach der ersten Konstante bei Google suchst, erhältst du Folgendes:

![](<../../.gitbook/assets/image (529).png>)

Daher kannst du annehmen, dass die dekompilierte Funktion ein **SHA256-Rechner** ist.\
Du kannst nach einer der anderen Konstanten suchen und wirst (wahrscheinlich) dasselbe Ergebnis erhalten.

### Dateninfo

Wenn der Code keine signifikante Konstante hat, könnte er **Informationen aus dem .data-Bereich laden**.\
Du kannst auf diese Daten zugreifen, **die erste DWORD gruppieren** und sie in Google suchen, wie wir es im vorherigen Abschnitt getan haben:

![](<../../.gitbook/assets/image (531).png>)

In diesem Fall, wenn du nach **0xA56363C6** suchst, kannst du herausfinden, dass es mit den **Tabellen des AES-Algorithmus** verbunden ist.

## RC4 **(Symmetrische Kryptografie)**

### Eigenschaften

Es besteht aus 3 Hauptteilen:

* **Initialisierungsphase/**: Erstellt eine **Tabelle von Werten von 0x00 bis 0xFF** (insgesamt 256 Bytes, 0x100). Diese Tabelle wird allgemein als **Substitutionsbox** (oder SBox) bezeichnet.
* **Scrambling-Phase**: Wird **durch die zuvor erstellte Tabelle** (Schleife von 0x100 Iterationen, erneut) schleifen und jeden Wert mit **semi-zufälligen** Bytes modifizieren. Um diese semi-zufälligen Bytes zu erstellen, wird der RC4 **Schlüssel verwendet**. RC4 **Schlüssel** können **zwischen 1 und 256 Bytes lang** sein, es wird jedoch normalerweise empfohlen, dass sie länger als 5 Bytes sind. Üblicherweise sind RC4-Schlüssel 16 Bytes lang.
* **XOR-Phase**: Schließlich wird der Klartext oder Chiffretext mit den zuvor erstellten Werten **XORed**. Die Funktion zum Verschlüsseln und Entschlüsseln ist dieselbe. Dazu wird eine **Schleife durch die erstellten 256 Bytes** so oft wie nötig durchgeführt. Dies wird normalerweise in einem dekompilierten Code mit einem **%256 (mod 256)** erkannt.

{% hint style="info" %}
**Um ein RC4 in einem Disassemblierungs-/dekompilierten Code zu identifizieren, kannst du nach 2 Schleifen der Größe 0x100 (unter Verwendung eines Schlüssels) suchen und dann ein XOR der Eingabedaten mit den 256 zuvor in den 2 Schleifen erstellten Werten, wahrscheinlich unter Verwendung eines %256 (mod 256)**
{% endhint %}

### **Initialisierungsphase/Substitutionsbox:** (Beachte die Zahl 256, die als Zähler verwendet wird, und wie eine 0 an jedem Platz der 256 Zeichen geschrieben wird)

![](<../../.gitbook/assets/image (584).png>)

### **Scrambling-Phase:**

![](<../../.gitbook/assets/image (835).png>)

### **XOR-Phase:**

![](<../../.gitbook/assets/image (904).png>)

## **AES (Symmetrische Kryptografie)**

### **Eigenschaften**

* Verwendung von **Substitutionsboxen und Nachschlagetabellen**
* Es ist möglich, **AES anhand der Verwendung spezifischer Nachschlagetabellenwerte** (Konstanten) zu unterscheiden. _Beachte, dass die **Konstante** **im Binärformat** **oder dynamisch** _**erstellt**_ werden kann._
* Der **Verschlüsselungsschlüssel** muss **durch 16** (normalerweise 32B) **teilbar** sein und es wird normalerweise ein **IV** von 16B verwendet.

### SBox-Konstanten

![](<../../.gitbook/assets/image (208).png>)

## Serpent **(Symmetrische Kryptografie)**

### Eigenschaften

* Es ist selten, Malware zu finden, die es verwendet, aber es gibt Beispiele (Ursnif)
* Einfach zu bestimmen, ob ein Algorithmus Serpent ist oder nicht, basierend auf seiner Länge (extrem lange Funktion)

### Identifizierung

In der folgenden Abbildung beachte, wie die Konstante **0x9E3779B9** verwendet wird (beachte, dass diese Konstante auch von anderen Krypto-Algorithmen wie **TEA** -Tiny Encryption Algorithm verwendet wird).\
Beachte auch die **Größe der Schleife** (**132**) und die **Anzahl der XOR-Operationen** in den **Disassemblierungs**-Anweisungen und im **Code**-Beispiel:

![](<../../.gitbook/assets/image (547).png>)

Wie bereits erwähnt, kann dieser Code in jedem Decompiler als **sehr lange Funktion** visualisiert werden, da es **keine Sprünge** darin gibt. Der dekompilierte Code kann wie folgt aussehen:

![](<../../.gitbook/assets/image (513).png>)

Daher ist es möglich, diesen Algorithmus zu identifizieren, indem man die **magische Zahl** und die **initialen XORs** überprüft, eine **sehr lange Funktion** sieht und einige **Anweisungen** der langen Funktion **mit einer Implementierung** (wie der Linksverschiebung um 7 und der Linksrotation um 22) vergleicht.

## RSA **(Asymmetrische Kryptografie)**

### Eigenschaften

* Komplexer als symmetrische Algorithmen
* Es gibt keine Konstanten! (benutzerdefinierte Implementierungen sind schwer zu bestimmen)
* KANAL (ein Krypto-Analyzer) kann keine Hinweise zu RSA geben, da er auf Konstanten angewiesen ist.

### Identifizierung durch Vergleiche

![](<../../.gitbook/assets/image (1113).png>)

* In Zeile 11 (links) gibt es ein `+7) >> 3`, das dasselbe ist wie in Zeile 35 (rechts): `+7) / 8`
* Zeile 12 (links) überprüft, ob `modulus_len < 0x040` und in Zeile 36 (rechts) wird überprüft, ob `inputLen+11 > modulusLen`

## MD5 & SHA (Hash)

### Eigenschaften

* 3 Funktionen: Init, Update, Final
* Ähnliche Initialisierungsfunktionen

### Identifizieren

**Init**

Du kannst beide identifizieren, indem du die Konstanten überprüfst. Beachte, dass die sha\_init eine Konstante hat, die MD5 nicht hat:

![](<../../.gitbook/assets/image (406).png>)

**MD5 Transform**

Beachte die Verwendung von mehr Konstanten

![](<../../.gitbook/assets/image (253) (1) (1).png>)

## CRC (Hash)

* Kleiner und effizienter, da seine Funktion darin besteht, zufällige Änderungen in Daten zu finden
* Verwendet Nachschlagetabellen (so kannst du Konstanten identifizieren)

### Identifizieren

Überprüfe **Nachschlagetabellenkonstanten**:

![](<../../.gitbook/assets/image (508).png>)

Ein CRC-Hash-Algorithmus sieht wie folgt aus:

![](<../../.gitbook/assets/image (391).png>)

## APLib (Kompression)

### Eigenschaften

* Keine erkennbaren Konstanten
* Du kannst versuchen, den Algorithmus in Python zu schreiben und online nach ähnlichen Dingen zu suchen

### Identifizieren

Der Graph ist ziemlich groß:

![](<../../.gitbook/assets/image (207) (2) (1).png>)

Überprüfe **3 Vergleiche, um ihn zu erkennen**:

![](<../../.gitbook/assets/image (430).png>)

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
