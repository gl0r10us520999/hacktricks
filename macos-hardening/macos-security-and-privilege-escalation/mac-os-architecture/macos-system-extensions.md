# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

W przeciwieństwie do rozszerzeń jądra, **rozszerzenia systemowe działają w przestrzeni użytkownika** zamiast w przestrzeni jądra, co zmniejsza ryzyko awarii systemu z powodu awarii rozszerzenia.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Istnieją trzy typy rozszerzeń systemowych: **Rozszerzenia DriverKit**, **Rozszerzenia Sieciowe** i **Rozszerzenia Bezpieczeństwa Końcówki**.

### **Rozszerzenia DriverKit**

DriverKit jest zamiennikiem rozszerzeń jądra, które **zapewniają wsparcie sprzętowe**. Umożliwia to działanie sterowników urządzeń (takich jak sterowniki USB, szeregowe, NIC i HID) w przestrzeni użytkownika zamiast w przestrzeni jądra. Ramy DriverKit zawierają **wersje w przestrzeni użytkownika niektórych klas I/O Kit**, a jądro przekazuje normalne zdarzenia I/O Kit do przestrzeni użytkownika, oferując bezpieczniejsze środowisko dla tych sterowników.

### **Rozszerzenia Sieciowe**

Rozszerzenia Sieciowe umożliwiają dostosowanie zachowań sieciowych. Istnieje kilka typów Rozszerzeń Sieciowych:

* **Proxy Aplikacji**: Używane do tworzenia klienta VPN, który implementuje protokół VPN oparty na przepływie. Oznacza to, że obsługuje ruch sieciowy na podstawie połączeń (lub przepływów) zamiast pojedynczych pakietów.
* **Tunel Pakietowy**: Używane do tworzenia klienta VPN, który implementuje protokół VPN oparty na pakietach. Oznacza to, że obsługuje ruch sieciowy na podstawie pojedynczych pakietów.
* **Filtruj Dane**: Używane do filtrowania "przepływów" sieciowych. Może monitorować lub modyfikować dane sieciowe na poziomie przepływu.
* **Filtruj Pakiet**: Używane do filtrowania pojedynczych pakietów sieciowych. Może monitorować lub modyfikować dane sieciowe na poziomie pakietu.
* **Proxy DNS**: Używane do tworzenia niestandardowego dostawcy DNS. Może być używane do monitorowania lub modyfikowania żądań i odpowiedzi DNS.

## Endpoint Security Framework

Endpoint Security to framework dostarczany przez Apple w macOS, który zapewnia zestaw API do zabezpieczeń systemowych. Jest przeznaczony do użytku przez **dostawców zabezpieczeń i programistów do budowy produktów, które mogą monitorować i kontrolować aktywność systemu** w celu identyfikacji i ochrony przed złośliwą działalnością.

Ten framework zapewnia **zbiór API do monitorowania i kontrolowania aktywności systemu**, takich jak wykonywanie procesów, zdarzenia systemu plików, zdarzenia sieciowe i jądra.

Rdzeń tego frameworka jest zaimplementowany w jądrze, jako Rozszerzenie Jądra (KEXT) znajdujące się w **`/System/Library/Extensions/EndpointSecurity.kext`**. Ten KEXT składa się z kilku kluczowych komponentów:

* **EndpointSecurityDriver**: Działa jako "punkt wejścia" dla rozszerzenia jądra. Jest głównym punktem interakcji między systemem operacyjnym a frameworkiem Endpoint Security.
* **EndpointSecurityEventManager**: Ten komponent jest odpowiedzialny za implementację haków jądra. Haki jądra pozwalają frameworkowi monitorować zdarzenia systemowe poprzez przechwytywanie wywołań systemowych.
* **EndpointSecurityClientManager**: Zarządza komunikacją z klientami w przestrzeni użytkownika, śledząc, którzy klienci są połączeni i potrzebują otrzymywać powiadomienia o zdarzeniach.
* **EndpointSecurityMessageManager**: Wysyła wiadomości i powiadomienia o zdarzeniach do klientów w przestrzeni użytkownika.

Zdarzenia, które framework Endpoint Security może monitorować, są klasyfikowane na:

* Zdarzenia plików
* Zdarzenia procesów
* Zdarzenia gniazd
* Zdarzenia jądra (takie jak ładowanie/odładowanie rozszerzenia jądra lub otwieranie urządzenia I/O Kit)

### Architektura Frameworka Endpoint Security

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Komunikacja w przestrzeni użytkownika** z frameworkiem Endpoint Security odbywa się za pośrednictwem klasy IOUserClient. Używane są dwie różne podklasy, w zależności od typu wywołującego:

* **EndpointSecurityDriverClient**: Wymaga uprawnienia `com.apple.private.endpoint-security.manager`, które jest posiadane tylko przez proces systemowy `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Wymaga uprawnienia `com.apple.developer.endpoint-security.client`. Zwykle byłoby używane przez oprogramowanie zabezpieczające firm trzecich, które musi współdziałać z frameworkiem Endpoint Security.

Rozszerzenia Endpoint Security:**`libEndpointSecurity.dylib`** to biblioteka C, której rozszerzenia systemowe używają do komunikacji z jądrem. Ta biblioteka używa I/O Kit (`IOKit`) do komunikacji z KEXT Endpoint Security.

**`endpointsecurityd`** to kluczowy demon systemowy zaangażowany w zarządzanie i uruchamianie rozszerzeń systemowych bezpieczeństwa końcówki, szczególnie podczas wczesnego procesu rozruchu. **Tylko rozszerzenia systemowe** oznaczone jako **`NSEndpointSecurityEarlyBoot`** w ich pliku `Info.plist` otrzymują tę wczesną obsługę rozruchu.

Inny demon systemowy, **`sysextd`**, **waliduje rozszerzenia systemowe** i przenosi je do odpowiednich lokalizacji systemowych. Następnie prosi odpowiedni demon o załadowanie rozszerzenia. **`SystemExtensions.framework`** jest odpowiedzialny za aktywację i dezaktywację rozszerzeń systemowych.

## Obejście ESF

ESF jest używane przez narzędzia zabezpieczające, które będą próbowały wykryć członka zespołu red, więc wszelkie informacje na temat tego, jak można to obejść, są interesujące.

### CVE-2021-30965

Problem polega na tym, że aplikacja zabezpieczająca musi mieć **uprawnienia do pełnego dostępu do dysku**. Więc jeśli atakujący mógłby to usunąć, mógłby zapobiec uruchomieniu oprogramowania:
```bash
tccutil reset All
```
Dla **więcej informacji** na temat tego obejścia i pokrewnych, sprawdź wykład [#OBTS v5.0: "Słaby punkt EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Na końcu to zostało naprawione poprzez nadanie nowego uprawnienia **`kTCCServiceEndpointSecurityClient`** aplikacji zabezpieczającej zarządzanej przez **`tccd`**, dzięki czemu `tccutil` nie usunie jej uprawnień, co uniemożliwi jej działanie.

## Odniesienia

* [**OBTS v3.0: "Bezpieczeństwo i niebezpieczeństwo Endpoint" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
