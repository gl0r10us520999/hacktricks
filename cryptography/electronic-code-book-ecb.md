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


# ECB

(ECB) इलेक्ट्रॉनिक कोड बुक - सममित एन्क्रिप्शन योजना जो **स्पष्ट पाठ के प्रत्येक ब्लॉक को** **साइफरटेक्स्ट के ब्लॉक से** **बदलती है**। यह **सबसे सरल** एन्क्रिप्शन योजना है। मुख्य विचार यह है कि **स्पष्ट पाठ को N बिट्स के ब्लॉकों में विभाजित करना** (इनपुट डेटा के ब्लॉक के आकार, एन्क्रिप्शन एल्गोरिदम पर निर्भर करता है) और फिर केवल एक कुंजी का उपयोग करके स्पष्ट पाठ के प्रत्येक ब्लॉक को एन्क्रिप्ट (डिक्रिप्ट) करना है।

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

ECB का उपयोग करने के कई सुरक्षा निहितार्थ हैं:

* **एन्क्रिप्टेड संदेश से ब्लॉक हटाए जा सकते हैं**
* **एन्क्रिप्टेड संदेश से ब्लॉक इधर-उधर किए जा सकते हैं**

# कमजोरियों का पता लगाना

कल्पना करें कि आप एक एप्लिकेशन में कई बार लॉगिन करते हैं और आपको **हमेशा वही कुकी मिलती है**। इसका कारण यह है कि एप्लिकेशन की कुकी **`<username>|<password>`** है।\
फिर, आप दो नए उपयोगकर्ताओं को उत्पन्न करते हैं, दोनों के पास **एक ही लंबा पासवर्ड** और **लगभग** वही **उपयोगकर्ता नाम** होता है।\
आप पाते हैं कि **8B के ब्लॉक** जहां **दोनों उपयोगकर्ताओं की जानकारी** समान है, वे **बराबर** हैं। फिर, आप कल्पना करते हैं कि यह **ECB के उपयोग के कारण हो सकता है**।

जैसे कि निम्नलिखित उदाहरण में। देखें कि ये **2 डिकोडेड कुकीज़** कई बार ब्लॉक **`\x23U\xE45K\xCB\x21\xC8`** हैं।
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
यह इसलिए है क्योंकि उन कुकीज़ का **उपयोगकर्ता नाम और पासवर्ड कई बार "a" अक्षर** को शामिल करते हैं (उदाहरण के लिए)। **विभिन्न** **ब्लॉक** वे ब्लॉक हैं जो **कम से कम 1 विभिन्न अक्षर** को शामिल करते हैं (शायद विभाजक "|" या उपयोगकर्ता नाम में कुछ आवश्यक अंतर)।

अब, हमलावर को केवल यह पता लगाना है कि प्रारूप `<username><delimiter><password>` है या `<password><delimiter><username>`। ऐसा करने के लिए, वह बस **कई उपयोगकर्ता नाम उत्पन्न कर सकता है** जिनमें **समान और लंबे उपयोगकर्ता नाम और पासवर्ड** हैं जब तक कि वह प्रारूप और विभाजक की लंबाई नहीं खोज लेता: 

| उपयोगकर्ता नाम की लंबाई: | पासवर्ड की लंबाई: | उपयोगकर्ता नाम+पासवर्ड की लंबाई: | कुकी की लंबाई (डिकोड करने के बाद): |
| ------------------------- | ------------------ | ------------------------------- | --------------------------------- |
| 2                       | 2                  | 4                               | 8                                 |
| 3                       | 3                  | 6                               | 8                                 |
| 3                       | 4                  | 7                               | 8                                 |
| 4                       | 4                  | 8                               | 16                                |
| 7                       | 7                  | 14                              | 16                                |

# भेद्यता का शोषण

## पूरे ब्लॉकों को हटाना

कुकी के प्रारूप को जानकर (`<username>|<password>`), उपयोगकर्ता नाम `admin` की नकल करने के लिए एक नया उपयोगकर्ता बनाएं जिसका नाम `aaaaaaaaadmin` हो और कुकी प्राप्त करें और इसे डिकोड करें:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
हम पहले देख सकते हैं कि पैटर्न `\x23U\xE45K\xCB\x21\xC8` पहले उस उपयोगकर्ता नाम के साथ बनाया गया था जिसमें केवल `a` था।\
फिर, आप 8B का पहला ब्लॉक हटा सकते हैं और आपको उपयोगकर्ता नाम `admin` के लिए एक मान्य कुकी मिलेगी:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Moving blocks

कई डेटाबेस में `WHERE username='admin';` या `WHERE username='admin    ';` के लिए खोज करना समान है। _(अतिरिक्त स्पेस पर ध्यान दें)_

तो, उपयोगकर्ता `admin` का अनुकरण करने का एक और तरीका होगा:

* एक उपयोगकर्ता नाम उत्पन्न करें जो: `len(<username>) + len(<delimiter) % len(block)`। `8B` के ब्लॉक आकार के साथ, आप `username       ` नाम का उपयोगकर्ता नाम उत्पन्न कर सकते हैं, जिसमें डिलिमिटर `|` है, तो भाग `<username><delimiter>` 8Bs के 2 ब्लॉकों का निर्माण करेगा।
* फिर, एक पासवर्ड उत्पन्न करें जो उस उपयोगकर्ता नाम को भरने के लिए एक सटीक संख्या में ब्लॉकों को भरता है जिसे हम अनुकरण करना चाहते हैं और स्पेस, जैसे: `admin   `

इस उपयोगकर्ता का कुकी 3 ब्लॉकों से बना होगा: पहले 2 उपयोगकर्ता नाम + डिलिमिटर के ब्लॉक हैं और तीसरा पासवर्ड (जो उपयोगकर्ता नाम का अनुकरण कर रहा है): `username       |admin   `

**फिर, बस पहले ब्लॉक को अंतिम समय के साथ बदलें और आप उपयोगकर्ता `admin` का अनुकरण कर रहे होंगे: `admin          |username`**

## References

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


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
