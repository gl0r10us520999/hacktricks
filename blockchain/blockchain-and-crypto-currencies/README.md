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


## Osnovni Koncepti

- **Pametni ugovori** definišu se kao programi koji se izvršavaju na blockchain-u kada su ispunjeni određeni uslovi, automatizujući izvršenje sporazuma bez posrednika.
- **Decentralizovane aplikacije (dApps)** se oslanjaju na pametne ugovore, imajući korisnički prijatan front-end i transparentan, auditable back-end.
- **Tokeni i novčići** se razlikuju, pri čemu novčići služe kao digitalni novac, dok tokeni predstavljaju vrednost ili vlasništvo u specifičnim kontekstima.
- **Utility tokeni** omogućavaju pristup uslugama, a **Security tokeni** označavaju vlasništvo nad imovinom.
- **DeFi** označava decentralizovane finansije, nudeći finansijske usluge bez centralnih vlasti.
- **DEX** i **DAO** se odnose na decentralizovane berzanske platforme i decentralizovane autonomne organizacije, redom.

## Mehanizmi Konsenzusa

Mehanizmi konsenzusa osiguravaju sigurnu i dogovorenu validaciju transakcija na blockchain-u:
- **Proof of Work (PoW)** se oslanja na računarsku snagu za verifikaciju transakcija.
- **Proof of Stake (PoS)** zahteva od validatora da drže određenu količinu tokena, smanjujući potrošnju energije u poređenju sa PoW.

## Osnovne Informacije o Bitcoinu

### Transakcije

Bitcoin transakcije uključuju prebacivanje sredstava između adresa. Transakcije se validiraju putem digitalnih potpisa, osiguravajući da samo vlasnik privatnog ključa može inicirati transfere.

#### Ključne Komponente:

- **Multisignature transakcije** zahtevaju više potpisa za autorizaciju transakcije.
- Transakcije se sastoje od **ulaza** (izvor sredstava), **izlaza** (odredište), **naknada** (plaćene rudarima) i **skripti** (pravila transakcije).

### Lightning Network

Cilj je poboljšati skalabilnost Bitcoina omogućavajući više transakcija unutar jednog kanala, samo emitovanjem konačnog stanja na blockchain.

## Problemi Privatnosti Bitcoina

Napadi na privatnost, kao što su **Common Input Ownership** i **UTXO Change Address Detection**, koriste obrasce transakcija. Strategije poput **Mixers** i **CoinJoin** poboljšavaju anonimnost zamagljujući veze transakcija između korisnika.

## Sticanje Bitcoina Anonimno

Metode uključuju gotovinske trgovine, rudarenje i korišćenje miksera. **CoinJoin** meša više transakcija kako bi otežao praćenje, dok **PayJoin** prikriva CoinJoins kao obične transakcije radi povećane privatnosti.


# Napadi na Privatnost Bitcoina

# Sažetak Napada na Privatnost Bitcoina

U svetu Bitcoina, privatnost transakcija i anonimnost korisnika često su predmet zabrinutosti. Evo pojednostavljenog pregleda nekoliko uobičajenih metoda putem kojih napadači mogu kompromitovati privatnost Bitcoina.

## **Pretpostavka Zajedničkog Vlasništva Ulaza**

Generalno je retko da se ulazi različitih korisnika kombinuju u jednoj transakciji zbog složenosti koja je uključena. Tako se **dve ulazne adrese u istoj transakciji često pretpostavljaju da pripadaju istom vlasniku**.

## **UTXO Adresa Promene**

UTXO, ili **Unspent Transaction Output**, mora biti potpuno potrošen u transakciji. Ako se samo deo pošalje na drugu adresu, ostatak ide na novu adresu promene. Posmatrači mogu pretpostaviti da ova nova adresa pripada pošiljaocu, kompromitujući privatnost.

### Primer
Da bi se to ublažilo, usluge mešanja ili korišćenje više adresa mogu pomoći u zamagljivanju vlasništva.

## **Izloženost Društvenih Mreža i Foruma**

Korisnici ponekad dele svoje Bitcoin adrese na mreži, što olakšava **povezivanje adrese sa njenim vlasnikom**.

## **Analiza Transakcionih Grafova**

Transakcije se mogu vizualizovati kao grafovi, otkrivajući potencijalne veze između korisnika na osnovu toka sredstava.

## **Heuristika Nepotrebnog Ulaza (Optimalna Heuristika Promene)**

Ova heuristika se zasniva na analizi transakcija sa više ulaza i izlaza kako bi se pogodilo koji izlaz je promena koja se vraća pošiljaocu.

### Primer
```bash
2 btc --> 4 btc
3 btc     1 btc
```
Ako dodavanje više ulaza čini izlaz promene većim od bilo kog pojedinačnog ulaza, to može zbuniti heuristiku.

## **Prisilna Ponovna Upotreba Adresa**

Napadači mogu slati male iznose na prethodno korišćene adrese, nadajući se da će primalac kombinovati ove sa drugim ulazima u budućim transakcijama, čime se povezuju adrese.

### Ispravno Ponašanje Novčanika
Novčanici bi trebali izbegavati korišćenje kovanica primljenih na već korišćenim, praznim adresama kako bi se sprečilo ovo curenje privatnosti.

## **Druge Tehnike Analize Blokčejna**

- **Tačni Iznosi Plaćanja:** Transakcije bez promene su verovatno između dve adrese koje poseduje isti korisnik.
- **Celi Brojevi:** Celi broj u transakciji sugeriše da je to plaćanje, pri čemu je ne-celi izlaz verovatno promena.
- **Otisak Novčanika:** Različiti novčanici imaju jedinstvene obrasce kreiranja transakcija, što omogućava analitičarima da identifikuju korišćen softver i potencijalno adresu promene.
- **Korelacije Iznosa i Vremena:** Otkriće vremena ili iznosa transakcija može učiniti transakcije tragovima.

## **Analiza Saobraćaja**

Praćenjem mrežnog saobraćaja, napadači mogu potencijalno povezati transakcije ili blokove sa IP adresama, ugrožavajući privatnost korisnika. Ovo je posebno tačno ako entitet upravlja mnogim Bitcoin čvorovima, što poboljšava njihovu sposobnost praćenja transakcija.

## Više
Za sveobuhvatan spisak napada na privatnost i odbrana, posetite [Bitcoin Privatnost na Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

# Anonimne Bitcoin Transakcije

## Načini za Sticanje Bitcoina Anonimno

- **Gotovinske Transakcije**: Sticanje bitcoina putem gotovine.
- **Alternativne Gotovine**: Kupovina poklon kartica i njihova razmena online za bitcoine.
- **Rudarenje**: Najprivatnija metoda za zarađivanje bitcoina je putem rudarenja, posebno kada se radi samostalno, jer rudarske grupe mogu znati IP adresu rudara. [Informacije o Rudarskim Grupama](https://en.bitcoin.it/wiki/Pooled_mining)
- **Krađa**: Teoretski, krađa bitcoina bi mogla biti još jedan način da se anonimno stekne, iako je to ilegalno i ne preporučuje se.

## Usluge Mešanja

Korišćenjem usluge mešanja, korisnik može **poslati bitcoine** i primiti **različite bitcoine u zamenu**, što otežava praćenje originalnog vlasnika. Ipak, ovo zahteva poverenje u uslugu da ne čuva evidenciju i da zaista vrati bitcoine. Alternativne opcije mešanja uključuju Bitcoin kockarnice.

## CoinJoin

**CoinJoin** spaja više transakcija od različitih korisnika u jednu, komplikujući proces za svakoga ko pokušava da uskladi ulaze sa izlazima. I pored svoje efikasnosti, transakcije sa jedinstvenim ulazima i izlazima i dalje se mogu potencijalno pratiti.

Primeri transakcija koje su možda koristile CoinJoin uključuju `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` i `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Za više informacija, posetite [CoinJoin](https://coinjoin.io/en). Za sličnu uslugu na Ethereum-u, pogledajte [Tornado Cash](https://tornado.cash), koja anonimizuje transakcije sa sredstvima od rudara.

## PayJoin

Varijanta CoinJoin, **PayJoin** (ili P2EP), prikriva transakciju između dve strane (npr. kupca i trgovca) kao redovnu transakciju, bez karakterističnih jednakih izlaza koji su specifični za CoinJoin. Ovo čini izuzetno teškim otkrivanje i moglo bi da poništi heuristiku zajedničkog vlasništva ulaza koju koriste entiteti za nadzor transakcija.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transakcije poput gornjih mogle bi biti PayJoin, poboljšavajući privatnost dok ostaju neprepoznatljive od standardnih bitcoin transakcija.

**Korišćenje PayJoin-a moglo bi značajno poremetiti tradicionalne metode nadzora**, čineći to obećavajućim razvojem u potrazi za transakcionom privatnošću.


# Najbolje prakse za privatnost u kriptovalutama

## **Tehnike sinhronizacije novčanika**

Da bi se održala privatnost i sigurnost, sinhronizacija novčanika sa blockchain-om je ključna. Dve metode se ističu:

- **Puni čvor**: Preuzimanjem celog blockchain-a, puni čvor osigurava maksimalnu privatnost. Sve transakcije ikada izvršene se čuvaju lokalno, što onemogućava protivnicima da identifikuju koje transakcije ili adrese korisnik zanima.
- **Filtriranje blokova na klijentskoj strani**: Ova metoda uključuje kreiranje filtera za svaki blok u blockchain-u, omogućavajući novčanicima da identifikuju relevantne transakcije bez izlaganja specifičnih interesa posmatračima mreže. Laki novčanici preuzimaju ove filtere, preuzimajući pune blokove samo kada se pronađe podudaranje sa adresama korisnika.

## **Korišćenje Tora za anonimnost**

S obzirom na to da Bitcoin funkcioniše na peer-to-peer mreži, preporučuje se korišćenje Tora za maskiranje vaše IP adrese, poboljšavajući privatnost prilikom interakcije sa mrežom.

## **Sprečavanje ponovne upotrebe adresa**

Da bi se zaštitila privatnost, važno je koristiti novu adresu za svaku transakciju. Ponovna upotreba adresa može kompromitovati privatnost povezivanjem transakcija sa istim entitetom. Moderni novčanici obeshrabruju ponovnu upotrebu adresa kroz svoj dizajn.

## **Strategije za privatnost transakcija**

- **Više transakcija**: Deljenje uplate na nekoliko transakcija može zamagliti iznos transakcije, ometajući napade na privatnost.
- **Izbegavanje promena**: Odabir transakcija koje ne zahtevaju promene poboljšava privatnost ometajući metode detekcije promena.
- **Više izlaza za promene**: Ako izbegavanje promena nije izvodljivo, generisanje više izlaza za promene može i dalje poboljšati privatnost.

# **Monero: Svetionik anonimnosti**

Monero odgovara na potrebu za apsolutnom anonimnošću u digitalnim transakcijama, postavljajući visoke standarde za privatnost.

# **Ethereum: Gas i transakcije**

## **Razumevanje gasa**

Gas meri računski napor potreban za izvršavanje operacija na Ethereum-u, a cena je u **gwei**. Na primer, transakcija koja košta 2,310,000 gwei (ili 0.00231 ETH) uključuje gas limit i osnovnu naknadu, uz napojnicu za podsticanje rudara. Korisnici mogu postaviti maksimalnu naknadu kako bi osigurali da ne preplate, a višak se vraća.

## **Izvršavanje transakcija**

Transakcije u Ethereum-u uključuju pošiljaoca i primaoca, koji mogu biti adrese korisnika ili pametnih ugovora. One zahtevaju naknadu i moraju biti rudarene. Osnovne informacije u transakciji uključuju primaoca, potpis pošiljaoca, vrednost, opcione podatke, gas limit i naknade. Značajno je da se adresa pošiljaoca deducira iz potpisa, čime se eliminiše potreba za njom u podacima transakcije.

Ove prakse i mehanizmi su osnovni za svakoga ko želi da se bavi kriptovalutama, a da pri tome prioritet daje privatnosti i sigurnosti.


## Reference

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)


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
