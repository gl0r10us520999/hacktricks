# Stego Tricks

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

## **Ekstrakcija podataka iz fajlova**

### **Binwalk**

Alat za pretraživanje binarnih fajlova za ugrađene skrivene fajlove i podatke. Instalira se putem `apt`, a njegov izvor je dostupan na [GitHub](https://github.com/ReFirmLabs/binwalk).
```bash
binwalk file # Displays the embedded data
binwalk -e file # Extracts the data
binwalk --dd ".*" file # Extracts all data
```
### **Foremost**

Obnavlja fajlove na osnovu njihovih zaglavlja i podnožja, korisno za png slike. Instalira se putem `apt` sa svojim izvorom na [GitHub](https://github.com/korczis/foremost).
```bash
foremost -i file # Extracts data
```
### **Exiftool**

Pomaže u prikazivanju metapodataka datoteke, dostupno [ovde](https://www.sno.phy.queensu.ca/\~phil/exiftool/).
```bash
exiftool file # Shows the metadata
```
### **Exiv2**

Slično kao exiftool, za pregled metapodataka. Instalira se putem `apt`, izvor na [GitHub](https://github.com/Exiv2/exiv2), i ima [službenu veb stranicu](http://www.exiv2.org/).
```bash
exiv2 file # Shows the metadata
```
### **Datoteka**

Identifikujte tip datoteke s kojom se bavite.

### **Stringovi**

Izvlači čitljive stringove iz datoteka, koristeći različite postavke kodiranja za filtriranje izlaza.
```bash
strings -n 6 file # Extracts strings with a minimum length of 6
strings -n 6 file | head -n 20 # First 20 strings
strings -n 6 file | tail -n 20 # Last 20 strings
strings -e s -n 6 file # 7bit strings
strings -e S -n 6 file # 8bit strings
strings -e l -n 6 file # 16bit strings (little-endian)
strings -e b -n 6 file # 16bit strings (big-endian)
strings -e L -n 6 file # 32bit strings (little-endian)
strings -e B -n 6 file # 32bit strings (big-endian)
```
### **Comparison (cmp)**

Korisno za upoređivanje izmenjene datoteke sa njenom originalnom verzijom koja se može pronaći na mreži.
```bash
cmp original.jpg stego.jpg -b -l
```
## **Izvlačenje Skrivenih Podataka u Tekstu**

### **Skriveni Podaci u Prostorima**

Nevidljivi karakteri u naizgled praznim prostorima mogu skrivati informacije. Da biste izvukli ove podatke, posetite [https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder](https://www.irongeek.com/i.php?page=security/unicode-steganography-homoglyph-encoder).

## **Izvlačenje Podataka iz Slika**

### **Identifikacija Detalja Slike sa GraphicMagick**

[GraphicMagick](https://imagemagick.org/script/download.php) služi za određivanje tipova fajlova slika i identifikaciju potencijalne korupcije. Izvršite komandu u nastavku da biste pregledali sliku:
```bash
./magick identify -verbose stego.jpg
```
Da biste pokušali da popravite oštećenu sliku, dodavanje komentara u metapodacima može pomoći:
```bash
./magick mogrify -set comment 'Extraneous bytes removed' stego.jpg
```
### **Steghide za Sakrivanje Podataka**

Steghide olakšava skrivanje podataka unutar `JPEG, BMP, WAV, i AU` fajlova, sposoban je za ugrađivanje i vađenje enkriptovanih podataka. Instalacija je jednostavna koristeći `apt`, a [izvorni kod je dostupan na GitHub-u](https://github.com/StefanoDeVuono/steghide).

**Komande:**

* `steghide info file` otkriva da li fajl sadrži skrivene podatke.
* `steghide extract -sf file [--passphrase password]` važi skrivene podatke, lozinka je opcionalna.

Za vađenje putem veba, posetite [ovu veb stranicu](https://futureboy.us/stegano/decinput.html).

**Bruteforce Napad sa Stegcracker-om:**

* Da biste pokušali da probijete lozinku na Steghide-u, koristite [stegcracker](https://github.com/Paradoxis/StegCracker.git) na sledeći način:
```bash
stegcracker <file> [<wordlist>]
```
### **zsteg za PNG i BMP fajlove**

zsteg se specijalizuje za otkrivanje skrivenih podataka u PNG i BMP fajlovima. Instalacija se vrši putem `gem install zsteg`, sa [izvorom na GitHub-u](https://github.com/zed-0xff/zsteg).

**Komande:**

* `zsteg -a file` primenjuje sve metode detekcije na fajl.
* `zsteg -E file` specificira payload za ekstrakciju podataka.

### **StegoVeritas i Stegsolve**

**stegoVeritas** proverava metapodatke, vrši transformacije slika i primenjuje LSB brute forcing među ostalim funkcijama. Koristite `stegoveritas.py -h` za punu listu opcija i `stegoveritas.py stego.jpg` da izvršite sve provere.

**Stegsolve** primenjuje razne filtere boja kako bi otkrio skrivene tekstove ili poruke unutar slika. Dostupan je na [GitHub-u](https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve/stegsolve).

### **FFT za detekciju skrivenog sadržaja**

Tehnike brze Furijeove transformacije (FFT) mogu otkriti skrivene sadržaje u slikama. Korisni resursi uključuju:

* [EPFL Demo](http://bigwww.epfl.ch/demo/ip/demos/FFT/)
* [Ejectamenta](https://www.ejectamenta.com/Fourifier-fullscreen/)
* [FFTStegPic na GitHub-u](https://github.com/0xcomposure/FFTStegPic)

### **Stegpy za audio i slikovne fajlove**

Stegpy omogućava umetanje informacija u slikovne i audio fajlove, podržavajući formate kao što su PNG, BMP, GIF, WebP i WAV. Dostupan je na [GitHub-u](https://github.com/dhsdshdhk/stegpy).

### **Pngcheck za analizu PNG fajlova**

Za analizu PNG fajlova ili za validaciju njihove autentičnosti, koristite:
```bash
apt-get install pngcheck
pngcheck stego.png
```
### **Додатни алати за анализу слика**

За даље истраживање, размислите о посети:

* [Magic Eye Solver](http://magiceye.ecksdee.co.uk/)
* [Image Error Level Analysis](https://29a.ch/sandbox/2012/imageerrorlevelanalysis/)
* [Outguess](https://github.com/resurrecting-open-source-projects/outguess)
* [OpenStego](https://www.openstego.com/)
* [DIIT](https://diit.sourceforge.net/)

## **Извлачење података из аудио записа**

**Аудио стеганографија** нуди јединствен метод за сакривање информација у звучним фајловима. Различити алати се користе за уграђивање или извлачење скривеног садржаја.

### **Steghide (JPEG, BMP, WAV, AU)**

Steghide је свестран алат дизајниран за сакривање података у JPEG, BMP, WAV и AU фајловима. Детаљна упутства су доступна у [документацији стего трикова](stego-tricks.md#steghide).

### **Stegpy (PNG, BMP, GIF, WebP, WAV)**

Овај алат је компатибилан са различитим форматима укључујући PNG, BMP, GIF, WebP и WAV. За више информација, погледајте [секцију Stegpy](stego-tricks.md#stegpy-png-bmp-gif-webp-wav).

### **ffmpeg**

ffmpeg је кључан за процену интегритета аудио фајлова, истичући детаљне информације и указујући на било какве несагласности.
```bash
ffmpeg -v info -i stego.mp3 -f null -
```
### **WavSteg (WAV)**

WavSteg se odlično snalazi u skrivanju i ekstrakciji podataka unutar WAV fajlova koristeći strategiju najmanje značajnog bita. Dostupan je na [GitHub](https://github.com/ragibson/Steganography#WavSteg). Komande uključuju:
```bash
python3 WavSteg.py -r -b 1 -s soundfile -o outputfile

python3 WavSteg.py -r -b 2 -s soundfile -o outputfile
```
### **Deepsound**

Deepsound omogućava enkripciju i detekciju informacija unutar zvučnih fajlova koristeći AES-256. Može se preuzeti sa [službene stranice](http://jpinsoft.net/deepsound/download.aspx).

### **Sonic Visualizer**

Neprocenjiv alat za vizuelnu i analitičku inspekciju audio fajlova, Sonic Visualizer može otkriti skrivene elemente koji su nevidljivi drugim sredstvima. Posetite [službenu veb stranicu](https://www.sonicvisualiser.org/) za više informacija.

### **DTMF Tones - Dial Tones**

Detekcija DTMF tonova u audio fajlovima može se postići putem online alata kao što su [ovaj DTMF detektor](https://unframework.github.io/dtmf-detect/) i [DialABC](http://dialabc.com/sound/detect/index.html).

## **Other Techniques**

### **Binary Length SQRT - QR Code**

Binarni podaci koji se kvadriraju u celoj broju mogu predstavljati QR kod. Koristite ovaj isječak za proveru:
```python
import math
math.sqrt(2500) #50
```
Za konverziju binarnih podataka u sliku, proverite [dcode](https://www.dcode.fr/binary-image). Da biste pročitali QR kodove, koristite [ovaj online čitač barkoda](https://online-barcode-reader.inliteresearch.com/).

### **Prevod Brajove Pisma**

Za prevođenje Brajove pisma, [Branah Braille Translator](https://www.branah.com/braille-translator) je odličan resurs.

## **Reference**

* [**https://0xrick.github.io/lists/stego/**](https://0xrick.github.io/lists/stego/)
* [**https://github.com/DominicBreuker/stego-toolkit**](https://github.com/DominicBreuker/stego-toolkit)

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
