{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}


# Базові Payloads

* **Простий список:** Просто список, що містить запис у кожному рядку.
* **Файл виконання:** Список, який читається під час виконання (не завантажується в пам'ять). Для підтримки великих списків.
* **Зміна регістру:** Застосувати зміни до списку рядків (без змін, до нижнього регістру, до ВЕРХНЬОГО регістру, до Правильного імені - перша літера з великої, решта до нижнього -, до Правильного імені - перша літера з великої, решта залишається такою ж -.
* **Числа:** Генерувати числа від X до Y з кроком Z або випадковим чином.
* **Брутфорсер:** Набір символів, мінімальна та максимальна довжина.

[https://github.com/0xC01DF00D/Collabfiltrator](https://github.com/0xC01DF00D/Collabfiltrator) : Payload для виконання команд та отримання виводу через запити DNS до burpcollab.

{% embed url="https://medium.com/@ArtsSEC/burp-suite-exporter-462531be24e" %}

[https://github.com/h3xstream/http-script-generator](https://github.com/h3xstream/http-script-generator)
