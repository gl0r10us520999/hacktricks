# FZ - Infrared

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **nas pratite na** **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Uvod <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Za više informacija o tome kako funkcioniše infracrveno, proverite:

{% content-ref url="../infrared.md" %}
[infrared.md](../infrared.md)
{% endcontent-ref %}

## IR Signal Receiver in Flipper Zero <a href="#ir-signal-receiver-in-flipper-zero" id="ir-signal-receiver-in-flipper-zero"></a>

Flipper koristi digitalni IR prijemnik TSOP, koji **omogućava presretanje signala sa IR daljinskih upravljača**. Postoje neki **smartfoni** poput Xiaomija, koji takođe imaju IR port, ali imajte na umu da **većina njih može samo da prenosi** signale i **nije u mogućnosti da ih primi**.

Flipperov infracrveni **prijemnik je prilično osetljiv**. Možete čak i **uhvatiti signal** dok se nalazite **neposredno između** daljinskog upravljača i televizora. Usmeravanje daljinskog upravljača direktno na Flipperov IR port nije neophodno. Ovo je korisno kada neko menja kanale dok stoji blizu televizora, a vi i Flipper ste na određenoj udaljenosti.

Kako se **dekodiranje infracrvenog** signala dešava na **softverskoj** strani, Flipper Zero potencijalno podržava **prijem i prenos bilo kojih IR daljinskih kodova**. U slučaju **nepoznatih** protokola koji nisu mogli biti prepoznati - on **snima i reprodukuje** sirovi signal tačno onako kako je primljen.

## Akcije

### Univerzalni daljinski upravljači

Flipper Zero može se koristiti kao **univerzalni daljinski upravljač za kontrolu bilo kog televizora, klima uređaja ili medijskog centra**. U ovom režimu, Flipper **bruteforcuje** sve **poznate kodove** svih podržanih proizvođača **prema rečniku sa SD kartice**. Nije potrebno odabrati određeni daljinski upravljač da biste isključili televizor u restoranu.

Dovoljno je pritisnuti dugme za napajanje u režimu Univerzalnog daljinskog upravljača, i Flipper će **uzastopno slati "Power Off"** komande svih televizora koje poznaje: Sony, Samsung, Panasonic... i tako dalje. Kada televizor primi svoj signal, reagovaće i isključiti se.

Takav bruteforce zahteva vreme. Što je rečnik veći, to će duže trajati da se završi. Nemoguće je saznati koji signal je tačno televizor prepoznao, pošto nema povratne informacije od televizora.

### Nauči novi daljinski

Moguće je **uhvatiti infracrveni signal** sa Flipper Zero. Ako **pronađe signal u bazi podataka**, Flipper će automatski **znati koji je to uređaj** i omogućiti vam da komunicirate s njim.\
Ako ne, Flipper može **sačuvati** **signal** i omogućiti vam da ga **ponovo reprodukujete**.

## Reference

* [https://blog.flipperzero.one/infrared/](https://blog.flipperzero.one/infrared/)

{% hint style="success" %}
Učite i vežbajte AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Učite i vežbajte GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **nas pratite na** **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakerske trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}
