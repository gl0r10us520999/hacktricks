# ZIPs tricks

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

**कमांड-लाइन उपकरण** **ज़िप फ़ाइलों** को प्रबंधित करने के लिए आवश्यक हैं, जो ज़िप फ़ाइलों का निदान, मरम्मत और क्रैक करने में मदद करते हैं। यहाँ कुछ प्रमुख उपयोगिताएँ हैं:

- **`unzip`**: बताता है कि ज़िप फ़ाइल क्यों डिकंप्रेस नहीं हो रही है।
- **`zipdetails -v`**: ज़िप फ़ाइल प्रारूप फ़ील्ड का विस्तृत विश्लेषण प्रदान करता है।
- **`zipinfo`**: बिना निकालें ज़िप फ़ाइल की सामग्री सूचीबद्ध करता है।
- **`zip -F input.zip --out output.zip`** और **`zip -FF input.zip --out output.zip`**: भ्रष्ट ज़िप फ़ाइलों को मरम्मत करने का प्रयास करते हैं।
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: ज़िप पासवर्ड के लिए ब्रूट-फोर्स क्रैकिंग का एक उपकरण, जो लगभग 7 वर्णों तक के पासवर्ड के लिए प्रभावी है।

[ज़िप फ़ाइल प्रारूप विनिर्देशन](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) ज़िप फ़ाइलों की संरचना और मानकों पर व्यापक विवरण प्रदान करता है।

यह ध्यान रखना महत्वपूर्ण है कि पासवर्ड-संरक्षित ज़िप फ़ाइलें **फाइल नामों या फ़ाइल आकारों को एन्क्रिप्ट नहीं करती हैं**, जो एक सुरक्षा दोष है जो RAR या 7z फ़ाइलों के साथ साझा नहीं किया जाता है, जो इस जानकारी को एन्क्रिप्ट करते हैं। इसके अलावा, पुराने ZipCrypto विधि के साथ एन्क्रिप्ट की गई ज़िप फ़ाइलें **प्लेनटेक्स्ट हमले** के प्रति संवेदनशील होती हैं यदि संकुचित फ़ाइल की एक अनएन्क्रिप्टेड प्रति उपलब्ध है। यह हमला ज्ञात सामग्री का उपयोग करके ज़िप के पासवर्ड को क्रैक करता है, जो [HackThis के लेख](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) में विस्तृत है और [इस शैक्षणिक पत्र](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) में और समझाया गया है। हालाँकि, **AES-256** एन्क्रिप्शन के साथ सुरक्षित ज़िप फ़ाइलें इस प्लेनटेक्स्ट हमले के प्रति प्रतिरक्षित हैं, जो संवेदनशील डेटा के लिए सुरक्षित एन्क्रिप्शन विधियों को चुनने के महत्व को दर्शाता है।

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

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
