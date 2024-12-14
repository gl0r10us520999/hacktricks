# macOS Chromium Injection

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

Przeglądarki oparte na Chromium, takie jak Google Chrome, Microsoft Edge, Brave i inne. Te przeglądarki są zbudowane na projekcie open-source Chromium, co oznacza, że dzielą wspólną bazę i mają podobne funkcjonalności oraz opcje dewelopera.

#### Flaga `--load-extension`

Flaga `--load-extension` jest używana podczas uruchamiania przeglądarki opartej na Chromium z linii poleceń lub skryptu. Ta flaga pozwala na **automatyczne załadowanie jednego lub więcej rozszerzeń** do przeglądarki przy starcie.

#### Flaga `--use-fake-ui-for-media-stream`

Flaga `--use-fake-ui-for-media-stream` to kolejna opcja wiersza poleceń, która może być używana do uruchamiania przeglądarek opartych na Chromium. Ta flaga jest zaprojektowana, aby **ominąć normalne monity użytkownika, które proszą o pozwolenie na dostęp do strumieni mediów z kamery i mikrofonu**. Gdy ta flaga jest używana, przeglądarka automatycznie przyznaje pozwolenie każdej stronie internetowej lub aplikacji, która prosi o dostęp do kamery lub mikrofonu.

### Narzędzia

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### Przykład
```bash
# Intercept traffic
voodoo intercept -b chrome
```
Find more examples in the tools links

## References

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Dziel się trikami hackingowymi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na githubie.

</details>
{% endhint %}
