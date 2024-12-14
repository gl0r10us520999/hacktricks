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

Différentes vulnérabilités telles que [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) ou [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) pourraient vous permettre de **lire les données internes de python mais ne vous permettront pas d'exécuter du code**. Par conséquent, un pentester devra tirer le meilleur parti de ces permissions de lecture pour **obtenir des privilèges sensibles et escalader la vulnérabilité**.

### Flask - Lire la clé secrète

La page principale d'une application Flask aura probablement l'objet global **`app`** où cette **clé secrète est configurée**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
Dans ce cas, il est possible d'accéder à cet objet en utilisant simplement n'importe quel gadget pour **accéder aux objets globaux** de la [**page de contournement des sandboxes Python**](bypass-python-sandboxes/).

Dans le cas où **la vulnérabilité se trouve dans un fichier python différent**, vous avez besoin d'un gadget pour traverser les fichiers afin d'atteindre le principal pour **accéder à l'objet global `app.secret_key`** afin de changer la clé secrète de Flask et pouvoir [**escalader les privilèges** en connaissant cette clé](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Une charge utile comme celle-ci [de ce rapport](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Utilisez ce payload pour **changer `app.secret_key`** (le nom dans votre application peut être différent) afin de pouvoir signer de nouveaux cookies flask avec plus de privilèges.

### Werkzeug - machine\_id et uuid de nœud

[**En utilisant ces payloads de cet article**](https://vozec.fr/writeups/tweedle-dum-dee/) vous pourrez accéder au **machine\_id** et au **uuid** de nœud, qui sont les **principaux secrets** dont vous avez besoin pour [**générer le pin Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md) que vous pouvez utiliser pour accéder à la console python dans `/console` si le **mode debug est activé :**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Notez que vous pouvez obtenir le **chemin local du serveur vers le `app.py`** en générant une **erreur** sur la page web qui vous **donnera le chemin**.
{% endhint %}

Si la vulnérabilité se trouve dans un autre fichier python, vérifiez le truc Flask précédent pour accéder aux objets depuis le fichier python principal.

{% hint style="success" %}
Apprenez et pratiquez le Hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le Hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supportez HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez-nous sur** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PRs aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts github.

</details>
{% endhint %}
