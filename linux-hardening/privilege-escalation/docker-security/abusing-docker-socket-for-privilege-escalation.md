# Зловживання Docker Socket для ескалації привілеїв

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

Існують випадки, коли у вас є **доступ до docker socket** і ви хочете використовувати його для **ескалації привілеїв**. Деякі дії можуть бути дуже підозрілими, і ви можете захотіти їх уникнути, тому тут ви можете знайти різні прапорці, які можуть бути корисними для ескалації привілеїв:

### Через монтування

Ви можете **монтувати** різні частини **файлової системи** в контейнері, що працює як root, і **доступатися** до них.\
Ви також можете **зловживати монтуванням для ескалації привілеїв** всередині контейнера.

* **`-v /:/host`** -> Монтуйте файлову систему хоста в контейнер, щоб ви могли **читати файлову систему хоста.**
* Якщо ви хочете **відчувати себе на хості**, але бути в контейнері, ви можете вимкнути інші механізми захисту, використовуючи прапорці, такі як:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> Це схоже на попередній метод, але тут ми **монтуємо диск пристрою**. Потім, всередині контейнера запустіть `mount /dev/sda1 /mnt`, і ви можете **доступатися** до **файлової системи хоста** в `/mnt`
* Запустіть `fdisk -l` на хості, щоб знайти пристрій `</dev/sda1>` для монтування
* **`-v /tmp:/host`** -> Якщо з якоїсь причини ви можете **просто змонтувати деяку директорію** з хоста і у вас є доступ всередині хоста. Змонтируйте її та створіть **`/bin/bash`** з **suid** у змонтованій директорії, щоб ви могли **виконати його з хоста та ескалувати до root**.

{% hint style="info" %}
Зверніть увагу, що, можливо, ви не можете змонтувати папку `/tmp`, але ви можете змонтувати **іншу записувану папку**. Ви можете знайти записувані директорії, використовуючи: `find / -writable -type d 2>/dev/null`

**Зверніть увагу, що не всі директорії в linux машині підтримують біт suid!** Щоб перевірити, які директорії підтримують біт suid, запустіть `mount | grep -v "nosuid"`. Наприклад, зазвичай `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` та `/var/lib/lxcfs` не підтримують біт suid.

Також зверніть увагу, що якщо ви можете **монтувати `/etc`** або будь-яку іншу папку **з конфігураційними файлами**, ви можете змінити їх з контейнера docker як root, щоб **зловживати ними на хості** та ескалувати привілеї (можливо, модифікувавши `/etc/shadow`)
{% endhint %}

### Втеча з контейнера

* **`--privileged`** -> З цим прапорцем ви [знімаєте всю ізоляцію з контейнера](docker-privileged.md#what-affects). Перевірте техніки, щоб [втекти з привілейованих контейнерів як root](docker-breakout-privilege-escalation/#automatic-enumeration-and-escape).
* **`--cap-add=<CAPABILITY/ALL> [--security-opt apparmor=unconfined] [--security-opt seccomp=unconfined] [-security-opt label:disable]`** -> Щоб [ескалувати, зловживаючи можливостями](../linux-capabilities.md), **надайте цю можливість контейнеру** та вимкніть інші методи захисту, які можуть заважати експлуатації.

### Curl

На цій сторінці ми обговорили способи ескалації привілеїв, використовуючи прапорці docker, ви можете знайти **способи зловживати цими методами, використовуючи команду curl** на сторінці:

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
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
</details>
{% endhint %}
