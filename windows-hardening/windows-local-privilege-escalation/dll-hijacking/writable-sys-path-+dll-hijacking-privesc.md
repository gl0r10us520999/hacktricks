# Writable Sys Path +Dll Hijacking Privesc

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

## Вступ

Якщо ви виявили, що можете **записувати в папку системного шляху** (зверніть увагу, що це не спрацює, якщо ви можете записувати в папку користувача), можливо, ви зможете **підвищити привілеї** в системі.

Для цього ви можете зловживати **Dll Hijacking**, де ви будете **викрадати бібліотеку, що завантажується** службою або процесом з **більшими привілеями**, ніж у вас, і оскільки ця служба завантажує Dll, яка, ймовірно, навіть не існує в системі, вона спробує завантажити її з системного шляху, в який ви можете записувати.

Для отримання додаткової інформації про **що таке Dll Hijacking** перегляньте:

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Підвищення привілеїв за допомогою Dll Hijacking

### Пошук відсутньої Dll

Перше, що вам потрібно зробити, це **виявити процес**, що працює з **більшими привілеями**, ніж у вас, який намагається **завантажити Dll з системного шляху**, в який ви можете записувати.

Проблема в таких випадках полягає в тому, що, ймовірно, ці процеси вже працюють. Щоб дізнатися, які Dll відсутні, вам потрібно запустити procmon якомога швидше (перед завантаженням процесів). Отже, щоб знайти відсутні .dll, зробіть:

* **Створіть** папку `C:\privesc_hijacking` і додайте шлях `C:\privesc_hijacking` до **системної змінної середовища Path**. Ви можете зробити це **вручну** або за допомогою **PS**:
```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```
* Запустіть **`procmon`** і перейдіть до **`Options`** --> **`Enable boot logging`** та натисніть **`OK`** у сповіщенні.
* Потім, **перезавантажте** комп'ютер. Коли комп'ютер перезавантажиться, **`procmon`** почне **записувати** події якомога швидше.
* Після того, як **Windows** буде **запущено, виконайте `procmon`** знову, він скаже вам, що працює, і **запитає, чи хочете ви зберегти** події у файл. Скажіть **так** і **збережіть події у файл**.
* **Після** того, як **файл** буде **згенеровано**, **закрийте** відкрите **вікно `procmon`** і **відкрийте файл подій**.
* Додайте ці **фільтри**, і ви знайдете всі Dll, які деякі **процеси намагалися завантажити** з папки записуваного системного шляху:

<figure><img src="../../../.gitbook/assets/image (945).png" alt=""><figcaption></figcaption></figure>

### Пропущені Dll

Запустивши це на безкоштовній **віртуальній (vmware) Windows 11 машині**, я отримав такі результати:

<figure><img src="../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

У цьому випадку .exe є марними, тому ігноруйте їх, пропущені DLL були з:

| Служба                         | Dll                | CMD рядок                                                             |
| ------------------------------- | ------------------ | -------------------------------------------------------------------- |
| Планувальник завдань (Schedule)       | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| Служба діагностичної політики (DPS) | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                             | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

Після того, як я це знайшов, я натрапив на цей цікавий блог, який також пояснює, як [**зловживати WptsExtensions.dll для підвищення привілеїв**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll). Що ми **зараз будемо робити**.

### Експлуатація

Отже, щоб **підвищити привілеї**, ми збираємося вкрасти бібліотеку **WptsExtensions.dll**. Маючи **шлях** і **ім'я**, нам просто потрібно **згенерувати шкідливий dll**.

Ви можете [**спробувати використати будь-який з цих прикладів**](./#creating-and-compiling-dlls). Ви можете запустити корисні навантаження, такі як: отримати rev shell, додати користувача, виконати beacon...

{% hint style="warning" %}
Зверніть увагу, що **не всі служби запускаються** з **`NT AUTHORITY\SYSTEM`**, деякі також запускаються з **`NT AUTHORITY\LOCAL SERVICE`**, які мають **менше привілеїв**, і ви **не зможете створити нового користувача**, зловживаючи його дозволами.\
Однак, цей користувач має привілей **`seImpersonate`**, тому ви можете використовувати [**потаточний набір для підвищення привілеїв**](../roguepotato-and-printspoofer.md). Отже, в цьому випадку rev shell є кращим варіантом, ніж спроба створити користувача.
{% endhint %}

На момент написання служба **Планувальник завдань** запускається з **Nt AUTHORITY\SYSTEM**.

Маючи **згенерований шкідливий Dll** (_в моєму випадку я використав x64 rev shell, і я отримав shell назад, але захисник його вбив, оскільки він був з msfvenom_), збережіть його в записуваному системному шляху з ім'ям **WptsExtensions.dll** і **перезавантажте** комп'ютер (або перезапустіть службу або зробіть все, що потрібно, щоб повторно запустити уражену службу/програму).

Коли служба буде перезапущена, **dll має бути завантажено та виконано** (ви можете **повторно використовувати** трюк з **procmon**, щоб перевірити, чи **бібліотека була завантажена, як очікувалося**).

{% hint style="success" %}
Вчіться та практикуйте Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вчіться та практикуйте Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
