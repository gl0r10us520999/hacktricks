# macOS Process Abuse

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

## Podstawowe informacje o procesach

Proces to instancja uruchamianego pliku wykonywalnego, jednak procesy nie wykonują kodu, to wątki. Dlatego **procesy są tylko kontenerami dla uruchamianych wątków**, zapewniając pamięć, deskryptory, porty, uprawnienia...

Tradycyjnie procesy były uruchamiane w ramach innych procesów (z wyjątkiem PID 1) poprzez wywołanie **`fork`**, co tworzyło dokładną kopię bieżącego procesu, a następnie **proces potomny** zazwyczaj wywoływał **`execve`**, aby załadować nowy plik wykonywalny i go uruchomić. Następnie wprowadzono **`vfork`**, aby przyspieszyć ten proces bez kopiowania pamięci.\
Następnie wprowadzono **`posix_spawn`**, łącząc **`vfork`** i **`execve`** w jednym wywołaniu i akceptując flagi:

* `POSIX_SPAWN_RESETIDS`: Resetuj efektywne identyfikatory do rzeczywistych identyfikatorów
* `POSIX_SPAWN_SETPGROUP`: Ustaw przynależność do grupy procesów
* `POSUX_SPAWN_SETSIGDEF`: Ustaw domyślne zachowanie sygnałów
* `POSIX_SPAWN_SETSIGMASK`: Ustaw maskę sygnałów
* `POSIX_SPAWN_SETEXEC`: Wykonaj w tym samym procesie (jak `execve` z dodatkowymi opcjami)
* `POSIX_SPAWN_START_SUSPENDED`: Rozpocznij wstrzymany
* `_POSIX_SPAWN_DISABLE_ASLR`: Rozpocznij bez ASLR
* `_POSIX_SPAWN_NANO_ALLOCATOR:` Użyj alokatora Nano z libmalloc
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Zezwól na `rwx` w segmentach danych
* `POSIX_SPAWN_CLOEXEC_DEFAULT`: Domyślnie zamknij wszystkie opisy plików przy exec(2)
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` Randomizuj wysokie bity przesunięcia ASLR

Ponadto `posix_spawn` pozwala określić tablicę **`posix_spawnattr`**, która kontroluje niektóre aspekty uruchamianego procesu, oraz **`posix_spawn_file_actions`**, aby zmodyfikować stan deskryptorów.

Gdy proces umiera, wysyła **kod zwrotu do procesu macierzystego** (jeśli proces macierzysty umarł, nowym procesem macierzystym jest PID 1) z sygnałem `SIGCHLD`. Proces macierzysty musi uzyskać tę wartość, wywołując `wait4()` lub `waitid()`, a do tego czasu proces potomny pozostaje w stanie zombie, gdzie nadal jest wymieniany, ale nie zużywa zasobów.

### PIDs

PID, identyfikatory procesów, identyfikuje unikalny proces. W XNU **PIDs** mają **64 bity**, rosną monotonnie i **nigdy się nie zawijają** (aby uniknąć nadużyć).

### Grupy procesów, sesje i koalicje

**Procesy** mogą być wstawiane do **grup**, aby ułatwić ich obsługę. Na przykład, polecenia w skrypcie powłoki będą w tej samej grupie procesów, więc możliwe jest **sygnalizowanie ich razem** za pomocą kill na przykład.\
Możliwe jest również **grupowanie procesów w sesje**. Gdy proces rozpoczyna sesję (`setsid(2)`), procesy potomne są umieszczane w sesji, chyba że rozpoczną własną sesję.

Koalicja to inny sposób grupowania procesów w Darwin. Proces dołączający do koalicji pozwala mu uzyskać dostęp do zasobów puli, dzieląc się księgą lub stawiając czoła Jetsam. Koalicje mają różne role: Lider, usługa XPC, Rozszerzenie.

### Uprawnienia i personae

Każdy proces posiada **uprawnienia**, które **identyfikują jego przywileje** w systemie. Każdy proces będzie miał jeden główny `uid` i jeden główny `gid` (chociaż może należeć do kilku grup).\
Możliwe jest również zmienienie identyfikatora użytkownika i grupy, jeśli binarka ma bit `setuid/setgid`.\
Istnieje kilka funkcji do **ustawiania nowych uids/gids**.

Wywołanie systemowe **`persona`** zapewnia **alternatywny** zestaw **uprawnień**. Przyjęcie persony zakłada jej uid, gid i przynależności grupowe **jednocześnie**. W [**kodzie źródłowym**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h) można znaleźć strukturę:
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## Podstawowe informacje o wątkach

1. **Wątki POSIX (pthreads):** macOS obsługuje wątki POSIX (`pthreads`), które są częścią standardowego API wątków dla C/C++. Implementacja pthreads w macOS znajduje się w `/usr/lib/system/libsystem_pthread.dylib`, która pochodzi z publicznie dostępnego projektu `libpthread`. Ta biblioteka zapewnia niezbędne funkcje do tworzenia i zarządzania wątkami.
2. **Tworzenie wątków:** Funkcja `pthread_create()` jest używana do tworzenia nowych wątków. Wewnątrz ta funkcja wywołuje `bsdthread_create()`, która jest wywołaniem systemowym na niższym poziomie specyficznym dla jądra XNU (na którym oparty jest macOS). To wywołanie systemowe przyjmuje różne flagi pochodzące z `pthread_attr` (atrybuty), które określają zachowanie wątku, w tym polityki planowania i rozmiar stosu.
* **Domyślny rozmiar stosu:** Domyślny rozmiar stosu dla nowych wątków wynosi 512 KB, co jest wystarczające dla typowych operacji, ale może być dostosowane za pomocą atrybutów wątku, jeśli potrzebna jest większa lub mniejsza przestrzeń.
3. **Inicjalizacja wątku:** Funkcja `__pthread_init()` jest kluczowa podczas konfiguracji wątku, wykorzystując argument `env[]` do analizy zmiennych środowiskowych, które mogą zawierać szczegóły dotyczące lokalizacji i rozmiaru stosu.

#### Zakończenie wątków w macOS

1. **Zakończenie wątków:** Wątki są zazwyczaj kończone przez wywołanie `pthread_exit()`. Ta funkcja pozwala wątkowi na czyste zakończenie, wykonując niezbędne czynności porządkowe i umożliwiając wątkowi przesłanie wartości zwrotnej do wszelkich oczekujących.
2. **Czyszczenie wątku:** Po wywołaniu `pthread_exit()`, wywoływana jest funkcja `pthread_terminate()`, która zajmuje się usunięciem wszystkich powiązanych struktur wątku. Zwalnia porty wątków Mach (Mach to podsystem komunikacyjny w jądrze XNU) i wywołuje `bsdthread_terminate`, wywołanie systemowe, które usuwa struktury na poziomie jądra związane z wątkiem.

#### Mechanizmy synchronizacji

Aby zarządzać dostępem do wspólnych zasobów i unikać warunków wyścigu, macOS zapewnia kilka prymitywów synchronizacji. Są one kluczowe w środowiskach wielowątkowych, aby zapewnić integralność danych i stabilność systemu:

1. **Mutexy:**
* **Zwykły mutex (Podpis: 0x4D555458):** Standardowy mutex o rozmiarze pamięci 60 bajtów (56 bajtów dla mutexa i 4 bajty dla podpisu).
* **Szybki mutex (Podpis: 0x4d55545A):** Podobny do zwykłego mutexa, ale zoptymalizowany do szybszych operacji, również 60 bajtów.
2. **Zmienne warunkowe:**
* Używane do oczekiwania na wystąpienie określonych warunków, o rozmiarze 44 bajtów (40 bajtów plus 4-bajtowy podpis).
* **Atrybuty zmiennych warunkowych (Podpis: 0x434e4441):** Atrybuty konfiguracyjne dla zmiennych warunkowych, o rozmiarze 12 bajtów.
3. **Zmienna Once (Podpis: 0x4f4e4345):**
* Zapewnia, że fragment kodu inicjalizacyjnego jest wykonywany tylko raz. Jej rozmiar wynosi 12 bajtów.
4. **Blokady do odczytu i zapisu:**
* Umożliwiają jednoczesny dostęp wielu czytelników lub jednego pisarza, ułatwiając efektywny dostęp do wspólnych danych.
* **Blokada do odczytu i zapisu (Podpis: 0x52574c4b):** O rozmiarze 196 bajtów.
* **Atrybuty blokady do odczytu i zapisu (Podpis: 0x52574c41):** Atrybuty dla blokad do odczytu i zapisu, o rozmiarze 20 bajtów.

{% hint style="success" %}
Ostatnie 4 bajty tych obiektów są używane do wykrywania przepełnień.
{% endhint %}

### Zmienne lokalne wątku (TLV)

**Zmienne lokalne wątku (TLV)** w kontekście plików Mach-O (format dla plików wykonywalnych w macOS) są używane do deklarowania zmiennych, które są specyficzne dla **każdego wątku** w aplikacji wielowątkowej. Zapewnia to, że każdy wątek ma swoją własną oddzielną instancję zmiennej, co pozwala unikać konfliktów i utrzymywać integralność danych bez potrzeby stosowania jawnych mechanizmów synchronizacji, takich jak mutexy.

W języku C i pokrewnych można zadeklarować zmienną lokalną wątku, używając słowa kluczowego **`__thread`**. Oto jak to działa w twoim przykładzie:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
Ten fragment definiuje `tlv_var` jako zmienną lokalną dla wątku. Każdy wątek uruchamiający ten kod będzie miał swoją własną `tlv_var`, a zmiany, które jeden wątek wprowadza do `tlv_var`, nie wpłyną na `tlv_var` w innym wątku.

W binarnym pliku Mach-O dane związane z zmiennymi lokalnymi dla wątku są zorganizowane w określonych sekcjach:

* **`__DATA.__thread_vars`**: Ta sekcja zawiera metadane dotyczące zmiennych lokalnych dla wątku, takie jak ich typy i status inicjalizacji.
* **`__DATA.__thread_bss`**: Ta sekcja jest używana dla zmiennych lokalnych dla wątku, które nie są jawnie inicjalizowane. Jest to część pamięci zarezerwowanej dla danych z inicjalizacją zerową.

Mach-O zapewnia również specyficzne API o nazwie **`tlv_atexit`**, aby zarządzać zmiennymi lokalnymi dla wątku, gdy wątek kończy działanie. To API pozwala na **rejestrowanie destruktorów** — specjalnych funkcji, które sprzątają dane lokalne dla wątku, gdy wątek kończy działanie.

### Priorytety Wątków

Zrozumienie priorytetów wątków polega na przyjrzeniu się, jak system operacyjny decyduje, które wątki uruchomić i kiedy. Ta decyzja jest wpływana przez poziom priorytetu przypisany do każdego wątku. W systemach macOS i podobnych do Uniksa, obsługiwane jest to za pomocą koncepcji takich jak `nice`, `renice` i klasy jakości usług (QoS).

#### Nice i Renice

1. **Nice:**
* Wartość `nice` procesu to liczba, która wpływa na jego priorytet. Każdy proces ma wartość nice w zakresie od -20 (najwyższy priorytet) do 19 (najniższy priorytet). Domyślna wartość nice, gdy proces jest tworzony, wynosi zazwyczaj 0.
* Niższa wartość nice (bliżej -20) sprawia, że proces jest bardziej "egoistyczny", przydzielając mu więcej czasu CPU w porównaniu do innych procesów z wyższymi wartościami nice.
2. **Renice:**
* `renice` to polecenie używane do zmiany wartości nice już działającego procesu. Może być używane do dynamicznego dostosowywania priorytetu procesów, zarówno zwiększając, jak i zmniejszając ich przydział czasu CPU na podstawie nowych wartości nice.
* Na przykład, jeśli proces potrzebuje tymczasowo więcej zasobów CPU, możesz obniżyć jego wartość nice za pomocą `renice`.

#### Klasy Jakości Usług (QoS)

Klasy QoS to nowocześniejsze podejście do zarządzania priorytetami wątków, szczególnie w systemach takich jak macOS, które wspierają **Grand Central Dispatch (GCD)**. Klasy QoS pozwalają programistom na **kategoryzowanie** pracy na różne poziomy w zależności od ich znaczenia lub pilności. macOS automatycznie zarządza priorytetami wątków na podstawie tych klas QoS:

1. **Interaktywne dla Użytkownika:**
* Ta klasa jest przeznaczona dla zadań, które aktualnie wchodzą w interakcję z użytkownikiem lub wymagają natychmiastowych wyników, aby zapewnić dobrą jakość doświadczenia użytkownika. Te zadania mają najwyższy priorytet, aby utrzymać responsywność interfejsu (np. animacje lub obsługa zdarzeń).
2. **Inicjowane przez Użytkownika:**
* Zadania, które użytkownik inicjuje i oczekuje natychmiastowych wyników, takie jak otwieranie dokumentu lub klikanie przycisku, który wymaga obliczeń. Te mają wysoki priorytet, ale są poniżej interaktywnych dla użytkownika.
3. **Użyteczność:**
* Te zadania są długoterminowe i zazwyczaj pokazują wskaźnik postępu (np. pobieranie plików, importowanie danych). Mają niższy priorytet niż zadania inicjowane przez użytkownika i nie muszą kończyć się natychmiast.
4. **Tło:**
* Ta klasa jest przeznaczona dla zadań, które działają w tle i nie są widoczne dla użytkownika. Mogą to być zadania takie jak indeksowanie, synchronizacja lub kopie zapasowe. Mają najniższy priorytet i minimalny wpływ na wydajność systemu.

Korzystając z klas QoS, programiści nie muszą zarządzać dokładnymi numerami priorytetów, ale raczej koncentrować się na naturze zadania, a system optymalizuje zasoby CPU odpowiednio.

Ponadto istnieją różne **polityki planowania wątków**, które określają zestaw parametrów planowania, które planista weźmie pod uwagę. Można to zrobić za pomocą `thread_policy_[set/get]`. Może to być przydatne w atakach warunków wyścigu.

## Nadużycie Procesów w MacOS

MacOS, jak każdy inny system operacyjny, zapewnia różnorodne metody i mechanizmy, aby **procesy mogły wchodzić w interakcje, komunikować się i dzielić danymi**. Chociaż te techniki są niezbędne do efektywnego funkcjonowania systemu, mogą być również nadużywane przez aktorów zagrożeń do **przeprowadzania złośliwych działań**.

### Wstrzykiwanie Bibliotek

Wstrzykiwanie bibliotek to technika, w której atakujący **zmusza proces do załadowania złośliwej biblioteki**. Po wstrzyknięciu biblioteka działa w kontekście docelowego procesu, dając atakującemu te same uprawnienia i dostęp, co proces.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Hookowanie Funkcji

Hookowanie funkcji polega na **przechwytywaniu wywołań funkcji** lub wiadomości w kodzie oprogramowania. Poprzez hookowanie funkcji, atakujący może **zmodyfikować zachowanie** procesu, obserwować wrażliwe dane lub nawet zyskać kontrolę nad przepływem wykonania.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Komunikacja Między Procesami

Komunikacja między procesami (IPC) odnosi się do różnych metod, za pomocą których oddzielne procesy **dzielą się i wymieniają danymi**. Chociaż IPC jest fundamentalne dla wielu legalnych aplikacji, może być również nadużywane do podważania izolacji procesów, wycieku wrażliwych informacji lub wykonywania nieautoryzowanych działań.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Wstrzykiwanie Aplikacji Electron

Aplikacje Electron uruchamiane z określonymi zmiennymi środowiskowymi mogą być podatne na wstrzykiwanie procesów:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Wstrzykiwanie Chromium

Możliwe jest użycie flag `--load-extension` i `--use-fake-ui-for-media-stream`, aby przeprowadzić **atak typu man in the browser**, co pozwala na kradzież naciśnięć klawiszy, ruchu, ciasteczek, wstrzykiwanie skryptów na stronach...:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### Brudny NIB

Pliki NIB **definiują elementy interfejsu użytkownika (UI)** i ich interakcje w aplikacji. Mogą jednak **wykonywać dowolne polecenia** i **Gatekeeper nie zatrzymuje** już uruchomionej aplikacji przed ponownym uruchomieniem, jeśli **plik NIB jest zmodyfikowany**. Dlatego mogą być używane do uruchamiania dowolnych programów w celu wykonania dowolnych poleceń:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Wstrzykiwanie Aplikacji Java

Możliwe jest nadużycie niektórych możliwości Javy (takich jak zmienna środowiskowa **`_JAVA_OPTS`**), aby sprawić, że aplikacja Java wykona **dowolny kod/polecenia**.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### Wstrzykiwanie Aplikacji .Net

Możliwe jest wstrzykiwanie kodu do aplikacji .Net poprzez **nadużycie funkcjonalności debugowania .Net** (niechronionej przez zabezpieczenia macOS, takie jak wzmocnienie czasu wykonania).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Wstrzykiwanie Perl

Sprawdź różne opcje, aby sprawić, że skrypt Perl wykona dowolny kod w:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Wstrzykiwanie Ruby

Możliwe jest również nadużycie zmiennych środowiskowych Ruby, aby sprawić, że dowolne skrypty wykonają dowolny kod:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Wstrzykiwanie Pythona

Jeśli zmienna środowiskowa **`PYTHONINSPECT`** jest ustawiona, proces Pythona przejdzie do interfejsu CLI Pythona, gdy zakończy działanie. Możliwe jest również użycie **`PYTHONSTARTUP`**, aby wskazać skrypt Pythona do wykonania na początku interaktywnej sesji.\
Należy jednak zauważyć, że skrypt **`PYTHONSTARTUP`** nie zostanie wykonany, gdy **`PYTHONINSPECT`** utworzy interaktywną sesję.

Inne zmienne środowiskowe, takie jak **`PYTHONPATH`** i **`PYTHONHOME`**, mogą być również przydatne do wykonania polecenia Pythona, które wykona dowolny kod.

Należy zauważyć, że pliki wykonywalne skompilowane za pomocą **`pyinstaller`** nie będą używać tych zmiennych środowiskowych, nawet jeśli działają przy użyciu osadzonego Pythona.

{% hint style="danger" %}
Ogólnie nie mogłem znaleźć sposobu, aby zmusić Pythona do wykonania dowolnego kodu, nadużywając zmiennych środowiskowych.\
Jednak większość ludzi instaluje Pythona za pomocą **Homebrew**, co zainstaluje Pythona w **zapisywalnej lokalizacji** dla domyślnego użytkownika administracyjnego. Możesz to przejąć za pomocą czegoś takiego:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Even **root** will run this code when running python.
{% endhint %}

## Wykrywanie

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) to aplikacja open source, która może **wykrywać i blokować działania związane z wstrzykiwaniem procesów**:

* Używając **zmiennych środowiskowych**: Będzie monitorować obecność którejkolwiek z następujących zmiennych środowiskowych: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** i **`ELECTRON_RUN_AS_NODE`**
* Używając wywołań **`task_for_pid`**: Aby znaleźć, kiedy jeden proces chce uzyskać **port zadania innego**, co pozwala na wstrzykiwanie kodu do procesu.
* **Parametry aplikacji Electron**: Ktoś może użyć argumentów wiersza poleceń **`--inspect`**, **`--inspect-brk`** i **`--remote-debugging-port`**, aby uruchomić aplikację Electron w trybie debugowania, a tym samym wstrzyknąć do niej kod.
* Używając **symlinków** lub **hardlinków**: Typowo najczęstszym nadużyciem jest **umieszczenie linku z naszymi uprawnieniami**, i **wskazanie go na lokalizację o wyższych uprawnieniach**. Wykrywanie jest bardzo proste zarówno dla hardlinków, jak i symlinków. Jeśli proces tworzący link ma **inny poziom uprawnień** niż plik docelowy, tworzymy **alert**. Niestety w przypadku symlinków blokowanie nie jest możliwe, ponieważ nie mamy informacji o docelowym miejscu linku przed jego utworzeniem. To jest ograniczenie frameworka EndpointSecurity firmy Apple.

### Wywołania wykonywane przez inne procesy

W [**tym wpisie na blogu**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) można znaleźć, jak można użyć funkcji **`task_name_for_pid`**, aby uzyskać informacje o innych **procesach wstrzykujących kod do procesu** i następnie uzyskać informacje o tym innym procesie.

Zauważ, że aby wywołać tę funkcję, musisz być **tym samym uid**, co ten, który uruchamia proces lub **root** (i zwraca informacje o procesie, a nie sposób wstrzykiwania kodu).

## Odniesienia

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

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
