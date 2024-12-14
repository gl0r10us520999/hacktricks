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


# ECB

(ECB) Elektronisches Codebuch - symmetrisches Verschlüsselungsschema, das **jeden Block des Klartexts** durch den **Block des Chiffretexts** **ersetzt**. Es ist das **einfachste** Verschlüsselungsschema. Die Hauptidee ist, den Klartext in **Blöcke von N Bits** (abhängig von der Größe des Eingabedatenblocks, Verschlüsselungsalgorithmus) zu **teilen** und dann jeden Block des Klartexts mit dem einzigen Schlüssel zu **verschlüsseln (entschlüsseln)**.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Die Verwendung von ECB hat mehrere Sicherheitsimplikationen:

* **Blöcke aus der verschlüsselten Nachricht können entfernt werden**
* **Blöcke aus der verschlüsselten Nachricht können verschoben werden**

# Erkennung der Schwachstelle

Stellen Sie sich vor, Sie melden sich mehrmals bei einer Anwendung an und Sie **erhalten immer dasselbe Cookie**. Das liegt daran, dass das Cookie der Anwendung **`<benutzername>|<passwort>`** ist.\
Dann generieren Sie zwei neue Benutzer, beide mit dem **gleichen langen Passwort** und **fast** dem **gleichen** **Benutzernamen**.\
Sie stellen fest, dass die **Blöcke von 8B**, in denen die **Informationen beider Benutzer** gleich sind, **gleich** sind. Dann stellen Sie sich vor, dass dies daran liegen könnte, dass **ECB verwendet wird**.

Wie im folgenden Beispiel. Beachten Sie, wie diese **2 decodierten Cookies** mehrmals den Block **`\x23U\xE45K\xCB\x21\xC8`** haben.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Dies liegt daran, dass der **Benutzername und das Passwort dieser Cookies mehrmals den Buchstaben "a" enthielten** (zum Beispiel). Die **Blöcke**, die **anders** sind, sind Blöcke, die **mindestens 1 anderes Zeichen** enthielten (vielleicht das Trennzeichen "|" oder einen notwendigen Unterschied im Benutzernamen).

Jetzt muss der Angreifer nur herausfinden, ob das Format `<username><delimiter><password>` oder `<password><delimiter><username>` ist. Um dies zu tun, kann er einfach **mehrere Benutzernamen generieren** mit **ähnlichen und langen Benutzernamen und Passwörtern, bis er das Format und die Länge des Trennzeichens findet:**

| Benutzername Länge: | Passwort Länge: | Benutzername+Passwort Länge: | Cookie-Länge (nach Dekodierung): |
| ------------------- | ---------------- | ----------------------------- | --------------------------------- |
| 2                   | 2                | 4                             | 8                                 |
| 3                   | 3                | 6                             | 8                                 |
| 3                   | 4                | 7                             | 8                                 |
| 4                   | 4                | 8                             | 16                                |
| 7                   | 7                | 14                            | 16                                |

# Ausnutzung der Schwachstelle

## Entfernen ganzer Blöcke

Wenn man das Format des Cookies kennt (`<username>|<password>`), um den Benutzernamen `admin` zu impersonifizieren, erstellt man einen neuen Benutzer namens `aaaaaaaaadmin` und erhält das Cookie und dekodiert es:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Wir können das Muster `\x23U\xE45K\xCB\x21\xC8` sehen, das zuvor mit dem Benutzernamen erstellt wurde, der nur `a` enthielt.\
Dann können Sie den ersten Block von 8B entfernen und Sie erhalten ein gültiges Cookie für den Benutzernamen `admin`:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Bewegende Blöcke

In vielen Datenbanken ist es dasselbe, nach `WHERE username='admin';` oder nach `WHERE username='admin    ';` zu suchen _(Beachten Sie die zusätzlichen Leerzeichen)_

Eine weitere Möglichkeit, den Benutzer `admin` zu impersonieren, wäre:

* Generieren Sie einen Benutzernamen, der: `len(<username>) + len(<delimiter) % len(block)`. Mit einer Blockgröße von `8B` können Sie einen Benutzernamen wie `username       ` generieren, mit dem Trennzeichen `|` wird der Chunk `<username><delimiter>` 2 Blöcke von 8Bs erzeugen.
* Dann generieren Sie ein Passwort, das eine genaue Anzahl von Blöcken enthält, die den Benutzernamen, den wir impersonieren möchten, und Leerzeichen füllt, wie: `admin   `

Das Cookie dieses Benutzers wird aus 3 Blöcken bestehen: die ersten 2 sind die Blöcke des Benutzernamens + Trennzeichen und der dritte das Passwort (das den Benutzernamen fälscht): `username       |admin   `

**Ersetzen Sie dann einfach den ersten Block mit dem letzten und Sie impersonieren den Benutzer `admin`: `admin          |username`**

## Referenzen

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


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
