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

## Osnovne informacije

Različite ranjivosti kao što su [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) ili [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) mogu vam omogućiti da **pročitate interne podatke u Pythonu, ali neće vam omogućiti da izvršite kod**. Stoga, pentester će morati da iskoristi ove dozvole za čitanje kako bi **dobio osetljive privilegije i eskalirao ranjivost**.

### Flask - Pročitajte tajni ključ

Glavna stranica Flask aplikacije verovatno će imati **`app`** globalni objekat gde je ovaj **tajni ključ konfigurisan**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
U ovom slučaju je moguće pristupiti ovom objektu koristeći bilo koji gadget za **pristup globalnim objektima** sa [**stranice za zaobilaženje Python sandboxes**](bypass-python-sandboxes/).

U slučaju kada **je ranjivost u drugom python fajlu**, potreban vam je gadget za prelazak između fajlova kako biste došli do glavnog da **pristupite globalnom objektu `app.secret_key`** da promenite Flask tajni ključ i budete u mogućnosti da [**povećate privilegije** znajući ovaj ključ](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Payload poput ovog [iz ovog izveštaja](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Koristite ovaj payload da **promenite `app.secret_key`** (ime u vašoj aplikaciji može biti drugačije) kako biste mogli da potpišete nove i privilegovanije flask kolačiće.

### Werkzeug - machine\_id i node uuid

[**Koristeći ove payload-ove iz ovog izveštaja**](https://vozec.fr/writeups/tweedle-dum-dee/) moći ćete da pristupite **machine\_id** i **uuid** node, koji su **glavne tajne** koje su vam potrebne da [**generišete Werkzeug pin**](../../network-services-pentesting/pentesting-web/werkzeug.md) koji možete koristiti za pristup python konzoli u `/console` ako je **debug mode omogućen:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Napomena da možete dobiti **lokalnu putanju servera do `app.py`** generišući neku **grešku** na veb stranici koja će **dati putanju**.
{% endhint %}

Ako je ranjivost u drugom python fajlu, proverite prethodni Flask trik za pristup objektima iz glavnog python fajla.

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podrška HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
