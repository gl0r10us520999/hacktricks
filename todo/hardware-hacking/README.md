# Hardware Hacking

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

## JTAG

JTAG omogućava izvođenje granice skeniranja. Granica skeniranja analizira određene sklopove, uključujući ugrađene ćelije za granice skeniranja i registre za svaki pin.

JTAG standard definiše **specifične komande za izvođenje granica skeniranja**, uključujući sledeće:

* **BYPASS** omogućava testiranje specifičnog čipa bez opterećenja prolaska kroz druge čipove.
* **SAMPLE/PRELOAD** uzima uzorak podataka koji ulaze i izlaze iz uređaja kada je u svom normalnom režimu rada.
* **EXTEST** postavlja i čita stanje pinova.

Takođe može podržati druge komande kao što su:

* **IDCODE** za identifikaciju uređaja
* **INTEST** za interno testiranje uređaja

Možda ćete naići na ove instrukcije kada koristite alat kao što je JTAGulator.

### Test Access Port

Granice skeniranja uključuju testove četvoržičnog **Test Access Port (TAP)**, opšteg port koji pruža **pristup JTAG test podršci** funkcijama ugrađenim u komponentu. TAP koristi sledećih pet signala:

* Ulaz testnog sata (**TCK**) TCK je **sat** koji definiše koliko često će TAP kontroler preuzeti jednu akciju (drugim rečima, skočiti na sledeće stanje u mašini stanja).
* Ulaz za odabir testnog moda (**TMS**) TMS kontroliše **konačnu mašinu stanja**. Na svakoj otkucaju sata, JTAG TAP kontroler uređaja proverava napon na TMS pinu. Ako je napon ispod određenog praga, signal se smatra niskim i tumači se kao 0, dok se, ako je napon iznad određenog praga, signal smatra visokim i tumači se kao 1.
* Ulaz testnih podataka (**TDI**) TDI je pin koji šalje **podatke u čip kroz ćelije skeniranja**. Svaki proizvođač je odgovoran za definisanje komunikacionog protokola preko ovog pina, jer JTAG to ne definiše.
* Izlaz testnih podataka (**TDO**) TDO je pin koji šalje **podatke iz čipa**.
* Ulaz za testni reset (**TRST**) Opcioni TRST resetuje konačnu mašinu stanja **na poznato dobro stanje**. Alternativno, ako se TMS drži na 1 tokom pet uzastopnih ciklusa sata, to poziva reset, na isti način na koji bi to uradio TRST pin, zbog čega je TRST opcioni.

Ponekad ćete moći da pronađete te pinove označene na PCB-u. U drugim slučajevima možda ćete morati da **pronađete ih**.

### Identifikacija JTAG pinova

Najbrži, ali najskuplji način za otkrivanje JTAG portova je korišćenje **JTAGulator**, uređaja kreiranog posebno za ovu svrhu (iako može **takođe otkriti UART pinove**).

Ima **24 kanala** koje možete povezati sa pinovima ploče. Zatim izvodi **BF napad** svih mogućih kombinacija šaljući **IDCODE** i **BYPASS** komande granice skeniranja. Ako primi odgovor, prikazuje kanal koji odgovara svakom JTAG signalu.

Jeftiniji, ali mnogo sporiji način identifikacije JTAG pinova je korišćenje [**JTAGenum**](https://github.com/cyphunk/JTAGenum/) učitanog na Arduino-kompatibilnom mikrokontroleru.

Korišćenjem **JTAGenum**, prvo biste **definisali pinove probnog** uređaja koji ćete koristiti za enumeraciju. Morali biste se osloniti na dijagram pinova uređaja, a zatim povezati te pinove sa testnim tačkama na vašem ciljanom uređaju.

**Treći način** za identifikaciju JTAG pinova je **inspekcija PCB-a** za jedan od pinova. U nekim slučajevima, PCB-ovi mogu povoljno pružiti **Tag-Connect interfejs**, što je jasan pokazatelj da ploča takođe ima JTAG konektor. Možete videti kako taj interfejs izgleda na [https://www.tag-connect.com/info/](https://www.tag-connect.com/info/). Pored toga, inspekcija **tehničkih listova čipova na PCB-u** može otkriti dijagrame pinova koji ukazuju na JTAG interfejse.

## SDW

SWD je ARM-specifičan protokol dizajniran za debagovanje.

SWD interfejs zahteva **dva pina**: dvosmerni **SWDIO** signal, koji je ekvivalent JTAG-ovim **TDI i TDO pinovima i satu**, i **SWCLK**, koji je ekvivalent **TCK** u JTAG-u. Mnogi uređaji podržavaju **Serial Wire ili JTAG Debug Port (SWJ-DP)**, kombinovani JTAG i SWD interfejs koji vam omogućava da povežete ili SWD ili JTAG probu sa ciljem.

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
