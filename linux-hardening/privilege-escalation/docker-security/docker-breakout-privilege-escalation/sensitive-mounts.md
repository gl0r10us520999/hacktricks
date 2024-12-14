# Чутливі монтування

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Витік `/proc` та `/sys` без належної ізоляції простору імен створює значні ризики безпеки, включаючи збільшення поверхні атаки та розкриття інформації. Ці каталоги містять чутливі файли, які, якщо неправильно налаштовані або доступні несанкціонованим користувачем, можуть призвести до втечі з контейнера, модифікації хоста або надати інформацію, що сприяє подальшим атакам. Наприклад, неправильне монтування `-v /proc:/host/proc` може обійти захист AppArmor через його залежність від шляху, залишаючи `/host/proc` незахищеним.

**Ви можете знайти додаткові деталі кожної потенційної уразливості в** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Уразливості procfs

### `/proc/sys`

Цей каталог дозволяє доступ для зміни змінних ядра, зазвичай через `sysctl(2)`, і містить кілька підкаталогів, які викликають занепокоєння:

#### **`/proc/sys/kernel/core_pattern`**

* Описано в [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Дозволяє визначити програму для виконання при генерації core-файлу з першими 128 байтами як аргументами. Це може призвести до виконання коду, якщо файл починається з пайпа `|`.
*   **Приклад тестування та експлуатації**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Тест доступу на запис
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Встановити власний обробник
sleep 5 && ./crash & # Викликати обробник
```

#### **`/proc/sys/kernel/modprobe`**

* Детально описано в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Містить шлях до завантажувача модулів ядра, який викликається для завантаження модулів ядра.
*   **Приклад перевірки доступу**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Перевірка доступу до modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Згадано в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Глобальний прапор, який контролює, чи панікує ядро або викликає OOM-убивцю, коли виникає умова OOM.

#### **`/proc/sys/fs`**

* Згідно з [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), містить параметри та інформацію про файлову систему.
* Доступ на запис може дозволити різні атаки відмови в обслуговуванні проти хоста.

#### **`/proc/sys/fs/binfmt_misc`**

* Дозволяє реєструвати інтерпретатори для неоригінальних бінарних форматів на основі їх магічного номера.
* Може призвести до підвищення привілеїв або доступу до кореневого шеллу, якщо `/proc/sys/fs/binfmt_misc/register` доступний для запису.
* Відповідний експлойт та пояснення:
* [Бідний чоловік rootkit через binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Глибокий туторіал: [Посилання на відео](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Інші в `/proc`

#### **`/proc/config.gz`**

* Може розкрити конфігурацію ядра, якщо `CONFIG_IKCONFIG_PROC` увімкнено.
* Корисно для атакуючих для виявлення уразливостей у запущеному ядрі.

#### **`/proc/sysrq-trigger`**

* Дозволяє викликати команди Sysrq, потенційно викликаючи негайні перезавантаження системи або інші критичні дії.
*   **Приклад перезавантаження хоста**:

```bash
echo b > /proc/sysrq-trigger # Перезавантажує хост
```

#### **`/proc/kmsg`**

* Відкриває повідомлення з кільцевого буфера ядра.
* Може допомогти в експлойтах ядра, витоках адрес та надати чутливу інформацію про систему.

#### **`/proc/kallsyms`**

* Перераховує експортовані символи ядра та їх адреси.
* Важливо для розробки експлойтів ядра, особливо для подолання KASLR.
* Інформація про адреси обмежена, якщо `kptr_restrict` встановлено на `1` або `2`.
* Деталі в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Інтерфейси з пристроєм пам'яті ядра `/dev/mem`.
* Історично вразливий до атак підвищення привілеїв.
* Більше на [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Представляє фізичну пам'ять системи у форматі ELF core.
* Читання може призвести до витоку вмісту пам'яті хоста та інших контейнерів.
* Великий розмір файлу може призвести до проблем з читанням або збоїв програмного забезпечення.
* Детальне використання в [Витягування /proc/kcore у 2019 році](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Альтернативний інтерфейс для `/dev/kmem`, що представляє віртуальну пам'ять ядра.
* Дозволяє читання та запис, отже, безпосередню модифікацію пам'яті ядра.

#### **`/proc/mem`**

* Альтернативний інтерфейс для `/dev/mem`, що представляє фізичну пам'ять.
* Дозволяє читання та запис, модифікація всієї пам'яті вимагає вирішення віртуальних адрес у фізичні.

#### **`/proc/sched_debug`**

* Повертає інформацію про планування процесів, обминаючи захисти простору PID.
* Відкриває імена процесів, ID та ідентифікатори cgroup.

#### **`/proc/[pid]/mountinfo`**

* Надає інформацію про точки монтування в просторі імен монтування процесу.
* Відкриває місцезнаходження контейнера `rootfs` або образу.

### Уразливості `/sys`

#### **`/sys/kernel/uevent_helper`**

* Використовується для обробки `uevents` пристроїв ядра.
* Запис у `/sys/kernel/uevent_helper` може виконати довільні скрипти при спрацьовуванні `uevent`.
*   **Приклад експлуатації**: %%%bash

#### Створює корисне навантаження

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Знаходить шлях хоста з монтування OverlayFS для контейнера

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Встановлює uevent\_helper на шкідливий помічник

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Викликає uevent

echo change > /sys/class/mem/null/uevent

#### Читає вихідні дані

cat /output %%%

#### **`/sys/class/thermal`**

* Контролює налаштування температури, потенційно викликаючи атаки DoS або фізичні пошкодження.

#### **`/sys/kernel/vmcoreinfo`**

* Витікає адреси ядра, потенційно компрометуючи KASLR.

#### **`/sys/kernel/security`**

* Містить інтерфейс `securityfs`, що дозволяє налаштування Linux Security Modules, таких як AppArmor.
* Доступ може дозволити контейнеру вимкнути свою MAC-систему.

#### **`/sys/firmware/efi/vars` та `/sys/firmware/efi/efivars`**

* Відкриває інтерфейси для взаємодії з EFI змінними в NVRAM.
* Неправильна конфігурація або експлуатація можуть призвести до "заблокованих" ноутбуків або неможливих для завантаження хост-машин.

#### **`/sys/kernel/debug`**

* `debugfs` пропонує "без правил" інтерфейс для налагодження ядра.
* Історія проблем з безпекою через його необмежений характер.

### Посилання

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Розуміння та зміцнення Linux контейнерів](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Зловживання привілейованими та непривілейованими Linux контейнерами](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
Вивчайте та практикуйте AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
