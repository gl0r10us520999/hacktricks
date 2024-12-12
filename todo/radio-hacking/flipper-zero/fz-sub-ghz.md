# FZ - Sub-GHz

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


## Intro <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero може **приймати та передавати радіочастоти в діапазоні 300-928 МГц** за допомогою вбудованого модуля, який може читати, зберігати та емулювати пульти дистанційного керування. Ці пульти використовуються для взаємодії з воротами, бар'єрами, радіозамками, пультами дистанційного керування, бездротовими дзвінками, розумними лампами та іншим. Flipper Zero може допомогти вам дізнатися, чи ваша безпека скомпрометована.

<figure><img src="../../../.gitbook/assets/image (714).png" alt=""><figcaption></figcaption></figure>

## Sub-GHz hardware <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero має вбудований модуль суб-1 ГГц на основі [﻿](https://www.st.com/en/nfc/st25r3916.html#overview)﻿[чіпа CC1101](https://www.ti.com/lit/ds/symlink/cc1101.pdf) та радіоантену (максимальна дальність - 50 метрів). Як чіп CC1101, так і антена призначені для роботи на частотах у діапазонах 300-348 МГц, 387-464 МГц та 779-928 МГц.

<figure><img src="../../../.gitbook/assets/image (923).png" alt=""><figcaption></figcaption></figure>

## Actions

### Frequency Analyser

{% hint style="info" %}
Як знайти, яка частота використовується пультом
{% endhint %}

Під час аналізу Flipper Zero сканує силу сигналу (RSSI) на всіх частотах, доступних у конфігурації частоти. Flipper Zero відображає частоту з найвищим значенням RSSI, зі силою сигналу вище -90 [dBm](https://en.wikipedia.org/wiki/DBm).

Щоб визначити частоту пульта, виконайте наступні дії:

1. Розмістіть пульт дуже близько до лівої частини Flipper Zero.
2. Перейдіть до **Головне меню** **→ Sub-GHz**.
3. Виберіть **Аналізатор частот**, потім натисніть і утримуйте кнопку на пульті, який ви хочете проаналізувати.
4. Перегляньте значення частоти на екрані.

### Read

{% hint style="info" %}
Знайдіть інформацію про використану частоту (також інший спосіб знайти, яка частота використовується)
{% endhint %}

Опція **Read** **прослуховує налаштовану частоту** на вказаній модуляції: 433.92 AM за замовчуванням. Якщо **щось знайдено** під час читання, **інформація надається** на екрані. Цю інформацію можна використовувати для відтворення сигналу в майбутньому.

Під час використання Read можна натиснути **ліву кнопку** та **налаштувати її**.\
На даний момент вона має **4 модуляції** (AM270, AM650, FM328 та FM476), і **кілька відповідних частот** збережено:

<figure><img src="../../../.gitbook/assets/image (947).png" alt=""><figcaption></figcaption></figure>

Ви можете встановити **будь-яку, яка вас цікавить**, однак, якщо ви **не впевнені, яка частота** може бути використана вашим пультом, **встановіть Hopping на ON** (за замовчуванням Off), і натискайте кнопку кілька разів, поки Flipper не захопить її і не надасть вам інформацію, необхідну для налаштування частоти.

{% hint style="danger" %}
Перемикання між частотами займає деякий час, тому сигнали, що передаються під час перемикання, можуть бути втрачені. Для кращого прийому сигналу встановіть фіксовану частоту, визначену Аналізатором частот.
{% endhint %}

### **Read Raw**

{% hint style="info" %}
Викрасти (і повторити) сигнал на налаштованій частоті
{% endhint %}

Опція **Read Raw** **записує сигнали**, що надсилаються на прослуховуваній частоті. Це можна використовувати для **викрадення** сигналу та **повторення** його.

За замовчуванням **Read Raw також на 433.92 в AM650**, але якщо з опцією Read ви виявили, що сигнал, який вас цікавить, знаходиться на **іншій частоті/модуляції, ви також можете змінити це**, натискаючи ліву кнопку (під час перебування в опції Read Raw).

### Brute-Force

Якщо ви знаєте протокол, який використовується, наприклад, гаражними воротами, можна **згенерувати всі коди та надіслати їх за допомогою Flipper Zero.** Це приклад, який підтримує загальні типи гаражів: [**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)

### Add Manually

{% hint style="info" %}
Додати сигнали з налаштованого списку протоколів
{% endhint %}

#### Список [підтримуваних протоколів](https://docs.flipperzero.one/sub-ghz/add-new-remote) <a href="#id-3iglu" id="id-3iglu"></a>

| Princeton\_433 (працює з більшістю систем статичного коду) | 433.92 | Статичний  |
| --------------------------------------------------------------- | ------ | ------- |
| Nice Flo 12bit\_433                                             | 433.92 | Статичний  |
| Nice Flo 24bit\_433                                             | 433.92 | Статичний  |
| CAME 12bit\_433                                                 | 433.92 | Статичний  |
| CAME 24bit\_433                                                 | 433.92 | Статичний  |
| Linear\_300                                                     | 300.00 | Статичний  |
| CAME TWEE                                                       | 433.92 | Статичний  |
| Gate TX\_433                                                    | 433.92 | Статичний  |
| DoorHan\_315                                                    | 315.00 | Динамічний |
| DoorHan\_433                                                    | 433.92 | Динамічний |
| LiftMaster\_315                                                 | 315.00 | Динамічний |
| LiftMaster\_390                                                 | 390.00 | Динамічний |
| Security+2.0\_310                                               | 310.00 | Динамічний |
| Security+2.0\_315                                               | 315.00 | Динамічний |
| Security+2.0\_390                                               | 390.00 | Динамічний |

### Підтримувані постачальники Sub-GHz

Перевірте список на [https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors)

### Підтримувані частоти за регіоном

Перевірте список на [https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies)

### Test

{% hint style="info" %}
Отримати dBms збережених частот
{% endhint %}

## Reference

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

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
