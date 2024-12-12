# Certificates

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

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## What is a Certificate

**Публічний ключовий сертифікат** — це цифровий ID, який використовується в криптографії для підтвердження того, що хтось володіє публічним ключем. Він містить деталі ключа, ідентичність власника (суб'єкт) та цифровий підпис від довіреного органу (видавця). Якщо програмне забезпечення довіряє видавцю, а підпис дійсний, можлива безпечна комунікація з власником ключа.

Сертифікати в основному видаються [сертифікаційними центрами](https://en.wikipedia.org/wiki/Certificate\_authority) (CA) в налаштуванні [інфраструктури відкритих ключів](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI). Інший метод — це [мережа довіри](https://en.wikipedia.org/wiki/Web\_of\_trust), де користувачі безпосередньо перевіряють ключі один одного. Загальний формат для сертифікатів — [X.509](https://en.wikipedia.org/wiki/X.509), який може бути адаптований для специфічних потреб, як зазначено в RFC 5280.

## x509 Common Fields

### **Загальні поля в сертифікатах x509**

У сертифікатах x509 кілька **полів** відіграють критичну роль у забезпеченні дійсності та безпеки сертифіката. Ось розподіл цих полів:

* **Номер версії** позначає версію формату x509.
* **Серійний номер** унікально ідентифікує сертифікат у системі сертифікаційного центру (CA), головним чином для відстеження анулювання.
* Поле **Суб'єкт** представляє власника сертифіката, яким може бути машина, особа або організація. Воно містить детальну ідентифікацію, таку як:
* **Загальна назва (CN)**: домени, охоплені сертифікатом.
* **Країна (C)**, **Місцевість (L)**, **Штат або провінція (ST, S або P)**, **Організація (O)** та **Організаційна одиниця (OU)** надають географічні та організаційні деталі.
* **Виділене ім'я (DN)** охоплює повну ідентифікацію суб'єкта.
* **Видавець** вказує, хто перевірив і підписав сертифікат, включаючи подібні підполя, як у Суб'єкта для CA.
* **Період дійсності** позначається часовими мітками **Не раніше** та **Не пізніше**, що забезпечує, щоб сертифікат не використовувався до або після певної дати.
* Розділ **Публічний ключ**, критично важливий для безпеки сертифіката, вказує алгоритм, розмір та інші технічні деталі публічного ключа.
* **Розширення x509v3** покращують функціональність сертифіката, вказуючи **Використання ключа**, **Розширене використання ключа**, **Альтернативне ім'я суб'єкта** та інші властивості для точного налаштування застосування сертифіката.

#### **Використання ключа та розширення**

* **Використання ключа** ідентифікує криптографічні застосування публічного ключа, такі як цифровий підпис або шифрування ключа.
* **Розширене використання ключа** ще більше звужує випадки використання сертифіката, наприклад, для аутентифікації сервера TLS.
* **Альтернативне ім'я суб'єкта** та **Основне обмеження** визначають додаткові імена хостів, охоплені сертифікатом, та чи є це сертифікатом CA або кінцевого суб'єкта відповідно.
* Ідентифікатори, такі як **Ідентифікатор ключа суб'єкта** та **Ідентифікатор ключа органу**, забезпечують унікальність та відстежуваність ключів.
* **Доступ до інформації про орган** та **Точки розподілу CRL** надають шляхи для перевірки видавця CA та перевірки статусу анулювання сертифіката.
* **SCT сертифікатів CT** пропонують журнали прозорості, критично важливі для публічної довіри до сертифіката.
```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```
### **Різниця між OCSP та CRL Distribution Points**

**OCSP** (**RFC 2560**) передбачає співпрацю клієнта та респондента для перевірки, чи був відкликаний цифровий сертифікат з публічним ключем, без необхідності завантажувати повний **CRL**. Цей метод є більш ефективним, ніж традиційний **CRL**, який надає список серійних номерів відкликаних сертифікатів, але вимагає завантаження потенційно великого файлу. CRL можуть містити до 512 записів. Більше деталей доступно [тут](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### **Що таке Прозорість Сертифікатів**

Прозорість сертифікатів допомагає боротися з загрозами, пов'язаними з сертифікатами, забезпечуючи видимість видачі та існування SSL сертифікатів для власників доменів, ЦС та користувачів. Її цілі:

* Запобігання видачі ЦС SSL сертифікатів для домену без відома власника домену.
* Встановлення відкритої системи аудиту для відстеження помилково або зловмисно виданих сертифікатів.
* Захист користувачів від шахрайських сертифікатів.

#### **Журнали Сертифікатів**

Журнали сертифікатів є публічно доступними, підлягають аудиту, записами сертифікатів, які ведуться мережевими службами. Ці журнали надають криптографічні докази для цілей аудиту. Як органи видачі, так і громадськість можуть подавати сертифікати до цих журналів або запитувати їх для перевірки. Хоча точна кількість серверів журналів не є фіксованою, очікується, що їх буде менше тисячі в усьому світі. Ці сервери можуть незалежно управлятися ЦС, ISP або будь-якою зацікавленою стороною.

#### **Запит**

Щоб дослідити журнали Прозорості Сертифікатів для будь-якого домену, відвідайте [https://crt.sh/](https://crt.sh).

Існують різні формати для зберігання сертифікатів, кожен з яких має свої випадки використання та сумісність. Це резюме охоплює основні формати та надає рекомендації щодо конвертації між ними.

## **Формати**

### **PEM Формат**

* Найбільш поширений формат для сертифікатів.
* Вимагає окремих файлів для сертифікатів та приватних ключів, закодованих у Base64 ASCII.
* Загальні розширення: .cer, .crt, .pem, .key.
* Переважно використовується Apache та подібними серверами.

### **DER Формат**

* Бінарний формат сертифікатів.
* Не має заяв "BEGIN/END CERTIFICATE", які є у файлах PEM.
* Загальні розширення: .cer, .der.
* Часто використовується з платформами Java.

### **P7B/PKCS#7 Формат**

* Зберігається у Base64 ASCII, з розширеннями .p7b або .p7c.
* Містить лише сертифікати та ланцюгові сертифікати, виключаючи приватний ключ.
* Підтримується Microsoft Windows та Java Tomcat.

### **PFX/P12/PKCS#12 Формат**

* Бінарний формат, який інкапсулює серверні сертифікати, проміжні сертифікати та приватні ключі в одному файлі.
* Розширення: .pfx, .p12.
* Переважно використовується на Windows для імпорту та експорту сертифікатів.

### **Конвертація Форматів**

**Конверсії PEM** є важливими для сумісності:

* **x509 до PEM**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEM до DER**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DER до PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEM до P7B**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7 до PEM**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX конверсії** є критично важливими для управління сертифікатами на Windows:

* **PFX до PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX до PKCS#8** включає два етапи:
1. Конвертувати PFX в PEM
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. Перетворення PEM в PKCS8
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7B до PFX** також вимагає двох команд:
1. Конвертувати P7B в CER
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. Перетворення CER та приватного ключа в PFX
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкого створення та **автоматизації робочих процесів**, підтримуваних **найсучаснішими** інструментами спільноти.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

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
