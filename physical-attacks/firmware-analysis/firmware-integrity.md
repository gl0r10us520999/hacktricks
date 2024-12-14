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

## Цілісність прошивки

**Користувацька прошивка та/або скомпільовані двійкові файли можуть бути завантажені для використання вразливостей перевірки цілісності або підпису**. Наступні кроки можна виконати для компіляції бекдору з прив'язкою до оболонки:

1. Прошивка може бути витягнута за допомогою firmware-mod-kit (FMK).
2. Слід визначити архітектуру прошивки та порядок байтів.
3. Можна створити крос-компіллятор за допомогою Buildroot або інших відповідних методів для середовища.
4. Бекдор може бути створений за допомогою крос-компіллятора.
5. Бекдор може бути скопійований до витягнутої прошивки в каталог /usr/bin.
6. Відповідний двійковий файл QEMU може бути скопійований до кореневої файлової системи витягнутої прошивки.
7. Бекдор може бути емульований за допомогою chroot та QEMU.
8. До бекдору можна отримати доступ через netcat.
9. Двійковий файл QEMU слід видалити з кореневої файлової системи витягнутої прошивки.
10. Модифікована прошивка може бути повторно упакована за допомогою FMK.
11. Прошивка з бекдором може бути протестована шляхом еймуляції її за допомогою інструментів аналізу прошивки (FAT) та підключення до IP-адреси та порту бекдору за допомогою netcat.

Якщо кореневий оболонка вже була отримана через динамічний аналіз, маніпуляцію завантажувачем або тестування безпеки апаратного забезпечення, можуть бути виконані попередньо скомпільовані шкідливі двійкові файли, такі як імпланти або реверсні оболонки. Автоматизовані інструменти для завантаження/імплантації, такі як фреймворк Metasploit та 'msfvenom', можуть бути використані за допомогою наступних кроків:

1. Слід визначити архітектуру прошивки та порядок байтів.
2. Msfvenom може бути використаний для вказівки цільового корисного навантаження, IP-адреси атакуючого хоста, номера порту для прослуховування, типу файлу, архітектури, платформи та вихідного файлу.
3. Корисне навантаження може бути передане на скомпрометований пристрій і переконатися, що воно має права на виконання.
4. Metasploit може бути підготовлений для обробки вхідних запитів, запустивши msfconsole та налаштувавши параметри відповідно до корисного навантаження.
5. Реверсна оболонка meterpreter може бути виконана на скомпрометованому пристрої.
6. Сесії meterpreter можуть бути моніторинговані під час їх відкриття.
7. Можна виконати дії після експлуатації.

Якщо можливо, вразливості в скриптах запуску можуть бути використані для отримання постійного доступу до пристрою під час перезавантажень. Ці вразливості виникають, коли скрипти запуску посилаються, [символічно посилаються](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data) або залежать від коду, розташованого в ненадійних змонтованих місцях, таких як SD-карти та флеш-обсяги, що використовуються для зберігання даних поза кореневими файловими системами.

## Посилання
* Для отримання додаткової інформації перегляньте [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

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
