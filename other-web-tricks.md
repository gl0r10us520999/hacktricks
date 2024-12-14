# Інші веб-трюки

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться трюками хакерів, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**Отримайте перспективу хакера на ваші веб-додатки, мережу та хмару**

**Знайдіть та повідомте про критичні, експлуатовані вразливості з реальним бізнес-імпактом.** Використовуйте наші 20+ спеціальних інструментів для картографування атакуючої поверхні, знаходження проблем безпеки, які дозволяють вам підвищити привілеї, та використовуйте автоматизовані експлойти для збору важливих доказів, перетворюючи вашу важку працю на переконливі звіти.

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

### Заголовок хоста

Декілька разів бекенд довіряє **заголовку хоста** для виконання деяких дій. Наприклад, він може використовувати його значення як **домен для надсилання скидання пароля**. Тож, коли ви отримуєте електронний лист з посиланням для скидання пароля, домен, що використовується, є тим, що ви вказали в заголовку хоста. Тоді ви можете запросити скидання пароля інших користувачів і змінити домен на той, що контролюється вами, щоб вкрасти їхні коди скидання пароля. [WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2).

{% hint style="warning" %}
Зверніть увагу, що можливо, вам навіть не потрібно чекати, поки користувач натисне на посилання для скидання пароля, щоб отримати токен, оскільки навіть **спам-фільтри або інші проміжні пристрої/боти можуть натискати на нього для аналізу**.
{% endhint %}

### Булеві сесії

Іноді, коли ви правильно проходите деяку перевірку, бекенд **просто додає булеве значення "True" до атрибута безпеки вашої сесії**. Тоді інша точка доступу знатиме, чи успішно ви пройшли цю перевірку.\
Однак, якщо ви **успішно проходите перевірку** і ваша сесія отримує це значення "True" в атрибуті безпеки, ви можете спробувати **доступитися до інших ресурсів**, які **залежить від того ж атрибута**, але до яких ви **не повинні мати доступу**. [WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a).

### Функціональність реєстрації

Спробуйте зареєструватися як вже існуючий користувач. Спробуйте також використовувати еквівалентні символи (крапки, багато пробілів та Unicode).

### Перехоплення електронних листів

Зареєструйте електронну пошту, перед підтвердженням змініть електронну пошту, тоді, якщо новий підтверджувальний електронний лист буде надіслано на першу зареєстровану електронну пошту, ви зможете перехопити будь-яку електронну пошту. Або, якщо ви можете активувати другу електронну пошту, підтверджуючи першу, ви також можете перехопити будь-який обліковий запис.

### Доступ до внутрішнього сервісного столу компаній, що використовують Atlassian

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### Метод TRACE

Розробники можуть забути вимкнути різні параметри налагодження в продуктивному середовищі. Наприклад, HTTP `TRACE` метод призначений для діагностичних цілей. Якщо він увімкнений, веб-сервер відповість на запити, які використовують метод `TRACE`, відображаючи в відповіді точний запит, який був отриманий. Ця поведінка часто безпечна, але іноді призводить до розкриття інформації, такої як назва внутрішніх заголовків аутентифікації, які можуть бути додані до запитів зворотними проксі.![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**Отримайте перспективу хакера на ваші веб-додатки, мережу та хмару**

**Знайдіть та повідомте про критичні, експлуатовані вразливості з реальним бізнес-імпактом.** Використовуйте наші 20+ спеціальних інструментів для картографування атакуючої поверхні, знаходження проблем безпеки, які дозволяють вам підвищити привілеї, та використовуйте автоматизовані експлойти для збору важливих доказів, перетворюючи вашу важку працю на переконливі звіти.

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться трюками хакерів, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
