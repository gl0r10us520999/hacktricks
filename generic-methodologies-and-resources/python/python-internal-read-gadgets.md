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

## Información Básica

Diferentes vulnerabilidades como [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) o [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) pueden permitirte **leer datos internos de python pero no te permitirán ejecutar código**. Por lo tanto, un pentester necesitará aprovechar al máximo estos permisos de lectura para **obtener privilegios sensibles y escalar la vulnerabilidad**.

### Flask - Leer clave secreta

La página principal de una aplicación Flask probablemente tendrá el **objeto global `app`** donde esta **secreta está configurada**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
En este caso, es posible acceder a este objeto utilizando cualquier gadget para **acceder a objetos globales** de la [**página de Bypass Python sandboxes**](bypass-python-sandboxes/).

En el caso de que **la vulnerabilidad esté en un archivo python diferente**, necesitas un gadget para recorrer archivos y llegar al principal para **acceder al objeto global `app.secret_key`** para cambiar la clave secreta de Flask y poder [**escalar privilegios** conociendo esta clave](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Una carga útil como esta [de este informe](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Utiliza esta carga útil para **cambiar `app.secret_key`** (el nombre en tu aplicación podría ser diferente) para poder firmar nuevas y más privilegiadas cookies de flask.

### Werkzeug - machine\_id y node uuid

[**Usando estas cargas útiles de este informe**](https://vozec.fr/writeups/tweedle-dum-dee/) podrás acceder al **machine\_id** y al **uuid** del nodo, que son los **secretos principales** que necesitas para [**generar el pin de Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md) que puedes usar para acceder a la consola de python en `/console` si el **modo de depuración está habilitado:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Tenga en cuenta que puede obtener la **ruta local del servidor al `app.py`** generando algún **error** en la página web que **le dará la ruta**.
{% endhint %}

Si la vulnerabilidad está en un archivo python diferente, consulte el truco de Flask anterior para acceder a los objetos desde el archivo python principal.

{% hint style="success" %}
Aprenda y practique Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda y practique Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoye a HackTricks</summary>

* Consulte los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únase al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síganos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparta trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}
