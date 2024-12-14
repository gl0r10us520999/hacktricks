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


# Інструменти карвінгу

## Autopsy

Найбільш поширений інструмент, що використовується в судовій експертизі для витягування файлів з образів, це [**Autopsy**](https://www.autopsy.com/download/). Завантажте його, встановіть і дайте йому обробити файл, щоб знайти "сховані" файли. Зверніть увагу, що Autopsy створено для підтримки образів дисків та інших видів образів, але не простих файлів.

## Binwalk <a id="binwalk"></a>

**Binwalk** - це інструмент для пошуку бінарних файлів, таких як зображення та аудіофайли, для вбудованих файлів і даних. Його можна встановити за допомогою `apt`, однак [джерело](https://github.com/ReFirmLabs/binwalk) можна знайти на github.  
**Корисні команди**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

Ще один поширений інструмент для знаходження прихованих файлів - це **foremost**. Ви можете знайти файл конфігурації foremost у `/etc/foremost.conf`. Якщо ви хочете шукати лише деякі конкретні файли, зніміть коментар з них. Якщо ви нічого не знімете, foremost буде шукати файли за замовчуванням.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** - це ще один інструмент, який можна використовувати для знаходження та вилучення **файлів, вбудованих у файл**. У цьому випадку вам потрібно буде зняти коментарі з файлів типів у конфігураційному файлі \(_/etc/scalpel/scalpel.conf_\), які ви хочете, щоб він вилучив.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

Цей інструмент входить до складу Kali, але ви можете знайти його тут: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

Цей інструмент може сканувати зображення і **витягувати pcaps** всередині нього, **мережеву інформацію (URL, домени, IP, MAC, електронні листи)** та інші **файли**. Вам потрібно лише:
```text
bulk_extractor memory.img -o out_folder
```
Navigate through **всю інформацію**, яку зібрав інструмент \(паролі?\), **аналізуйте** **пакети** \(читайте[ **аналіз Pcaps**](../pcap-inspection/)\), шукайте **дивні домени** \(домени, пов'язані з **шкідливим ПЗ** або **неіснуючі**\).

## PhotoRec

Ви можете знайти його за [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)

Він постачається з версією GUI та CLI. Ви можете вибрати **типи файлів**, які хочете, щоб PhotoRec шукав.

![](../../../.gitbook/assets/image%20%28524%29.png)

# Специфічні інструменти для карвінгу даних

## FindAES

Шукає ключі AES, досліджуючи їх графіки ключів. Може знаходити ключі 128, 192 та 256 біт, такі як ті, що використовуються TrueCrypt та BitLocker.

Завантажити [тут](https://sourceforge.net/projects/findaes/).

# Додаткові інструменти

Ви можете використовувати [**viu** ](https://github.com/atanunq/viu), щоб переглядати зображення з терміналу.  
Ви можете використовувати командний рядок linux **pdftotext**, щоб перетворити pdf у текст і прочитати його.

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
