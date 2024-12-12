{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа хакінгу HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа хакінгу HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>
{% endhint %}


# Перевірка BSSIDs

Коли ви отримуєте захоплення, головний трафік якого є Wifi, використовуючи WireShark, ви можете почати досліджувати всі SSID захоплення за допомогою _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## Brute Force

Один із стовпчиків на цьому екрані показує, чи **була знайдена будь-яка аутентифікація всередині pcap**. Якщо це так, ви можете спробувати використати Brute force за допомогою `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
# Дані в маяках / Бічний канал

Якщо ви підозрюєте, що **дані витікають в маяках мережі Wifi**, ви можете перевірити маяки мережі, використовуючи фільтр, подібний наступному: `wlan contains <ІМЯМЕРЕЖІ>`, або `wlan.ssid == "ІМЯМЕРЕЖІ"`, шукаючи підозрілі рядки відфільтрованих пакетів.

# Знаходження невідомих MAC-адрес в мережі Wifi

Наступне посилання буде корисним для пошуку **машин, які надсилають дані в мережі Wifi**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Якщо ви вже знаєте **MAC-адреси, ви можете видалити їх з виводу**, додавши перевірки, подібні до цієї: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Після виявлення **невідомих MAC-адрес**, які спілкуються в мережі, ви можете використовувати **фільтри**, подібні до наступного: `wlan.addr==<MAC-адрес> && (ftp || http || ssh || telnet)` для фільтрації трафіку. Зверніть увагу, що фільтри ftp/http/ssh/telnet корисні, якщо ви розшифрували трафік.

# Розшифрування трафіку

Редагування --> Налаштування --> Протоколи --> IEEE 802.11--> Редагувати

![](<../../../.gitbook/assets/image (426).png>)
