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

## Basic Information

macOS의 번들은 애플리케이션, 라이브러리 및 기타 필요한 파일을 포함하는 컨테이너 역할을 하여 Finder에서 `*.app` 파일과 같은 단일 객체로 나타납니다. 가장 일반적으로 접하는 번들은 `.app` 번들이지만, `.framework`, `.systemextension`, `.kext`와 같은 다른 유형도 널리 사용됩니다.

### Essential Components of a Bundle

번들 내, 특히 `<application>.app/Contents/` 디렉토리 내에는 다양한 중요한 리소스가 포함되어 있습니다:

* **\_CodeSignature**: 이 디렉토리는 애플리케이션의 무결성을 검증하는 데 중요한 코드 서명 세부정보를 저장합니다. 다음과 같은 명령어를 사용하여 코드 서명 정보를 검사할 수 있습니다: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: 사용자 상호작용 시 실행되는 애플리케이션의 실행 가능한 바이너리를 포함합니다.
* **Resources**: 이미지, 문서 및 인터페이스 설명(nib/xib 파일)을 포함한 애플리케이션의 사용자 인터페이스 구성 요소를 위한 저장소입니다.
* **Info.plist**: 시스템이 애플리케이션을 적절하게 인식하고 상호작용하는 데 중요한 애플리케이션의 주요 구성 파일 역할을 합니다.

#### Important Keys in Info.plist

`Info.plist` 파일은 애플리케이션 구성의 초석으로, 다음과 같은 키를 포함합니다:

* **CFBundleExecutable**: `Contents/MacOS` 디렉토리에 위치한 주요 실행 파일의 이름을 지정합니다.
* **CFBundleIdentifier**: 애플리케이션에 대한 전역 식별자를 제공하며, macOS에서 애플리케이션 관리를 위해 광범위하게 사용됩니다.
* **LSMinimumSystemVersion**: 애플리케이션이 실행되기 위해 필요한 최소 macOS 버전을 나타냅니다.

### Exploring Bundles

`Safari.app`와 같은 번들의 내용을 탐색하려면 다음 명령어를 사용할 수 있습니다: `bash ls -lR /Applications/Safari.app/Contents`

이 탐색은 `_CodeSignature`, `MacOS`, `Resources`와 같은 디렉토리 및 `Info.plist`와 같은 파일을 드러내며, 각각 애플리케이션 보안에서 사용자 인터페이스 및 운영 매개변수 정의에 이르기까지 고유한 목적을 수행합니다.

#### Additional Bundle Directories

일반 디렉토리 외에도 번들은 다음을 포함할 수 있습니다:

* **Frameworks**: 애플리케이션에서 사용하는 번들 프레임워크를 포함합니다. 프레임워크는 추가 리소스가 있는 dylibs와 같습니다.
* **PlugIns**: 애플리케이션의 기능을 향상시키는 플러그인 및 확장을 위한 디렉토리입니다.
* **XPCServices**: 애플리케이션이 프로세스 외부 통신을 위해 사용하는 XPC 서비스를 보유합니다.

이 구조는 모든 필요한 구성 요소가 번들 내에 캡슐화되어 모듈화되고 안전한 애플리케이션 환경을 촉진하도록 보장합니다.

`Info.plist` 키와 그 의미에 대한 더 자세한 정보는 Apple 개발자 문서에서 광범위한 리소스를 제공합니다: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
