# euid, ruid, suid

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

<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Deepen your expertise in **Mobile Security** with 8kSec Academy. Master iOS and Android security through our self-paced courses and get certified:

{% embed url="https://academy.8ksec.io/" %}


### Змінні ідентифікації користувача

- **`ruid`**: **реальний ідентифікатор користувача** позначає користувача, який ініціював процес.
- **`euid`**: Відомий як **ефективний ідентифікатор користувача**, він представляє ідентичність користувача, яку система використовує для визначення привілеїв процесу. Зазвичай `euid` відображає `ruid`, за винятком випадків, таких як виконання бінарного файлу з SetUID, коли `euid` приймає ідентичність власника файлу, надаючи таким чином певні операційні дозволи.
- **`suid`**: Цей **збережений ідентифікатор користувача** є важливим, коли процес з високими привілеями (зазвичай працює як root) тимчасово повинен відмовитися від своїх привілеїв для виконання певних завдань, а потім знову відновити свій початковий підвищений статус.

#### Важлива примітка
Процес, що не працює під root, може змінювати свій `euid` лише для того, щоб відповідати поточному `ruid`, `euid` або `suid`.

### Розуміння функцій set*uid

- **`setuid`**: На відміну від початкових припущень, `setuid` в основному змінює `euid`, а не `ruid`. Конкретно, для привілейованих процесів він вирівнює `ruid`, `euid` і `suid` з вказаним користувачем, часто root, ефективно закріплюючи ці ідентифікатори через переважаючий `suid`. Детальну інформацію можна знайти на [сторінці man setuid](https://man7.org/linux/man-pages/man2/setuid.2.html).
- **`setreuid`** та **`setresuid`**: Ці функції дозволяють для тонкого налаштування `ruid`, `euid` і `suid`. Однак їх можливості залежать від рівня привілеїв процесу. Для процесів, що не є root, зміни обмежуються поточними значеннями `ruid`, `euid` і `suid`. У свою чергу, процеси root або ті, що мають можливість `CAP_SETUID`, можуть призначати довільні значення цим ідентифікаторам. Більше інформації можна отримати з [сторінки man setresuid](https://man7.org/linux/man-pages/man2/setresuid.2.html) та [сторінки man setreuid](https://man7.org/linux/man-pages/man2/setreuid.2.html).

Ці функціональні можливості не призначені як механізм безпеки, а для полегшення запланованого операційного потоку, наприклад, коли програма приймає ідентичність іншого користувача, змінюючи свій ефективний ідентифікатор користувача.

Варто зазначити, що хоча `setuid` може бути звичайним способом підвищення привілеїв до root (оскільки він вирівнює всі ідентифікатори до root), важливо розрізняти ці функції для розуміння та маніпулювання поведінкою ідентифікаторів користувача в різних сценаріях.

### Механізми виконання програм у Linux

#### **Системний виклик `execve`**
- **Функціональність**: `execve` ініціює програму, визначену першим аргументом. Він приймає два масиви аргументів, `argv` для аргументів і `envp` для середовища.
- **Поведение**: Він зберігає простір пам'яті виклику, але оновлює стек, купу та сегменти даних. Код програми замінюється новою програмою.
- **Збереження ідентифікатора користувача**:
- `ruid`, `euid` та додаткові групові ідентифікатори залишаються незмінними.
- `euid` може мати нюанси змін, якщо нова програма має встановлений біт SetUID.
- `suid` оновлюється з `euid` після виконання.
- **Документація**: Детальну інформацію можна знайти на [сторінці man `execve`](https://man7.org/linux/man-pages/man2/execve.2.html).

#### **Функція `system`**
- **Функціональність**: На відміну від `execve`, `system` створює дочірній процес за допомогою `fork` і виконує команду в цьому дочірньому процесі за допомогою `execl`.
- **Виконання команди**: Виконує команду через `sh` з `execl("/bin/sh", "sh", "-c", command, (char *) NULL);`.
- **Поведение**: Оскільки `execl` є формою `execve`, він працює подібно, але в контексті нового дочірнього процесу.
- **Документація**: Додаткову інформацію можна отримати з [сторінки man `system`](https://man7.org/linux/man-pages/man3/system.3.html).

#### **Поведение `bash` і `sh` з SUID**
- **`bash`**:
- Має опцію `-p`, що впливає на те, як обробляються `euid` і `ruid`.
- Без `-p` `bash` встановлює `euid` на `ruid`, якщо вони спочатку відрізняються.
- З `-p` початковий `euid` зберігається.
- Більше деталей можна знайти на [сторінці man `bash`](https://linux.die.net/man/1/bash).
- **`sh`**:
- Не має механізму, подібного до `-p` в `bash`.
- Поведінка щодо ідентифікаторів користувача не згадується явно, за винятком опції `-i`, що підкреслює збереження рівності `euid` і `ruid`.
- Додаткова інформація доступна на [сторінці man `sh`](https://man7.org/linux/man-pages/man1/sh.1p.html).

Ці механізми, які відрізняються за своєю роботою, пропонують різноманітні варіанти для виконання та переходу між програмами, з конкретними нюансами в тому, як управляються та зберігаються ідентифікатори користувачів.

### Тестування поведінки ідентифікаторів користувача в виконаннях

Приклади взято з https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail, перевірте його для отримання додаткової інформації

#### Випадок 1: Використання `setuid` з `system`

**Мета**: Розуміння впливу `setuid` у поєднанні з `system` і `bash` як `sh`.

**C Код**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
system("id");
return 0;
}
```
**Компиляція та дозволи:**
```bash
oxdf@hacky$ gcc a.c -o /mnt/nfsshare/a;
oxdf@hacky$ chmod 4755 /mnt/nfsshare/a
```

```bash
bash-4.2$ $ ./a
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* `ruid` та `euid` починають з 99 (ніхто) та 1000 (франк) відповідно.
* `setuid` вирівнює обидва до 1000.
* `system` виконує `/bin/bash -c id` через символічне посилання з sh на bash.
* `bash`, без `-p`, коригує `euid`, щоб відповідати `ruid`, в результаті чого обидва стають 99 (ніхто).

#### Випадок 2: Використання setreuid з system

**C Код**:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setreuid(1000, 1000);
system("id");
return 0;
}
```
**Компиляція та дозволи:**
```bash
oxdf@hacky$ gcc b.c -o /mnt/nfsshare/b; chmod 4755 /mnt/nfsshare/b
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* `setreuid` встановлює як ruid, так і euid на 1000.
* `system` викликає bash, який підтримує ідентифікатори користувачів через їхню рівність, фактично діючи як frank.

#### Випадок 3: Використання setuid з execve
Мета: Дослідження взаємодії між setuid та execve.
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/usr/bin/id", NULL, NULL);
return 0;
}
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* `ruid` залишається 99, але euid встановлено на 1000, відповідно до ефекту setuid.

**C Код Приклад 2 (Виклик Bash):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/bin/bash", NULL, NULL);
return 0;
}
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
**Аналіз:**

* Хоча `euid` встановлено на 1000 за допомогою `setuid`, `bash` скидає euid на `ruid` (99) через відсутність `-p`.

**C Code Example 3 (Using bash -p):**
```bash
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
setuid(1000);
execve(paramList[0], paramList, NULL);
return 0;
}
```
**Виконання та Результат:**
```bash
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=100
```
## References
* [https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail)


<figure><img src="/.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Поглибте свої знання в **Mobile Security** з 8kSec Academy. Опануйте безпеку iOS та Android через наші курси з самостійним навчанням та отримайте сертифікат:

{% embed url="https://academy.8ksec.io/" %}


{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
