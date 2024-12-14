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

## Podstawowe informacje

Różne luki, takie jak [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) lub [**Class Pollution**](class-pollution-pythons-prototype-pollution.md), mogą pozwolić na **odczyt danych wewnętrznych Pythona, ale nie pozwolą na wykonanie kodu**. Dlatego pentester musi maksymalnie wykorzystać te uprawnienia do odczytu, aby **uzyskać wrażliwe uprawnienia i eskalować lukę**.

### Flask - Odczyt klucza tajnego

Główna strona aplikacji Flask prawdopodobnie będzie miała globalny obiekt **`app`**, w którym **ten sekret jest skonfigurowany**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
W tym przypadku możliwe jest uzyskanie dostępu do tego obiektu, używając dowolnego gadżetu do **uzyskiwania dostępu do obiektów globalnych** z [**strony Bypass Python sandboxes**](bypass-python-sandboxes/).

W przypadku, gdy **vulnerability znajduje się w innym pliku python**, potrzebujesz gadżetu do przeszukiwania plików, aby dotrzeć do głównego, aby **uzyskać dostęp do obiektu globalnego `app.secret_key`**, aby zmienić klucz tajny Flask i móc [**eskalować uprawnienia** znając ten klucz](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Payload taki jak ten [z tego opisu](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Użyj tego ładunku, aby **zmienić `app.secret_key`** (nazwa w twojej aplikacji może być inna), aby móc podpisywać nowe i bardziej uprzywilejowane ciasteczka flask.

### Werkzeug - machine\_id i node uuid

[**Używając tych ładunków z tego opisu**](https://vozec.fr/writeups/tweedle-dum-dee/) będziesz mógł uzyskać dostęp do **machine\_id** i **uuid** węzła, które są **głównymi sekretami**, których potrzebujesz, aby [**wygenerować pin Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md), którego możesz użyć do uzyskania dostępu do konsoli python w `/console`, jeśli **tryb debugowania jest włączony:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Zauważ, że możesz uzyskać **lokalną ścieżkę serwera do `app.py`** generując jakiś **błąd** na stronie internetowej, co **da ci ścieżkę**.
{% endhint %}

Jeśli luka znajduje się w innym pliku python, sprawdź poprzedni trik Flask, aby uzyskać dostęp do obiektów z głównego pliku python.

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
