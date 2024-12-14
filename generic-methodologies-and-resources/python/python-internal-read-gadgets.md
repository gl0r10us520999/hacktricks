# Python Internal Read Gadgets

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Basic Information

Verskillende kwesbaarhede soos [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) of [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) mag jou toelaat om **python interne data te lees maar sal jou nie toelaat om kode uit te voer nie**. Daarom sal 'n pentester die meeste van hierdie lees toestemmings moet maak om **sensitiewe voorregte te verkry en die kwesbaarheid te eskaleer**.

### Flask - Lees geheime sleutel

Die hoofblad van 'n Flask-toepassing sal waarskynlik die **`app`** globale objek hê waar hierdie **geheime sleutel geconfigureer is**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
In hierdie geval is dit moontlik om toegang tot hierdie objek te verkry net deur enige gadget te gebruik om **globale objek te benader** vanaf die [**Bypass Python sandboxes bladsy**](bypass-python-sandboxes/).

In die geval waar **die kwesbaarheid in 'n ander python-lêer is**, benodig jy 'n gadget om lêers te traverseer om by die hoof een te kom om **toegang tot die globale objek `app.secret_key`** te verkry om die Flask geheime sleutel te verander en in staat te wees om [**privileges te verhoog** deur hierdie sleutel te ken](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

'n Payload soos hierdie [uit hierdie skrywe](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Gebruik hierdie payload om **`app.secret_key`** te **verander** (die naam in jou app mag dalk anders wees) om nuwe en meer bevoegdhede flask koekies te kan teken.

### Werkzeug - machine\_id en node uuid

[**Deur hierdie payloads uit hierdie skrywe te gebruik**](https://vozec.fr/writeups/tweedle-dum-dee/) sal jy toegang hê tot die **machine\_id** en die **uuid** node, wat die **hoofgeheime** is wat jy nodig het om [**die Werkzeug pin te genereer**](../../network-services-pentesting/pentesting-web/werkzeug.md) wat jy kan gebruik om toegang te verkry tot die python-konsol in `/console` as die **debug-modus geaktiveer is:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Let daarop dat jy die **bediener se plaaslike pad na die `app.py`** kan kry deur 'n **fout** op die webblad te genereer wat jou **die pad** sal **gee**.
{% endhint %}

As die kwesbaarheid in 'n ander python-lêer is, kyk na die vorige Flask-truk om toegang tot die objekte van die hoof python-lêer te verkry.

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
