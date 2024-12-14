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

Різні вразливості, такі як [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) або [**Class Pollution**](class-pollution-pythons-prototype-pollution.md), можуть дозволити вам **читати внутрішні дані python, але не дозволять виконувати код**. Тому, пентестер повинен максимально використати ці дозволи на читання, щоб **отримати чутливі привілеї та ескалувати вразливість**.

### Flask - Читання секретного ключа

Головна сторінка додатку Flask, ймовірно, міститиме глобальний об'єкт **`app`**, де цей **секрет налаштовано**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
У цьому випадку можливо отримати доступ до цього об'єкта, просто використовуючи будь-який гаджет для **доступу до глобальних об'єктів** з [**сторінки обходу пісочниць Python**](bypass-python-sandboxes/).

У випадку, коли **вразливість знаходиться в іншому файлі python**, вам потрібен гаджет для переходу між файлами, щоб дістатися до основного, щоб **отримати доступ до глобального об'єкта `app.secret_key`**, щоб змінити секретний ключ Flask і мати можливість [**ескалації привілеїв** знаючи цей ключ](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Пейлоад, подібний до цього [з цього опису](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Використовуйте цей payload, щоб **змінити `app.secret_key`** (ім'я у вашому додатку може бути іншим), щоб мати можливість підписувати нові та більш привілейовані flask cookies.

### Werkzeug - machine\_id та node uuid

[**Використовуючи ці payload з цього опису**](https://vozec.fr/writeups/tweedle-dum-dee/) ви зможете отримати доступ до **machine\_id** та **uuid** node, які є **основними секретами**, необхідними для [**генерації Werkzeug pin**](../../network-services-pentesting/pentesting-web/werkzeug.md), який ви можете використовувати для доступу до python консолі в `/console`, якщо **режим налагодження увімкнено:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Зверніть увагу, що ви можете отримати **локальний шлях сервера до `app.py`**, викликавши деяку **помилку** на веб-сторінці, яка **дасть вам шлях**.
{% endhint %}

Якщо вразливість знаходиться в іншому файлі python, перевірте попередній трюк Flask для доступу до об'єктів з основного файлу python.

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
