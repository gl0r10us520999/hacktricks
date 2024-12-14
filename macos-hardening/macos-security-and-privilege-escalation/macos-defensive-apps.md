# macOS Defensive Apps

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Firewalls

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): Він буде моніторити кожне з'єднання, яке здійснює кожен процес. Залежно від режиму (тихий дозвіл з'єднань, тихе відхилення з'єднання та сповіщення) він **показуватиме вам сповіщення** щоразу, коли встановлюється нове з'єднання. Він також має дуже зручний GUI для перегляду всієї цієї інформації.
* [**LuLu**](https://objective-see.org/products/lulu.html): Брандмауер Objective-See. Це базовий брандмауер, який сповіщатиме вас про підозрілі з'єднання (він має GUI, але не такий вишуканий, як у Little Snitch).

## Persistence detection

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Додаток Objective-See, який шукатиме в кількох місцях, де **шкідливе ПЗ може зберігатися** (це одноразовий інструмент, а не сервіс моніторингу).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): Як KnockKnock, моніторячи процеси, які генерують збереження.

## Keyloggers detection

* [**ReiKey**](https://objective-see.org/products/reikey.html): Додаток Objective-See для виявлення **кейлогерів**, які встановлюють "event taps" клавіатури.
