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
- **`Software\Microsoft\Windows NT\CurrentVersion`** に位置し、Windowsのバージョン、サービスパック、インストール時間、登録された所有者の名前が簡潔に表示されます。

### **Computer Name**
- ホスト名は **`System\ControlSet001\Control\ComputerName\ComputerName`** にあります。

### **Time Zone Setting**
- システムのタイムゾーンは **`System\ControlSet001\Control\TimeZoneInformation`** に保存されています。

### **Access Time Tracking**
- デフォルトでは、最終アクセス時間の追跡はオフになっています (**`NtfsDisableLastAccessUpdate=1`**)。これを有効にするには、次のコマンドを使用します：
`fsutil behavior set disablelastaccess 0`

### Windows Versions and Service Packs
- **Windowsのバージョン** はエディション（例：Home、Pro）とそのリリース（例：Windows 10、Windows 11）を示し、**サービスパック** は修正や時には新機能を含む更新です。

### Enabling Last Access Time
- 最終アクセス時間の追跡を有効にすると、ファイルが最後に開かれた時刻を確認でき、法医学的分析やシステム監視にとって重要です。

### Network Information Details
- レジストリには、**ネットワークの種類（無線、ケーブル、3G）** や **ネットワークカテゴリ（パブリック、プライベート/ホーム、ドメイン/ワーク）** など、ネットワーク構成に関する詳細なデータが保存されており、ネットワークセキュリティ設定や権限を理解するために重要です。

### Client Side Caching (CSC)
- **CSC** は共有ファイルのコピーをキャッシュすることでオフラインファイルアクセスを向上させます。異なる **CSCFlags** 設定は、どのファイルがどのようにキャッシュされるかを制御し、特に接続が不安定な環境でのパフォーマンスやユーザー体験に影響を与えます。

### AutoStart Programs
- 様々な `Run` および `RunOnce` レジストリキーにリストされているプログラムは、起動時に自動的に起動され、システムのブート時間に影響を与え、マルウェアや不要なソフトウェアを特定するための興味深いポイントとなる可能性があります。

### Shellbags
- **Shellbags** はフォルダビューの設定を保存するだけでなく、フォルダが存在しなくてもフォルダアクセスの法医学的証拠を提供します。これは調査にとって非常に貴重で、他の手段では明らかでないユーザーの活動を明らかにします。

### USB Information and Forensics
- レジストリに保存されたUSBデバイスに関する詳細は、どのデバイスがコンピュータに接続されていたかを追跡するのに役立ち、機密ファイルの転送や不正アクセスの事件にデバイスを関連付ける可能性があります。

### Volume Serial Number
- **ボリュームシリアル番号** は、ファイルシステムの特定のインスタンスを追跡するのに重要であり、異なるデバイス間でファイルの起源を確立する必要がある法医学的シナリオで役立ちます。

### **Shutdown Details**
- シャットダウン時間とカウント（後者はXPのみ）は **`System\ControlSet001\Control\Windows`** および **`System\ControlSet001\Control\Watchdog\Display`** に保存されています。

### **Network Configuration**
- 詳細なネットワークインターフェース情報は **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`** を参照してください。
- VPN接続を含む最初と最後のネットワーク接続時間は、**`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`** のさまざまなパスに記録されています。

### **Shared Folders**
- 共有フォルダと設定は **`System\ControlSet001\Services\lanmanserver\Shares`** にあります。クライアントサイドキャッシング（CSC）設定はオフラインファイルの可用性を決定します。

### **Programs that Start Automatically**
- **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** のようなパスや、`Software\Microsoft\Windows\CurrentVersion` の下の類似のエントリは、起動時に実行されるプログラムを詳細に示します。

### **Searches and Typed Paths**
- エクスプローラーの検索と入力されたパスは、**`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** の下で WordwheelQuery と TypedPaths に記録されています。

### **Recent Documents and Office Files**
- 最近アクセスされた文書やOfficeファイルは、`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` および特定のOfficeバージョンのパスに記録されています。

### **Most Recently Used (MRU) Items**
- 最近使用されたファイルパスやコマンドを示すMRUリストは、`NTUSER.DAT` のさまざまな `ComDlg32` および `Explorer` サブキーに保存されています。

### **User Activity Tracking**
- User Assist機能は、実行回数や最終実行時間を含む詳細なアプリケーション使用統計を **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`** に記録します。

### **Shellbags Analysis**
- フォルダアクセスの詳細を明らかにするShellbagsは、`USRCLASS.DAT` および `NTUSER.DAT` の下の `Software\Microsoft\Windows\Shell` に保存されています。分析には **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** を使用してください。

### **USB Device History**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** および **`HKLM\SYSTEM\ControlSet001\Enum\USB`** には、接続されたUSBデバイスに関する豊富な詳細が含まれており、製造元、製品名、接続タイムスタンプが含まれます。
- 特定のUSBデバイスに関連付けられたユーザーは、デバイスの **{GUID}** を検索することで特定できます。
- 最後にマウントされたデバイスとそのボリュームシリアル番号は、それぞれ `System\MountedDevices` および `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt` を通じて追跡できます。

このガイドは、Windowsシステム上の詳細なシステム、ネットワーク、およびユーザー活動情報にアクセスするための重要なパスと方法を要約し、明確さと使いやすさを目指しています。



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
