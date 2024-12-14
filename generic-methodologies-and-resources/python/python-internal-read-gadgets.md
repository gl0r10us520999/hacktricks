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

## 基本信息

不同的漏洞，例如 [**Python 格式字符串**](bypass-python-sandboxes/#python-format-string) 或 [**类污染**](class-pollution-pythons-prototype-pollution.md)，可能允许你 **读取 Python 内部数据，但不允许你执行代码**。因此，渗透测试人员需要充分利用这些读取权限，以 **获取敏感权限并升级漏洞**。

### Flask - 读取密钥

Flask 应用程序的主页面可能会有 **`app`** 全局对象，其中配置了这个 **密钥**。
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
在这种情况下，只需使用任何小工具即可从[**绕过 Python 沙箱页面**](bypass-python-sandboxes/)访问此对象以**访问全局对象**。

在**漏洞位于不同的 Python 文件**的情况下，您需要一个小工具来遍历文件，以便到达主文件以**访问全局对象 `app.secret_key`**，以更改 Flask 秘密密钥并能够[**利用此密钥提升权限**](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign)。

像这样的有效载荷[来自这篇文章](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

使用此有效载荷来**更改 `app.secret_key`**（您应用中的名称可能不同），以便能够签署新的和更高级的 flask cookies。

### Werkzeug - machine\_id 和 node uuid

[**使用此写作中的有效载荷**](https://vozec.fr/writeups/tweedle-dum-dee/)，您将能够访问**machine\_id**和**uuid**节点，这些是您需要的**主要秘密**，以[**生成 Werkzeug pin**](../../network-services-pentesting/pentesting-web/werkzeug.md)，您可以在**调试模式启用**时使用它访问 `/console` 中的 python 控制台：
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
注意，您可以通过在网页上生成一些**错误**来获取**服务器本地路径到 `app.py`**，这将**给您路径**。
{% endhint %}

如果漏洞在另一个python文件中，请检查之前的Flask技巧以访问主python文件中的对象。

{% hint style="success" %}
学习和实践AWS黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践GCP黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f)或[**电报群组**](https://t.me/peass)或**关注**我们在**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github库提交PR来分享黑客技巧。

</details>
{% endhint %}
