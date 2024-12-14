# ZIPs tricks

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

**ZIP 파일**을 관리하기 위한 **명령줄 도구**는 ZIP 파일을 진단, 복구 및 해킹하는 데 필수적입니다. 다음은 몇 가지 주요 유틸리티입니다:

- **`unzip`**: ZIP 파일이 압축 해제되지 않는 이유를 밝힙니다.
- **`zipdetails -v`**: ZIP 파일 형식 필드에 대한 자세한 분석을 제공합니다.
- **`zipinfo`**: ZIP 파일의 내용을 추출하지 않고 나열합니다.
- **`zip -F input.zip --out output.zip`** 및 **`zip -FF input.zip --out output.zip`**: 손상된 ZIP 파일을 복구하려고 시도합니다.
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: ZIP 비밀번호를 무차별 대입으로 해킹하는 도구로, 약 7자까지의 비밀번호에 효과적입니다.

[ZIP 파일 형식 사양](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)은 ZIP 파일의 구조와 표준에 대한 포괄적인 세부정보를 제공합니다.

비밀번호로 보호된 ZIP 파일은 **파일 이름이나 파일 크기를 암호화하지 않**는다는 점을 주목하는 것이 중요합니다. 이는 RAR 또는 7z 파일과는 다른 보안 결함입니다. 또한, 구식 ZipCrypto 방법으로 암호화된 ZIP 파일은 압축된 파일의 암호화되지 않은 복사본이 있는 경우 **평문 공격**에 취약합니다. 이 공격은 알려진 내용을 활용하여 ZIP의 비밀번호를 해킹하며, [HackThis의 기사](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files)와 [이 학술 논문](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf)에서 자세히 설명되어 있습니다. 그러나 **AES-256** 암호화로 보호된 ZIP 파일은 이 평문 공격에 면역이 있어, 민감한 데이터에 대한 안전한 암호화 방법 선택의 중요성을 보여줍니다.

## References
* [https://michael-myers.github.io/blog/categories/ctf/](https://michael-myers.github.io/blog/categories/ctf/)

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
