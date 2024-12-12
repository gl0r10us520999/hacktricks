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

<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Поглибте свої знання в **Мобільній безпеці** з 8kSec Academy. Опануйте безпеку iOS та Android через наші курси з самостійним навчанням та отримайте сертифікат:

{% embed url="https://academy.8ksec.io/" %}

**Маніпуляція аудіо та відео файлами** є основою в **CTF forensics challenges**, використовуючи **стеганографію** та аналіз метаданих для приховування або виявлення секретних повідомлень. Інструменти, такі як **[mediainfo](https://mediaarea.net/en/MediaInfo)** та **`exiftool`**, є необхідними для перевірки метаданих файлів та ідентифікації типів контенту.

Для аудіо завдань **[Audacity](http://www.audacityteam.org/)** виділяється як провідний інструмент для перегляду форм хвиль та аналізу спектрограм, що є важливим для виявлення тексту, закодованого в аудіо. **[Sonic Visualiser](http://www.sonicvisualiser.org/)** настійно рекомендується для детального аналізу спектрограм. **Audacity** дозволяє маніпулювати аудіо, наприклад, сповільнювати або реверсувати треки для виявлення прихованих повідомлень. **[Sox](http://sox.sourceforge.net/)**, утиліта командного рядка, відзначається в конвертації та редагуванні аудіофайлів.

**Маніпуляція найменш значущими бітами (LSB)** є поширеною технікою в стеганографії аудіо та відео, що використовує фіксовані розміри частин медіафайлів для прихованого вбудовування даних. **[Multimon-ng](http://tools.kali.org/wireless-attacks/multimon-ng)** корисний для декодування повідомлень, прихованих як **DTMF тони** або **Морзе**.

Відеозавдання часто включають контейнерні формати, які об'єднують аудіо та відеопотоки. **[FFmpeg](http://ffmpeg.org/)** є основним інструментом для аналізу та маніпуляції цими форматами, здатним до демультиплексування та відтворення контенту. Для розробників **[ffmpy](http://ffmpy.readthedocs.io/en/latest/examples.html)** інтегрує можливості FFmpeg у Python для розширених сценарних взаємодій.

Цей набір інструментів підкреслює універсальність, необхідну в CTF завданнях, де учасники повинні використовувати широкий спектр технік аналізу та маніпуляції для виявлення прихованих даних в аудіо та відеофайлах.

## Посилання
* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)


<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Поглибте свої знання в **Мобільній безпеці** з 8kSec Academy. Опануйте безпеку iOS та Android через наші курси з самостійним навчанням та отримайте сертифікат:

{% embed url="https://academy.8ksec.io/" %}

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
