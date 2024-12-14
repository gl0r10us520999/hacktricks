# Skeleton Key

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

## Atak Skeleton Key

Atak **Skeleton Key** to zaawansowana technika, która pozwala atakującym **obejść uwierzytelnianie Active Directory** poprzez **wstrzyknięcie hasła głównego** do kontrolera domeny. Umożliwia to atakującemu **uwierzytelnienie się jako dowolny użytkownik** bez jego hasła, co skutkuje **przyznaniem mu nieograniczonego dostępu** do domeny.

Można go przeprowadzić za pomocą [Mimikatz](https://github.com/gentilkiwi/mimikatz). Aby zrealizować ten atak, **wymagane są uprawnienia administratora domeny**, a atakujący musi celować w każdy kontroler domeny, aby zapewnić kompleksowe naruszenie. Jednak efekt ataku jest tymczasowy, ponieważ **ponowne uruchomienie kontrolera domeny eliminuje złośliwe oprogramowanie**, co wymaga ponownej implementacji dla utrzymania dostępu.

**Wykonanie ataku** wymaga jednego polecenia: `misc::skeleton`.

## Środki zaradcze

Strategie łagodzenia takich ataków obejmują monitorowanie określonych identyfikatorów zdarzeń, które wskazują na instalację usług lub użycie wrażliwych uprawnień. W szczególności, poszukiwanie identyfikatora zdarzenia systemowego 7045 lub identyfikatora zdarzenia zabezpieczeń 4673 może ujawnić podejrzane działania. Dodatkowo, uruchomienie `lsass.exe` jako chronionego procesu może znacznie utrudnić wysiłki atakujących, ponieważ wymaga to od nich użycia sterownika w trybie jądra, co zwiększa złożoność ataku.

Oto polecenia PowerShell, aby wzmocnić środki bezpieczeństwa:

- Aby wykryć instalację podejrzanych usług, użyj: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*"}`

- W szczególności, aby wykryć sterownik Mimikatz, można wykorzystać następujące polecenie: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*" -and $_.message -like "*mimidrv*"}`

- Aby wzmocnić `lsass.exe`, zaleca się włączenie go jako chronionego procesu: `New-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name RunAsPPL -Value 1 -Verbose`

Weryfikacja po ponownym uruchomieniu systemu jest kluczowa, aby upewnić się, że środki ochronne zostały pomyślnie zastosowane. Można to osiągnąć poprzez: `Get-WinEvent -FilterHashtable @{Logname='System';ID=12} | ?{$_.message -like "*protected process*`

## Źródła
* [https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/](https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/)

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
