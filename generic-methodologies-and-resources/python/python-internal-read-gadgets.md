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

## Temel Bilgiler

[**Python Format Strings**](bypass-python-sandboxes/#python-format-string) veya [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) gibi farklı güvenlik açıkları, **python iç verilerini okumanıza izin verebilir ancak kod çalıştırmanıza izin vermez**. Bu nedenle, bir pentester bu okuma izinlerini en iyi şekilde kullanarak **hassas ayrıcalıkları elde etmeli ve güvenlik açığını yükseltmelidir**.

### Flask - Gizli anahtarı oku

Bir Flask uygulamasının ana sayfasında muhtemelen bu **gizli anahtarın yapılandırıldığı** **`app`** global nesnesi olacaktır.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
Bu durumda, bu nesneye erişmek için [**Python kumandalarını atlama sayfasından**](bypass-python-sandboxes/) herhangi bir gadget kullanarak **global nesnelere erişmek** mümkündür.

**Açığın farklı bir python dosyasında olduğu** durumda, ana dosyaya ulaşmak için dosyaları geçmek için bir gadget'a ihtiyacınız var, böylece **global nesne `app.secret_key`'e erişebilir** ve Flask gizli anahtarını değiştirebilir ve bu anahtarı bilerek [**yetki yükseltme**] yapabilirsiniz. 

Bu yazıdan [şu yükleme](https://ctftime.org/writeup/36082) gibi bir payload:

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Bu yükü kullanarak **`app.secret_key`**'i (uygulamanızdaki adı farklı olabilir) değiştirin, böylece yeni ve daha fazla yetkiye sahip flask çerezlerini imzalayabilirsiniz.

### Werkzeug - machine\_id ve node uuid

[**Bu yazıdan bu yükleri kullanarak**](https://vozec.fr/writeups/tweedle-dum-dee/) **machine\_id** ve **uuid** node'una erişebileceksiniz, bu da [**Werkzeug pin'ini oluşturmak için**](../../network-services-pentesting/pentesting-web/werkzeug.md) ihtiyaç duyduğunuz **ana sırlar**dır. Eğer **hata ayıklama modu etkinse**, `/console`'da python konsoluna erişmek için kullanabilirsiniz:
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Not edin ki **sunucunun `app.py`'ye olan yerel yolu** web sayfasında bazı **hatalar** oluşturarak **yolu alabilirsiniz**.
{% endhint %}

Eğer zafiyet farklı bir python dosyasındaysa, ana python dosyasından nesnelere erişmek için önceki Flask numarasını kontrol edin.

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **Bize katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya **bizi** **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking numaralarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
