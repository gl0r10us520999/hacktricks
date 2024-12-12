# FZ - Інфрачервоний

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Вступ <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Для отримання додаткової інформації про те, як працює інфрачервоний, перевірте:

{% content-ref url="../infrared.md" %}
[infrared.md](../infrared.md)
{% endcontent-ref %}

## ІЧ-приймач сигналу в Flipper Zero <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Flipper використовує цифровий ІЧ-приймач сигналу TSOP, який **дозволяє перехоплювати сигнали від ІЧ-пультів**. Є деякі **смартфони**, такі як Xiaomi, які також мають ІЧ-порт, але майте на увазі, що **більшість з них можуть лише передавати** сигнали і **не можуть їх приймати**.

ІЧ-приймач Flipper **досить чутливий**. Ви навіть можете **піймати сигнал**, залишаючись **десь посередині** між пультом і телевізором. Необхідно вказувати пульт безпосередньо на ІЧ-порт Flipper. Це зручно, коли хтось переключає канали, стоячи біля телевізора, а ви і Flipper знаходитесь на деякій відстані.

Оскільки **декодування інфрачервоного** сигналу відбувається на **програмному** рівні, Flipper Zero потенційно підтримує **прийом і передачу будь-яких кодів ІЧ-пультів**. У випадку **невідомих** протоколів, які не можуть бути розпізнані - він **записує та відтворює** сирий сигнал точно так, як його отримано.

## Дії

### Універсальні пульти

Flipper Zero може використовуватися як **універсальний пульт для керування будь-яким телевізором, кондиціонером або медіацентром**. У цьому режимі Flipper **брутфорсить** всі **відомі коди** всіх підтримуваних виробників **згідно з словником з SD-карти**. Вам не потрібно вибирати конкретний пульт, щоб вимкнути телевізор у ресторані.

Досить натиснути кнопку живлення в режимі універсального пульта, і Flipper **послідовно надсилатиме команди "Вимкнути живлення"** всіх телевізорів, які він знає: Sony, Samsung, Panasonic... і так далі. Коли телевізор отримає свій сигнал, він відреагує і вимкнеться.

Такий брутфорс займає час. Чим більший словник, тим довше це займе. Неможливо дізнатися, який саме сигнал телевізор розпізнав, оскільки немає зворотного зв'язку від телевізора.

### Навчити новий пульт

Можливо **захопити інфрачервоний сигнал** за допомогою Flipper Zero. Якщо він **знайде сигнал у базі даних**, Flipper автоматично **знатиме, який це пристрій** і дозволить вам взаємодіяти з ним.\
Якщо ні, Flipper може **зберегти** **сигнал** і дозволить вам **відтворити його**.

## Посилання

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
