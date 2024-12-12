# Інфрачервоний

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

## Як працює інфрачервоний <a href="#how-the-infrared-port-works" id="how-the-infrared-port-works"></a>

**Інфрачервоне світло невидиме для людей**. Довжина хвилі ІЧ становить **від 0.7 до 1000 мікрон**. Пульт дистанційного керування використовує ІЧ-сигнал для передачі даних і працює в діапазоні довжин хвиль 0.75..1.4 мікрон. Мікроконтролер у пульті змушує інфрачервоний світлодіод блимати з певною частотою, перетворюючи цифровий сигнал в ІЧ-сигнал.

Для прийому ІЧ-сигналів використовується **фотоприймач**. Він **перетворює ІЧ-світло в імпульси напруги**, які вже є **цифровими сигналами**. Зазвичай у **приймачі є фільтр темного світла**, який пропускає **тільки бажану довжину хвилі** і відсікає шум.

### Різноманітність ІЧ-протоколів <a href="#variety-of-ir-protocols" id="variety-of-ir-protocols"></a>

ІЧ-протоколи відрізняються за 3 факторами:

* кодування бітів
* структура даних
* несуча частота — часто в діапазоні 36..38 кГц

#### Способи кодування бітів <a href="#bit-encoding-ways" id="bit-encoding-ways"></a>

**1. Кодування відстані імпульсів**

Біти кодуються шляхом модуляції тривалості простору між імпульсами. Ширина самого імпульсу є сталою.

<figure><img src="../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

**2. Кодування ширини імпульсів**

Біти кодуються шляхом модуляції ширини імпульсу. Ширина простору після сплеску імпульсу є сталою.

<figure><img src="../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

**3. Кодування фази**

Це також відомо як кодування Манчестера. Логічне значення визначається полярністю переходу між сплеском імпульсу та простором. "Простір до сплеску імпульсу" позначає логіку "0", "сплеск імпульсу до простору" позначає логіку "1".

<figure><img src="../../.gitbook/assets/image (634).png" alt=""><figcaption></figcaption></figure>

**4. Комбінація попередніх і інших екзотичних**

{% hint style="info" %}
Існують ІЧ-протоколи, які **намагаються стати універсальними** для кількох типів пристроїв. Найвідомішими є RC5 і NEC. На жаль, найвідоміше **не означає найпоширеніше**. У моєму оточенні я зустрів лише два пульти NEC і жодного RC5.

Виробники люблять використовувати свої унікальні ІЧ-протоколи, навіть у межах одного і того ж діапазону пристроїв (наприклад, ТВ-бокси). Тому пульти від різних компаній і іноді від різних моделей однієї компанії не можуть працювати з іншими пристроями того ж типу.
{% endhint %}

### Дослідження ІЧ-сигналу

Найнадійніший спосіб побачити, як виглядає ІЧ-сигнал пульта, — це використовувати осцилограф. Він не демодулює і не інвертує отриманий сигнал, він просто відображається "як є". Це корисно для тестування та налагодження. Я покажу очікуваний сигнал на прикладі ІЧ-протоколу NEC.

<figure><img src="../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

Зазвичай на початку закодованого пакета є преамбула. Це дозволяє приймачеві визначити рівень підсилення та фону. Існують також протоколи без преамбули, наприклад, Sharp.

Потім передаються дані. Структура, преамбула та метод кодування бітів визначаються конкретним протоколом.

**ІЧ-протокол NEC** містить коротку команду та код повторення, який надсилається під час натискання кнопки. Як команда, так і код повторення мають однакову преамбулу на початку.

Команда NEC, крім преамбули, складається з байта адреси та байта номера команди, за якими пристрій розуміє, що потрібно виконати. Байти адреси та номера команди дублюються з інверсними значеннями, щоб перевірити цілісність передачі. В кінці команди є додатковий стоп-біт.

Код повторення має "1" після преамбули, що є стоп-бітом.

Для **логіки "0" і "1"** NEC використовує кодування відстані імпульсів: спочатку передається сплеск імпульсу, після якого йде пауза, довжина якої задає значення біта.

### Кондиціонери

На відміну від інших пультів, **кондиціонери не передають лише код натиснутої кнопки**. Вони також **передають всю інформацію** під час натискання кнопки, щоб забезпечити, що **кондиціонер і пульт синхронізовані**.\
Це дозволить уникнути ситуації, коли машина, налаштована на 20ºC, підвищується до 21ºC з одного пульта, а потім, коли використовується інший пульт, який все ще має температуру 20ºC, температура підвищується до 21ºC (а не до 22ºC, вважаючи, що вона на 21ºC).

### Атаки

Ви можете атакувати інфрачервоний з Flipper Zero:

{% content-ref url="flipper-zero/fz-infrared.md" %}
[fz-infrared.md](flipper-zero/fz-infrared.md)
{% endcontent-ref %}

## Посилання

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

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
