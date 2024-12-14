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


# CBC

Jeśli **ciasteczko** to **tylko** **nazwa użytkownika** (lub pierwsza część ciasteczka to nazwa użytkownika) i chcesz podszyć się pod nazwę użytkownika "**admin**". W takim razie możesz stworzyć nazwę użytkownika **"bdmin"** i **bruteforce** **pierwszy bajt** ciasteczka.

# CBC-MAC

**Kod uwierzytelniania wiadomości z łańcuchowaniem bloków szyfrowych** (**CBC-MAC**) to metoda stosowana w kryptografii. Działa poprzez wzięcie wiadomości i szyfrowanie jej blok po bloku, gdzie szyfrowanie każdego bloku jest powiązane z poprzednim. Proces ten tworzy **łańcuch bloków**, zapewniając, że zmiana nawet jednego bitu oryginalnej wiadomości spowoduje nieprzewidywalną zmianę w ostatnim bloku zaszyfrowanych danych. Aby wprowadzić lub cofnąć taką zmianę, wymagany jest klucz szyfrowania, co zapewnia bezpieczeństwo.

Aby obliczyć CBC-MAC wiadomości m, szyfruje się m w trybie CBC z zerowym wektorem inicjalizacyjnym i zachowuje ostatni blok. Poniższy rysunek szkicuje obliczenie CBC-MAC wiadomości składającej się z bloków![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) przy użyciu tajnego klucza k i szyfru blokowego E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

W przypadku CBC-MAC zazwyczaj **IV używany to 0**.\
To jest problem, ponieważ 2 znane wiadomości (`m1` i `m2`) niezależnie wygenerują 2 podpisy (`s1` i `s2`). Tak więc:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Następnie wiadomość składająca się z m1 i m2 połączonych (m3) wygeneruje 2 podpisy (s31 i s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Co jest możliwe do obliczenia bez znajomości klucza szyfrowania.**

Wyobraź sobie, że szyfrujesz nazwisko **Administrator** w blokach **8-bajtowych**:

* `Administ`
* `rator\00\00\00`

Możesz stworzyć nazwę użytkownika o nazwie **Administ** (m1) i odzyskać podpis (s1).\
Następnie możesz stworzyć nazwę użytkownika, która jest wynikiem `rator\00\00\00 XOR s1`. To wygeneruje `E(m2 XOR s1 XOR 0)` co jest s32.\
Teraz możesz użyć s32 jako podpisu pełnej nazwy **Administrator**.

### Podsumowanie

1. Uzyskaj podpis nazwy użytkownika **Administ** (m1), który to s1
2. Uzyskaj podpis nazwy użytkownika **rator\x00\x00\x00 XOR s1 XOR 0**, który to s32**.**
3. Ustaw ciasteczko na s32, a będzie to ważne ciasteczko dla użytkownika **Administrator**.

# Attack Controlling IV

Jeśli możesz kontrolować używany IV, atak może być bardzo łatwy.\
Jeśli ciasteczka to tylko zaszyfrowana nazwa użytkownika, aby podszyć się pod użytkownika "**administrator**", możesz stworzyć użytkownika "**Administrator**" i otrzymasz jego ciasteczko.\
Teraz, jeśli możesz kontrolować IV, możesz zmienić pierwszy bajt IV, tak aby **IV\[0] XOR "A" == IV'\[0] XOR "a"** i ponownie wygenerować ciasteczko dla użytkownika **Administrator.** To ciasteczko będzie ważne do **podszywania się** pod użytkownika **administrator** z początkowym **IV**.

## References

Więcej informacji na [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


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
