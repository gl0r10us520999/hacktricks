# Certificates

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

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## What is a Certificate

**공개 키 인증서**는 암호화에서 누군가가 공개 키를 소유하고 있음을 증명하는 데 사용되는 디지털 ID입니다. 여기에는 키의 세부정보, 소유자의 신원(주체) 및 신뢰할 수 있는 기관(발급자)의 디지털 서명이 포함됩니다. 소프트웨어가 발급자를 신뢰하고 서명이 유효하면 키 소유자와의 안전한 통신이 가능합니다.

인증서는 주로 [인증 기관](https://en.wikipedia.org/wiki/Certificate\_authority) (CA)에서 [공개 키 인프라](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI) 설정에서 발급됩니다. 또 다른 방법은 사용자가 서로의 키를 직접 검증하는 [신뢰의 웹](https://en.wikipedia.org/wiki/Web\_of\_trust)입니다. 인증서의 일반적인 형식은 [X.509](https://en.wikipedia.org/wiki/X.509)이며, RFC 5280에 설명된 특정 요구 사항에 맞게 조정될 수 있습니다.

## x509 Common Fields

### **x509 인증서의 일반 필드**

x509 인증서에서 여러 **필드**는 인증서의 유효성과 보안을 보장하는 데 중요한 역할을 합니다. 이러한 필드의 세부 사항은 다음과 같습니다:

* **버전 번호**는 x509 형식의 버전을 나타냅니다.
* **일련 번호**는 인증서를 인증 기관(CA) 시스템 내에서 고유하게 식별하며, 주로 폐기 추적을 위해 사용됩니다.
* **주체** 필드는 인증서의 소유자를 나타내며, 이는 기계, 개인 또는 조직일 수 있습니다. 여기에는 다음과 같은 자세한 식별 정보가 포함됩니다:
* **공통 이름(CN)**: 인증서가 적용되는 도메인.
* **국가(C)**, **지역(L)**, **주 또는 지방(ST, S 또는 P)**, **조직(O)** 및 **조직 단위(OU)**는 지리적 및 조직적 세부정보를 제공합니다.
* **구별 이름(DN)**은 전체 주체 식별을 포함합니다.
* **발급자**는 인증서를 검증하고 서명한 사람을 나타내며, CA에 대한 주체와 유사한 하위 필드를 포함합니다.
* **유효 기간**은 **Not Before** 및 **Not After** 타임스탬프로 표시되어 인증서가 특정 날짜 이전이나 이후에 사용되지 않도록 보장합니다.
* **공개 키** 섹션은 인증서의 보안에 중요한 부분으로, 공개 키의 알고리즘, 크기 및 기타 기술적 세부정보를 지정합니다.
* **x509v3 확장**은 인증서의 기능을 향상시키며, **키 사용**, **확장 키 사용**, **주체 대체 이름** 및 인증서의 적용을 세부 조정하는 기타 속성을 지정합니다.

#### **키 사용 및 확장**

* **키 사용**은 공개 키의 암호화 응용 프로그램을 식별하며, 디지털 서명 또는 키 암호화와 같은 용도로 사용됩니다.
* **확장 키 사용**은 인증서의 사용 사례를 더욱 좁혀 TLS 서버 인증과 같은 용도로 사용됩니다.
* **주체 대체 이름** 및 **기본 제약 조건**은 인증서가 적용되는 추가 호스트 이름과 인증서가 CA인지 최종 엔티티 인증서인지를 정의합니다.
* **주체 키 식별자** 및 **권한 키 식별자**와 같은 식별자는 키의 고유성과 추적 가능성을 보장합니다.
* **권한 정보 접근** 및 **CRL 배포 지점**은 발급 CA를 검증하고 인증서 폐기 상태를 확인하는 경로를 제공합니다.
* **CT 사전 인증서 SCT**는 인증서에 대한 공공 신뢰를 위해 중요한 투명성 로그를 제공합니다.
```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```
### **OCSP와 CRL 배포 지점의 차이**

**OCSP** (**RFC 2560**)는 클라이언트와 응답자가 협력하여 디지털 공개 키 인증서가 취소되었는지 확인하는 방법으로, 전체 **CRL**을 다운로드할 필요가 없습니다. 이 방법은 취소된 인증서 일련 번호 목록을 제공하지만 잠재적으로 큰 파일을 다운로드해야 하는 전통적인 **CRL**보다 더 효율적입니다. CRL은 최대 512개의 항목을 포함할 수 있습니다. 더 많은 세부정보는 [여기](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm)에서 확인할 수 있습니다.

### **인증서 투명성이란**

인증서 투명성은 SSL 인증서의 발급 및 존재가 도메인 소유자, CA 및 사용자에게 가시화되도록 하여 인증서 관련 위협에 대응하는 데 도움을 줍니다. 그 목표는 다음과 같습니다:

* CA가 도메인 소유자의 지식 없이 도메인에 대한 SSL 인증서를 발급하는 것을 방지합니다.
* 실수로 또는 악의적으로 발급된 인증서를 추적하기 위한 공개 감사 시스템을 구축합니다.
* 사용자들을 사기성 인증서로부터 보호합니다.

#### **인증서 로그**

인증서 로그는 네트워크 서비스에 의해 유지되는 공개 감사 가능하고 추가 전용 기록입니다. 이러한 로그는 감사 목적으로 암호학적 증거를 제공합니다. 발급 기관과 대중 모두 이러한 로그에 인증서를 제출하거나 검증을 위해 쿼리할 수 있습니다. 로그 서버의 정확한 수는 고정되어 있지 않지만, 전 세계적으로 천 개 미만일 것으로 예상됩니다. 이러한 서버는 CA, ISP 또는 관심 있는 어떤 주체에 의해 독립적으로 관리될 수 있습니다.

#### **쿼리**

어떤 도메인에 대한 인증서 투명성 로그를 탐색하려면 [https://crt.sh/](https://crt.sh) 를 방문하세요.

인증서를 저장하는 다양한 형식이 있으며, 각 형식은 고유한 사용 사례와 호환성을 가지고 있습니다. 이 요약에서는 주요 형식을 다루고 이들 간의 변환에 대한 지침을 제공합니다.

## **형식**

### **PEM 형식**

* 인증서에 가장 널리 사용되는 형식입니다.
* 인증서와 개인 키를 위해 별도의 파일이 필요하며, Base64 ASCII로 인코딩됩니다.
* 일반적인 확장자: .cer, .crt, .pem, .key.
* 주로 Apache 및 유사한 서버에서 사용됩니다.

### **DER 형식**

* 인증서의 이진 형식입니다.
* PEM 파일에서 발견되는 "BEGIN/END CERTIFICATE" 문이 없습니다.
* 일반적인 확장자: .cer, .der.
* 종종 Java 플랫폼과 함께 사용됩니다.

### **P7B/PKCS#7 형식**

* Base64 ASCII로 저장되며, 확장자는 .p7b 또는 .p7c입니다.
* 개인 키를 제외하고 인증서와 체인 인증서만 포함됩니다.
* Microsoft Windows 및 Java Tomcat에서 지원됩니다.

### **PFX/P12/PKCS#12 형식**

* 서버 인증서, 중간 인증서 및 개인 키를 하나의 파일에 캡슐화하는 이진 형식입니다.
* 확장자: .pfx, .p12.
* 주로 Windows에서 인증서 가져오기 및 내보내기에 사용됩니다.

### **형식 변환**

**PEM 변환**은 호환성을 위해 필수적입니다:

* **x509 to PEM**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEM에서 DER로**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DER to PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEM에서 P7B로**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7를 PEM으로**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX 변환**은 Windows에서 인증서를 관리하는 데 중요합니다:

* **PFX에서 PEM으로**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX to PKCS#8**는 두 단계로 이루어집니다:
1. PFX를 PEM으로 변환
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. PEM을 PKCS8로 변환하기
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7B to PFX** 또한 두 개의 명령이 필요합니다:
1. P7B를 CER로 변환
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. CER 및 개인 키를 PFX로 변환하기
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)를 사용하여 세계에서 **가장 진보된** 커뮤니티 도구로 구동되는 **워크플로우**를 쉽게 구축하고 **자동화**하세요.\
오늘 바로 접근하세요:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

{% hint style="success" %}
AWS 해킹 배우기 및 연습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우기 및 연습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 계획**](https://github.com/sponsors/carlospolop) 확인하기!
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 리포지토리에 PR을 제출하여 해킹 팁을 공유하세요.**

</details>
{% endhint %}
