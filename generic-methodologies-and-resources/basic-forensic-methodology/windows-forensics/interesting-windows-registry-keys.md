# Цікаві ключі реєстру Windows

### Цікаві ключі реєстру Windows

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


### **Версія Windows та інформація про власника**
- Розташована за адресою **`Software\Microsoft\Windows NT\CurrentVersion`**, ви знайдете версію Windows, Service Pack, час установки та ім'я зареєстрованого власника у зрозумілій формі.

### **Ім'я комп'ютера**
- Ім'я хоста знаходиться під **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Налаштування часового поясу**
- Часовий пояс системи зберігається в **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Відстеження часу доступу**
- За замовчуванням, відстеження останнього часу доступу вимкнено (**`NtfsDisableLastAccessUpdate=1`**). Щоб увімкнути його, використовуйте:
`fsutil behavior set disablelastaccess 0`

### Версії Windows та Service Packs
- **Версія Windows** вказує на вид (наприклад, Home, Pro) та його випуск (наприклад, Windows 10, Windows 11), тоді як **Service Packs** є оновленнями, які включають виправлення та, іноді, нові функції.

### Увімкнення часу останнього доступу
- Увімкнення відстеження часу останнього доступу дозволяє бачити, коли файли були востаннє відкриті, що може бути критично важливим для судової експертизи або моніторингу системи.

### Деталі інформації про мережу
- Реєстр містить обширні дані про мережеві конфігурації, включаючи **типи мереж (бездротові, кабельні, 3G)** та **категорії мереж (Публічна, Приватна/Домашня, Доменна/Робоча)**, які є важливими для розуміння налаштувань безпеки мережі та дозволів.

### Кешування на стороні клієнта (CSC)
- **CSC** покращує доступ до офлайн-файлів, кешуючи копії спільних файлів. Різні налаштування **CSCFlags** контролюють, як і які файли кешуються, впливаючи на продуктивність та досвід користувача, особливо в середовищах з переривчастим з'єднанням.

### Програми автозапуску
- Програми, перераховані в різних ключах реєстру `Run` та `RunOnce`, автоматично запускаються під час завантаження, впливаючи на час завантаження системи та потенційно будучи точками інтересу для виявлення шкідливого програмного забезпечення або небажаного програмного забезпечення.

### Shellbags
- **Shellbags** не лише зберігають налаштування для перегляду папок, але й надають судову експертизу доступу до папок, навіть якщо папка більше не існує. Вони є безцінними для розслідувань, виявляючи активність користувача, яка не є очевидною через інші засоби.

### Інформація про USB та судова експертиза
- Деталі, збережені в реєстрі про USB-пристрої, можуть допомогти відстежити, які пристрої були підключені до комп'ютера, потенційно пов'язуючи пристрій з чутливими передачами файлів або інцидентами несанкціонованого доступу.

### Серійний номер тому
- **Серійний номер тому** може бути критично важливим для відстеження конкретного екземпляра файлової системи, корисним у судових сценаріях, де потрібно встановити походження файлу на різних пристроях.

### **Деталі завершення роботи**
- Час завершення роботи та кількість (остання лише для XP) зберігаються в **`System\ControlSet001\Control\Windows`** та **`System\ControlSet001\Control\Watchdog\Display`**.

### **Конфігурація мережі**
- Для детальної інформації про мережеві інтерфейси зверніться до **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Перші та останні часи підключення до мережі, включаючи VPN-з'єднання, реєструються під різними шляхами в **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Спільні папки**
- Спільні папки та налаштування знаходяться під **`System\ControlSet001\Services\lanmanserver\Shares`**. Налаштування кешування на стороні клієнта (CSC) визначають доступність офлайн-файлів.

### **Програми, які запускаються автоматично**
- Шляхи, такі як **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** та подібні записи під `Software\Microsoft\Windows\CurrentVersion`, детально описують програми, які налаштовані на запуск під час завантаження.

### **Пошуки та введені шляхи**
- Пошуки в Провіднику та введені шляхи відстежуються в реєстрі під **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** для WordwheelQuery та TypedPaths відповідно.

### **Недавні документи та файли Office**
- Недавні документи та файли Office, до яких було отримано доступ, зазначені в `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` та специфічних шляхах версії Office.

### **Найбільш нещодавно використані (MRU) елементи**
- Списки MRU, що вказують на нещодавні шляхи файлів та команди, зберігаються в різних підключах `ComDlg32` та `Explorer` під `NTUSER.DAT`.

### **Відстеження активності користувача**
- Функція User Assist реєструє детальну статистику використання програм, включаючи кількість запусків та час останнього запуску, за адресою **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Аналіз Shellbags**
- Shellbags, що розкривають деталі доступу до папок, зберігаються в `USRCLASS.DAT` та `NTUSER.DAT` під `Software\Microsoft\Windows\Shell`. Використовуйте **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** для аналізу.

### **Історія USB-пристроїв**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** та **`HKLM\SYSTEM\ControlSet001\Enum\USB`** містять багаті деталі про підключені USB-пристрої, включаючи виробника, назву продукту та часові мітки підключення.
- Користувача, пов'язаного з конкретним USB-пристроєм, можна визначити, шукаючи в хівах `NTUSER.DAT` для **{GUID}** пристрою.
- Останній змонтований пристрій та його серійний номер тому можна відстежити через `System\MountedDevices` та `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt` відповідно.

Цей посібник узагальнює ключові шляхи та методи для доступу до детальної інформації про систему, мережу та активність користувачів на системах Windows, прагнучи до ясності та зручності використання.



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
