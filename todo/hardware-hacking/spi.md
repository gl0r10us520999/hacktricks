# SPI

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

## Basic Information

SPI (Serial Peripheral Interface) — це синхронний послідовний комунікаційний протокол, що використовується в вбудованих системах для короткострокової комунікації між ІС (інтегральними схемами). Протокол комунікації SPI використовує архітектуру майстер-раб, яка координується сигналами годинника та вибору чіпа. Архітектура майстер-раб складається з майстра (зазвичай мікропроцесора), який керує зовнішніми периферійними пристроями, такими як EEPROM, датчики, контрольні пристрої тощо, які вважаються рабами.

До майстра можна підключити кілька рабів, але раби не можуть спілкуватися один з одним. Раби керуються двома виводами: годинником і вибором чіпа. Оскільки SPI є синхронним комунікаційним протоколом, виводи вводу та виводу слідують за сигналами годинника. Вибір чіпа використовується майстром для вибору раба та взаємодії з ним. Коли вибір чіпа високий, пристрій-раб не вибрано, тоді як коли він низький, чіп вибрано, і майстер взаємодіє з рабом.

MOSI (Master Out, Slave In) та MISO (Master In, Slave Out) відповідають за передачу та отримання даних. Дані надсилаються до пристрою-раба через вивід MOSI, поки вибір чіпа залишається низьким. Вхідні дані містять інструкції, адреси пам'яті або дані відповідно до специфікації постачальника пристрою-раба. Після дійсного вводу вивід MISO відповідає за передачу даних до майстра. Вихідні дані надсилаються точно на наступному тактовому циклі після завершення вводу. Вивід MISO передає дані, поки дані повністю не передані або поки майстер не встановить вивід вибору чіпа високим (в цьому випадку раб перестане передавати, і майстер більше не слухатиме після цього тактового циклу).

## Dumping Firmware from EEPROMs

Скидання прошивки може бути корисним для аналізу прошивки та виявлення вразливостей у ній. Часто прошивка недоступна в Інтернеті або є нерелевантною через різні фактори, такі як номер моделі, версія тощо. Тому безпосереднє витягування прошивки з фізичного пристрою може бути корисним для конкретизації під час пошуку загроз.

Отримання серійної консолі може бути корисним, але часто трапляється, що файли є тільки для читання. Це обмежує аналіз з різних причин. Наприклад, інструменти, які потрібні для надсилання та отримання пакетів, можуть бути відсутніми в прошивці. Тому витягування бінарних файлів для реверс-інжинірингу є недоцільним. Отже, наявність всієї прошивки на системі та витягування бінарних файлів для аналізу може бути дуже корисним.

Крім того, під час червоного тестування та отримання фізичного доступу до пристроїв скидання прошивки може допомогти в модифікації файлів або інжекції шкідливих файлів, а потім повторного запису їх у пам'ять, що може бути корисним для впровадження бекдору в пристрій. Отже, існує безліч можливостей, які можна відкрити за допомогою скидання прошивки.

### CH341A EEPROM Programmer and Reader

Цей пристрій є недорогим інструментом для скидання прошивок з EEPROM та повторного запису їх з файлами прошивки. Це був популярний вибір для роботи з чіпами BIOS комп'ютера (які є просто EEPROM). Цей пристрій підключається через USB і потребує мінімальних інструментів для початку. Крім того, він зазвичай виконує завдання швидко, тому може бути корисним і для фізичного доступу до пристроїв.

![drawing](../../.gitbook/assets/board\_image\_ch341a.jpg)

Підключіть пам'ять EEPROM до програматора CH341a та підключіть пристрій до комп'ютера. Якщо пристрій не виявляється, спробуйте встановити драйвери на комп'ютер. Також переконайтеся, що EEPROM підключено в правильній орієнтації (зазвичай, розмістіть вивід VCC у зворотній орієнтації до USB-роз'єму), інакше програмне забезпечення не зможе виявити чіп. За потреби зверніться до діаграми:

![drawing](../../.gitbook/assets/connect\_wires\_ch341a.jpg) ![drawing](../../.gitbook/assets/eeprom\_plugged\_ch341a.jpg)

Нарешті, використовуйте програмне забезпечення, таке як flashrom, G-Flash (GUI) тощо, для скидання прошивки. G-Flash — це мінімальний GUI-інструмент, який швидкий і автоматично виявляє EEPROM. Це може бути корисним, якщо прошивку потрібно витягнути швидко, без зайвих налаштувань документації.

![drawing](../../.gitbook/assets/connected\_status\_ch341a.jpg)

Після скидання прошивки аналіз можна провести на бінарних файлах. Інструменти, такі як strings, hexdump, xxd, binwalk тощо, можуть бути використані для витягнення великої кількості інформації про прошивку, а також про всю файлову систему.

Для витягнення вмісту з прошивки можна використовувати binwalk. Binwalk аналізує шістнадцяткові сигнатури та ідентифікує файли в бінарному файлі та здатний їх витягувати.
```
binwalk -e <filename>
```
Файл може бути .bin або .rom в залежності від використовуваних інструментів та конфігурацій.

{% hint style="danger" %}
Зверніть увагу, що витягування прошивки є делікатним процесом і вимагає багато терпіння. Будь-яке неналежне поводження може потенційно пошкодити прошивку або навіть повністю її стерти, зробивши пристрій непридатним для використання. Рекомендується вивчити конкретний пристрій перед спробою витягнути прошивку.
{% endhint %}

### Bus Pirate + flashrom

![](<../../.gitbook/assets/image (910).png>)

Зверніть увагу, що навіть якщо PINOUT Pirate Bus вказує на контакти для **MOSI** та **MISO** для підключення до SPI, деякі SPI можуть вказувати контакти як DI та DO. **MOSI -> DI, MISO -> DO**

![](<../../.gitbook/assets/image (360).png>)

У Windows або Linux ви можете використовувати програму [**`flashrom`**](https://www.flashrom.org/Flashrom) для скидання вмісту флеш-пам'яті, запустивши щось на зразок:
```bash
# In this command we are indicating:
# -VV Verbose
# -c <chip> The chip (if you know it better, if not, don'tindicate it and the program might be able to find it)
# -p <programmer> In this case how to contact th chip via the Bus Pirate
# -r <file> Image to save in the filesystem
flashrom -VV -c "W25Q64.V" -p buspirate_spi:dev=COM3 -r flash_content.img
```
{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
