{% hint style="success" %}
AWS 해킹을 배우고 실습하세요:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹을 배우고 실습하세요: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* 해킹 팁을 공유하려면 **HackTricks** 및 **HackTricks Cloud** github 저장소로 PR을 제출하세요.

</details>
{% endhint %}


# BSSID 확인

WireShark를 사용하여 Wifi 주요 트래픽을 포함하는 캡처를 받았을 때, _Wireless --> WLAN Traffic_를 사용하여 캡처의 모든 SSID를 조사할 수 있습니다:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## 브루트 포스

해당 화면의 열 중 하나는 **pcap 내에서 인증이 발견되었는지 여부**를 나타냅니다. 그렇다면 `aircrack-ng`를 사용하여 브루트 포스를 시도할 수 있습니다:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
# 데이터 비콘 / 사이드 채널 내의 데이터 누출 의심

만약 **Wifi 네트워크의 비콘 내에서 데이터가 누출되고 있다고 의심**한다면 다음과 같은 필터를 사용하여 네트워크의 비콘을 확인할 수 있습니다: `wlan contains <네트워크이름>`, 또는 `wlan.ssid == "네트워크이름"` 필터된 패킷 내에서 의심스러운 문자열을 찾습니다.

# Wifi 네트워크 내의 알 수 없는 MAC 주소 찾기

다음 링크는 **Wifi 네트워크 내에서 데이터를 보내는 기기를 찾는 데 유용**할 것입니다:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

이미 알고 있는 **MAC 주소가 있다면 출력에서 제거**할 수 있도록 다음과 같은 확인 사항을 추가할 수 있습니다: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

네트워크 내에서 통신하는 **알 수 없는 MAC 주소를 감지**하면 다음과 같은 필터를 사용하여 해당 트래픽을 필터링할 수 있습니다: `wlan.addr==<MAC 주소> && (ftp || http || ssh || telnet)` ftp/http/ssh/telnet 필터는 트래픽을 해독했다면 유용합니다.

# 트래픽 해독

편집 --> 환경 설정 --> 프로토콜 --> IEEE 802.11--> 편집

![](<../../../.gitbook/assets/image (426).png>)
