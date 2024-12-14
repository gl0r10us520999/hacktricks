# Озброєння Distroless

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Що таке Distroless

Контейнер distroless — це тип контейнера, який **містить лише необхідні залежності для запуску конкретного застосунку**, без будь-якого додаткового програмного забезпечення або інструментів, які не є необхідними. Ці контейнери розроблені, щоб бути **легкими** та **безпечними** наскільки це можливо, і вони прагнуть **мінімізувати поверхню атаки**, видаляючи будь-які непотрібні компоненти.

Контейнери distroless часто використовуються в **виробничих середовищах, де безпека та надійність є найважливішими**.

Деякі **приклади** **контейнерів distroless**:

* Надано **Google**: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* Надано **Chainguard**: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Озброєння Distroless

Мета озброєння контейнера distroless полягає в тому, щоб мати можливість **виконувати довільні двійкові файли та корисні навантаження, навіть з обмеженнями**, які накладає **distroless** (відсутність загальних двійкових файлів у системі), а також захистами, які зазвичай зустрічаються в контейнерах, такими як **тільки для читання** або **без виконання** в `/dev/shm`.

### Через пам'ять

Приблизно в якийсь момент 2023 року...

### Через існуючі двійкові файли

#### openssl

****[**У цьому пості,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) пояснюється, що двійковий файл **`openssl`** часто зустрічається в цих контейнерах, можливо, тому що він **потрібен** програмному забезпеченню, яке буде працювати всередині контейнера.


{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
