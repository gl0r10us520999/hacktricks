# CGroups

{% hint style="success" %}
Ucz się i praktykuj Hacking w AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i praktykuj Hacking w GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wesprzyj HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na githubie.

</details>
{% endhint %}

## Podstawowe informacje

**Linux Control Groups**, czyli **cgroups**, to funkcja jądra Linuxa, która umożliwia alokację, ograniczanie i priorytetyzację zasobów systemowych, takich jak CPU, pamięć i wejście/wyjście dysku, między grupami procesów. Oferują one mechanizm **zarządzania i izolowania użycia zasobów** kolekcji procesów, korzystny w celu takich jak ograniczanie zasobów, izolacja obciążenia i priorytetyzacja zasobów między różnymi grupami procesów.

Istnieją **dwie wersje cgroups**: wersja 1 i wersja 2. Obydwie mogą być używane równocześnie na systemie. Główną różnicą jest to, że **cgroups wersji 2** wprowadzają **hierarchiczną strukturę przypominającą drzewo**, umożliwiając bardziej subtelne i szczegółowe rozdział zasobów między grupami procesów. Dodatkowo, wersja 2 wprowadza różne ulepszenia, w tym:

Oprócz nowej organizacji hierarchicznej, cgroups wersji 2 wprowadziły również **kilka innych zmian i ulepszeń**, takich jak wsparcie dla **nowych kontrolerów zasobów**, lepsze wsparcie dla aplikacji z przeszłości oraz poprawiona wydajność.

Ogólnie rzecz biorąc, cgroups **wersji 2 oferują więcej funkcji i lepszą wydajność** niż wersja 1, ale ta ostatnia nadal może być używana w pewnych scenariuszach, gdzie istnieje obawa o zgodność z starszymi systemami.

Możesz wyświetlić cgroups v1 i v2 dla dowolnego procesu, patrząc na jego plik cgroup w /proc/\<pid>. Możesz zacząć od sprawdzenia cgroups swojego powłoki tym poleceniem:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
### Przeglądanie cgroups

Struktura wyjściowa prezentuje się następująco:

* **Numery 2–12**: cgroups v1, gdzie każda linia reprezentuje inny cgroup. Kontrolery dla nich są określone obok numeru.
* **Numer 1**: Również cgroups v1, ale wyłącznie do celów zarządzania (ustawiane przez np. systemd) i brak kontrolera.
* **Numer 0**: Reprezentuje cgroups v2. Nie wymieniono żadnych kontrolerów, a ta linia jest wyłączna dla systemów działających wyłącznie na cgroups v2.
* **Nazwy są hierarchiczne**, przypominające ścieżki plików, wskazując strukturę i relacje między różnymi cgroups.
* **Nazwy takie jak /user.slice lub /system.slice** określają kategoryzację cgroups, gdzie user.slice zazwyczaj jest przeznaczony dla sesji logowania zarządzanych przez systemd, a system.slice dla usług systemowych.

System plików jest zazwyczaj wykorzystywany do dostępu do **cgroups**, odbiegając od interfejsu wywołań systemowych Unix tradycyjnie używanego do interakcji z jądrem. Aby zbadać konfigurację cgroup powłoki, należy przejrzeć plik **/proc/self/cgroup**, który ujawnia cgroup powłoki. Następnie, przechodząc do katalogu **/sys/fs/cgroup** (lub **`/sys/fs/cgroup/unified`**) i lokalizując katalog o nazwie cgroup, można obserwować różne ustawienia i informacje o użyciu zasobów istotne dla cgroup.

![System plików Cgroup](<../../../.gitbook/assets/image (1128).png>)

Kluczowe pliki interfejsu dla cgroups mają prefiks **cgroup**. Plik **cgroup.procs**, który można przeglądać za pomocą standardowych poleceń takich jak cat, wymienia procesy wewnątrz cgroup. Inny plik, **cgroup.threads**, zawiera informacje o wątkach.

![Cgroup Procesy](<../../../.gitbook/assets/image (281).png>)

Cgroups zarządzające powłokami zazwyczaj obejmują dwa kontrolery regulujące użycie pamięci i liczbę procesów. Aby współdziałać z kontrolerem, należy skonsultować pliki z prefiksem kontrolera. Na przykład, **pids.current** byłby odniesieniem do ustalenia liczby wątków w cgroup.

![Pamięć Cgroup](<../../../.gitbook/assets/image (677).png>)

Wskazanie **max** w wartości sugeruje brak określonego limitu dla cgroup. Jednakże, ze względu na hierarchiczną naturę cgroups, limity mogą być narzucane przez cgroup na niższym poziomie w hierarchii katalogów.

### Manipulowanie i Tworzenie cgroups

Procesy są przypisywane do cgroups poprzez **zapisanie ich identyfikatora procesu (PID) do pliku `cgroup.procs`**. Wymaga to uprawnień roota. Na przykład, aby dodać proces:
```bash
echo [pid] > cgroup.procs
```
Podobnie, **modyfikowanie atrybutów cgroup, takich jak ustawienie limitu PID**, odbywa się poprzez zapisanie żądanej wartości do odpowiedniego pliku. Aby ustawić maksymalnie 3 000 PID-ów dla cgroup:
```bash
echo 3000 > pids.max
```
**Tworzenie nowych cgroups** polega na utworzeniu nowego podkatalogu w hierarchii cgroup, co powoduje automatyczne wygenerowanie niezbędnych plików interfejsu przez jądro. Chociaż cgroups bez aktywnych procesów można usunąć za pomocą `rmdir`, należy pamiętać o pewnych ograniczeniach:

* **Procesy mogą być umieszczone tylko w liściastych cgroups** (czyli tych najbardziej zagnieżdżonych w hierarchii).
* **Cgroup nie może posiadać kontrolera nieobecnego w swoim rodzicu**.
* **Kontrolery dla dzieci cgroups muszą być jawnie zadeklarowane** w pliku `cgroup.subtree_control`. Na przykład, aby włączyć kontrolery CPU i PID w dziecku cgroup:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
**Korzeń cgroup** jest wyjątkiem od tych zasad, pozwalającym na bezpośrednie umieszczanie procesów. Może to być użyte do usunięcia procesów z zarządzania przez systemd.

**Monitorowanie użycia CPU** wewnątrz cgroup jest możliwe za pomocą pliku `cpu.stat`, wyświetlającego całkowity czas CPU zużyty, co jest pomocne do śledzenia użycia przez podprocesy usługi:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>Statystyki użycia CPU widoczne w pliku cpu.stat</p></figcaption></figure>

## Referencje

* **Książka: Jak działa Linux, 3. wydanie: Co każdy superużytkownik powinien wiedzieć autorstwa Briana Warda**
