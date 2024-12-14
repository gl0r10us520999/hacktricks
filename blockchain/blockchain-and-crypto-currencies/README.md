{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na githubie.

</details>
{% endhint %}


## Podstawowe pojęcia

- **Smart Contracts** są definiowane jako programy, które wykonują się na blockchainie, gdy spełnione są określone warunki, automatyzując realizację umów bez pośredników.
- **Decentralized Applications (dApps)** opierają się na smart contracts, oferując przyjazny interfejs użytkownika oraz przejrzysty, audytowalny backend.
- **Tokens & Coins** różnią się tym, że monety służą jako cyfrowe pieniądze, podczas gdy tokeny reprezentują wartość lub własność w określonych kontekstach.
- **Utility Tokens** dają dostęp do usług, a **Security Tokens** oznaczają własność aktywów.
- **DeFi** oznacza Decentralized Finance, oferując usługi finansowe bez centralnych władz.
- **DEX** i **DAOs** odnoszą się do Decentralized Exchange Platforms i Decentralized Autonomous Organizations, odpowiednio.

## Mechanizmy konsensusu

Mechanizmy konsensusu zapewniają bezpieczne i uzgodnione walidacje transakcji na blockchainie:
- **Proof of Work (PoW)** polega na mocy obliczeniowej do weryfikacji transakcji.
- **Proof of Stake (PoS)** wymaga, aby walidatorzy posiadali określoną ilość tokenów, co zmniejsza zużycie energii w porównaniu do PoW.

## Podstawy Bitcoina

### Transakcje

Transakcje Bitcoinowe polegają na transferze środków między adresami. Transakcje są weryfikowane za pomocą podpisów cyfrowych, co zapewnia, że tylko właściciel klucza prywatnego może inicjować transfery.

#### Kluczowe komponenty:

- **Transakcje wielopodpisowe** wymagają wielu podpisów do autoryzacji transakcji.
- Transakcje składają się z **wejść** (źródło funduszy), **wyjść** (cel), **opłat** (płatnych górnikom) oraz **skryptów** (reguły transakcji).

### Lightning Network

Ma na celu zwiększenie skalowalności Bitcoina, pozwalając na wiele transakcji w ramach jednego kanału, transmitując tylko ostateczny stan do blockchaina.

## Problemy z prywatnością Bitcoina

Ataki na prywatność, takie jak **Common Input Ownership** i **UTXO Change Address Detection**, wykorzystują wzorce transakcji. Strategie takie jak **Mixers** i **CoinJoin** poprawiają anonimowość, zaciemniając powiązania transakcji między użytkownikami.

## Nabywanie Bitcoinów anonimowo

Metody obejmują transakcje gotówkowe, kopanie oraz korzystanie z mixerów. **CoinJoin** łączy wiele transakcji, aby skomplikować śledzenie, podczas gdy **PayJoin** maskuje CoinJoins jako zwykłe transakcje dla zwiększonej prywatności.


# Ataki na prywatność Bitcoina

# Podsumowanie ataków na prywatność Bitcoina

W świecie Bitcoina prywatność transakcji i anonimowość użytkowników są często przedmiotem zmartwień. Oto uproszczony przegląd kilku powszechnych metod, za pomocą których atakujący mogą naruszyć prywatność Bitcoina.

## **Założenie wspólnej własności wejść**

Zazwyczaj rzadko zdarza się, aby wejścia od różnych użytkowników były łączone w jednej transakcji z powodu związanej z tym złożoności. Dlatego **dwa adresy wejściowe w tej samej transakcji często zakłada się, że należą do tego samego właściciela**.

## **Wykrywanie adresu zmiany UTXO**

UTXO, czyli **Unspent Transaction Output**, musi być całkowicie wydany w transakcji. Jeśli tylko część z niego jest wysyłana na inny adres, reszta trafia na nowy adres zmiany. Obserwatorzy mogą założyć, że ten nowy adres należy do nadawcy, co narusza prywatność.

### Przykład
Aby to złagodzić, usługi mieszania lub korzystanie z wielu adresów mogą pomóc w zaciemnieniu własności.

## **Ekspozycja w sieciach społecznościowych i na forach**

Użytkownicy czasami dzielą się swoimi adresami Bitcoinowymi w Internecie, co sprawia, że **łatwo jest powiązać adres z jego właścicielem**.

## **Analiza grafów transakcji**

Transakcje można wizualizować jako grafy, ujawniające potencjalne powiązania między użytkownikami na podstawie przepływu funduszy.

## **Heurystyka niepotrzebnych wejść (Heurystyka optymalnej zmiany)**

Ta heurystyka opiera się na analizie transakcji z wieloma wejściami i wyjściami, aby zgadnąć, które wyjście jest zmianą wracającą do nadawcy.

### Przykład
```bash
2 btc --> 4 btc
3 btc     1 btc
```
If adding more inputs makes the change output larger than any single input, it can confuse the heuristic.

## **Wymuszone ponowne użycie adresu**

Atakujący mogą wysyłać małe kwoty na wcześniej używane adresy, mając nadzieję, że odbiorca połączy je z innymi wejściami w przyszłych transakcjach, łącząc w ten sposób adresy.

### Poprawne zachowanie portfela
Portfele powinny unikać używania monet otrzymanych na już używanych, pustych adresach, aby zapobiec temu wyciekowi prywatności.

## **Inne techniki analizy blockchaina**

- **Dokładne kwoty płatności:** Transakcje bez reszty prawdopodobnie odbywają się między dwoma adresami należącymi do tego samego użytkownika.
- **Okrągłe liczby:** Okrągła liczba w transakcji sugeruje, że jest to płatność, a nieokrągły wynik prawdopodobnie jest resztą.
- **Odcisk portfela:** Różne portfele mają unikalne wzorce tworzenia transakcji, co pozwala analitykom zidentyfikować używane oprogramowanie i potencjalnie adres zmiany.
- **Korelacje kwoty i czasu:** Ujawnienie czasów lub kwot transakcji może sprawić, że transakcje będą śledzone.

## **Analiza ruchu**

Monitorując ruch w sieci, atakujący mogą potencjalnie powiązać transakcje lub bloki z adresami IP, naruszając prywatność użytkowników. Jest to szczególnie prawdziwe, jeśli podmiot obsługuje wiele węzłów Bitcoin, co zwiększa ich zdolność do monitorowania transakcji.

## Więcej
Aby uzyskać pełną listę ataków na prywatność i obrony, odwiedź [Prywatność Bitcoina na Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

# Anonimowe transakcje Bitcoin

## Sposoby na uzyskanie Bitcoinów anonimowo

- **Transakcje gotówkowe**: Nabywanie bitcoinów za gotówkę.
- **Alternatywy gotówkowe**: Zakup kart podarunkowych i wymiana ich online na bitcoiny.
- **Kopanie**: Najbardziej prywatną metodą zarabiania bitcoinów jest kopanie, szczególnie gdy jest wykonywane samodzielnie, ponieważ pule wydobywcze mogą znać adres IP górnika. [Informacje o pulach wydobywczych](https://en.bitcoin.it/wiki/Pooled_mining)
- **Kradzież**: Teoretycznie kradzież bitcoinów mogłaby być inną metodą na ich anonimowe zdobycie, chociaż jest to nielegalne i niezalecane.

## Usługi mieszania

Korzystając z usługi mieszania, użytkownik może **wysłać bitcoiny** i otrzymać **inne bitcoiny w zamian**, co utrudnia śledzenie oryginalnego właściciela. Jednak wymaga to zaufania do usługi, aby nie prowadziła logów i rzeczywiście zwróciła bitcoiny. Alternatywne opcje mieszania obejmują kasyna Bitcoin.

## CoinJoin

**CoinJoin** łączy wiele transakcji od różnych użytkowników w jedną, co komplikuje proces dla każdego, kto próbuje dopasować wejścia do wyjść. Mimo swojej skuteczności, transakcje z unikalnymi rozmiarami wejść i wyjść mogą nadal być potencjalnie śledzone.

Przykładowe transakcje, które mogły używać CoinJoin, to `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` i `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Aby uzyskać więcej informacji, odwiedź [CoinJoin](https://coinjoin.io/en). Dla podobnej usługi na Ethereum, sprawdź [Tornado Cash](https://tornado.cash), które anonimizuje transakcje z funduszami od górników.

## PayJoin

Wariant CoinJoin, **PayJoin** (lub P2EP), maskuje transakcję między dwiema stronami (np. klientem a sprzedawcą) jako zwykłą transakcję, bez charakterystycznych równych wyjść typowych dla CoinJoin. To sprawia, że jest niezwykle trudne do wykrycia i może unieważnić heurystykę wspólnego posiadania wejść używaną przez podmioty monitorujące transakcje.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transakcje takie jak powyższe mogą być PayJoin, zwiększając prywatność, jednocześnie pozostając nieodróżnialnymi od standardowych transakcji bitcoinowych.

**Wykorzystanie PayJoin może znacząco zakłócić tradycyjne metody nadzoru**, co czyni to obiecującym rozwojem w dążeniu do prywatności transakcyjnej.


# Najlepsze praktyki dotyczące prywatności w kryptowalutach

## **Techniki synchronizacji portfeli**

Aby zachować prywatność i bezpieczeństwo, synchronizacja portfeli z blockchainem jest kluczowa. Dwie metody wyróżniają się:

- **Pełny węzeł**: Pobierając cały blockchain, pełny węzeł zapewnia maksymalną prywatność. Wszystkie transakcje kiedykolwiek dokonane są przechowywane lokalnie, co uniemożliwia przeciwnikom zidentyfikowanie, które transakcje lub adresy interesują użytkownika.
- **Filtrowanie bloków po stronie klienta**: Ta metoda polega na tworzeniu filtrów dla każdego bloku w blockchainie, co pozwala portfelom identyfikować odpowiednie transakcje bez ujawniania konkretnych zainteresowań obserwatorom sieci. Lekkie portfele pobierają te filtry, ściągając pełne bloki tylko wtedy, gdy znajdą dopasowanie z adresami użytkownika.

## **Wykorzystanie Tora dla anonimowości**

Biorąc pod uwagę, że Bitcoin działa w sieci peer-to-peer, zaleca się korzystanie z Tora, aby ukryć swój adres IP, co zwiększa prywatność podczas interakcji z siecią.

## **Zapobieganie ponownemu używaniu adresów**

Aby chronić prywatność, ważne jest, aby używać nowego adresu dla każdej transakcji. Ponowne używanie adresów może narazić prywatność, łącząc transakcje z tą samą jednostką. Nowoczesne portfele zniechęcają do ponownego używania adresów poprzez swój design.

## **Strategie dla prywatności transakcji**

- **Wiele transakcji**: Podzielenie płatności na kilka transakcji może zaciemnić kwotę transakcji, utrudniając ataki na prywatność.
- **Unikanie reszty**: Wybieranie transakcji, które nie wymagają zwrotu reszty, zwiększa prywatność, zakłócając metody wykrywania reszty.
- **Wiele zwrotów reszty**: Jeśli unikanie reszty nie jest możliwe, generowanie wielu zwrotów reszty może nadal poprawić prywatność.

# **Monero: Latarnia anonimowości**

Monero odpowiada na potrzebę absolutnej anonimowości w transakcjach cyfrowych, ustanawiając wysoki standard prywatności.

# **Ethereum: Gaz i transakcje**

## **Zrozumienie gazu**

Gaz mierzy wysiłek obliczeniowy potrzebny do wykonania operacji na Ethereum, wyceniany w **gwei**. Na przykład, transakcja kosztująca 2,310,000 gwei (lub 0.00231 ETH) wiąże się z limitem gazu i opłatą podstawową, z napiwkiem dla zachęcenia górników. Użytkownicy mogą ustawić maksymalną opłatę, aby upewnić się, że nie przepłacają, a nadwyżka jest zwracana.

## **Wykonywanie transakcji**

Transakcje w Ethereum obejmują nadawcę i odbiorcę, którymi mogą być adresy użytkowników lub smart kontraktów. Wymagają one opłaty i muszą być wydobywane. Kluczowe informacje w transakcji obejmują odbiorcę, podpis nadawcy, wartość, opcjonalne dane, limit gazu i opłaty. Co ważne, adres nadawcy jest dedukowany z podpisu, eliminując potrzebę jego umieszczania w danych transakcji.

Te praktyki i mechanizmy są podstawowe dla każdego, kto chce zaangażować się w kryptowaluty, priorytetowo traktując prywatność i bezpieczeństwo.


## Źródła

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

<summary>Wsparcie HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się trikami hakerskimi, przesyłając PR-y do repozytoriów** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
