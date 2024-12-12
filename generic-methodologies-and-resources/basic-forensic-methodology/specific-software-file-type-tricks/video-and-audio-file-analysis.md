{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
{% endhint %}

**Маніпулювання аудіо та відео файлами** є основою викликів **CTF forensics**, використовуючи **стеганографію** та аналіз метаданих для приховання або розкриття секретних повідомлень. Інструменти, такі як **[mediainfo](https://mediaarea.net/en/MediaInfo)** та **`exiftool`**, є необхідними для інспекції метаданих файлів та ідентифікації типів контенту.

Для викликів, пов'язаних з аудіо, **[Audacity](http://www.audacityteam.org/)** виділяється як провідний інструмент для перегляду хвиль та аналізу спектрограм, які є важливими для виявлення тексту, закодованого в аудіо. **[Sonic Visualiser](http://www.sonicvisualiser.org/)** високо рекомендується для детального аналізу спектрограм. **Audacity** дозволяє маніпулювати аудіо, такі як уповільнення або реверс треків для виявлення схованих повідомлень. **[Sox](http://sox.sourceforge.net/)**, утиліта командного рядка, відмінно підходить для конвертації та редагування аудіо файлів.

Маніпулювання **найменш значущими бітами (LSB)** є поширеною технікою в аудіо та відео стеганографії, використовуючи фіксовані частини файлів медіа для приховання даних непомітно. **[Multimon-ng](http://tools.kali.org/wireless-attacks/multimon-ng)** корисний для декодування повідомлень, прихованих у вигляді **DTMF сигналів** або **коду Морзе**.

Відео виклики часто включають контейнерні формати, які об'єднують аудіо та відео потоки. **[FFmpeg](http://ffmpeg.org/)** є основним інструментом для аналізу та маніпулювання цими форматами, здатним демультиплексувати та відтворювати вміст. Для розробників, **[ffmpy](http://ffmpy.readthedocs.io/en/latest/examples.html)** інтегрує можливості FFmpeg в Python для продвинутих сценаріїв взаємодії.

Цей набір інструментів підкреслює необхідну універсальність в CTF викликах, де учасники повинні використовувати широкий спектр аналізу та маніпуляційних технік для виявлення схованих даних у аудіо та відео файлах.

## Посилання
* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)
{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
{% endhint %}
