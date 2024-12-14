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


# CBC

Якщо **cookie** є **тільки** **іменем користувача** (або перша частина cookie є іменем користувача) і ви хочете видати себе за користувача з ім'ям "**admin**". Тоді ви можете створити ім'я користувача **"bdmin"** і **bruteforce** **перший байт** cookie.

# CBC-MAC

**Код автентифікації повідомлень з ланцюгом блоку шифру** (**CBC-MAC**) є методом, що використовується в криптографії. Він працює, беручи повідомлення і шифруючи його блок за блоком, де шифрування кожного блоку пов'язане з попереднім. Цей процес створює **ланцюг блоків**, що забезпечує, що зміна навіть одного біта оригінального повідомлення призведе до непередбачуваної зміни в останньому блоці зашифрованих даних. Щоб внести або скасувати таку зміну, потрібен ключ шифрування, що забезпечує безпеку.

Щоб обчислити CBC-MAC повідомлення m, шифрують m в режимі CBC з нульовим вектором ініціалізації і зберігають останній блок. Наступна фігура ілюструє обчислення CBC-MAC повідомлення, що складається з блоків![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) з використанням секретного ключа k і блочного шифру E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Уразливість

Зазвичай з CBC-MAC **IV, що використовується, дорівнює 0**.\
Це є проблемою, оскільки 2 відомі повідомлення (`m1` і `m2`) незалежно генеруватимуть 2 підписи (`s1` і `s2`). Отже:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Тоді повідомлення, що складається з m1 і m2, конкатенованих (m3), генеруватиме 2 підписи (s31 і s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Що можливо обчислити без знання ключа шифрування.**

Уявіть, що ви шифруєте ім'я **Administrator** в **8байтових** блоках:

* `Administ`
* `rator\00\00\00`

Ви можете створити ім'я користувача **Administ** (m1) і отримати підпис (s1).\
Потім ви можете створити ім'я користувача, яке є результатом `rator\00\00\00 XOR s1`. Це згенерує `E(m2 XOR s1 XOR 0)`, що є s32.\
Тепер ви можете використовувати s32 як підпис повного імені **Administrator**.

### Резюме

1. Отримайте підпис імені користувача **Administ** (m1), який є s1
2. Отримайте підпис імені користувача **rator\x00\x00\x00 XOR s1 XOR 0**, що є s32**.**
3. Встановіть cookie на s32, і це буде дійсним cookie для користувача **Administrator**.

# Атака на контроль IV

Якщо ви можете контролювати використовуваний IV, атака може бути дуже простою.\
Якщо cookie є просто зашифрованим ім'ям користувача, щоб видати себе за користувача "**administrator**", ви можете створити користувача "**Administrator**" і отримати його cookie.\
Тепер, якщо ви можете контролювати IV, ви можете змінити перший байт IV так, що **IV\[0] XOR "A" == IV'\[0] XOR "a"** і відновити cookie для користувача **Administrator.** Цей cookie буде дійсним для **видачі** себе за користувача **administrator** з початковим **IV**.

## Посилання

Більше інформації на [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
