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

## Informazioni di base

Diverse vulnerabilità come [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) o [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) potrebbero consentirti di **leggere i dati interni di python ma non ti permetteranno di eseguire codice**. Pertanto, un pentester dovrà sfruttare al massimo questi permessi di lettura per **ottenere privilegi sensibili e aumentare la vulnerabilità**.

### Flask - Leggi la chiave segreta

La pagina principale di un'applicazione Flask avrà probabilmente l'oggetto globale **`app`** dove questa **segreta è configurata**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
In questo caso è possibile accedere a questo oggetto semplicemente utilizzando qualsiasi gadget per **accedere agli oggetti globali** dalla pagina [**Bypass Python sandboxes**](bypass-python-sandboxes/).

Nel caso in cui **la vulnerabilità si trovi in un file python diverso**, hai bisogno di un gadget per attraversare i file per arrivare a quello principale e **accedere all'oggetto globale `app.secret_key`** per cambiare la chiave segreta di Flask e poter [**escalare i privilegi** conoscendo questa chiave](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Un payload come questo [da questo writeup](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Usa questo payload per **cambiare `app.secret_key`** (il nome nella tua app potrebbe essere diverso) per poter firmare nuovi e più privilegiati cookie flask.

### Werkzeug - machine\_id e uuid nodo

[**Utilizzando questi payload da questo writeup**](https://vozec.fr/writeups/tweedle-dum-dee/) sarai in grado di accedere al **machine\_id** e al **uuid** nodo, che sono i **principali segreti** di cui hai bisogno per [**generare il pin di Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md) che puoi usare per accedere alla console python in `/console` se la **modalità debug è abilitata:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Nota che puoi ottenere il **percorso locale del server per `app.py`** generando qualche **errore** nella pagina web che ti **darà il percorso**.
{% endhint %}

Se la vulnerabilità si trova in un file python diverso, controlla il trucco Flask precedente per accedere agli oggetti dal file python principale.

{% hint style="success" %}
Impara e pratica il Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica il Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos su github.

</details>
{% endhint %}
