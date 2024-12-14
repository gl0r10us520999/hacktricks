# Interesting Windows Registry Keys

### Interesting Windows Registry Keys

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


### **Windows Version and Owner Info**
- **`Software\Microsoft\Windows NT\CurrentVersion`**에 위치하며, Windows 버전, 서비스 팩, 설치 시간 및 등록된 소유자의 이름을 간단하게 확인할 수 있습니다.

### **Computer Name**
- 호스트 이름은 **`System\ControlSet001\Control\ComputerName\ComputerName`** 아래에서 찾을 수 있습니다.

### **Time Zone Setting**
- 시스템의 시간대는 **`System\ControlSet001\Control\TimeZoneInformation`**에 저장됩니다.

### **Access Time Tracking**
- 기본적으로 마지막 접근 시간 추적은 꺼져 있습니다 (**`NtfsDisableLastAccessUpdate=1`**). 이를 활성화하려면 다음을 사용하세요:
`fsutil behavior set disablelastaccess 0`

### Windows Versions and Service Packs
- **Windows 버전**은 에디션(예: Home, Pro)과 릴리스를 나타내며(예: Windows 10, Windows 11), **서비스 팩**은 수정 사항과 때때로 새로운 기능을 포함하는 업데이트입니다.

### Enabling Last Access Time
- 마지막 접근 시간 추적을 활성화하면 파일이 마지막으로 열렸던 시간을 확인할 수 있어, 포렌식 분석이나 시스템 모니터링에 중요할 수 있습니다.

### Network Information Details
- 레지스트리는 네트워크 구성에 대한 광범위한 데이터를 보유하고 있으며, 여기에는 **네트워크 유형(무선, 케이블, 3G)** 및 **네트워크 범주(공용, 개인/가정, 도메인/작업)**가 포함되어 있어 네트워크 보안 설정 및 권한을 이해하는 데 필수적입니다.

### Client Side Caching (CSC)
- **CSC**는 공유 파일의 복사본을 캐싱하여 오프라인 파일 접근을 향상시킵니다. 다양한 **CSCFlags** 설정은 어떤 파일이 어떻게 캐시되는지를 제어하여 성능과 사용자 경험에 영향을 미칩니다, 특히 간헐적인 연결이 있는 환경에서.

### AutoStart Programs
- 다양한 `Run` 및 `RunOnce` 레지스트리 키에 나열된 프로그램은 시작 시 자동으로 실행되며, 시스템 부팅 시간에 영향을 미치고 악성 소프트웨어나 원치 않는 소프트웨어를 식별하는 데 관심의 지점이 될 수 있습니다.

### Shellbags
- **Shellbags**는 폴더 보기 설정을 저장할 뿐만 아니라 폴더가 더 이상 존재하지 않더라도 폴더 접근에 대한 포렌식 증거를 제공합니다. 이는 조사를 위해 매우 귀중하며, 다른 방법으로는 명확하지 않은 사용자 활동을 드러냅니다.

### USB Information and Forensics
- 레지스트리에 저장된 USB 장치에 대한 세부정보는 어떤 장치가 컴퓨터에 연결되었는지를 추적하는 데 도움이 되며, 이는 민감한 파일 전송이나 무단 접근 사건과 연결될 수 있습니다.

### Volume Serial Number
- **볼륨 일련 번호**는 파일 시스템의 특정 인스턴스를 추적하는 데 중요할 수 있으며, 이는 다양한 장치 간에 파일 출처를 확인해야 하는 포렌식 시나리오에서 유용합니다.

### **Shutdown Details**
- 종료 시간 및 횟수(후자는 XP 전용)는 **`System\ControlSet001\Control\Windows`** 및 **`System\ControlSet001\Control\Watchdog\Display`**에 저장됩니다.

### **Network Configuration**
- 자세한 네트워크 인터페이스 정보는 **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**를 참조하세요.
- 첫 번째 및 마지막 네트워크 연결 시간, VPN 연결을 포함하여, 다양한 경로에 기록됩니다 **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`** 아래에.

### **Shared Folders**
- 공유 폴더 및 설정은 **`System\ControlSet001\Services\lanmanserver\Shares`** 아래에 있습니다. 클라이언트 측 캐싱(CSC) 설정은 오프라인 파일 가용성을 결정합니다.

### **Programs that Start Automatically**
- **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`**와 `Software\Microsoft\Windows\CurrentVersion` 아래의 유사한 항목은 시작 시 실행되도록 설정된 프로그램의 경로를 자세히 설명합니다.

### **Searches and Typed Paths**
- 탐색기 검색 및 입력된 경로는 **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** 아래에서 WordwheelQuery 및 TypedPaths에 대해 추적됩니다.

### **Recent Documents and Office Files**
- 최근에 접근한 문서 및 Office 파일은 `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` 및 특정 Office 버전 경로에 기록됩니다.

### **Most Recently Used (MRU) Items**
- 최근 파일 경로 및 명령을 나타내는 MRU 목록은 `NTUSER.DAT`의 다양한 `ComDlg32` 및 `Explorer` 하위 키에 저장됩니다.

### **User Activity Tracking**
- User Assist 기능은 실행 횟수 및 마지막 실행 시간을 포함한 자세한 애플리케이션 사용 통계를 **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**에 기록합니다.

### **Shellbags Analysis**
- 폴더 접근 세부정보를 드러내는 Shellbags는 `USRCLASS.DAT` 및 `NTUSER.DAT`의 `Software\Microsoft\Windows\Shell` 아래에 저장됩니다. 분석을 위해 **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)**를 사용하세요.

### **USB Device History**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** 및 **`HKLM\SYSTEM\ControlSet001\Enum\USB`**는 연결된 USB 장치에 대한 풍부한 세부정보를 포함하고 있으며, 여기에는 제조업체, 제품 이름 및 연결 타임스탬프가 포함됩니다.
- 특정 USB 장치와 관련된 사용자는 장치의 **{GUID}**에 대해 `NTUSER.DAT` 하이브를 검색하여 확인할 수 있습니다.
- 마지막으로 마운트된 장치와 그 볼륨 일련 번호는 각각 `System\MountedDevices` 및 `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`를 통해 추적할 수 있습니다.

이 가이드는 Windows 시스템에서 상세한 시스템, 네트워크 및 사용자 활동 정보를 접근하기 위한 중요한 경로와 방법을 요약하여 명확성과 사용성을 목표로 합니다.



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
