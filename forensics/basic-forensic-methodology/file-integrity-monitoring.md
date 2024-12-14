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


# 기준선

기준선은 시스템의 특정 부분을 스냅샷으로 찍어 **미래 상태와 비교하여 변경 사항을 강조하는** 것입니다.

예를 들어, 파일 시스템의 각 파일의 해시를 계산하고 저장하여 어떤 파일이 수정되었는지 확인할 수 있습니다.\
이것은 생성된 사용자 계정, 실행 중인 프로세스, 실행 중인 서비스 및 크게 변경되지 않아야 하는 다른 모든 것에 대해서도 수행할 수 있습니다.

## 파일 무결성 모니터링

파일 무결성 모니터링(FIM)은 파일의 변경 사항을 추적하여 IT 환경과 데이터를 보호하는 중요한 보안 기술입니다. 두 가지 주요 단계가 포함됩니다:

1. **기준선 비교:** 파일 속성 또는 암호화 체크섬(MD5 또는 SHA-2와 같은)을 사용하여 기준선을 설정하고, 수정 사항을 감지하기 위한 미래 비교를 수행합니다.
2. **실시간 변경 알림:** 파일이 접근되거나 변경될 때 즉각적인 알림을 받습니다. 일반적으로 OS 커널 확장을 통해 이루어집니다.

## 도구

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

## 참고자료

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


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
