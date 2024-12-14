{% hint style="success" %}
Ucz się i ćwicz AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na GitHubie.

</details>
{% endhint %}


<a rel="license" href="https://creativecommons.org/licenses/by-nc/4.0/"><img alt="Licencja Creative Commons" style="border-width:0" src="https://licensebuttons.net/l/by-nc/4.0/88x31.png" /></a><br>Prawa autorskie © Carlos Polop 2021.  Z wyjątkiem miejsc, gdzie wskazano inaczej (informacje zewnętrzne skopiowane do książki należą do oryginalnych autorów), tekst na <a href="https://github.com/carlospolop/hacktricks">HACK TRICKS</a> autorstwa Carlosa Polopa jest licencjonowany na podstawie <a href="https://creativecommons.org/licenses/by-nc/4.0/">licencji Creative Commons Uznanie autorstwa - Użycie niekomercyjne 4.0 Międzynarodowej (CC BY-NC 4.0)</a>.

Licencja: Uznanie autorstwa - Użycie niekomercyjne 4.0 Międzynarodowej (CC BY-NC 4.0)<br>
Ludzka wersja licencji: https://creativecommons.org/licenses/by-nc/4.0/<br>
Pełne warunki prawne: https://creativecommons.org/licenses/by-nc/4.0/legalcode<br>
Formatowanie: https://github.com/jmatsushita/Creative-Commons-4.0-Markdown/blob/master/licenses/by-nc.markdown<br>

# creative commons

# Uznanie autorstwa - Użycie niekomercyjne 4.0 Międzynarodowe

Creative Commons Corporation (“Creative Commons”) nie jest kancelarią prawną i nie świadczy usług prawnych ani porad prawnych. Dystrybucja publicznych licencji Creative Commons nie tworzy relacji adwokat-klient ani innej relacji. Creative Commons udostępnia swoje licencje i powiązane informacje na zasadzie "tak jak jest". Creative Commons nie udziela żadnych gwarancji dotyczących swoich licencji, jakiegokolwiek materiału licencjonowanego na podstawie ich warunków oraz jakichkolwiek powiązanych informacji. Creative Commons zrzeka się wszelkiej odpowiedzialności za szkody wynikające z ich użycia w najszerszym możliwym zakresie.

## Używanie publicznych licencji Creative Commons

Publiczne licencje Creative Commons zapewniają standardowy zestaw warunków, które twórcy i inni posiadacze praw mogą wykorzystać do dzielenia się oryginalnymi dziełami autorskimi oraz innymi materiałami objętymi prawem autorskim i innymi określonymi prawami wskazanymi w publicznej licencji poniżej. Poniższe rozważania mają charakter informacyjny, nie są wyczerpujące i nie stanowią części naszych licencji.

* __Rozważania dla licencjodawców:__ Nasze publiczne licencje są przeznaczone do użytku przez osoby upoważnione do udzielania publiczności zgody na korzystanie z materiałów w sposób, który w inny sposób byłby ograniczony przez prawo autorskie i inne określone prawa. Nasze licencje są nieodwołalne. Licencjodawcy powinni przeczytać i zrozumieć warunki licencji, którą wybierają, przed jej zastosowaniem. Licencjodawcy powinni również zabezpieczyć wszystkie niezbędne prawa przed zastosowaniem naszych licencji, aby publiczność mogła ponownie wykorzystać materiał zgodnie z oczekiwaniami. Licencjodawcy powinni wyraźnie oznaczyć wszelkie materiały, które nie podlegają licencji. Obejmuje to inne materiały licencjonowane na podstawie CC lub materiały używane na podstawie wyjątku lub ograniczenia do prawa autorskiego. [Więcej rozważań dla licencjodawców](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensors).

* __Rozważania dla publiczności:__ Korzystając z jednej z naszych publicznych licencji, licencjodawca udziela publiczności zgody na korzystanie z licencjonowanego materiału na określonych warunkach. Jeśli zgoda licencjodawcy nie jest konieczna z jakiegokolwiek powodu – na przykład z powodu jakiegokolwiek stosownego wyjątku lub ograniczenia do prawa autorskiego – to takie użycie nie jest regulowane przez licencję. Nasze licencje przyznają jedynie uprawnienia w ramach prawa autorskiego i innych praw, które licencjodawca ma prawo przyznać. Użycie licencjonowanego materiału może być nadal ograniczone z innych powodów, w tym dlatego, że inni mają prawa autorskie lub inne prawa do materiału. Licencjodawca może zgłaszać specjalne prośby, takie jak prośba o oznaczenie lub opisanie wszystkich zmian. Chociaż nie jest to wymagane przez nasze licencje, zachęcamy do poszanowania tych próśb, gdzie to rozsądne. [Więcej rozważań dla publiczności](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensees).

# Publiczna Licencja Creative Commons Uznanie autorstwa - Użycie niekomercyjne 4.0 Międzynarodowa

Korzystając z Praw Licencjonowanych (zdefiniowanych poniżej), akceptujesz i zgadzasz się na przestrzeganie warunków tej Publicznej Licencji Creative Commons Uznanie autorstwa - Użycie niekomercyjne 4.0 Międzynarodowej ("Publiczna Licencja"). W zakresie, w jakim ta Publiczna Licencja może być interpretowana jako umowa, przyznaje się Tobie Prawa Licencjonowane w zamian za Twoją akceptację tych warunków, a Licencjodawca przyznaje Ci takie prawa w zamian za korzyści, jakie Licencjodawca otrzymuje z udostępnienia Materiału Licencjonowanego na tych warunkach.

## Sekcja 1 – Definicje.

a. __Materiał Adaptowany__ oznacza materiał objęty prawem autorskim i podobnymi prawami, który pochodzi z Materiału Licencjonowanego lub jest na nim oparty i w którym Materiał Licencjonowany jest tłumaczony, zmieniany, aranżowany, przekształcany lub w inny sposób modyfikowany w sposób wymagający zgody na podstawie praw autorskich i podobnych praw posiadanych przez Licencjodawcę. Na potrzeby tej Publicznej Licencji, gdy Materiał Licencjonowany jest dziełem muzycznym, występem lub nagraniem dźwiękowym, Materiał Adaptowany zawsze powstaje, gdy Materiał Licencjonowany jest synchronizowany w czasowym związku z obrazem w ruchu.

b. __Licencja Adaptatora__ oznacza licencję, którą stosujesz do swoich praw autorskich i podobnych praw w swoich wkładach do Materiału Adaptowanego zgodnie z warunkami tej Publicznej Licencji.

c. __Prawa autorskie i podobne prawa__ oznaczają prawa autorskie i/lub podobne prawa ściśle związane z prawem autorskim, w tym, bez ograniczeń, prawa do występu, nadawania, nagrania dźwiękowego oraz prawa do baz danych Sui Generis, bez względu na to, jak prawa są oznaczane lub klasyfikowane. Na potrzeby tej Publicznej Licencji, prawa określone w Sekcji 2(b)(1)-(2) nie są Prawami Autorskimi i Podobnymi Prawami.

d. __Skuteczne Środki Technologiczne__ oznaczają te środki, które, w przypadku braku odpowiedniej władzy, nie mogą być omijane na podstawie przepisów wypełniających zobowiązania na podstawie Artykułu 11 Traktatu WIPO o prawie autorskim przyjętego 20 grudnia 1996 roku i/lub podobnych międzynarodowych umów.

e. __Wyjątki i Ograniczenia__ oznaczają dozwolony użytek, dozwolone korzystanie i/lub jakiekolwiek inne wyjątki lub ograniczenia do Prawa Autorskiego i Podobnych Praw, które mają zastosowanie do Twojego użycia Materiału Licencjonowanego.

f. __Materiał Licencjonowany__ oznacza dzieło artystyczne lub literackie, bazę danych lub inny materiał, do którego Licencjodawca zastosował tę Publiczną Licencję.

g. __Prawa Licencjonowane__ oznaczają prawa przyznane Tobie na podstawie warunków tej Publicznej Licencji, które są ograniczone do wszystkich Praw Autorskich i Podobnych Praw, które mają zastosowanie do Twojego użycia Materiału Licencjonowanego i które Licencjodawca ma prawo licencjonować.

h. __Licencjodawca__ oznacza osobę/osoby lub podmiot/ty, które przyznają prawa na podstawie tej Publicznej Licencji.

i. __Niekomercyjny__ oznacza, że nie jest głównie przeznaczony ani skierowany na korzyść komercyjną lub wynagrodzenie pieniężne. Na potrzeby tej Publicznej Licencji wymiana Materiału Licencjonowanego na inny materiał objęty Prawem Autorskim i Podobnymi Prawami za pomocą cyfrowego udostępniania plików lub podobnych środków jest Niekomercyjna, pod warunkiem, że nie ma płatności wynagrodzenia pieniężnego w związku z wymianą.

j. __Udostępniać__ oznacza dostarczanie materiału publiczności wszelkimi środkami lub procesami, które wymagają zgody na podstawie Praw Licencjonowanych, takimi jak reprodukcja, publiczne wyświetlanie, publiczne występy, dystrybucja, rozpowszechnianie, komunikacja lub import, oraz udostępnianie materiału publiczności, w tym w sposób, który umożliwia członkom publiczności dostęp do materiału z miejsca i w czasie przez nich indywidualnie wybranym.

k. __Prawa do baz danych Sui Generis__ oznaczają prawa inne niż prawa autorskie wynikające z Dyrektywy 96/9/WE Parlamentu Europejskiego i Rady z dnia 11 marca 1996 roku w sprawie prawnej ochrony baz danych, w zmienionym brzmieniu i/lub w następstwie, a także inne zasadniczo równoważne prawa w dowolnym miejscu na świecie.

l. __Ty__ oznacza osobę lub podmiot wykonujący Prawa Licencjonowane na podstawie tej Publicznej Licencji. Twoje ma odpowiadające znaczenie.

## Sekcja 2 – Zakres.

a. ___Przyznanie licencji.___

1. Z zastrzeżeniem warunków tej Publicznej Licencji, Licencjodawca niniejszym przyznaje Tobie na całym świecie, bezpłatną, niepodlicencjonowalną, niewyłączną, nieodwołalną licencję na wykonywanie Praw Licencjonowanych w Materiale Licencjonowanym w celu:

A. reprodukcji i Udostępniania Materiału Licencjonowanego, w całości lub w części, wyłącznie w celach Niekomercyjnych; oraz

B. produkcji, reprodukcji i Udostępniania Materiału Adaptowanego wyłącznie w celach Niekomercyjnych.

2. __Wyjątki i Ograniczenia.__ Dla uniknięcia wątpliwości, gdzie Wyjątki i Ograniczenia mają zastosowanie do Twojego użycia, ta Publiczna Licencja nie ma zastosowania, a Ty nie musisz przestrzegać jej warunków.

3. __Okres obowiązywania.__ Okres obowiązywania tej Publicznej Licencji określony jest w Sekcji 6(a).

4. __Media i formaty; dozwolone modyfikacje techniczne.__ Licencjodawca upoważnia Cię do wykonywania Praw Licencjonowanych we wszystkich mediach i formatach, które są obecnie znane lub będą stworzone w przyszłości, oraz do dokonywania niezbędnych modyfikacji technicznych w tym celu. Licencjodawca zrzeka się i/lub zgadza się nie twierdzić, że ma prawo lub władzę zabraniać Ci dokonywania niezbędnych modyfikacji technicznych w celu wykonywania Praw Licencjonowanych, w tym modyfikacji technicznych niezbędnych do ominięcia Skutecznych Środków Technologicznych. Na potrzeby tej Publicznej Licencji, dokonanie modyfikacji dozwolonych na podstawie tej Sekcji 2(a)(4) nigdy nie produkuje Materiału Adaptowanego.

5. __Odbiorcy dalsi.__

A. __Oferta od Licencjodawcy – Materiał Licencjonowany.__ Każdy odbiorca Materiału Licencjonowanego automatycznie otrzymuje ofertę od Licencjodawcy na wykonywanie Praw Licencjonowanych na warunkach tej Publicznej Licencji.

B. __Brak ograniczeń dalszych.__ Nie możesz oferować ani nakładać żadnych dodatkowych lub innych warunków na Materiał Licencjonowany, ani stosować żadnych Skutecznych Środków Technologicznych do Materiału Licencjonowanego, jeśli ogranicza to wykonywanie Praw Licencjonowanych przez jakiegokolwiek odbiorcę Materiału Licencjonowanego.

6. __Brak poparcia.__ Nic w tej Publicznej Licencji nie stanowi ani nie może być interpretowane jako zgoda na twierdzenie lub sugerowanie, że jesteś, lub że Twoje użycie Materiału Licencjonowanego jest, związane z, lub sponsorowane, popierane, lub przyznane oficjalny status przez Licencjodawcę lub innych wyznaczonych do otrzymania uznania, jak przewidziano w Sekcji 3(a)(1)(A)(i).

b. ___Inne prawa.___

1. Prawa moralne, takie jak prawo do integralności, nie są licencjonowane na podstawie tej Publicznej Licencji, ani prawa do reklamy, prywatności i/lub inne podobne prawa osobiste; jednak w miarę możliwości Licencjodawca zrzeka się i/lub zgadza się nie twierdzić, że ma jakiekolwiek takie prawa, które są w posiadaniu Licencjodawcy w ograniczonym zakresie niezbędnym do umożliwienia Ci wykonywania Praw Licencjonowanych, ale nie w innym przypadku.

2. Prawa patentowe i prawa do znaków towarowych nie są licencjonowane na podstawie tej Publicznej Licencji.

3. W miarę możliwości Licencjodawca zrzeka się wszelkich praw do pobierania tantiem od Ciebie za wykonywanie Praw Licencjonowanych, czy to bezpośrednio, czy poprzez organizację zbiorowego zarządzania na podstawie jakiegokolwiek dobrowolnego lub zrzeczonego ustawowego lub obowiązkowego systemu licencjonowania. We wszystkich innych przypadkach Licencjodawca wyraźnie zastrzega sobie wszelkie prawa do pobierania takich tantiem, w tym gdy Materiał Licencjonowany jest używany w sposób inny niż w celach Niekomercyjnych.

## Sekcja 3 – Warunki licencji.

Twoje wykonywanie Praw Licencjonowanych jest wyraźnie uzależnione od następujących warunków.

a. ___Uznanie.___

1. Jeśli Udostępniasz Materiał Licencjonowany (w tym w zmienionej formie), musisz:

A. zachować następujące, jeśli zostało dostarczone przez Licencjodawcę z Materiałem Licencjonowanym:

i. identyfikacja twórcy/twórców Materiału Licencjonowanego i wszelkich innych wyznaczonych do otrzymania uznania, w sposób rozsądny wymagany przez Licencjodawcę (w tym przez pseudonim, jeśli wyznaczony);

ii. informacja o prawach autorskich;

iii. informacja odnosząca się do tej Publicznej Licencji;

iv. informacja odnosząca się do zrzeczenia się gwarancji;

v. URI lub hiperłącze do Materiału Licencjonowanego w miarę rozsądnej wykonalności;

B. wskazać, czy zmodyfikowałeś Materiał Licencjonowany i zachować wskazanie wszelkich wcześniejszych modyfikacji; oraz

C. wskazać, że Materiał Licencjonowany jest licencjonowany na podstawie tej Publicznej Licencji, i dołączyć tekst tej Publicznej Licencji lub URI lub hiperłącze do tej Publicznej Licencji.

2. Możesz spełnić warunki w Sekcji 3(a)(1) w sposób rozsądny, w zależności od medium, środków i kontekstu, w którym Udostępniasz Materiał Licencjonowany. Na przykład, może być rozsądne spełnienie warunków poprzez podanie URI lub hiperłącza do zasobu, który zawiera wymagane informacje.

3. Na żądanie Licencjodawcy musisz usunąć wszelkie informacje wymagane przez Sekcję 3(a)(1)(A) w miarę rozsądnej wykonalności.

4. Jeśli Udostępniasz Materiał Adaptowany, który produkujesz, Licencja Adaptatora, którą stosujesz, nie może uniemożliwić odbiorcom Materiału Adaptowanego przestrzegania tej Publicznej Licencji.

## Sekcja 4 – Prawa do baz danych Sui Generis.

Gdy Prawa Licencjonowane obejmują Prawa do baz danych Sui Generis, które mają zastosowanie do Twojego użycia Materiału Licencjonowanego:

a. dla uniknięcia wątpliwości, Sekcja 2(a)(1) przyznaje Ci prawo do wydobywania, ponownego używania, reprodukcji i Udostępniania całości lub znacznej części zawartości bazy danych wyłącznie w celach Niekomercyjnych;

b. jeśli uwzględniasz całość lub znaczna część zawartości bazy danych w bazie danych, w której masz Prawa do baz danych Sui Generis, to baza danych, w której masz Prawa do baz danych Sui Generis (ale nie jej poszczególne zawartości) jest Materiałem Adaptowanym; oraz

c. musisz przestrzegać warunków w Sekcji 3(a), jeśli Udostępniasz całość lub znaczna część zawartości bazy danych.

Dla uniknięcia wątpliwości, ta Sekcja 4 uzupełnia i nie zastępuje Twoich zobowiązań na podstawie tej Publicznej Licencji, gdy Prawa Licencjonowane obejmują inne Prawa Autorskie i Podobne Prawa.

## Sekcja 5 – Zrzeczenie się gwarancji i ograniczenie odpowiedzialności.

a. __O ile nie zostało to oddzielnie podjęte przez Licencjodawcę, w miarę możliwości Licencjodawca oferuje Materiał Licencjonowany w stanie "tak jak jest" i "w miarę dostępności", i nie składa żadnych oświadczeń ani gwarancji jakiegokolwiek rodzaju dotyczących Materiału Licencjonowanego, czy to wyraźnych, dorozumianych, ustawowych, czy innych. Obejmuje to, bez ograniczeń, gwarancje tytułu, przydatności handlowej, przydatności do określonego celu, braku naruszenia, braku ukrytych lub innych wad, dokładności, czy obecności lub braku błędów, niezależnie od tego, czy są znane, czy odkrywalne. Gdzie zrzeczenia się gwarancji nie są dozwolone w całości lub w części, to zrzeczenie się może nie mieć zastosowania do Ciebie.__

b. __W miarę możliwości, w żadnym przypadku Licencjodawca nie będzie odpowiedzialny wobec Ciebie na jakiejkolwiek podstawie prawnej (w tym, bez ograniczeń, z tytułu niedbalstwa) ani w inny sposób za jakiekolwiek bezpośrednie, szczególne, pośrednie, przypadkowe, wynikowe, karne, wzorcowe lub inne straty, koszty, wydatki lub szkody wynikające z tej Publicznej Licencji lub użycia Materiału Licencjonowanego, nawet jeśli Licencjodawca został poinformowany o możliwości takich strat, kosztów, wydatków lub szkód. Gdzie ograniczenie odpowiedzialności nie jest dozwolone w całości lub w części, to ograniczenie może nie mieć zastosowania do Ciebie.__

c. Zrzeczenie się gwarancji i ograniczenie odpowiedzialności przewidziane powyżej będzie interpretowane w sposób, który, w miarę możliwości, najbliżej przybliża całkowite zrzeczenie się i zrzeczenie się wszelkiej odpowiedzialności.

## Sekcja 6 – Okres obowiązywania i rozwiązanie.

a. Ta Publiczna Licencja ma zastosowanie przez okres obowiązywania Praw Autorskich i Podobnych Praw licencjonowanych tutaj. Jednakże, jeśli nie przestrzegasz tej Publicznej Licencji, Twoje prawa na podstawie tej Publicznej Licencji wygasają automatycznie.

b. Gdy Twoje prawo do korzystania z Materiału Licencjonowanego wygasło na podstawie Sekcji 6(a), przywraca się:

1. automatycznie od daty, w której naruszenie zostało naprawione, pod warunkiem, że zostanie naprawione w ciągu 30 dni od odkrycia naruszenia; lub

2. na wyraźne przywrócenie przez Licencjodawcę.

Dla uniknięcia wątpliwości, ta Sekcja 6(b) nie wpływa na jakiekolwiek prawo, które Licencjodawca może mieć do dochodzenia roszczeń za Twoje naruszenia tej Publicznej Licencji.

c. Dla uniknięcia wątpliwości, Licencjodawca może również oferować Materiał Licencjonowany na oddzielnych warunkach lub zaprzestać dystrybucji Materiału Licencjonowanego w dowolnym momencie; jednakże, czynienie tego nie zakończy tej Publicznej Licencji.

d. Sekcje 1, 5, 6, 7 i 8 przetrwają rozwiązanie tej Publicznej Licencji.

## Sekcja 7 – Inne warunki i zasady.

a. Licencjodawca nie będzie związany żadnymi dodatkowymi lub innymi warunkami komunikowanymi przez Ciebie, chyba że wyraźnie uzgodnione.

b. Jakiekolwiek ustalenia, porozumienia lub umowy dotyczące Materiału Licencjonowanego, które nie są tutaj wymienione, są oddzielne i niezależne od warunków tej Publicznej Licencji.

## Sekcja 8 – Interpretacja.

a. Dla uniknięcia wątpliwości, ta Publiczna Licencja nie zmniejsza, nie ogranicza, nie restrykcjonuje ani nie nakłada warunków na jakiekolwiek użycie Materiału Licencjonowanego, które mogłoby być legalnie dokonane bez zgody na podstawie tej Publicznej Licencji.

b. W miarę możliwości, jeśli jakiekolwiek postanowienie tej Publicznej Licencji uznane zostanie za niewykonalne, zostanie automatycznie zmienione w minimalnym zakresie niezbędnym do uczynienia go wykonalnym. Jeśli postanowienie nie może być zmienione, zostanie ono oddzielone od tej Publicznej Licencji bez wpływu na wykonalność pozostałych warunków.

c. Żaden warunek tej Publicznej Licencji nie będzie zrzeczony, a żadne niedopełnienie nie będzie zaakceptowane, chyba że wyraźnie uzgodnione przez Licencjodawcę.

d. Nic w tej Publicznej Licencji nie stanowi ani nie może być interpretowane jako ograniczenie lub zrzeczenie się jakichkolwiek przywilejów i immunitetów, które mają zastosowanie do Licencjodawcy lub Ciebie, w tym od procesów prawnych jakiejkolwiek jurysdykcji lub władzy.
```
Creative Commons is not a party to its public licenses. Notwithstanding, Creative Commons may elect to apply one of its public licenses to material it publishes and in those instances will be considered the “Licensor.” Except for the limited purpose of indicating that material is shared under a Creative Commons public license or as otherwise permitted by the Creative Commons policies published at [creativecommons.org/policies](http://creativecommons.org/policies), Creative Commons does not authorize the use of the trademark “Creative Commons” or any other trademark or logo of Creative Commons without its prior written consent including, without limitation, in connection with any unauthorized modifications to any of its public licenses or any other arrangements, understandings, or agreements concerning use of licensed material. For the avoidance of doubt, this paragraph does not form part of the public licenses.

Creative Commons may be contacted at [creativecommons.org](http://creativecommons.org/).
```
{% hint style="success" %}
Ucz się i ćwicz AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na githubie.

</details>
{% endhint %}
