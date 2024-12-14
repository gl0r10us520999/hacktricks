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

## Basic Information

Ukatili tofauti kama [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) au [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) zinaweza kukuruhusu **kusoma data za ndani za python lakini hazitakuruhusu kuendesha msimbo**. Hivyo, pentester atahitaji kutumia ruhusa hizi za kusoma ili **kupata mamlaka nyeti na kuongeza ukatili**.

### Flask - Read secret key

Ukurasa mkuu wa programu ya Flask huenda ukawa na **`app`** kitu cha kimataifa ambapo **siri hii imewekwa**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
Katika kesi hii, inawezekana kufikia kitu hiki kwa kutumia gadget yoyote ili **kufikia vitu vya kimataifa** kutoka kwenye [**ukurasa wa Bypass Python sandboxes**](bypass-python-sandboxes/).

Katika kesi ambapo **udhaifu uko kwenye faili tofauti la python**, unahitaji gadget ili kupita kwenye faili ili kufikia faili kuu ili **kufikia kitu cha kimataifa `app.secret_key`** kubadilisha funguo za siri za Flask na kuwa na uwezo wa [**kuinua mamlaka** ukijua funguo hii](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Payload kama hii [kutoka kwenye andiko hili](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Tumia payload hii ili **kubadilisha `app.secret_key`** (jina katika programu yako linaweza kuwa tofauti) ili uweze kusaini vidakuzi vya flask vipya na vya kibali zaidi.

### Werkzeug - machine\_id na node uuid

[**Kwa kutumia payload hizi kutoka kwa andiko hili**](https://vozec.fr/writeups/tweedle-dum-dee/) utaweza kufikia **machine\_id** na **uuid** node, ambazo ni **siri kuu** unazohitaji [**kuunda pin ya Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md) unaweza kutumia kufikia konso ya python katika `/console` ikiwa **mode ya debug imewezeshwa:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Kumbuka kwamba unaweza kupata **njia ya ndani ya seva kwa `app.py`** kwa kuzalisha **makosa** kwenye ukurasa wa wavuti ambayo itakupa **njia**.
{% endhint %}

Ikiwa udhaifu uko katika faili tofauti la python, angalia hila ya Flask ya awali ili kufikia vitu kutoka kwa faili kuu la python.

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **fuata** sisi kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki hila za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
