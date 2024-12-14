# macOS MDM

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

**Aby dowiedzieć się o MDM macOS, sprawdź:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Podstawy

### **Przegląd MDM (Zarządzanie Urządzeniami Mobilnymi)**

[Zarządzanie Urządzeniami Mobilnymi](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) jest wykorzystywane do nadzorowania różnych urządzeń końcowych, takich jak smartfony, laptopy i tablety. Szczególnie dla platform Apple (iOS, macOS, tvOS) obejmuje zestaw specjalistycznych funkcji, interfejsów API i praktyk. Działanie MDM opiera się na kompatybilnym serwerze MDM, który jest dostępny komercyjnie lub jako open-source, i musi wspierać [Protokół MDM](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Kluczowe punkty obejmują:

* Centralne zarządzanie urządzeniami.
* Zależność od serwera MDM, który przestrzega protokołu MDM.
* Zdolność serwera MDM do wysyłania różnych poleceń do urządzeń, na przykład zdalnego usuwania danych lub instalacji konfiguracji.

### **Podstawy DEP (Program Rejestracji Urządzeń)**

[Program Rejestracji Urządzeń](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) oferowany przez Apple upraszcza integrację Zarządzania Urządzeniami Mobilnymi (MDM) poprzez umożliwienie konfiguracji bezdotykowej dla urządzeń iOS, macOS i tvOS. DEP automatyzuje proces rejestracji, pozwalając urządzeniom na działanie od razu po wyjęciu z pudełka, z minimalną interwencją użytkownika lub administratora. Kluczowe aspekty obejmują:

* Umożliwia urządzeniom autonomiczne rejestrowanie się na wcześniej zdefiniowanym serwerze MDM po pierwszej aktywacji.
* Głównie korzystne dla nowych urządzeń, ale również stosowane dla urządzeń poddawanych rekonfiguracji.
* Ułatwia prostą konfigurację, szybko przygotowując urządzenia do użytku w organizacji.

### **Rozważania dotyczące bezpieczeństwa**

Ważne jest, aby zauważyć, że łatwość rejestracji zapewniana przez DEP, choć korzystna, może również stwarzać ryzyko bezpieczeństwa. Jeśli środki ochronne nie są odpowiednio egzekwowane dla rejestracji MDM, napastnicy mogą wykorzystać ten uproszczony proces do zarejestrowania swojego urządzenia na serwerze MDM organizacji, podszywając się pod urządzenie korporacyjne.

{% hint style="danger" %}
**Alert bezpieczeństwa**: Uproszczona rejestracja DEP może potencjalnie umożliwić nieautoryzowaną rejestrację urządzenia na serwerze MDM organizacji, jeśli odpowiednie zabezpieczenia nie są wdrożone.
{% endhint %}

### Podstawy Czym jest SCEP (Protokół Prośby o Certyfikat)?

* Stosunkowo stary protokół, stworzony przed powszechnym wprowadzeniem TLS i HTTPS.
* Daje klientom ustandaryzowany sposób wysyłania **Prośby o Podpisanie Certyfikatu** (CSR) w celu uzyskania certyfikatu. Klient poprosi serwer o wydanie podpisanego certyfikatu.

### Czym są Profile Konfiguracji (aka mobileconfigs)?

* Oficjalny sposób Apple na **ustawianie/egzekwowanie konfiguracji systemu.**
* Format pliku, który może zawierać wiele ładunków.
* Oparty na listach właściwości (w rodzaju XML).
* „może być podpisany i zaszyfrowany, aby zweryfikować ich pochodzenie, zapewnić ich integralność i chronić ich zawartość.” Podstawy — Strona 70, Przewodnik po Bezpieczeństwie iOS, styczeń 2018.

## Protokoły

### MDM

* Połączenie APNs (**serwery Apple**) + RESTful API (**serwery dostawców MDM**)
* **Komunikacja** zachodzi między **urządzeniem** a serwerem związanym z **produktem zarządzania urządzeniami**
* **Polecenia** dostarczane z MDM do urządzenia w **słownikach zakodowanych w plist**
* Całość przez **HTTPS**. Serwery MDM mogą być (i zazwyczaj są) przypinane.
* Apple przyznaje dostawcy MDM **certyfikat APNs** do uwierzytelniania

### DEP

* **3 API**: 1 dla sprzedawców, 1 dla dostawców MDM, 1 dla tożsamości urządzenia (nieudokumentowane):
* Tzw. [API "usługi chmurowej" DEP](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Używane przez serwery MDM do kojarzenia profili DEP z konkretnymi urządzeniami.
* [API DEP używane przez autoryzowanych sprzedawców Apple](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html) do rejestracji urządzeń, sprawdzania statusu rejestracji i statusu transakcji.
* Nieudokumentowane prywatne API DEP. Używane przez urządzenia Apple do żądania swojego profilu DEP. Na macOS, binarny `cloudconfigurationd` jest odpowiedzialny za komunikację przez to API.
* Bardziej nowoczesne i oparte na **JSON** (w porównaniu do plist)
* Apple przyznaje dostawcy MDM **token OAuth**

**API "usługi chmurowej" DEP**

* RESTful
* synchronizuje rekordy urządzeń z Apple do serwera MDM
* synchronizuje „profile DEP” do Apple z serwera MDM (dostarczane przez Apple do urządzenia później)
* Profil DEP zawiera:
* URL serwera dostawcy MDM
* Dodatkowe zaufane certyfikaty dla URL serwera (opcjonalne przypinanie)
* Dodatkowe ustawienia (np. które ekrany pominąć w Asystencie Konfiguracji)

## Numer seryjny

Urządzenia Apple wyprodukowane po 2010 roku zazwyczaj mają **12-znakowe alfanumeryczne** numery seryjne, przy czym **pierwsze trzy cyfry reprezentują miejsce produkcji**, następne **dwie** wskazują **rok** i **tydzień** produkcji, następne **trzy** cyfry dostarczają **unikalny** **identyfikator**, a **ostatnie** **cztery** cyfry reprezentują **numer modelu**.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## Kroki rejestracji i zarządzania

1. Utworzenie rekordu urządzenia (Sprzedawca, Apple): Rekord nowego urządzenia jest tworzony
2. Przypisanie rekordu urządzenia (Klient): Urządzenie jest przypisywane do serwera MDM
3. Synchronizacja rekordu urządzenia (dostawca MDM): MDM synchronizuje rekordy urządzeń i przesyła profile DEP do Apple
4. Rejestracja DEP (Urządzenie): Urządzenie otrzymuje swój profil DEP
5. Pobieranie profilu (Urządzenie)
6. Instalacja profilu (Urządzenie) a. w tym ładunki MDM, SCEP i root CA
7. Wydanie polecenia MDM (Urządzenie)

![](<../../../.gitbook/assets/image (694).png>)

Plik `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` eksportuje funkcje, które można uznać za **wysokopoziomowe "kroki"** procesu rejestracji.

### Krok 4: Rejestracja DEP - Uzyskiwanie Rekordu Aktywacji

Ta część procesu zachodzi, gdy **użytkownik uruchamia Maca po raz pierwszy** (lub po całkowitym wyczyszczeniu)

![](<../../../.gitbook/assets/image (1044).png>)

lub podczas wykonywania `sudo profiles show -type enrollment`

* Określenie **czy urządzenie jest włączone w DEP**
* Rekord Aktywacji to wewnętrzna nazwa dla **profilu DEP**
* Rozpoczyna się, gdy urządzenie jest podłączone do Internetu
* Napędzane przez **`CPFetchActivationRecord`**
* Zrealizowane przez **`cloudconfigurationd`** za pośrednictwem XPC. **"Asystent Konfiguracji"** (gdy urządzenie jest uruchamiane po raz pierwszy) lub polecenie **`profiles`** skontaktuje się z tym demonem, aby uzyskać rekord aktywacji.
* LaunchDaemon (zawsze działa jako root)

Wykonuje kilka kroków, aby uzyskać Rekord Aktywacji, realizowanych przez **`MCTeslaConfigurationFetcher`**. Proces ten wykorzystuje szyfrowanie zwane **Absinthe**

1. Pobierz **certyfikat**
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. **Zainicjuj** stan z certyfikatu (**`NACInit`**)
1. Używa różnych danych specyficznych dla urządzenia (tj. **Numer Seryjny za pomocą `IOKit`**)
3. Pobierz **klucz sesji**
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. Ustanów sesję (**`NACKeyEstablishment`**)
5. Złóż żądanie
1. POST do [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) wysyłając dane `{ "action": "RequestProfileConfiguration", "sn": "" }`
2. Ładunek JSON jest szyfrowany za pomocą Absinthe (**`NACSign`**)
3. Wszystkie żądania przez HTTPs, używane są wbudowane certyfikaty root

![](<../../../.gitbook/assets/image (566).png>)

Odpowiedź to słownik JSON z ważnymi danymi, takimi jak:

* **url**: URL hosta dostawcy MDM dla profilu aktywacji
* **anchor-certs**: Tablica certyfikatów DER używanych jako zaufane kotwice

### **Krok 5: Pobieranie profilu**

![](<../../../.gitbook/assets/image (444).png>)

* Żądanie wysłane do **url podanego w profilu DEP**.
* **Certyfikaty kotwiczne** są używane do **oceny zaufania**, jeśli są podane.
* Przypomnienie: właściwość **anchor\_certs** profilu DEP
* **Żądanie to prosty .plist** z identyfikacją urządzenia
* Przykłady: **UDID, wersja OS**.
* Podpisane CMS, zakodowane DER
* Podpisane za pomocą **certyfikatu tożsamości urządzenia (z APNS)**
* **Łańcuch certyfikatów** zawiera wygasły **Apple iPhone Device CA**

![](<../../../.gitbook/assets/image (567).png>)

### Krok 6: Instalacja profilu

* Po pobraniu, **profil jest przechowywany w systemie**
* Ten krok rozpoczyna się automatycznie (jeśli w **asystencie konfiguracji**)
* Napędzany przez **`CPInstallActivationProfile`**
* Zrealizowane przez mdmclient za pośrednictwem XPC
* LaunchDaemon (jako root) lub LaunchAgent (jako użytkownik), w zależności od kontekstu
* Profile konfiguracji mają wiele ładunków do zainstalowania
* Framework ma architekturę opartą na wtyczkach do instalacji profili
* Każdy typ ładunku jest powiązany z wtyczką
* Może być XPC (w frameworku) lub klasyczny Cocoa (w ManagedClient.app)
* Przykład:
* Ładunki certyfikatów używają CertificateService.xpc

Typowo, **profil aktywacji** dostarczany przez dostawcę MDM będzie **zawierał następujące ładunki**:

* `com.apple.mdm`: aby **zarejestrować** urządzenie w MDM
* `com.apple.security.scep`: aby bezpiecznie dostarczyć **certyfikat klienta** do urządzenia.
* `com.apple.security.pem`: aby **zainstalować zaufane certyfikaty CA** w systemowym Keychain urządzenia.
* Instalacja ładunku MDM odpowiada **rejestracji MDM w dokumentacji**
* Ładunek **zawiera kluczowe właściwości**:
*
* URL rejestracji MDM (**`CheckInURL`**)
* URL Polling Komend MDM (**`ServerURL`**) + temat APNs do jego wyzwolenia
* Aby zainstalować ładunek MDM, żądanie jest wysyłane do **`CheckInURL`**
* Zrealizowane w **`mdmclient`**
* Ładunek MDM może zależeć od innych ładunków
* Umożliwia **przypinanie żądań do konkretnych certyfikatów**:
* Właściwość: **`CheckInURLPinningCertificateUUIDs`**
* Właściwość: **`ServerURLPinningCertificateUUIDs`**
* Dostarczane przez ładunek PEM
* Umożliwia urządzeniu przypisanie certyfikatu tożsamości:
* Właściwość: IdentityCertificateUUID
* Dostarczane przez ładunek SCEP

### **Krok 7: Nasłuchiwanie poleceń MDM**

* Po zakończeniu rejestracji MDM, dostawca może **wysyłać powiadomienia push za pomocą APNs**
* Po odebraniu, obsługiwane przez **`mdmclient`**
* Aby sprawdzić polecenia MDM, żądanie jest wysyłane do ServerURL
* Wykorzystuje wcześniej zainstalowany ładunek MDM:
* **`ServerURLPinningCertificateUUIDs`** do przypinania żądania
* **`IdentityCertificateUUID`** do certyfikatu klienta TLS

## Ataki

### Rejestracja urządzeń w innych organizacjach

Jak wcześniej wspomniano, aby spróbować zarejestrować urządzenie w organizacji **wystarczy tylko numer seryjny należący do tej organizacji**. Gdy urządzenie jest zarejestrowane, wiele organizacji zainstaluje wrażliwe dane na nowym urządzeniu: certyfikaty, aplikacje, hasła WiFi, konfiguracje VPN [i tak dalej](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Dlatego może to być niebezpieczny punkt wejścia dla napastników, jeśli proces rejestracji nie jest odpowiednio chroniony:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

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
