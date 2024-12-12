# Вихід з KIOSKів

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}



---

## Перевірка фізичного пристрою

|   Компонент   | Дія                                                               |
| ------------- | ------------------------------------------------------------------ |
| Кнопка живлення  | Вимкнення та повторне вмикнення пристрою може відкрити стартовий екран      |
| Кабель живлення   | Перевірте, чи перезавантажується пристрій, коли живлення короткочасно вимкнено   |
| USB порти     | Підключіть фізичну клавіатуру з більшою кількістю комбінацій клавіш                        |
| Ethernet      | Сканування мережі або перехоплення може дозволити подальшу експлуатацію             |


## Перевірка можливих дій всередині GUI програми

**Звичайні діалоги** - це ті опції **збереження файлу**, **відкриття файлу**, вибір шрифту, кольору... Більшість з них **пропонують повну функціональність Провідника**. Це означає, що ви зможете отримати доступ до функцій Провідника, якщо зможете отримати доступ до цих опцій:

* Закрити/Закрити як
* Відкрити/Відкрити за допомогою
* Друк
* Експорт/Імпорт
* Пошук
* Сканування

Вам слід перевірити, чи можете ви:

* Модифікувати або створювати нові файли
* Створювати символічні посилання
* Отримати доступ до обмежених зон
* Виконувати інші програми

### Виконання команд

Можливо, **використовуючи опцію `Відкрити за допомогою`** ви зможете відкрити/виконати якийсь вид оболонки.

#### Windows

Наприклад _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ знайдіть більше бінарних файлів, які можна використовувати для виконання команд (та виконання несподіваних дій) тут: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ Більше тут: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### Обхід обмежень шляху

* **Змінні середовища**: Існує багато змінних середовища, які вказують на певний шлях
* **Інші протоколи**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Символічні посилання**
* **Ярлики**: CTRL+N (відкрити нову сесію), CTRL+R (виконати команди), CTRL+SHIFT+ESC (Диспетчер завдань), Windows+E (відкрити провідник), CTRL-B, CTRL-I (Улюблені), CTRL-H (Історія), CTRL-L, CTRL-O (Файл/Діалог відкриття), CTRL-P (Діалог друку), CTRL-S (Зберегти як)
* Схований адміністративний меню: CTRL-ALT-F8, CTRL-ESC-F9
* **Shell URIs**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC шляхи**: Шляхи для підключення до спільних папок. Вам слід спробувати підключитися до C$ локального комп'ютера ("\\\127.0.0.1\c$\Windows\System32")
* **Більше UNC шляхів:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

### Завантажте свої бінарні файли

Консоль: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Провідник: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Редактор реєстру: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### Доступ до файлової системи з браузера

| ШЛЯХ                | ШЛЯХ              | ШЛЯХ               | ШЛЯХ                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

### Ярлики

* Sticky Keys – натисніть SHIFT 5 разів
* Mouse Keys – SHIFT+ALT+NUMLOCK
* High Contrast – SHIFT+ALT+PRINTSCN
* Toggle Keys – утримуйте NUMLOCK протягом 5 секунд
* Filter Keys – утримуйте правий SHIFT протягом 12 секунд
* WINDOWS+F1 – Пошук Windows
* WINDOWS+D – Показати робочий стіл
* WINDOWS+E – Запустити Провідник Windows
* WINDOWS+R – Виконати
* WINDOWS+U – Центр доступності
* WINDOWS+F – Пошук
* SHIFT+F10 – Контекстне меню
* CTRL+SHIFT+ESC – Диспетчер завдань
* CTRL+ALT+DEL – Екран завантаження на новіших версіях Windows
* F1 – Допомога F3 – Пошук
* F6 – Адресний рядок
* F11 – Перемикання на повний екран у Internet Explorer
* CTRL+H – Історія Internet Explorer
* CTRL+T – Internet Explorer – Нова вкладка
* CTRL+N – Internet Explorer – Нова сторінка
* CTRL+O – Відкрити файл
* CTRL+S – Зберегти CTRL+N – Новий RDP / Citrix

### Проведення

* Проведіть з лівого боку вправо, щоб побачити всі відкриті вікна, зменшивши програму KIOSK і отримавши доступ до всієї ОС безпосередньо;
* Проведіть з правого боку вліво, щоб відкрити Центр дій, зменшивши програму KIOSK і отримавши доступ до всієї ОС безпосередньо;
* Проведіть з верхнього краю, щоб зробити панель заголовка видимою для програми, відкритої в повноекранному режимі;
* Проведіть вгору знизу, щоб показати панель завдань у повноекранній програмі.

### Трюки Internet Explorer

#### 'Панель інструментів зображень'

Це панель інструментів, яка з'являється у верхньому лівому куті зображення, коли на нього натискають. Ви зможете Зберегти, Друкувати, Відправити по електронній пошті, Відкрити "Мої зображення" в Провіднику. Kiosk повинен використовувати Internet Explorer.

#### Протокол Shell

Введіть ці URL-адреси, щоб отримати вигляд Провідника:

* `shell:Administrative Tools`
* `shell:DocumentsLibrary`
* `shell:Libraries`
* `shell:UserProfiles`
* `shell:Personal`
* `shell:SearchHomeFolder`
* `shell:NetworkPlacesFolder`
* `shell:SendTo`
* `shell:UserProfiles`
* `shell:Common Administrative Tools`
* `shell:MyComputerFolder`
* `shell:InternetFolder`
* `Shell:Profile`
* `Shell:ProgramFiles`
* `Shell:System`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Панель управління
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Мій комп'ютер
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Мої мережеві місця
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

### Показати розширення файлів

Перевірте цю сторінку для отримання додаткової інформації: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## Трюки браузерів

Резервні версії iKat:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

Створіть загальний діалог, використовуючи JavaScript, і отримайте доступ до провідника файлів: `document.write('<input/type=file>')`\
Джерело: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## iPad

### Жести та кнопки

* Проведіть вгору чотирма (або п'ятьма) пальцями / Подвійне натискання кнопки Home: Щоб переглянути вид багатозадачності та змінити програму
* Проведіть в один бік або інший чотирма або п'ятьма пальцями: Щоб змінити на наступну/остання програму
* Зробіть стиснення екрану п'ятьма пальцями / Натисніть кнопку Home / Проведіть вгору одним пальцем знизу екрану швидким рухом вгору: Щоб отримати доступ до головного екрану
* Проведіть одним пальцем знизу екрану всього на 1-2 дюйми (повільно): Показується док
* Проведіть вниз з верхньої частини дисплея одним пальцем: Щоб переглянути ваші сповіщення
* Проведіть вниз одним пальцем у верхньому правому куті екрану: Щоб побачити центр управління iPad Pro
* Проведіть одним пальцем з лівого боку екрану на 1-2 дюйми: Щоб побачити вид Сьогодні
* Проведіть швидко одним пальцем з центру екрану вправо або вліво: Щоб змінити на наступну/остання програму
* Натисніть і утримуйте кнопку Увімкнення/**Вимкнення**/Сну в верхньому правому куті **iPad +** Перемістіть повзунок Вимкнути **вимкнення** до правого краю: Щоб вимкнути
* Натисніть кнопку Увімкнення/**Вимкнення**/Сну в верхньому правому куті **iPad і кнопку Home на кілька секунд**: Щоб примусово вимкнути
* Натисніть кнопку Увімкнення/**Вимкнення**/Сну в верхньому правому куті **iPad і кнопку Home швидко**: Щоб зробити знімок екрана, який з'явиться в нижньому лівому куті дисплея. Натискайте обидві кнопки одночасно дуже коротко, оскільки якщо ви утримаєте їх кілька секунд, буде виконано жорстке вимкнення.

### Ярлики

Вам слід мати клавіатуру iPad або адаптер USB клавіатури. Тут будуть показані лише ярлики, які можуть допомогти вийти з програми.

| Клавіша | Назва         |
| --- | ------------ |
| ⌘   | Команда      |
| ⌥   | Опція (Alt) |
| ⇧   | Shift        |
| ↩   | Повернення       |
| ⇥   | Табуляція          |
| ^   | Контроль      |
| ←   | Ліва стрілка   |
| →   | Права стрілка  |
| ↑   | Вверх     |
| ↓   | Вниз     |

#### Системні ярлики

Ці ярлики призначені для візуальних налаштувань та налаштувань звуку, залежно від використання iPad.

| Ярлик | Дія                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Зменшити яскравість екрана                                                                    |
| F2       | Збільшити яскравість екрана                                                                |
| F7       | Назад на одну пісню                                                                  |
| F8       | Відтворити/пауза                                                                     |
| F9       | Пропустити пісню                                                                      |
| F10      | Вимкнути                                                                           |
| F11      | Зменшити гучність                                                                |
| F12      | Збільшити гучність                                                                |
| ⌘ Space  | Відобразити список доступних мов; щоб вибрати одну, натисніть пробіл ще раз. |

#### Навігація iPad

| Ярлик                                           | Дія                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Перейти на головний екран                                              |
| ⌘⇧H (Command-Shift-H)                              | Перейти на головний екран                                              |
| ⌘ (Space)                                          | Відкрити Spotlight                                          |
| ⌘⇥ (Command-Tab)                                   | Список останніх десяти використаних програм                                 |
| ⌘\~                                                | Перейти до останньої програми                                       |
| ⌘⇧3 (Command-Shift-3)                              | Знімок екрана (з'являється в нижньому лівому куті для збереження або дій) |
| ⌘⇧4                                                | Знімок екрана та відкриття його в редакторі                    |
| Натисніть і утримуйте ⌘                                   | Список доступних ярликів для програми                 |
| ⌘⌥D (Command-Option/Alt-D)                         | Відкриває док                                      |
| ^⌥H (Control-Option-H)                             | Кнопка Home                                             |
| ^⌥H H (Control-Option-H-H)                         | Показати панель багатозадачності                                      |
| ^⌥I (Control-Option-i)                             | Вибір елемента                                            |
| Escape                                             | Кнопка назад                                             |
| → (Права стрілка)                                    | Наступний елемент                                               |
| ← (Ліва стрілка)                                     | Попередній елемент                                           |
| ↑↓ (Вверх, Вниз)                          | Одночасно натискайте на вибраний елемент                        |
| ⌥ ↓ (Option-Down arrow)                            | Прокрутити вниз                                             |
| ⌥↑ (Option-Up arrow)                               | Прокрутити вгору                                               |
| ⌥← або ⌥→ (Option-Left arrow або Option-Right arrow) | Прокрутити вліво або вправо                                    |
| ^⌥S (Control-Option-S)                             | Увімкнути або вимкнути голосовий супровід                         |
| ⌘⇧⇥ (Command-Shift-Tab)                            | Перемикання на попередню програму                              |
| ⌘⇥ (Command-Tab)                                   | Повернутися до оригінальної програми                         |
| ←+→, потім Option + ← або Option+→                   | Навігація через док                                   |

#### Ярлики Safari

| Ярлик                | Дія                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Command-L)          | Відкрити місцезнаходження                                    |
| ⌘T                      | Відкрити нову вкладку                                   |
| ⌘W                      | Закрити поточну вкладку                            |
| ⌘R                      | Оновити поточну вкладку                          |
| ⌘.                      | Зупинити завантаження поточної вкладки                     |
| ^⇥                      | Перемикання на наступну вкладку                           |
| ^⇧⇥ (Control-Shift-Tab) | Перейти до попередньої вкладки                         |
| ⌘L                      | Вибрати текстове поле/URL для редагування     |
| ⌘⇧T (Command-Shift-T)   | Відкрити останню закриту вкладку (можна використовувати кілька разів) |
| ⌘\[                     | Повернутися на одну сторінку в історії перегляду      |
| ⌘]                      | Перейти вперед на одну сторінку в історії перегляду   |
| ⌘⇧R                     | Активувати режим читання                             |

#### Ярлики Mail

| Ярлик                   | Дія                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | Відкрити місцезнаходження                |
| ⌘T                         | Відкрити нову вкладку               |
| ⌘W                         | Закрити поточну вкладку        |
| ⌘R                         | Оновити поточну вкладку      |
| ⌘.                         | Зупинити завантаження поточної вкладки |
| ⌘⌥F (Command-Option/Alt-F) | Шукати у вашій поштовій скриньці       |

## Посилання

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)



{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримка HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
