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


# Alati za carving

## Autopsy

Najčešći alat korišćen u forenzici za ekstrakciju fajlova iz slika je [**Autopsy**](https://www.autopsy.com/download/). Preuzmite ga, instalirajte i omogućite mu da učita fajl kako bi pronašao "sakrivene" fajlove. Imajte na umu da je Autopsy napravljen da podržava disk slike i druge vrste slika, ali ne i obične fajlove.

## Binwalk <a id="binwalk"></a>

**Binwalk** je alat za pretraživanje binarnih fajlova kao što su slike i audio fajlovi za ugrađene fajlove i podatke. Može se instalirati sa `apt`, međutim [izvor](https://github.com/ReFirmLabs/binwalk) se može naći na github-u.  
**Korisne komande**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

Još jedan uobičajen alat za pronalaženje skrivenih fajlova je **foremost**. Konfiguracioni fajl foremost-a možete pronaći u `/etc/foremost.conf`. Ako želite da pretražujete samo neke specifične fajlove, otkomentarišite ih. Ako ne otkomentarišete ništa, foremost će pretraživati svoje podrazumevane konfiguracione tipove fajlova.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** je još jedan alat koji se može koristiti za pronalaženje i ekstrakciju **datoteka ugrađenih u datoteku**. U ovom slučaju, potrebno je da otkomentarišete tipove datoteka iz konfiguracione datoteke \(_/etc/scalpel/scalpel.conf_\) koje želite da ekstraktujete.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

Ovaj alat dolazi unutar kali, ali ga možete pronaći ovde: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

Ovaj alat može skenirati sliku i **izvući pcaps** unutar nje, **mrežne informacije (URL-ovi, domene, IP adrese, MAC adrese, e-mailovi)** i još **datoteka**. Samo treba da uradite:
```text
bulk_extractor memory.img -o out_folder
```
Navigirajte kroz **sve informacije** koje je alat prikupio \(lozinke?\), **analizirajte** **pakete** \(pročitajte[ **analizu Pcaps**](../pcap-inspection/)\), pretražujte **čudne domene** \(domene povezane sa **malverom** ili **nepostojećim**\).

## PhotoRec

Možete ga pronaći na [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)

Dolazi sa GUI i CLI verzijom. Možete odabrati **tipove fajlova** koje želite da PhotoRec pretražuje.

![](../../../.gitbook/assets/image%20%28524%29.png)

# Specifični alati za vađenje podataka

## FindAES

Pretražuje AES ključeve tražeći njihove rasporede ključeva. Sposoban je da pronađe 128, 192 i 256 bitne ključeve, kao što su oni koje koriste TrueCrypt i BitLocker.

Preuzmite [ovde](https://sourceforge.net/projects/findaes/).

# Dodatni alati

Možete koristiti [**viu** ](https://github.com/atanunq/viu) da vidite slike iz terminala.  
Možete koristiti linux alat komandne linije **pdftotext** da transformišete pdf u tekst i pročitate ga.

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
