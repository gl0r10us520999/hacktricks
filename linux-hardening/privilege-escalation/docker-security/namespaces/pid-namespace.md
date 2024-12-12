# PID Namespace

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
{% endhint %}

## Основна інформація

PID (Process IDentifier) namespace — це функція в ядрі Linux, яка забезпечує ізоляцію процесів, дозволяючи групі процесів мати свій власний набір унікальних PID, відокремлений від PID в інших просторах імен. Це особливо корисно в контейнеризації, де ізоляція процесів є важливою для безпеки та управління ресурсами.

Коли створюється новий PID namespace, першому процесу в цьому просторі імен присвоюється PID 1. Цей процес стає "init" процесом нового простору імен і відповідає за управління іншими процесами в межах цього простору. Кожен наступний процес, створений у просторі імен, матиме унікальний PID у цьому просторі, і ці PID будуть незалежні від PID в інших просторах імен.

З точки зору процесу в PID namespace, він може бачити лише інші процеси в тому ж просторі імен. Він не знає про процеси в інших просторах імен і не може взаємодіяти з ними за допомогою традиційних інструментів управління процесами (наприклад, `kill`, `wait` тощо). Це забезпечує рівень ізоляції, який допомагає запобігти взаємодії процесів один з одним.

### Як це працює:

1. Коли створюється новий процес (наприклад, за допомогою системного виклику `clone()`), процес може бути призначений новому або існуючому PID namespace. **Якщо створюється новий простір імен, процес стає "init" процесом цього простору**.
2. **Ядро** підтримує **відображення між PID у новому просторі імен та відповідними PID** у батьківському просторі (тобто, просторі імен, з якого був створений новий простір). Це відображення **дозволяє ядру перекладати PID за необхідності**, наприклад, при надсиланні сигналів між процесами в різних просторах імен.
3. **Процеси в PID namespace можуть бачити та взаємодіяти лише з іншими процесами в тому ж просторі імен**. Вони не знають про процеси в інших просторах імен, і їхні PID є унікальними в межах їхнього простору.
4. Коли **PID namespace знищується** (наприклад, коли "init" процес простору виходить), **всі процеси в цьому просторі імен завершуються**. Це забезпечує належне очищення всіх ресурсів, пов'язаних з простором імен.

## Лабораторія:

### Створити різні простори імен

#### CLI
```bash
sudo unshare -pf --mount-proc /bin/bash
```
<details>

<summary>Помилка: bash: fork: Не вдається виділити пам'ять</summary>

Коли `unshare` виконується без параметра `-f`, виникає помилка через те, як Linux обробляє нові PID (ідентифікатори процесів) простори. Основні деталі та рішення наведені нижче:

1. **Пояснення проблеми**:
- Ядро Linux дозволяє процесу створювати нові простори за допомогою системного виклику `unshare`. Однак процес, який ініціює створення нового PID простору (який називається "процесом unshare"), не входить до нового простору; лише його дочірні процеси входять.
- Виконання `%unshare -p /bin/bash%` запускає `/bin/bash` в тому ж процесі, що й `unshare`. Відповідно, `/bin/bash` та його дочірні процеси знаходяться в оригінальному PID просторі.
- Перший дочірній процес `/bin/bash` у новому просторі стає PID 1. Коли цей процес завершується, це викликає очищення простору, якщо немає інших процесів, оскільки PID 1 має особливу роль усиновлення сирітських процесів. Ядро Linux тоді вимкне виділення PID у цьому просторі.

2. **Наслідок**:
- Завершення PID 1 у новому просторі призводить до очищення прапора `PIDNS_HASH_ADDING`. Це призводить до того, що функція `alloc_pid` не може виділити новий PID при створенні нового процесу, що викликає помилку "Не вдається виділити пам'ять".

3. **Рішення**:
- Проблему можна вирішити, використовуючи параметр `-f` з `unshare`. Цей параметр змушує `unshare` створити новий процес після створення нового PID простору.
- Виконання `%unshare -fp /bin/bash%` забезпечує, що команда `unshare` сама стає PID 1 у новому просторі. `/bin/bash` та його дочірні процеси тоді безпечно містяться в цьому новому просторі, запобігаючи передчасному завершенню PID 1 та дозволяючи нормальне виділення PID.

Забезпечивши, що `unshare` виконується з прапором `-f`, новий PID простір правильно підтримується, що дозволяє `/bin/bash` та його підпроцесам працювати без виникнення помилки виділення пам'яті.

</details>

Монтування нової інстанції файлової системи `/proc`, якщо ви використовуєте параметр `--mount-proc`, забезпечує, що новий простір монтування має **точний та ізольований вигляд інформації про процеси, специфічної для цього простору**.

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;Перевірте, в якому просторі імен знаходиться ваш процес
```bash
ls -l /proc/self/ns/pid
lrwxrwxrwx 1 root root 0 Apr  3 18:45 /proc/self/ns/pid -> 'pid:[4026532412]'
```
### Знайти всі PID простори

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name pid -exec readlink {} \; 2>/dev/null | sort -u
```
{% endcode %}

Зверніть увагу, що користувач root з початкового (за замовчуванням) PID простору може бачити всі процеси, навіть ті, що в нових PID просторах, тому ми можемо бачити всі PID простори.

### Увійти всередину PID простору
```bash
nsenter -t TARGET_PID --pid /bin/bash
```
Коли ви входите в PID простір з за замовчуванням, ви все ще зможете бачити всі процеси. І процес з цього PID ns зможе бачити новий bash у PID ns.

Також, ви можете **входити в інший PID простір процесу тільки якщо ви root**. І ви **не можете** **входити** в інший простір **без дескриптора**, що вказує на нього (як `/proc/self/ns/pid`)

## References
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)
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
</details>
{% endhint %}
