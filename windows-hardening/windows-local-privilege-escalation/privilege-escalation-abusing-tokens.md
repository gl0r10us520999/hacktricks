# 滥用令牌

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在 Twitter 上关注** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}

## 令牌

如果你**不知道 Windows 访问令牌是什么**，请在继续之前阅读此页面：

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

**也许你可以通过滥用你已经拥有的令牌来提升权限**

### SeImpersonatePrivilege

这是任何进程持有的特权，允许对任何令牌进行 impersonation（但不允许创建），前提是可以获得其句柄。可以通过诱使 Windows 服务（DCOM）对一个漏洞执行 NTLM 认证来获取特权令牌，从而启用以 SYSTEM 权限执行进程。可以使用各种工具利用此漏洞，例如 [juicy-potato](https://github.com/ohpe/juicy-potato)、[RogueWinRM](https://github.com/antonioCoco/RogueWinRM)（需要禁用 winrm）、[SweetPotato](https://github.com/CCob/SweetPotato) 和 [PrintSpoofer](https://github.com/itm4n/PrintSpoofer)。

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="juicypotato.md" %}
[juicypotato.md](juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege

它与 **SeImpersonatePrivilege** 非常相似，将使用 **相同的方法** 来获取特权令牌。\
然后，此特权允许**将主令牌分配**给新的/挂起的进程。使用特权 impersonation 令牌可以派生出主令牌（DuplicateTokenEx）。\
使用该令牌，可以通过 'CreateProcessAsUser' 创建一个 **新进程**，或创建一个挂起的进程并**设置令牌**（通常，无法修改正在运行的进程的主令牌）。

### SeTcbPrivilege

如果你启用了此令牌，可以使用 **KERB\_S4U\_LOGON** 为任何其他用户获取 **impersonation 令牌**，而无需知道凭据，**向令牌添加任意组**（管理员），将令牌的 **完整性级别** 设置为 "**中等**"，并将此令牌分配给 **当前线程**（SetThreadToken）。

### SeBackupPrivilege

此特权使系统能够**授予对任何文件的所有读取访问**控制（仅限读取操作）。它用于**从注册表中读取本地管理员**帐户的密码哈希，随后可以使用像 "**psexec**" 或 "**wmiexec**" 这样的工具与哈希一起使用（Pass-the-Hash 技术）。然而，在以下两种情况下，此技术会失败：当本地管理员帐户被禁用，或当有政策限制本地管理员的远程连接的管理权限时。\
你可以通过以下方式**滥用此特权**：

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* 关注 **IppSec** 在 [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec)
* 或如在以下内容中所述的 **使用备份操作员提升权限** 部分：

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege

此特权提供对任何系统文件的 **写访问** 权限，无论文件的访问控制列表（ACL）如何。它为提升权限打开了许多可能性，包括**修改服务**、执行 DLL 劫持以及通过图像文件执行选项设置 **调试器**等多种技术。

### SeCreateTokenPrivilege

SeCreateTokenPrivilege 是一种强大的权限，特别是在用户具备 impersonate 令牌的能力时，但在缺乏 SeImpersonatePrivilege 的情况下也很有用。此能力依赖于能够 impersonate 代表同一用户的令牌，并且其完整性级别不超过当前进程的完整性级别。

**关键点：**

* **在没有 SeImpersonatePrivilege 的情况下进行 impersonation：** 可以利用 SeCreateTokenPrivilege 在特定条件下通过 impersonate 令牌实现权限提升。
* **令牌 impersonation 的条件：** 成功的 impersonation 要求目标令牌属于同一用户，并且其完整性级别小于或等于尝试 impersonation 的进程的完整性级别。
* **创建和修改 impersonation 令牌：** 用户可以创建一个 impersonation 令牌，并通过添加特权组的 SID（安全标识符）来增强它。

### SeLoadDriverPrivilege

此特权允许**加载和卸载设备驱动程序**，通过创建具有特定值的注册表项 `ImagePath` 和 `Type`。由于对 `HKLM`（HKEY\_LOCAL\_MACHINE）的直接写访问受到限制，因此必须使用 `HKCU`（HKEY\_CURRENT\_USER）。然而，为了使 `HKCU` 对内核可识别以进行驱动程序配置，必须遵循特定路径。

此路径为 `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName`，其中 `<RID>` 是当前用户的相对标识符。在 `HKCU` 中，必须创建整个路径，并设置两个值：

* `ImagePath`，即要执行的二进制文件的路径
* `Type`，值为 `SERVICE_KERNEL_DRIVER`（`0x00000001`）。

**遵循的步骤：**

1. 由于写访问受限，访问 `HKCU` 而不是 `HKLM`。
2. 在 `HKCU` 中创建路径 `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName`，其中 `<RID>` 代表当前用户的相对标识符。
3. 将 `ImagePath` 设置为二进制文件的执行路径。
4. 将 `Type` 设置为 `SERVICE_KERNEL_DRIVER`（`0x00000001`）。
```python
# Example Python code to set the registry values
import winreg as reg

# Define the path and values
path = r'Software\YourPath\System\CurrentControlSet\Services\DriverName' # Adjust 'YourPath' as needed
key = reg.OpenKey(reg.HKEY_CURRENT_USER, path, 0, reg.KEY_WRITE)
reg.SetValueEx(key, "ImagePath", 0, reg.REG_SZ, "path_to_binary")
reg.SetValueEx(key, "Type", 0, reg.REG_DWORD, 0x00000001)
reg.CloseKey(key)
```
更多滥用此权限的方法在 [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege)

### SeTakeOwnershipPrivilege

这与 **SeRestorePrivilege** 类似。其主要功能允许一个进程 **假定对象的所有权**，绕过通过提供 WRITE\_OWNER 访问权限的明确自由裁量访问要求。该过程首先确保获得所需注册表项的所有权以进行写入，然后更改 DACL 以启用写入操作。
```bash
takeown /f 'C:\some\file.txt' #Now the file is owned by you
icacls 'C:\some\file.txt' /grant <your_username>:F #Now you have full access
# Use this with files that might contain credentials such as
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software
%WINDIR%\repair\security
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
c:\inetpub\wwwwroot\web.config
```
### SeDebugPrivilege

此权限允许**调试其他进程**，包括读取和写入内存。可以利用此权限采用各种内存注入策略，能够规避大多数杀毒软件和主机入侵防御解决方案。

#### Dump memory

您可以使用[ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)来自[SysInternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)来**捕获进程的内存**。具体来说，这可以应用于**本地安全授权子系统服务（**[**LSASS**](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)**）**进程，该进程负责在用户成功登录系统后存储用户凭据。

然后，您可以在mimikatz中加载此转储以获取密码：
```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```
#### RCE

如果你想获得一个 `NT SYSTEM` shell，你可以使用：

* [**SeDebugPrivilege-Exploit (C++)**](https://github.com/bruno-1337/SeDebugPrivilege-Exploit)
* [**SeDebugPrivilegePoC (C#)**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)
* [**psgetsys.ps1 (Powershell Script)**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)
```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```
## 检查权限
```
whoami /priv
```
**显示为禁用的令牌**可以被启用，您实际上可以利用_启用_和_禁用_令牌。

### 启用所有令牌

如果您有禁用的令牌，可以使用脚本[**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1)来启用所有令牌：
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
或嵌入在这个[**帖子**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/)中的**脚本**。

## 表格

完整的令牌权限备忘单在[https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)，下面的摘要将仅列出直接利用该权限以获得管理员会话或读取敏感文件的方法。

| 权限                      | 影响        | 工具                    | 执行路径                                                                                                                                                                                                                                                                                                                                     | 备注                                                                                                                                                                                                                                                                                                                        |
| ------------------------ | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**管理员**_ | 第三方工具              | _"这将允许用户模拟令牌并使用诸如potato.exe、rottenpotato.exe和juicypotato.exe等工具提升到nt系统"_                                                                                                                                                                                                      | 感谢[Aurélien Chalot](https://twitter.com/Defte\_)的更新。我会尽快尝试将其重新表述为更像食谱的内容。                                                                                                                                                                                        |
| **`SeBackup`**           | **威胁**    | _**内置命令**_         | 使用`robocopy /b`读取敏感文件                                                                                                                                                                                                                                                                                                             | <p>- 如果您可以读取%WINDIR%\MEMORY.DMP，可能会更有趣<br><br>- <code>SeBackupPrivilege</code>（和robocopy）在打开文件时没有帮助。<br><br>- Robocopy需要同时具有SeBackup和SeRestore才能使用/b参数。</p>                                                                      |
| **`SeCreateToken`**      | _**管理员**_ | 第三方工具              | 使用`NtCreateToken`创建任意令牌，包括本地管理员权限。                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**            | _**管理员**_ | **PowerShell**          | 复制`lsass.exe`令牌。                                                                                                                                                                                                                                                                                                                   | 脚本可以在[FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1)找到                                                                                                                                                                                                         |
| **`SeLoadDriver`**       | _**管理员**_ | 第三方工具              | <p>1. 加载有缺陷的内核驱动程序，如<code>szkg64.sys</code><br>2. 利用驱动程序漏洞<br><br>或者，该权限可用于卸载与安全相关的驱动程序，使用<code>ftlMC</code>内置命令。即：<code>fltMC sysmondrv</code></p>                                                                           | <p>1. <code>szkg64</code>漏洞被列为<a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a><br>2. <code>szkg64</code> <a href="https://www.greyhathacker.net/?p=1025">利用代码</a>由<a href="https://twitter.com/parvezghh">Parvez Anwar</a>创建</p> |
| **`SeRestore`**          | _**管理员**_ | **PowerShell**          | <p>1. 启动具有SeRestore权限的PowerShell/ISE。<br>2. 使用<a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a>启用该权限。<br>3. 将utilman.exe重命名为utilman.old<br>4. 将cmd.exe重命名为utilman.exe<br>5. 锁定控制台并按Win+U</p> | <p>攻击可能会被某些AV软件检测到。</p><p>替代方法依赖于使用相同权限替换存储在“Program Files”中的服务二进制文件</p>                                                                                                                                                            |
| **`SeTakeOwnership`**    | _**管理员**_ | _**内置命令**_         | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. 将cmd.exe重命名为utilman.exe<br>4. 锁定控制台并按Win+U</p>                                                                                                                                       | <p>攻击可能会被某些AV软件检测到。</p><p>替代方法依赖于使用相同权限替换存储在“Program Files”中的服务二进制文件。</p>                                                                                                                                                           |
| **`SeTcb`**              | _**管理员**_ | 第三方工具              | <p>操纵令牌以包含本地管理员权限。可能需要SeImpersonate。</p><p>待验证。</p>                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                |

## 参考

* 查看定义Windows令牌的此表： [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* 查看关于使用令牌进行privesc的[**这篇论文**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt)。

{% hint style="success" %}
学习和实践AWS黑客攻击：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践GCP黑客攻击： <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f)或[**电报群组**](https://t.me/peass)或**在Twitter上关注** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github库提交PR来分享黑客技巧。

</details>
{% endhint %}
