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

विभिन्न कमजोरियों जैसे कि [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) या [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) आपको **पायथन आंतरिक डेटा पढ़ने की अनुमति दे सकती हैं लेकिन कोड निष्पादित करने की अनुमति नहीं देंगी**। इसलिए, एक पेंटेस्टर को **संवेदनशील विशेषाधिकार प्राप्त करने और कमजोरियों को बढ़ाने के लिए इन पढ़ने की अनुमतियों का अधिकतम लाभ उठाने की आवश्यकता होगी**।

### Flask - Read secret key

एक Flask एप्लिकेशन का मुख्य पृष्ठ शायद **`app`** वैश्विक वस्तु होगा जहाँ यह **गुप्त कुंजी कॉन्फ़िगर की गई है**।
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
इस मामले में, आप इस ऑब्जेक्ट तक पहुँच सकते हैं केवल किसी भी गैजेट का उपयोग करके **वैश्विक ऑब्जेक्ट्स तक पहुँचने के लिए** [**Python सैंडबॉक्स को बायपास करने के पृष्ठ**](bypass-python-sandboxes/) से।

उस मामले में जहाँ **कमजोरी एक अलग पायथन फ़ाइल में है**, आपको फ़ाइलों को पार करने के लिए एक गैजेट की आवश्यकता है ताकि मुख्य फ़ाइल तक पहुँच सकें और **वैश्विक ऑब्जेक्ट `app.secret_key`** तक पहुँच सकें ताकि Flask गुप्त कुंजी को बदल सकें और इस कुंजी को जानकर [**अधिकार बढ़ा सकें**](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign)।

इस तरह का एक पेलोड [इस लेख से](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

इस पेलोड का उपयोग करें **`app.secret_key`** को **बदलने** के लिए (आपके ऐप में नाम अलग हो सकता है) ताकि आप नए और अधिक विशेषाधिकार प्राप्त फ्लास्क कुकीज़ पर हस्ताक्षर कर सकें।

### Werkzeug - machine\_id और node uuid

[**इस लेख से ये पेलोड का उपयोग करते हुए**](https://vozec.fr/writeups/tweedle-dum-dee/) आप **machine\_id** और **uuid** नोड तक पहुँच सकते हैं, जो कि **मुख्य रहस्य** हैं जिनकी आपको [**Werkzeug पिन उत्पन्न करने**](../../network-services-pentesting/pentesting-web/werkzeug.md) की आवश्यकता है जिसे आप `/console` में पहुँचने के लिए उपयोग कर सकते हैं यदि **डिबग मोड सक्षम** है:
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
ध्यान दें कि आप **`app.py`** के लिए **सर्वर का स्थानीय पथ** कुछ **त्रुटि** उत्पन्न करके प्राप्त कर सकते हैं जो **आपको पथ देगा**।
{% endhint %}

यदि कमजोरियां किसी अन्य पायथन फ़ाइल में हैं, तो मुख्य पायथन फ़ाइल से वस्तुओं तक पहुँचने के लिए पिछले Flask ट्रिक की जाँच करें।

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
