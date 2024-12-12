# Хитрощі Wireshark

## Хитрощі Wireshark

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Покращуйте свої навички Wireshark

### Навчальні посібники

Наступні навчальні посібники дивовижні для вивчення деяких цікавих базових трюків:

* [https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
* [https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* [https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* [https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)

### Аналізована інформація

**Інформація експерта**

Клацнувши на _**Аналізувати** --> **Інформація експерта**_ ви отримаєте **огляд** того, що відбувається в **аналізованих** пакетах:

![](<../../../.gitbook/assets/image (570).png>)

**Вирішені адреси**

Під _**Статистика --> Вирішені адреси**_ ви можете знайти кілька **інформації**, яка була "**вирішена**" Wireshark, такі як порт/транспорт до протоколу, MAC до виробника тощо. Цікаво знати, що включено в комунікацію.

![](<../../../.gitbook/assets/image (571).png>)

**Ієрархія протоколів**

Під _**Статистика --> Ієрархія протоколів**_ ви можете знайти **протоколи**, що **залучені** в комунікацію та дані про них.

![](<../../../.gitbook/assets/image (572).png>)

**Розмови**

Під _**Статистика --> Розмови**_ ви можете знайти **короткий огляд розмов** в комунікації та дані про них.

![](<../../../.gitbook/assets/image (573).png>)

**Кінцеві точки**

Під _**Статистика --> Кінцеві точки**_ ви можете знайти **короткий огляд кінцевих точок** в комунікації та дані про кожну з них.

![](<../../../.gitbook/assets/image (575).png>)

**Інформація DNS**

Під _**Статистика --> DNS**_ ви можете знайти статистику про захоплені запити DNS.

![](<../../../.gitbook/assets/image (577).png>)

**Графік введення/виведення**

Під _**Статистика --> Графік введення/виведення**_ ви можете знайти **графік комунікації.**

![](<../../../.gitbook/assets/image (574).png>)

### Фільтри

Тут ви можете знайти фільтр Wireshark в залежності від протоколу: [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)\
Інші цікаві фільтри:

* `(http.request or ssl.handshake.type == 1) and !(udp.port eq 1900)`
* Трафік HTTP та початковий трафік HTTPS
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002) and !(udp.port eq 1900)`
* Трафік HTTP та початковий трафік HTTPS + TCP SYN
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002 or dns) and !(udp.port eq 1900)`
* Трафік HTTP та початковий трафік HTTPS + TCP SYN + запити DNS

### Пошук

Якщо ви хочете **шукати** **вміст** всередині **пакетів** сеансів, натисніть _CTRL+f_. Ви можете додати нові шари до головної панелі інформації (№, Час, Джерело тощо), натиснувши праву кнопку, а потім редагувати стовпець.

### Безкоштовні лабораторії pcap

**Практикуйте з безкоштовними викликами: [https://www.malware-traffic-analysis.net/](https://www.malware-traffic-analysis.net)**

## Визначення доменів

Ви можете додати стовпець, який показує заголовок Host HTTP:

![](<../../../.gitbook/assets/image (403).png>)

І стовпець, який додає ім'я сервера з початкового з'єднання HTTPS (**ssl.handshake.type == 1**):

![](<../../../.gitbook/assets/image (408) (1).png>)

## Визначення локальних імен хостів

### З DHCP

У поточному Wireshark замість `bootp` вам потрібно шукати `DHCP`

![](<../../../.gitbook/assets/image (404).png>)

### З NBNS

![](<../../../.gitbook/assets/image (405).png>)

## Розшифрування TLS

### Розшифрування трафіку https за допомогою приватного ключа сервера

_редагування>параметри>протокол>ssl>_

![](<../../../.gitbook/assets/image (98).png>)

Натисніть _Редагувати_ та додайте всі дані сервера та приватного ключа (_IP, Порт, Протокол, Файл ключа та пароль_)

### Розшифрування трафіку https за допомогою симетричних ключів сеансу

Як Firefox, так і Chrome мають можливість реєструвати ключі сеансу TLS, які можна використовувати з Wireshark для розшифрування трафіку TLS. Це дозволяє проводити глибинний аналіз безпечних комунікацій. Докладнішу інформацію про те, як виконати це розшифрування, можна знайти в посібнику на [Red Flag Security](https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/).

Щоб виявити це, шукайте всередині середовища змінну `SSLKEYLOGFILE`

Файл спільних ключів буде виглядати так:

![](<../../../.gitbook/assets/image (99).png>)

Щоб імпортувати це в Wireshark, перейдіть до \_редагувати > параметри > протокол > ssl > та імпортуйте його в (Pre)-Master-Secret log filename:

![](<../../../.gitbook/assets/image (100).png>)

## Комунікація ADB

Витягніть APK з комунікації ADB, де APK було відправлено:
```python
from scapy.all import *

pcap = rdpcap("final2.pcapng")

def rm_data(data):
splitted = data.split(b"DATA")
if len(splitted) == 1:
return data
else:
return splitted[0]+splitted[1][4:]

all_bytes = b""
for pkt in pcap:
if Raw in pkt:
a = pkt[Raw]
if b"WRTE" == bytes(a)[:4]:
all_bytes += rm_data(bytes(a)[24:])
else:
all_bytes += rm_data(bytes(a))
print(all_bytes)

f = open('all_bytes.data', 'w+b')
f.write(all_bytes)
f.close()
```
<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
