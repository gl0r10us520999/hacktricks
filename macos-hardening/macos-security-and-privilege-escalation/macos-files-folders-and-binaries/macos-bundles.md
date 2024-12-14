# macOS Bundles

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

Bundli w macOS służą jako kontenery dla różnych zasobów, w tym aplikacji, bibliotek i innych niezbędnych plików, co sprawia, że wyglądają jak pojedyncze obiekty w Finderze, takie jak znane pliki `*.app`. Najczęściej spotykanym bundlem jest bundle `.app`, chociaż inne typy, takie jak `.framework`, `.systemextension` i `.kext`, są również powszechne.

### Kluczowe komponenty bundla

W obrębie bundla, szczególnie w katalogu `<application>.app/Contents/`, znajduje się wiele ważnych zasobów:

* **\_CodeSignature**: Ten katalog przechowuje szczegóły dotyczące podpisu kodu, które są niezbędne do weryfikacji integralności aplikacji. Możesz sprawdzić informacje o podpisie kodu, używając poleceń takich jak: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Zawiera wykonywalny plik binarny aplikacji, który uruchamia się po interakcji użytkownika.
* **Resources**: Repozytorium komponentów interfejsu użytkownika aplikacji, w tym obrazów, dokumentów i opisów interfejsu (pliki nib/xib).
* **Info.plist**: Działa jako główny plik konfiguracyjny aplikacji, kluczowy dla systemu, aby odpowiednio rozpoznać i współdziałać z aplikacją.

#### Ważne klucze w Info.plist

Plik `Info.plist` jest fundamentem konfiguracji aplikacji, zawierając klucze takie jak:

* **CFBundleExecutable**: Określa nazwę głównego pliku wykonywalnego znajdującego się w katalogu `Contents/MacOS`.
* **CFBundleIdentifier**: Dostarcza globalny identyfikator dla aplikacji, szeroko stosowany przez macOS do zarządzania aplikacjami.
* **LSMinimumSystemVersion**: Wskazuje minimalną wersję macOS wymaganą do uruchomienia aplikacji.

### Badanie bundli

Aby zbadać zawartość bundla, takiego jak `Safari.app`, można użyć następującego polecenia: `bash ls -lR /Applications/Safari.app/Contents`

To badanie ujawnia katalogi takie jak `_CodeSignature`, `MacOS`, `Resources` oraz pliki takie jak `Info.plist`, z których każdy pełni unikalną rolę, od zabezpieczania aplikacji po definiowanie jej interfejsu użytkownika i parametrów operacyjnych.

#### Dodatkowe katalogi bundla

Poza powszechnymi katalogami, bundle mogą również zawierać:

* **Frameworks**: Zawiera zintegrowane frameworki używane przez aplikację. Frameworki są jak dyliby z dodatkowymi zasobami.
* **PlugIns**: Katalog dla wtyczek i rozszerzeń, które zwiększają możliwości aplikacji.
* **XPCServices**: Zawiera usługi XPC używane przez aplikację do komunikacji poza procesem.

Ta struktura zapewnia, że wszystkie niezbędne komponenty są zamknięte w bundlu, co ułatwia modułowe i bezpieczne środowisko aplikacji.

Aby uzyskać bardziej szczegółowe informacje na temat kluczy `Info.plist` i ich znaczenia, dokumentacja dewelopera Apple oferuje obszerne zasoby: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
