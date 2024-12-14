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

## Basic Information

Chromium 기반 브라우저인 Google Chrome, Microsoft Edge, Brave 등. 이러한 브라우저는 Chromium 오픈 소스 프로젝트를 기반으로 구축되었으며, 이는 공통 기반을 공유하고 따라서 유사한 기능과 개발자 옵션을 가지고 있음을 의미합니다.

#### `--load-extension` Flag

`--load-extension` 플래그는 명령줄이나 스크립트에서 Chromium 기반 브라우저를 시작할 때 사용됩니다. 이 플래그는 브라우저 시작 시 **하나 이상의 확장 프로그램을 자동으로 로드**할 수 있게 해줍니다.

#### `--use-fake-ui-for-media-stream` Flag

`--use-fake-ui-for-media-stream` 플래그는 Chromium 기반 브라우저를 시작하는 데 사용할 수 있는 또 다른 명령줄 옵션입니다. 이 플래그는 **카메라와 마이크에서 미디어 스트림에 접근하기 위한 권한을 요청하는 일반 사용자 프롬프트를 우회**하도록 설계되었습니다. 이 플래그가 사용되면 브라우저는 카메라나 마이크에 접근을 요청하는 모든 웹사이트나 애플리케이션에 자동으로 권한을 부여합니다.

### Tools

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### Example
```bash
# Intercept traffic
voodoo intercept -b chrome
```
Find more examples in the tools links

## References

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

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
