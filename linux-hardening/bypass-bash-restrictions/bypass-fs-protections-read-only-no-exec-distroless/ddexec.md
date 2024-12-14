# DDexec / EverythingExec

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

## Kontekst

W systemie Linux, aby uruchomić program, musi on istnieć jako plik, musi być w jakiś sposób dostępny w hierarchii systemu plików (tak działa `execve()`). Plik ten może znajdować się na dysku lub w pamięci RAM (tmpfs, memfd), ale potrzebujesz ścieżki do pliku. To bardzo ułatwiło kontrolowanie tego, co jest uruchamiane w systemie Linux, co ułatwia wykrywanie zagrożeń i narzędzi atakujących lub zapobieganie ich próbom uruchomienia czegokolwiek (_np._ nie pozwalając użytkownikom bez uprawnień na umieszczanie plików wykonywalnych gdziekolwiek).

Ale ta technika ma na celu zmianę tego wszystkiego. Jeśli nie możesz uruchomić procesu, którego chcesz... **to przejmujesz już istniejący**.

Ta technika pozwala na **obejście powszechnych technik ochrony, takich jak tylko do odczytu, noexec, biała lista nazw plików, biała lista hashy...**

## Zależności

Ostateczny skrypt zależy od następujących narzędzi, które muszą być dostępne w systemie, który atakujesz (domyślnie znajdziesz je wszędzie):
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## Technika

Jeśli jesteś w stanie dowolnie modyfikować pamięć procesu, możesz go przejąć. Może to być użyte do przejęcia już istniejącego procesu i zastąpienia go innym programem. Możemy to osiągnąć, używając wywołania systemowego `ptrace()` (co wymaga posiadania możliwości wykonywania wywołań systemowych lub dostępności gdb w systemie) lub, co bardziej interesujące, pisząc do `/proc/$pid/mem`.

Plik `/proc/$pid/mem` jest mapowaniem jeden do jednego całej przestrzeni adresowej procesu (_np._ od `0x0000000000000000` do `0x7ffffffffffff000` w x86-64). Oznacza to, że odczyt lub zapis do tego pliku w przesunięciu `x` jest tym samym, co odczyt lub modyfikacja zawartości pod adresem wirtualnym `x`.

Teraz mamy cztery podstawowe problemy do rozwiązania:

* Generalnie tylko root i właściciel programu pliku mogą go modyfikować.
* ASLR.
* Jeśli spróbujemy odczytać lub zapisać do adresu, który nie jest mapowany w przestrzeni adresowej programu, otrzymamy błąd I/O.

Te problemy mają rozwiązania, które, chociaż nie są doskonałe, są dobre:

* Większość interpreterów powłok pozwala na tworzenie deskryptorów plików, które będą dziedziczone przez procesy potomne. Możemy stworzyć fd wskazujący na plik `mem` powłoki z uprawnieniami do zapisu... więc procesy potomne, które używają tego fd, będą mogły modyfikować pamięć powłoki.
* ASLR nie jest nawet problemem, możemy sprawdzić plik `maps` powłoki lub jakikolwiek inny z procfs, aby uzyskać informacje o przestrzeni adresowej procesu.
* Musimy więc użyć `lseek()` na pliku. Z powłoki nie można tego zrobić, chyba że używając infamnego `dd`.

### W większych szczegółach

Kroki są stosunkowo łatwe i nie wymagają żadnego rodzaju ekspertyzy, aby je zrozumieć:

* Przeanalizuj binarny plik, który chcemy uruchomić, oraz loader, aby dowiedzieć się, jakie mapowania są potrzebne. Następnie stwórz "shell" kod, który będzie wykonywał, ogólnie mówiąc, te same kroki, które jądro wykonuje przy każdym wywołaniu `execve()`:
* Utwórz wspomniane mapowania.
* Odczytaj binaria do nich.
* Ustaw uprawnienia.
* Na koniec zainicjuj stos z argumentami dla programu i umieść wektor pomocniczy (potrzebny loaderowi).
* Skocz do loadera i pozwól mu zrobić resztę (załaduj biblioteki potrzebne programowi).
* Uzyskaj z pliku `syscall` adres, do którego proces wróci po wywołaniu systemowym, które wykonuje.
* Nadpisz to miejsce, które będzie wykonywalne, naszym shellcode (poprzez `mem` możemy modyfikować strony, które nie mogą być zapisywane).
* Przekaż program, który chcemy uruchomić, do stdin procesu (zostanie `read()` przez wspomniany "shell" kod).
* W tym momencie to loader jest odpowiedzialny za załadowanie niezbędnych bibliotek dla naszego programu i skok do niego.

**Sprawdź narzędzie w** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

Istnieje kilka alternatyw dla `dd`, z których jedna, `tail`, jest obecnie domyślnym programem używanym do `lseek()` przez plik `mem` (co było jedynym celem użycia `dd`). Wspomniane alternatywy to:
```bash
tail
hexdump
cmp
xxd
```
Ustawiając zmienną `SEEKER`, możesz zmienić używanego poszukiwacza, _np._:
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Jeśli znajdziesz inny ważny seeker, który nie został zaimplementowany w skrypcie, możesz go nadal użyć, ustawiając zmienną `SEEKER_ARGS`:
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Zablokuj to, EDRs.

## References
* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
