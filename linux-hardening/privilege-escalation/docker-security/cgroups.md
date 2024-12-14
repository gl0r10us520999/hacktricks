# CGroups

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

## Podstawowe informacje

**Linux Control Groups**, czyli **cgroups**, to funkcja jądra Linux, która umożliwia alokację, ograniczenie i priorytetyzację zasobów systemowych, takich jak CPU, pamięć i I/O dysku, wśród grup procesów. Oferują mechanizm do **zarządzania i izolowania wykorzystania zasobów** przez zbiory procesów, co jest korzystne w takich celach jak ograniczenie zasobów, izolacja obciążenia i priorytetyzacja zasobów wśród różnych grup procesów.

Istnieją **dwie wersje cgroups**: wersja 1 i wersja 2. Obie mogą być używane jednocześnie w systemie. Główna różnica polega na tym, że **cgroups wersja 2** wprowadza **hierarchiczną, drzewiastą strukturę**, umożliwiając bardziej zniuansowaną i szczegółową dystrybucję zasobów wśród grup procesów. Dodatkowo, wersja 2 wprowadza różne ulepszenia, w tym:

Oprócz nowej hierarchicznej organizacji, cgroups wersja 2 wprowadziła również **kilka innych zmian i ulepszeń**, takich jak wsparcie dla **nowych kontrolerów zasobów**, lepsze wsparcie dla aplikacji starszej generacji oraz poprawioną wydajność.

Ogólnie rzecz biorąc, cgroups **wersja 2 oferuje więcej funkcji i lepszą wydajność** niż wersja 1, ale ta ostatnia może być nadal używana w niektórych scenariuszach, gdzie zgodność ze starszymi systemami jest istotna.

Możesz wylistować cgroups v1 i v2 dla dowolnego procesu, przeglądając jego plik cgroup w /proc/\<pid>. Możesz zacząć od sprawdzenia cgroups swojego powłoki za pomocą tego polecenia:
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
The output structure is as follows:

* **Liczby 2–12**: cgroups v1, gdzie każda linia reprezentuje inny cgroup. Kontrolery dla nich są określone obok numeru.
* **Liczba 1**: Również cgroups v1, ale wyłącznie do celów zarządzania (ustawiane przez np. systemd) i nie ma kontrolera.
* **Liczba 0**: Reprezentuje cgroups v2. Żadne kontrolery nie są wymienione, a ta linia jest wyłączna dla systemów działających tylko na cgroups v2.
* **Nazwy są hierarchiczne**, przypominające ścieżki plików, wskazujące na strukturę i relacje między różnymi cgroups.
* **Nazwy takie jak /user.slice lub /system.slice** określają kategoryzację cgroups, przy czym user.slice zazwyczaj dotyczy sesji logowania zarządzanych przez systemd, a system.slice dotyczy usług systemowych.

### Viewing cgroups

System plików jest zazwyczaj wykorzystywany do uzyskiwania dostępu do **cgroups**, różniąc się od interfejsu wywołań systemowych Unix tradycyjnie używanego do interakcji z jądrem. Aby zbadać konfigurację cgroup powłoki, należy sprawdzić plik **/proc/self/cgroup**, który ujawnia cgroup powłoki. Następnie, przechodząc do katalogu **/sys/fs/cgroup** (lub **`/sys/fs/cgroup/unified`**) i znajdując katalog, który dzieli nazwę cgroup, można zaobserwować różne ustawienia i informacje o wykorzystaniu zasobów związane z cgroup.

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

Kluczowe pliki interfejsu dla cgroups są poprzedzone prefiksem **cgroup**. Plik **cgroup.procs**, który można przeglądać za pomocą standardowych poleceń, takich jak cat, wymienia procesy w cgroup. Inny plik, **cgroup.threads**, zawiera informacje o wątkach.

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Cgroups zarządzające powłokami zazwyczaj obejmują dwa kontrolery, które regulują wykorzystanie pamięci i liczbę procesów. Aby interagować z kontrolerem, należy skonsultować się z plikami noszącymi prefiks kontrolera. Na przykład, **pids.current** byłby odniesiony, aby ustalić liczbę wątków w cgroup.

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

Wskazanie **max** w wartości sugeruje brak konkretnego limitu dla cgroup. Jednak z powodu hierarchicznej natury cgroups, limity mogą być narzucane przez cgroup na niższym poziomie w hierarchii katalogów.

### Manipulating and Creating cgroups

Procesy są przypisywane do cgroups przez **zapisanie ich identyfikatora procesu (PID) do pliku `cgroup.procs`**. Wymaga to uprawnień roota. Na przykład, aby dodać proces:
```bash
echo [pid] > cgroup.procs
```
Podobnie, **modyfikacja atrybutów cgroup, takich jak ustawienie limitu PID**, odbywa się poprzez zapisanie żądanej wartości do odpowiedniego pliku. Aby ustawić maksymalnie 3,000 PID dla cgroup:
```bash
echo 3000 > pids.max
```
**Tworzenie nowych cgroups** polega na utworzeniu nowego podkatalogu w hierarchii cgroup, co powoduje, że jądro automatycznie generuje niezbędne pliki interfejsu. Chociaż cgroups bez aktywnych procesów można usunąć za pomocą `rmdir`, należy być świadomym pewnych ograniczeń:

* **Procesy mogą być umieszczane tylko w liściastych cgroups** (tj. w najbardziej zagnieżdżonych w hierarchii).
* **Cgroup nie może posiadać kontrolera, który nie występuje w jej rodzicu**.
* **Kontrolery dla cgroups podrzędnych muszą być wyraźnie zadeklarowane** w pliku `cgroup.subtree_control`. Na przykład, aby włączyć kontrolery CPU i PID w cgroup podrzędnej:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
**root cgroup** jest wyjątkiem od tych zasad, pozwalającym na bezpośrednie umieszczanie procesów. Może to być użyte do usunięcia procesów z zarządzania systemd.

**Monitorowanie użycia CPU** w cgroup jest możliwe poprzez plik `cpu.stat`, który wyświetla całkowity czas CPU zużyty, co jest pomocne w śledzeniu użycia w podprocesach usługi:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>Statystyki użycia CPU przedstawione w pliku cpu.stat</p></figcaption></figure>

## References

* **Książka: Jak działa Linux, 3. wydanie: Co każdy superużytkownik powinien wiedzieć autorstwa Briana Warda**

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
