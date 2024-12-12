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


# Identifikacija pakovanih binarnih datoteka

* **nedostatak stringova**: Uobičajeno je pronaći da pakovane binarne datoteke gotovo da nemaju nikakve stringove
* Puno **neiskorišćenih stringova**: Takođe, kada malware koristi neku vrstu komercijalnog pakera, uobičajeno je pronaći puno stringova bez međureferenci. Čak i ako ovi stringovi postoje, to ne znači da binarna datoteka nije pakovana.
* Takođe možete koristiti neke alate da pokušate da otkrijete koji je pakera korišćen za pakovanje binarne datoteke:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# Osnovne preporuke

* **Počnite** analizirati pakovanu binarnu datoteku **od dna u IDA i pomerajte se ka vrhu**. Rasklopnici izlaze kada rasklopljeni kod izađe, tako da je malo verovatno da će rasklopnik preneti izvršenje na rasklopljeni kod na početku.
* Pretražujte za **JMP-ovima** ili **CALL-ovima** ka **registrima** ili **regionima** **memorije**. Takođe pretražujte za **funkcijama koje prosleđuju argumente i adresu, a zatim pozivaju `retn`**, jer povratak funkcije u tom slučaju može pozvati adresu koja je upravo stavljena na stek pre nego što je pozvana.
* Postavite **prekidač** na `VirtualAlloc` jer ovo alocira prostor u memoriji gde program može pisati rasklopljeni kod. "Pokreni do korisničkog koda" ili koristite F8 da **dobijete vrednost unutar EAX** nakon izvršavanja funkcije i "**pratite tu adresu u dump-u**". Nikada ne znate da li je to region gde će biti sačuvan rasklopljeni kod.
* **`VirtualAlloc`** sa vrednošću "**40**" kao argument znači Čitanje+Pisanje+Izvršavanje (neki kod koji treba da se izvrši će biti kopiran ovde).
* **Dok rasklapate** kod, normalno je pronaći **several calls** ka **aritmetičkim operacijama** i funkcijama kao što su **`memcopy`** ili **`Virtual`**`Alloc`. Ako se nađete u funkciji koja očigledno samo vrši aritmetičke operacije i možda neki `memcopy`, preporuka je da pokušate da **pronađete kraj funkcije** (možda JMP ili poziv nekog registra) **ili** barem **poziv poslednje funkcije** i pokrenete do tada jer kod nije zanimljiv.
* Dok rasklapate kod **napomena** kada god **promenite region memorije** jer promena regiona memorije može ukazivati na **početak rasklapajućeg koda**. Možete lako dumpovati region memorije koristeći Process Hacker (proces --> svojstva --> memorija).
* Dok pokušavate da rasklopite kod, dobar način da **znate da li već radite sa rasklopljenim kodom** (tako da ga možete samo dumpovati) je da **proverite stringove binarne datoteke**. Ako u nekom trenutku izvršite skok (možda menjajući region memorije) i primetite da su **dodati mnogi više stringova**, tada možete znati **da radite sa rasklopljenim kodom**.\
Međutim, ako pakera već sadrži puno stringova, možete videti koliko stringova sadrži reč "http" i videti da li se ovaj broj povećava.
* Kada dumpujete izvršnu datoteku iz regiona memorije, možete popraviti neke zaglavlja koristeći [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases).
