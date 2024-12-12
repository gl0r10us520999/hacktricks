# Kriptografski/Kompresioni Algoritmi

## Kriptografski/Kompresioni Algoritmi

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Obuka AWS Crveni Tim Stručnjak (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Obuka GCP Crveni Tim Stručnjak (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Identifikacija Algoritama

Ako završite u kodu **koristeći pomeranja udesno i ulevo, ekskluzivne ili operacije i nekoliko aritmetičkih operacija**, vrlo je moguće da je to implementacija **kriptografskog algoritma**. Ovde će biti prikazane neke metode za **identifikaciju algoritma koji se koristi bez potrebe za vraćanjem svakog koraka**.

### API funkcije

**CryptDeriveKey**

Ako se koristi ova funkcija, možete pronaći koji **algoritam se koristi** proverom vrednosti drugog parametra:

![](<../../.gitbook/assets/image (156).png>)

Pogledajte ovde tabelu mogućih algoritama i njihove dodeljene vrednosti: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

Kompresuje i dekompresuje dati bafer podataka.

**CryptAcquireContext**

Iz [dokumenata](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta): Funkcija **CryptAcquireContext** se koristi za dobijanje ručke ka određenom kontejneru ključeva unutar određenog provajdera kriptografskih usluga (CSP). **Ova vraćena ručka se koristi u pozivima funkcija CryptoAPI** koje koriste izabrani CSP.

**CryptCreateHash**

Pokreće heširanje toka podataka. Ako se koristi ova funkcija, možete pronaći koji **algoritam se koristi** proverom vrednosti drugog parametra:

![](<../../.gitbook/assets/image (549).png>)

\
Pogledajte ovde tabelu mogućih algoritama i njihove dodeljene vrednosti: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### Konstante koda

Ponekad je zaista lako identifikovati algoritam zahvaljujući činjenici da mora koristiti posebnu i jedinstvenu vrednost.

![](<../../.gitbook/assets/image (833).png>)

Ako pretražite prvu konstantu na Google-u, dobićete ovo:

![](<../../.gitbook/assets/image (529).png>)

Stoga, možete pretpostaviti da je dekompilirana funkcija **kalkulator sha256**.\
Možete pretražiti bilo koju od drugih konstanti i dobićete (verovatno) isti rezultat.

### Informacije o podacima

Ako kod nema značajne konstante, možda **učitava informacije iz odeljka .data**.\
Možete pristupiti tim podacima, **grupisati prvi dvojni reč** i pretražiti ga na Google-u kao što smo uradili u prethodnom odeljku:

![](<../../.gitbook/assets/image (531).png>)

U ovom slučaju, ako potražite **0xA56363C6** možete saznati da je povezano sa **tabelama algoritma AES**.

## RC4 **(Simetrična Kriptografija)**

### Karakteristike

Sastoji se od 3 glavna dela:

* **Faza inicijalizacije/**: Kreira **tabelu vrednosti od 0x00 do 0xFF** (ukupno 256 bajtova, 0x100). Ova tabela se obično naziva **Substitution Box** (ili SBox).
* **Faza mešanja**: Proći će **kroz prethodno kreiranu tabelu** (petlja od 0x100 iteracija, ponovo) modifikujući svaku vrednost sa **polu-slučajnim** bajtovima. Da bi se kreirali ovi polu-slučajni bajtovi, koristi se RC4 **ključ**. RC4 **ključevi** mogu biti **između 1 i 256 bajtova u dužini**, međutim obično se preporučuje da bude iznad 5 bajtova. Obično, RC4 ključevi su dužine 16 bajtova.
* **XOR faza**: Na kraju, plain-text ili cipertekst je **XOR-ovan sa vrednostima kreiranim pre**. Funkcija za šifrovanje i dešifrovanje je ista. Za ovo će se izvršiti **petlja kroz kreiranih 256 bajtova** koliko god puta je potrebno. Ovo se obično prepoznaje u dekompiliranom kodu sa **%256 (mod 256)**.

{% hint style="info" %}
**Da biste identifikovali RC4 u disasembliranom/dekompiliranom kodu, možete proveriti 2 petlje veličine 0x100 (sa korišćenjem ključa) i zatim XOR ulaznih podataka sa 256 vrednosti kreiranih pre u 2 petlje verovatno koristeći %256 (mod 256)**
{% endhint %}

### **Faza inicijalizacije/Substitution Box:** (Obratite pažnju na broj 256 korišćen kao brojač i kako je 0 upisan na svako mesto od 256 karaktera)

![](<../../.gitbook/assets/image (584).png>)

### **Faza mešanja:**

![](<../../.gitbook/assets/image (835).png>)

### **XOR faza:**

![](<../../.gitbook/assets/image (904).png>)

## **AES (Simetrična Kriptografija)**

### **Karakteristike**

* Korišćenje **substitution boxes i lookup tabela**
* Moguće je **razlikovati AES zahvaljujući korišćenju specifičnih vrednosti lookup tabela** (konstanti). _Imajte na umu da se **konstanta** može **skladištiti** u binarnom **ili kreirati**_ _**dinamički**._
* **Ključ za šifrovanje** mora biti **deljiv** sa **16** (obično 32B) i obično se koristi IV od 16B.

### SBox konstante

![](<../../.gitbook/assets/image (208).png>)

## Serpent **(Simetrična Kriptografija)**

### Karakteristike

* Retko je pronaći malver koji ga koristi ali postoje primeri (Ursnif)
* Jednostavno je odrediti da li je algoritam Serpent ili ne na osnovu njegove dužine (izuzetno duga funkcija)

### Identifikacija

U sledećoj slici primetite kako se koristi konstanta **0x9E3779B9** (imajte na umu da se ova konstanta takođe koristi i u drugim kripto algoritmima poput **TEA** -Tiny Encryption Algorithm).\
Takođe obratite pažnju na **veličinu petlje** (**132**) i **broj XOR operacija** u instrukcijama **disasemblera** i u **primeru koda**:

![](<../../.gitbook/assets/image (547).png>)

Kao što je pomenuto ranije, ovaj kod može biti vizualizovan unutar bilo kog dekompajlera kao **vrlo duga funkcija** jer **nema skokova** unutar nje. Dekompilirani kod može izgledati ovako:

![](<../../.gitbook/assets/image (513).png>)

Stoga je moguće identifikovati ovaj algoritam proverom **magičnog broja** i **početnih XOR-ova**, videći **vrlo dugu funkciju** i **upoređujući** neke **instrukcije** iz duge funkcije **sa implementacijom** (kao što je pomeranje ulevo za 7 i rotacija ulevo za 22).
## RSA **(Asimetrična kriptografija)**

### Karakteristike

* Složenija od simetričnih algoritama
* Nema konstanti! (prilagođene implementacije su teške za određivanje)
* KANAL (kripto analizator) ne uspeva da pokaže tragove o RSA jer se oslanja na konstante.

### Identifikacija poređenjem

![](<../../.gitbook/assets/image (1113).png>)

* U liniji 11 (levo) postoji `+7) >> 3` što je isto kao u liniji 35 (desno): `+7) / 8`
* Linija 12 (levo) proverava da li je `modulus_len < 0x040` i u liniji 36 (desno) proverava da li je `inputLen+11 > modulusLen`

## MD5 & SHA (heš)

### Karakteristike

* 3 funkcije: Init, Update, Final
* Slične inicijalizacijske funkcije

### Identifikacija

**Init**

Možete ih identifikovati proverom konstanti. Imajte na umu da sha\_init ima 1 konstantu koju MD5 nema:

![](<../../.gitbook/assets/image (406).png>)

**MD5 Transform**

Primetite korišćenje više konstanti

![](<../../.gitbook/assets/image (253) (1) (1).png>)

## CRC (heš)

* Manji i efikasniji jer je njegova funkcija da pronađe slučajne promene u podacima
* Koristi tabele za pretragu (tako da možete identifikovati konstante)

### Identifikacija

Proverite **konstante u tabeli za pretragu**:

![](<../../.gitbook/assets/image (508).png>)

Algoritam za CRC heš izgleda ovako:

![](<../../.gitbook/assets/image (391).png>)

## APLib (Kompresija)

### Karakteristike

* Nerecognoscibilne konstante
* Možete pokušati da napišete algoritam u Pythonu i tražite slične stvari online

### Identifikacija

Grafikon je prilično velik:

![](<../../.gitbook/assets/image (207) (2) (1).png>)

Proverite **3 poređenja da biste ga prepoznali**:

![](<../../.gitbook/assets/image (430).png>)
