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


# 카빙 도구

## 오토프시

이미지에서 파일을 추출하는 데 가장 일반적으로 사용되는 포렌식 도구는 [**Autopsy**](https://www.autopsy.com/download/)입니다. 다운로드하여 설치한 후 파일을 가져와 "숨겨진" 파일을 찾습니다. Autopsy는 디스크 이미지 및 기타 종류의 이미지를 지원하도록 설계되었지만 단순 파일은 지원하지 않습니다.

## 빈워크 <a id="binwalk"></a>

**Binwalk**는 이미지 및 오디오 파일과 같은 이진 파일에서 내장된 파일 및 데이터를 검색하는 도구입니다. `apt`로 설치할 수 있으며, [소스](https://github.com/ReFirmLabs/binwalk)는 github에서 찾을 수 있습니다.  
**유용한 명령어**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

또 다른 일반적인 도구는 **foremost**입니다. foremost의 구성 파일은 `/etc/foremost.conf`에 있습니다. 특정 파일을 검색하려면 주석을 제거하면 됩니다. 아무것도 주석을 제거하지 않으면 foremost는 기본적으로 구성된 파일 유형을 검색합니다.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **스칼펠**

**스칼펠**은 **파일에 포함된 파일**을 찾고 추출하는 데 사용할 수 있는 또 다른 도구입니다. 이 경우 추출하려는 파일 유형을 구성 파일 \(_/etc/scalpel/scalpel.conf_\)에서 주석을 제거해야 합니다.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

이 도구는 칼리 안에 포함되어 있지만, 여기에서 찾을 수 있습니다: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

이 도구는 이미지를 스캔하고 그 안에서 **pcaps**를 **추출**하며, **네트워크 정보\(URLs, 도메인, IPs, MACs, 메일\)** 및 더 많은 **파일**을 추출할 수 있습니다. 당신이 해야 할 일은:
```text
bulk_extractor memory.img -o out_folder
```
모든 정보를 탐색하십시오 \(비밀번호?\), 패킷을 분석하십시오 \(읽기[ **Pcaps 분석**](../pcap-inspection/)\), 이상한 도메인을 검색하십시오 \(**악성코드** 또는 **존재하지 않는** 도메인과 관련된\).

## PhotoRec

[https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)에서 찾을 수 있습니다.

GUI 및 CLI 버전이 함께 제공됩니다. PhotoRec이 검색할 **파일 유형**을 선택할 수 있습니다.

![](../../../.gitbook/assets/image%20%28524%29.png)

# 특정 데이터 카빙 도구

## FindAES

키 스케줄을 검색하여 AES 키를 검색합니다. TrueCrypt 및 BitLocker에서 사용하는 것과 같은 128, 192 및 256 비트 키를 찾을 수 있습니다.

[여기에서 다운로드](https://sourceforge.net/projects/findaes/)하십시오.

# 보조 도구

[**viu**](https://github.com/atanunq/viu)를 사용하여 터미널에서 이미지를 볼 수 있습니다.  
리눅스 명령줄 도구 **pdftotext**를 사용하여 PDF를 텍스트로 변환하고 읽을 수 있습니다.



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
