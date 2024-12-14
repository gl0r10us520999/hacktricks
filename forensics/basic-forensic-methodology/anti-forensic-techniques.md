{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# 时间戳

攻击者可能会对**更改文件的时间戳**感兴趣，以避免被检测。\
可以在 MFT 中的属性 `$STANDARD_INFORMATION` __ 和 __ `$FILE_NAME` 中找到时间戳。

这两个属性都有 4 个时间戳：**修改**、**访问**、**创建**和 **MFT 注册修改**（MACE 或 MACB）。

**Windows 资源管理器**和其他工具显示来自 **`$STANDARD_INFORMATION`** 的信息。

## TimeStomp - 反取证工具

该工具**修改** **`$STANDARD_INFORMATION`** 中的时间戳信息**但**不修改 **`$FILE_NAME`** 中的信息。因此，可以**识别** **可疑** **活动**。

## Usnjrnl

**USN 日志**（更新序列号日志）是 NTFS（Windows NT 文件系统）的一个特性，用于跟踪卷更改。 [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) 工具允许检查这些更改。

![](<../../.gitbook/assets/image (449).png>)

上图是**工具**显示的**输出**，可以观察到对文件进行了**某些更改**。

## $LogFile

**对文件系统的所有元数据更改都被记录**在一个称为 [预写日志](https://en.wikipedia.org/wiki/Write-ahead_logging) 的过程中。记录的元数据保存在名为 `**$LogFile**` 的文件中，该文件位于 NTFS 文件系统的根目录。可以使用 [LogFileParser](https://github.com/jschicht/LogFileParser) 等工具解析此文件并识别更改。

![](<../../.gitbook/assets/image (450).png>)

同样，在工具的输出中可以看到**进行了某些更改**。

使用同一工具可以识别**时间戳被修改到哪个时间**：

![](<../../.gitbook/assets/image (451).png>)

* CTIME：文件的创建时间
* ATIME：文件的修改时间
* MTIME：文件的 MFT 注册修改
* RTIME：文件的访问时间

## `$STANDARD_INFORMATION` 和 `$FILE_NAME` 比较

识别可疑修改文件的另一种方法是比较两个属性上的时间，寻找**不匹配**。

## 纳秒

**NTFS** 时间戳的**精度**为**100 纳秒**。因此，找到时间戳如 2010-10-10 10:10:**00.000:0000 的文件是非常可疑的。

## SetMace - 反取证工具

该工具可以修改两个属性 `$STARNDAR_INFORMATION` 和 `$FILE_NAME`。然而，从 Windows Vista 开始，必须在活动操作系统中修改此信息。

# 数据隐藏

NFTS 使用集群和最小信息大小。这意味着如果一个文件占用一个集群和一半，**剩下的一半将永远不会被使用**，直到文件被删除。因此，可以在这个松弛空间中**隐藏数据**。

有像 slacker 这样的工具可以在这个“隐藏”空间中隐藏数据。然而，对 `$logfile` 和 `$usnjrnl` 的分析可以显示某些数据被添加：

![](<../../.gitbook/assets/image (452).png>)

然后，可以使用像 FTK Imager 这样的工具检索松弛空间。请注意，这种工具可以保存内容为模糊或甚至加密。

# UsbKill

这是一个工具，如果检测到 USB 端口的任何更改，将**关闭计算机**。\
发现这一点的一种方法是检查正在运行的进程并**审查每个正在运行的 Python 脚本**。

# 实时 Linux 发行版

这些发行版在**RAM**内存中**执行**。检测它们的唯一方法是**如果 NTFS 文件系统以写入权限挂载**。如果仅以读取权限挂载，则无法检测到入侵。

# 安全删除

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Windows 配置

可以禁用多种 Windows 日志记录方法，以使取证调查变得更加困难。

## 禁用时间戳 - UserAssist

这是一个注册表项，维护用户运行每个可执行文件的日期和时间。

禁用 UserAssist 需要两个步骤：

1. 设置两个注册表项，`HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` 和 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`，都设置为零，以表示我们希望禁用 UserAssist。
2. 清除看起来像 `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>` 的注册表子树。

## 禁用时间戳 - Prefetch

这将保存有关执行的应用程序的信息，目的是提高 Windows 系统的性能。然而，这对于取证实践也可能有用。

* 执行 `regedit`
* 选择文件路径 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* 右键单击 `EnablePrefetcher` 和 `EnableSuperfetch`
* 选择修改，将每个值从 1（或 3）更改为 0
* 重启

## 禁用时间戳 - 最后访问时间

每当从 NTFS 卷打开文件夹时，系统会花时间**更新每个列出文件夹的时间戳字段**，称为最后访问时间。在使用频繁的 NTFS 卷上，这可能会影响性能。

1. 打开注册表编辑器 (Regedit.exe)。
2. 浏览到 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`。
3. 查找 `NtfsDisableLastAccessUpdate`。如果不存在，请添加此 DWORD 并将其值设置为 1，这将禁用该过程。
4. 关闭注册表编辑器，并重启服务器。

## 删除 USB 历史

所有**USB 设备条目**都存储在 Windows 注册表中的 **USBSTOR** 注册表项下，该项包含在您将 USB 设备插入 PC 或笔记本电脑时创建的子项。您可以在这里找到此项 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`。**删除此项**将删除 USB 历史。\
您还可以使用工具 [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) 确保您已删除它们（并删除它们）。

另一个保存 USB 信息的文件是 `C:\Windows\INF` 中的 `setupapi.dev.log`。这也应该被删除。

## 禁用影子副本

**列出**影子副本使用 `vssadmin list shadowstorage`\
**删除**它们运行 `vssadmin delete shadow`

您还可以通过 GUI 删除它们，按照 [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html) 中提出的步骤。

要禁用影子副本，请参见 [这里的步骤](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows)：

1. 通过在单击 Windows 开始按钮后在文本搜索框中输入“services”打开服务程序。
2. 从列表中找到“卷影复制”，选择它，然后右键单击访问属性。
3. 从“启动类型”下拉菜单中选择禁用，然后通过单击应用和确定确认更改。

还可以在注册表中修改将要在影子副本中复制的文件的配置 `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

## 重写已删除的文件

* 您可以使用**Windows 工具**：`cipher /w:C` 这将指示 cipher 从 C 盘的可用未使用磁盘空间中删除任何数据。
* 您还可以使用像 [**Eraser**](https://eraser.heidi.ie) 这样的工具。

## 删除 Windows 事件日志

* Windows + R --> eventvwr.msc --> 展开“Windows 日志” --> 右键单击每个类别并选择“清除日志”
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## 禁用 Windows 事件日志

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* 在服务部分禁用“Windows 事件日志”服务
* `WEvtUtil.exec clear-log` 或 `WEvtUtil.exe cl`

## 禁用 $UsnJrnl

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}
