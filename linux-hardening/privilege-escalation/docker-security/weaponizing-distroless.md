# Weaponizing Distroless

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

## What is Distroless

एक डिस्ट्रोलैस कंटेनर एक प्रकार का कंटेनर है जो **विशिष्ट एप्लिकेशन को चलाने के लिए आवश्यक निर्भरताओं** को **ही** शामिल करता है, बिना किसी अतिरिक्त सॉफ़्टवेयर या उपकरणों के जो आवश्यक नहीं हैं। ये कंटेनर **हल्के** और **सुरक्षित** होने के लिए डिज़ाइन किए गए हैं, और वे **हमले की सतह को कम करने** का लक्ष्य रखते हैं, किसी भी अनावश्यक घटकों को हटा कर।

डिस्ट्रोलैस कंटेनर अक्सर **उत्पादन वातावरण में उपयोग किए जाते हैं जहां सुरक्षा और विश्वसनीयता सर्वोपरि हैं**।

कुछ **उदाहरण** **डिस्ट्रोलैस कंटेनरों** के हैं:

* **गूगल** द्वारा प्रदान किया गया: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* **चेनगार्ड** द्वारा प्रदान किया गया: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Weaponizing Distroless

डिस्ट्रोलैस कंटेनर को हथियार बनाने का लक्ष्य यह है कि **मनमाने बाइनरी और पेलोड को निष्पादित किया जा सके, भले ही डिस्ट्रोलैस** (सिस्टम में सामान्य बाइनरी की कमी) द्वारा लगाए गए प्रतिबंध हों और साथ ही कंटेनरों में आमतौर पर पाए जाने वाले सुरक्षा उपाय जैसे **पढ़ने के लिए केवल** या **निष्पादित न करें** `/dev/shm` में।

### Through memory

2023 के किसी बिंदु पर आ रहा है...

### Via Existing binaries

#### openssl

****[**इस पोस्ट में,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) यह बताया गया है कि बाइनरी **`openssl`** अक्सर इन कंटेनरों में पाई जाती है, संभवतः क्योंकि यह **जरूरी** है उस सॉफ़्टवेयर के लिए जो कंटेनर के अंदर चलने वाला है।


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
