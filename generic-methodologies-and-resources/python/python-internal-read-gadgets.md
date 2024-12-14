# Python Internal Read Gadgets

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

## Grundinformationen

Verschiedene Schwachstellen wie [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) oder [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) könnten es Ihnen ermöglichen, **interne Python-Daten zu lesen, aber nicht, Code auszuführen**. Daher muss ein Pentester das Beste aus diesen Leserechten machen, um **sensible Berechtigungen zu erhalten und die Schwachstelle zu eskalieren**.

### Flask - Geheimen Schlüssel lesen

Die Hauptseite einer Flask-Anwendung wird wahrscheinlich das **`app`** globale Objekt haben, wo dieses **Geheimnis konfiguriert ist**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
In diesem Fall ist es möglich, auf dieses Objekt zuzugreifen, indem man einfach ein beliebiges Gadget verwendet, um **globale Objekte** von der [**Seite zum Umgehen von Python-Sandboxen**](bypass-python-sandboxes/) zuzugreifen.

Im Fall, dass **die Schwachstelle in einer anderen Python-Datei** liegt, benötigt man ein Gadget, um Dateien zu durchlaufen, um zur Hauptdatei zu gelangen, um **auf das globale Objekt `app.secret_key`** zuzugreifen, um den Flask-Geheimschlüssel zu ändern und in der Lage zu sein, [**Privilegien zu eskalieren**, indem man diesen Schlüssel kennt](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Eine Payload wie diese [aus diesem Bericht](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Verwenden Sie dieses Payload, um **`app.secret_key`** (der Name in Ihrer App könnte anders sein) zu ändern, um neue und privilegierte Flask-Cookies signieren zu können.

### Werkzeug - machine\_id und node uuid

[**Mit diesen Payloads aus diesem Bericht**](https://vozec.fr/writeups/tweedle-dum-dee/) können Sie auf die **machine\_id** und die **uuid** des Knotens zugreifen, die die **Hauptgeheimnisse** sind, die Sie benötigen, um [**den Werkzeug-Pin zu generieren**](../../network-services-pentesting/pentesting-web/werkzeug.md), den Sie verwenden können, um auf die Python-Konsole in `/console` zuzugreifen, wenn der **Debug-Modus aktiviert ist:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Beachten Sie, dass Sie den **lokalen Pfad des Servers zu `app.py`** erhalten können, indem Sie einen **Fehler** auf der Webseite erzeugen, der Ihnen **den Pfad** gibt.
{% endhint %}

Wenn die Schwachstelle in einer anderen Python-Datei liegt, überprüfen Sie den vorherigen Flask-Trick, um auf die Objekte aus der Haupt-Python-Datei zuzugreifen.

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
