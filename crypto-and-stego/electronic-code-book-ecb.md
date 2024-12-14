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


# ECB

(ECB) 전자 코드 북 - **명확한 텍스트의 각 블록을** **암호문 블록으로 대체하는** 대칭 암호화 방식입니다. 가장 **간단한** 암호화 방식입니다. 주요 아이디어는 **명확한 텍스트를 N 비트 블록으로 나누고** (입력 데이터의 블록 크기, 암호화 알고리즘에 따라 다름) 각 명확한 텍스트 블록을 **단 하나의 키**를 사용하여 암호화(복호화)하는 것입니다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

ECB를 사용하는 것은 여러 가지 보안 문제를 야기합니다:

* **암호화된 메시지의 블록을 제거할 수 있습니다.**
* **암호화된 메시지의 블록을 이동할 수 있습니다.**

# 취약점 탐지

애플리케이션에 여러 번 로그인하면 **항상 같은 쿠키를 받는다고 상상해 보세요.** 이는 애플리케이션의 쿠키가 **`<username>|<password>`**이기 때문입니다.\
그런 다음, **같은 긴 비밀번호**와 **거의** **같은** **사용자 이름**을 가진 두 명의 새로운 사용자를 생성합니다.\
두 사용자의 **정보가 같은 8B 블록**이 **같다는 것을 알게 됩니다.** 그런 다음, **ECB가 사용되고 있을 수 있다고 상상합니다.**

다음 예제와 같이. 이 **2개의 디코딩된 쿠키**가 여러 번 블록 **`\x23U\xE45K\xCB\x21\xC8`**를 가지고 있는지 관찰하세요.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
이것은 **그 쿠키의 사용자 이름과 비밀번호에 "a"라는 글자가 여러 번 포함되어 있기 때문입니다** (예를 들어). **다른** **블록**은 **최소 1개의 다른 문자**가 포함된 블록입니다 (아마도 구분자 "|" 또는 사용자 이름의 필요한 차이일 수 있습니다).

이제 공격자는 형식이 `<username><delimiter><password>`인지 `<password><delimiter><username>`인지 알아내기만 하면 됩니다. 이를 위해 그는 **유사하고 긴 사용자 이름과 비밀번호로 여러 사용자 이름을 생성하여 형식과 구분자의 길이를 찾을 수 있습니다:**

| 사용자 이름 길이: | 비밀번호 길이: | 사용자 이름+비밀번호 길이: | 쿠키의 길이 (디코딩 후): |
| ---------------- | ---------------- | ------------------------- | --------------------------------- |
| 2                | 2                | 4                         | 8                                 |
| 3                | 3                | 6                         | 8                                 |
| 3                | 4                | 7                         | 8                                 |
| 4                | 4                | 8                         | 16                                |
| 7                | 7                | 14                        | 16                                |

# 취약점 악용

## 전체 블록 제거

쿠키의 형식(` <username>|<password>`)을 알고, 사용자 이름 `admin`을 가장하기 위해 `aaaaaaaaadmin`이라는 새 사용자를 만들고 쿠키를 가져와서 디코딩합니다:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
우리는 이전에 `a`만 포함된 사용자 이름으로 생성된 패턴 `\x23U\xE45K\xCB\x21\xC8`를 볼 수 있습니다.\
그런 다음, 8B의 첫 번째 블록을 제거하면 사용자 이름 `admin`에 대한 유효한 쿠키를 얻을 수 있습니다:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## 블록 이동

많은 데이터베이스에서 `WHERE username='admin';`을 검색하는 것과 `WHERE username='admin    ';`을 검색하는 것은 동일합니다. _(여분의 공백에 주목하세요)_

따라서, 사용자 `admin`을 가장하는 또 다른 방법은 다음과 같습니다:

* `len(<username>) + len(<delimiter) % len(block)`인 사용자 이름을 생성합니다. 블록 크기가 `8B`인 경우, `username       `이라는 사용자 이름을 생성할 수 있으며, 구분자 `|`를 사용하면 청크 `<username><delimiter>`가 2개의 8B 블록을 생성합니다.
* 그런 다음, 우리가 가장하고자 하는 사용자 이름과 공백을 포함하는 정확한 블록 수를 채우는 비밀번호를 생성합니다: `admin   `

이 사용자의 쿠키는 3개의 블록으로 구성됩니다: 처음 2개는 사용자 이름 + 구분자의 블록이고, 세 번째는 (사용자 이름을 가장하는) 비밀번호입니다: `username       |admin   `

**그런 다음, 첫 번째 블록을 마지막 블록으로 교체하면 사용자 `admin`을 가장하게 됩니다: `admin          |username`**

## 참고문헌

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


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
