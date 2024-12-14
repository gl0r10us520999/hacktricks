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

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# Timestamps

공격자는 **파일의 타임스탬프를 변경**하여 탐지를 피하는 데 관심이 있을 수 있습니다.\
타임스탬프는 MFT의 속성 `$STANDARD_INFORMATION` __ 및 __ `$FILE_NAME` 내에서 찾을 수 있습니다.

두 속성 모두 4개의 타임스탬프를 가지고 있습니다: **수정**, **접근**, **생성**, 및 **MFT 레지스트리 수정** (MACE 또는 MACB).

**Windows 탐색기** 및 기타 도구는 **`$STANDARD_INFORMATION`**의 정보를 표시합니다.

## TimeStomp - Anti-forensic Tool

이 도구는 **`$STANDARD_INFORMATION`** 내의 타임스탬프 정보를 **수정**하지만 **`$FILE_NAME`** 내의 정보는 **수정하지 않습니다**. 따라서 **의심스러운** **활동을 식별**할 수 있습니다.

## Usnjrnl

**USN 저널** (Update Sequence Number Journal)은 NTFS (Windows NT 파일 시스템)의 기능으로, 볼륨 변경 사항을 추적합니다. [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) 도구를 사용하면 이러한 변경 사항을 검사할 수 있습니다.

![](<../../.gitbook/assets/image (449).png>)

이전 이미지는 **도구**에서 표시된 **출력**으로, 파일에 대해 **일부 변경이 수행되었음을** 관찰할 수 있습니다.

## $LogFile

**파일 시스템에 대한 모든 메타데이터 변경 사항은** [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead_logging)이라는 프로세스에 기록됩니다. 기록된 메타데이터는 NTFS 파일 시스템의 루트 디렉토리에 위치한 `**$LogFile**`이라는 파일에 저장됩니다. [LogFileParser](https://github.com/jschicht/LogFileParser)와 같은 도구를 사용하여 이 파일을 구문 분석하고 변경 사항을 식별할 수 있습니다.

![](<../../.gitbook/assets/image (450).png>)

다시 말해, 도구의 출력에서 **일부 변경이 수행되었음을** 확인할 수 있습니다.

같은 도구를 사용하여 **타임스탬프가 수정된 시간을 식별**할 수 있습니다:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: 파일의 생성 시간
* ATIME: 파일의 수정 시간
* MTIME: 파일의 MFT 레지스트리 수정
* RTIME: 파일의 접근 시간

## `$STANDARD_INFORMATION` 및 `$FILE_NAME` 비교

의심스러운 수정된 파일을 식별하는 또 다른 방법은 두 속성의 시간을 비교하여 **불일치를 찾는 것**입니다.

## 나노초

**NTFS** 타임스탬프는 **100 나노초**의 **정밀도**를 가지고 있습니다. 따라서 2010-10-10 10:10:**00.000:0000와 같은 타임스탬프를 가진 파일을 찾는 것은 매우 **의심스럽습니다**.

## SetMace - Anti-forensic Tool

이 도구는 두 속성 `$STARNDAR_INFORMATION` 및 `$FILE_NAME`을 수정할 수 있습니다. 그러나 Windows Vista부터는 이 정보를 수정하기 위해 라이브 OS가 필요합니다.

# Data Hiding

NFTS는 클러스터와 최소 정보 크기를 사용합니다. 즉, 파일이 클러스터와 반 개를 사용하면, **남은 반 개는 파일이 삭제될 때까지 절대 사용되지 않습니다**. 따라서 이 여유 공간에 **데이터를 숨길 수 있습니다**.

slacker와 같은 도구를 사용하면 이 "숨겨진" 공간에 데이터를 숨길 수 있습니다. 그러나 `$logfile` 및 `$usnjrnl`의 분석을 통해 일부 데이터가 추가되었음을 보여줄 수 있습니다:

![](<../../.gitbook/assets/image (452).png>)

그런 다음 FTK Imager와 같은 도구를 사용하여 여유 공간을 복구할 수 있습니다. 이러한 종류의 도구는 내용을 난독화하거나 심지어 암호화된 상태로 저장할 수 있습니다.

# UsbKill

이 도구는 **USB** 포트에서 변경 사항이 감지되면 컴퓨터를 **꺼**는 도구입니다.\
이를 발견하는 방법은 실행 중인 프로세스를 검사하고 **실행 중인 각 파이썬 스크립트를 검토하는 것**입니다.

# Live Linux Distributions

이 배포판은 **RAM** 메모리 내에서 **실행됩니다**. 이를 감지하는 유일한 방법은 **NTFS 파일 시스템이 쓰기 권한으로 마운트된 경우**입니다. 읽기 권한으로만 마운트되면 침입을 감지할 수 없습니다.

# Secure Deletion

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Windows Configuration

여러 Windows 로깅 방법을 비활성화하여 포렌식 조사를 훨씬 더 어렵게 만들 수 있습니다.

## Disable Timestamps - UserAssist

이것은 사용자가 각 실행 파일을 실행한 날짜와 시간을 유지하는 레지스트리 키입니다.

UserAssist를 비활성화하려면 두 단계를 수행해야 합니다:

1. 두 개의 레지스트리 키, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` 및 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`를 모두 0으로 설정하여 UserAssist를 비활성화하겠다는 신호를 보냅니다.
2. `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`와 같은 레지스트리 하위 트리를 지웁니다.

## Disable Timestamps - Prefetch

이것은 Windows 시스템의 성능을 향상시키기 위해 실행된 애플리케이션에 대한 정보를 저장합니다. 그러나 이것은 포렌식 관행에도 유용할 수 있습니다.

* `regedit` 실행
* 파일 경로 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters` 선택
* `EnablePrefetcher` 및 `EnableSuperfetch`를 마우스 오른쪽 버튼으로 클릭
* 각 항목에서 값을 1(또는 3)에서 0으로 변경하기 위해 수정 선택
* 재시작

## Disable Timestamps - Last Access Time

NTFS 볼륨에서 폴더가 열릴 때마다 시스템은 **각 나열된 폴더의 타임스탬프 필드를 업데이트하는 데 시간을 소요합니다**, 이를 마지막 접근 시간이라고 합니다. 사용량이 많은 NTFS 볼륨에서는 성능에 영향을 줄 수 있습니다.

1. 레지스트리 편집기(Regedit.exe)를 엽니다.
2. `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`으로 이동합니다.
3. `NtfsDisableLastAccessUpdate`를 찾습니다. 존재하지 않으면 이 DWORD를 추가하고 값을 1로 설정하여 프로세스를 비활성화합니다.
4. 레지스트리 편집기를 닫고 서버를 재부팅합니다.

## Delete USB History

모든 **USB 장치 항목**은 USB 장치를 PC 또는 노트북에 연결할 때 생성되는 하위 키를 포함하는 **USBSTOR** 레지스트리 키 아래에 Windows 레지스트리에 저장됩니다. 이 키는 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`에 있습니다. **이것을 삭제하면** USB 기록이 삭제됩니다.\
또한 [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) 도구를 사용하여 삭제되었는지 확인할 수 있습니다 (그리고 삭제할 수 있습니다).

USB에 대한 정보를 저장하는 또 다른 파일은 `C:\Windows\INF` 내의 `setupapi.dev.log` 파일입니다. 이것도 삭제해야 합니다.

## Disable Shadow Copies

**쉐도우 복사** 목록을 보려면 `vssadmin list shadowstorage`를 실행하세요.\
**삭제**하려면 `vssadmin delete shadow`를 실행하세요.

GUI를 통해 삭제하려면 [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)에서 제안된 단계를 따르세요.

쉐도우 복사를 비활성화하려면 [여기에서 단계](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows)를 따르세요:

1. Windows 시작 버튼을 클릭한 후 텍스트 검색 상자에 "services"를 입력하여 서비스 프로그램을 엽니다.
2. 목록에서 "Volume Shadow Copy"를 찾아 선택한 후 마우스 오른쪽 버튼을 클릭하여 속성에 접근합니다.
3. "시작 유형" 드롭다운 메뉴에서 비활성화를 선택하고 변경 사항을 적용하고 확인을 클릭합니다.

어떤 파일이 쉐도우 복사에 복사될지를 레지스트리 `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`에서 수정할 수도 있습니다.

## Overwrite deleted files

* **Windows 도구**를 사용할 수 있습니다: `cipher /w:C` 이는 C 드라이브 내의 사용되지 않는 디스크 공간에서 모든 데이터를 제거하도록 지시합니다.
* [**Eraser**](https://eraser.heidi.ie)와 같은 도구를 사용할 수도 있습니다.

## Delete Windows event logs

* Windows + R --> eventvwr.msc --> "Windows Logs" 확장 --> 각 카테고리를 마우스 오른쪽 버튼으로 클릭하고 "Clear Log" 선택
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Disable Windows event logs

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* 서비스 섹션에서 "Windows Event Log" 서비스를 비활성화합니다.
* `WEvtUtil.exec clear-log` 또는 `WEvtUtil.exe cl`

## Disable $UsnJrnl

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


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
