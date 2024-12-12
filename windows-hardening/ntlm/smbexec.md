# SmbExec/ScExec

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

## Jak to działa

**Smbexec** to narzędzie używane do zdalnego wykonywania poleceń na systemach Windows, podobne do **Psexec**, ale unika umieszczania jakichkolwiek złośliwych plików na docelowym systemie.

### Kluczowe punkty dotyczące **SMBExec**

- Działa poprzez tworzenie tymczasowej usługi (na przykład "BTOBTO") na docelowej maszynie, aby wykonywać polecenia za pomocą cmd.exe (%COMSPEC%), bez zrzucania jakichkolwiek binariów.
- Pomimo swojego dyskretnego podejścia, generuje dzienniki zdarzeń dla każdego wykonanego polecenia, oferując formę nieinteraktywnego "shella".
- Polecenie do połączenia za pomocą **Smbexec** wygląda tak:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Wykonywanie poleceń bez binariów

- **Smbexec** umożliwia bezpośrednie wykonywanie poleceń za pomocą binPaths usług, eliminując potrzebę posiadania fizycznych binariów na docelowym systemie.
- Metoda ta jest przydatna do wykonywania jednorazowych poleceń na docelowym systemie Windows. Na przykład, połączenie jej z modułem `web_delivery` Metasploit pozwala na wykonanie ładunku zwrotnego Meterpreter skierowanego na PowerShell.
- Tworząc zdalną usługę na maszynie atakującego z binPath ustawionym na uruchomienie podanego polecenia przez cmd.exe, możliwe jest pomyślne wykonanie ładunku, osiągając callback i wykonanie ładunku z nasłuchiwaczem Metasploit, nawet jeśli wystąpią błędy odpowiedzi usługi.

### Przykład poleceń

Utworzenie i uruchomienie usługi można zrealizować za pomocą następujących poleceń:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
Aby uzyskać więcej szczegółów, sprawdź [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## Odniesienia
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

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
