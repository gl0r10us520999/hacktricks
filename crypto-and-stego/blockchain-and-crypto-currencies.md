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
- **Tokeni i kovanice** se razlikuju, pri čemu kovanice služe kao digitalni novac, dok tokeni predstavljaju vrednost ili vlasništvo u specifičnim kontekstima.
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

Metode uključuju trgovinu gotovinom, rudarenje i korišćenje miksera. **CoinJoin** meša više transakcija kako bi otežao praćenje, dok **PayJoin** prikriva CoinJoins kao obične transakcije radi povećane privatnosti.


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
If adding more inputs makes the change output larger than any single input, it can confuse the heuristic.

## **Forced Address Reuse**

Napadači mogu slati male iznose na prethodno korišćene adrese, nadajući se da će primalac kombinovati ove sa drugim ulazima u budućim transakcijama, čime se povezuju adrese.

### Correct Wallet Behavior
Novčanici bi trebali izbegavati korišćenje kovanica primljenih na već korišćenim, praznim adresama kako bi sprečili ovaj gubitak privatnosti.

## **Other Blockchain Analysis Techniques**

- **Exact Payment Amounts:** Transakcije bez promena su verovatno između dve adrese koje poseduje isti korisnik.
- **Round Numbers:** Okružni broj u transakciji sugeriše da je to uplata, pri čemu je neokružni izlaz verovatno promena.
- **Wallet Fingerprinting:** Različiti novčanici imaju jedinstvene obrasce kreiranja transakcija, što omogućava analitičarima da identifikuju korišćen softver i potencijalno adresu promene.
- **Amount & Timing Correlations:** Otkriće vremena ili iznosa transakcija može učiniti transakcije tragovima.

## **Traffic Analysis**

Prateći mrežni saobraćaj, napadači mogu potencijalno povezati transakcije ili blokove sa IP adresama, ugrožavajući privatnost korisnika. Ovo je posebno tačno ako entitet upravlja mnogim Bitcoin čvorovima, što poboljšava njihovu sposobnost praćenja transakcija.

## More
For a comprehensive list of privacy attacks and defenses, visit [Bitcoin Privacy on Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

# Anonymous Bitcoin Transactions

## Ways to Get Bitcoins Anonymously

- **Cash Transactions**: Sticanje bitcoina putem gotovine.
- **Cash Alternatives**: Kupovina poklon kartica i njihova razmena online za bitcoine.
- **Mining**: Najprivatnija metoda za zarađivanje bitcoina je putem rudarenja, posebno kada se radi samostalno jer rudarske grupe mogu znati IP adresu rudara. [Mining Pools Information](https://en.bitcoin.it/wiki/Pooled_mining)
- **Theft**: Teoretski, krađa bitcoina bi mogla biti još jedna metoda za sticanje anonimno, iako je to ilegalno i nije preporučljivo.

## Mixing Services

Korišćenjem usluge mešanja, korisnik može **slati bitcoine** i primati **različite bitcoine u zamenu**, što otežava praćenje originalnog vlasnika. Ipak, ovo zahteva poverenje u uslugu da ne čuva evidenciju i da zaista vrati bitcoine. Alternativne opcije mešanja uključuju Bitcoin kockarnice.

## CoinJoin

**CoinJoin** spaja više transakcija od različitih korisnika u jednu, komplikujući proces za svakoga ko pokušava da uskladi ulaze sa izlazima. I pored svoje efikasnosti, transakcije sa jedinstvenim ulaznim i izlaznim veličinama i dalje se potencijalno mogu pratiti.

Primeri transakcija koje su možda koristile CoinJoin uključuju `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` i `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

For more information, visit [CoinJoin](https://coinjoin.io/en). For a similar service on Ethereum, check out [Tornado Cash](https://tornado.cash), which anonymizes transactions with funds from miners.

## PayJoin

Varijanta CoinJoin, **PayJoin** (ili P2EP), prikriva transakciju između dve strane (npr. kupca i trgovca) kao redovnu transakciju, bez karakterističnih jednakih izlaza koji su specifični za CoinJoin. Ovo čini izuzetno teškim otkrivanje i moglo bi da poništi heuristiku zajedničkog vlasništva ulaza koju koriste entiteti za nadzor transakcija.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transakcije poput gornjih mogle bi biti PayJoin, poboljšavajući privatnost dok ostaju neprepoznatljive od standardnih bitcoin transakcija.

**Korišćenje PayJoin moglo bi značajno poremetiti tradicionalne metode nadzora**, čineći to obećavajućim razvojem u potrazi za transakcionom privatnošću.


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

Ove prakse i mehanizmi su osnovni za svakoga ko želi da se bavi kriptovalutama dok prioritet daje privatnosti i sigurnosti.


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
