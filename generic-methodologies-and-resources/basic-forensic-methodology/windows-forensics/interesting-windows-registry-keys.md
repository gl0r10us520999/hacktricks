# Цікаві ключі реєстру Windows

### Цікаві ключі реєстру Windows

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

### **Інформація про версію Windows та власника**
- У розділі **`Software\Microsoft\Windows NT\CurrentVersion`** ви знайдете версію Windows, пакунок обслуговування, час встановлення та ім'я зареєстрованого власника простим способом.

### **Ім'я комп'ютера**
- Ім'я хоста знаходиться в розділі **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Налаштування часового поясу**
- Часовий пояс системи зберігається в **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Відстеження часу доступу**
- За замовчуванням відстеження часу останнього доступу вимкнено (**`NtfsDisableLastAccessUpdate=1`**). Щоб ввімкнути його, використовуйте:
`fsutil behavior set disablelastaccess 0`

### Версії Windows та пакунки обслуговування
- **Версія Windows** вказує на вид (наприклад, Home, Pro) та її випуск (наприклад, Windows 10, Windows 11), тоді як **пакунки обслуговування** - це оновлення, які включають виправлення та іноді нові функції.

### Ввімкнення часу останнього доступу
- Ввімкнення відстеження часу останнього доступу дозволяє вам бачити, коли файли востаннє відкривалися, що може бути критичним для судового аналізу або моніторингу системи.

### Деталі мережевої інформації
- Реєстр містить велику кількість даних про мережеві конфігурації, включаючи **типи мереж (бездротові, кабельні, 3G)** та **категорії мереж (Публічна, Приватна/Домашня, Домен/Робоча)**, які є важливими для розуміння налаштувань мережевої безпеки та дозволів.

### Кешування клієнтської сторони (CSC)
- **CSC** покращує доступ до файлів в автономному режимі шляхом кешування копій спільних файлів. Різні налаштування **CSCFlags** контролюють те, які файли кешуються та як це впливає на продуктивність та користувацький досвід, особливо в середовищах з періодичним з'єднанням.

### Програми автозапуску
- Програми, перераховані в різних ключах реєстру `Run` та `RunOnce`, автоматично запускаються при запуску системи, впливаючи на час завантаження системи та потенційно бувши точками інтересу для ідентифікації шкідливого ПЗ або небажаного програмного забезпечення.

### Shellbags
- **Shellbags** не лише зберігають налаштування для перегляду папок, але також надають судові докази доступу до папок навіть у випадку, якщо папка більше не існує. Вони є невичерпним джерелом для розслідувань, розкриваючи діяльність користувача, яка не є очевидною іншими засобами.

### Інформація та судовий аналіз USB
- Деталі, збережені в реєстрі про USB-пристрої, можуть допомогти відстежити, які пристрої були підключені до комп'ютера, потенційно пов'язуючи пристрій з передачею чутливих файлів або випадками несанкціонованого доступу.

### Серійний номер тома
- **Серійний номер тома** може бути вирішальним для відстеження конкретного екземпляра файлової системи, корисний в судових сценаріях, де потрібно встановити походження файлу на різних пристроях.

### **Деталі вимкнення**
- Час вимкнення та кількість (останній лише для XP) зберігаються в **`System\ControlSet001\Control\Windows`** та **`System\ControlSet001\Control\Watchdog\Display`**.

### **Конфігурація мережі**
- Для докладної інформації про мережевий інтерфейс звертайтеся до **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Перші та останні часи підключення до мережі, включаючи підключення VPN, реєструються в різних шляхах в **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Спільні папки**
- Спільні папки та налаштування знаходяться в розділі **`System\ControlSet001\Services\lanmanserver\Shares`**. Налаштування кешування клієнтської сторони (CSC) визначають доступність файлів в автономному режимі.

### **Програми, які автоматично запускаються**
- Шляхи, такі як **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** та подібні записи в `Software\Microsoft\Windows\CurrentVersion` деталізують програми, призначені для запуску при завантаженні.

### **Пошуки та введені шляхи**
- Пошуки та введені шляхи в Explorer відстежуються в реєстрі під **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** для WordwheelQuery та TypedPaths відповідно.

### **Нещодавні документи та файли Office**
- Недавні документи та файли Office, до яких зверталися, відзначені в `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` та конкретні шляхи версій Office.

### **Останні використані елементи (MRU)**
- Списки MRU, що вказують на останні шляхи файлів та команд, зберігаються в різних підключеннях `ComDlg32` та `Explorer` під `NTUSER.DAT`.

### **Відстеження активності користувача**
- Функція User Assist веде детальну статистику використання програм, включаючи кількість запусків та час останнього запуску, в **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Аналіз Shellbags**
- Shellbags, які розкривають деталі доступу до папок, зберігаються в `USRCLASS.DAT` та `NTUSER.DAT` під `Software\Microsoft\Windows\Shell`. Використовуйте **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** для аналізу.

### **Історія USB-пристроїв**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** та **`HKLM\SYSTEM\ControlSet001\Enum\USB`** містять важливі деталі про підключені USB-пристрої, включаючи виробника, назву продукту та часи підключення.
- Користувач, пов'язаний з певним USB-пристроєм, може бути визначений шляхом пошуку уламків `NTUSER.DAT` для **{GUID}** пристрою.
- Останній підключений пристрій та його серійний номер тома можна відстежити через `System\MountedDevices` та `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt` відповідно.

Цей посібник узагальнює важливі шляхи та методи доступу до детальної інформації про систему, мережу та активність користувача в системах Windows, спрямований на зрозумілість та використовуваність. 

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
