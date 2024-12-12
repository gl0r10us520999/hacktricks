# Python आंतरिक पठन गैजेट्स

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## मूल जानकारी

विभिन्न सुरक्षा गंभीरताएं जैसे कि [**Python Format Strings**](bypass-python-sandboxes/#python-format-string) या [**Class Pollution**](class-pollution-pythons-prototype-pollution.md) आपको **Python आंतरिक डेटा पठन करने की अनुमति देती हैं लेकिन आपको कोड को नहीं चलाने देगी**। इसलिए, एक पेंटेस्टर को इन पठन अनुमतियों का सबसे अधिक उपयोग करने की आवश्यकता होगी **संवेदनशील विशेषाधिकार प्राप्त करने और सुरक्षा गंभीरता को उन्नत करने** के लिए।

### Flask - गुप्त कुंजी पठन

एक Flask एप्लिकेशन का मुख्य पृष्ठ शायद **`app`** ग्लोबल ऑब्जेक्ट होगा जहाँ यह **गुप्त कूट समाकृत होता है**।
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
इस मामले में, इस ऑब्जेक्ट तक पहुंचना संभव है जिसके लिए केवल किसी गैजेट का उपयोग करना है **ग्लोबल ऑब्जेक्ट्स तक पहुंचने** के लिए [**बाइपास पायथन सैंडबॉक्स पेज**](bypass-python-sandboxes/) से।

उस मामले में जहां **वलनरेबिलिटी एक विभिन्न पायथन फ़ाइल में है**, आपको मुख्य फ़ाइल तक पहुंचने के लिए फ़ाइलों को चलने के लिए एक गैजेट की आवश्यकता है ताकि **ग्लोबल ऑब्जेक्ट `app.secret_key`** तक पहुंच सकें और फ्लास्क सीक्रेट कुंजी को बदल सकें और [**इस कुंजी को जानकर विशेषाधिकारों को बढ़ाने**](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign) में सक्षम हों।

इस लेख से एक ऐसा पेलोड:

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

इस payload का उपयोग करें **`app.secret_key` को बदलने** के लिए (आपके ऐप में नाम अलग हो सकता है) ताकि आप नए और अधिक विशेषाधिकार वाले flask cookies को साइन कर सकें।

### Werkzeug - machine\_id और node uuid

[**इस लेख से इन payload का उपयोग करके**](https://vozec.fr/writeups/tweedle-dum-dee/) आप **machine\_id** और **uuid** नोड तक पहुँच सकेंगे, जो हैं **मुख्य रहस्य** जिनकी आपको [**Werkzeug पिन उत्पन्न करने के लिए**](../../network-services-pentesting/pentesting-web/werkzeug.md) आवश्यकता है जिसका उपयोग आप `/console` में पहुँचने के लिए कर सकते हैं अगर **डीबग मोड सक्षम है:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
ध्यान दें कि आप **`app.py`** के **सर्वर का स्थानीय पथ** प्राप्त कर सकते हैं वेब पेज में कुछ **त्रुटि** उत्पन्न करके जो आपको **पथ देगी**।
{% endhint %}

यदि कमजोरी एक विभिन्न पायथन फ़ाइल में है, तो मुख्य पायथन फ़ाइल से ऑब्जेक्ट तक पहुंचने के लिए पिछले फ्लास्क ट्रिक की जाँच करें।

{% hint style="success" %}
एडब्ल्यूएस हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण एडब्ल्यूएस रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
जीसीपी हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण जीसीपी रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
{% endhint %}
