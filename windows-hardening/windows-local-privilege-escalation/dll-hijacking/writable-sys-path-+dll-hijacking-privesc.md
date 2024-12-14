# Writable Sys Path +Dll Hijacking Privesc

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

## 介绍

如果你发现你可以**在系统路径文件夹中写入**（注意，如果你只能在用户路径文件夹中写入，这将无效），那么你可能可以**提升权限**。

为了做到这一点，你可以利用**Dll劫持**，你将**劫持一个被服务或进程加载的库**，该服务或进程的**权限高于你的**，而且因为该服务正在加载一个可能在整个系统中根本不存在的Dll，它将尝试从你可以写入的系统路径加载它。

有关**什么是Dll劫持**的更多信息，请查看：

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## 使用Dll劫持进行权限提升

### 查找缺失的Dll

你需要做的第一件事是**识别一个运行权限高于你的进程**，该进程正在尝试**从你可以写入的系统路径加载Dll**。

在这种情况下的问题是，这些进程可能已经在运行。要找出哪些Dll缺失，你需要尽快启动procmon（在进程加载之前）。因此，要查找缺失的.dll，请执行：

* **创建**文件夹`C:\privesc_hijacking`并将路径`C:\privesc_hijacking`添加到**系统路径环境变量**。你可以**手动**完成此操作或使用**PS**：
```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```
* 启动 **`procmon`**，然后转到 **`Options`** --> **`Enable boot logging`**，在提示中按 **`OK`**。
* 然后，**重启**。当计算机重新启动时，**`procmon`** 将尽快开始 **记录** 事件。
* 一旦 **Windows** 启动，再次执行 **`procmon`**，它会告诉你它已经在运行，并会 **询问你是否想将** 事件存储在文件中。选择 **是** 并 **将事件存储在文件中**。
* **在** **文件** 生成后，**关闭** 打开的 **`procmon`** 窗口并 **打开事件文件**。
* 添加这些 **过滤器**，你将找到一些 **进程尝试从可写的系统路径文件夹加载的所有Dll**：

<figure><img src="../../../.gitbook/assets/image (945).png" alt=""><figcaption></figcaption></figure>

### 漏掉的 Dlls

在一台免费的 **虚拟 (vmware) Windows 11 机器** 上运行此命令，我得到了以下结果：

<figure><img src="../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

在这种情况下，.exe 是无用的，所以忽略它们，漏掉的 DLL 来自：

| 服务                             | Dll                | CMD 行                                                               |
| -------------------------------- | ------------------ | -------------------------------------------------------------------- |
| 任务调度程序 (Schedule)          | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| 诊断策略服务 (DPS)              | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                              | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

找到这个之后，我发现了一篇有趣的博客文章，它也解释了如何 [**滥用 WptsExtensions.dll 进行权限提升**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll)。这正是我们 **现在要做的**。

### 利用

因此，为了 **提升权限**，我们将劫持库 **WptsExtensions.dll**。拥有 **路径** 和 **名称** 后，我们只需 **生成恶意 dll**。

你可以 [**尝试使用这些示例中的任何一个**](./#creating-and-compiling-dlls)。你可以运行有效载荷，例如：获取反向 shell，添加用户，执行信标...

{% hint style="warning" %}
请注意 **并非所有服务都以** **`NT AUTHORITY\SYSTEM`** 运行，有些服务也以 **`NT AUTHORITY\LOCAL SERVICE`** 运行，该服务具有 **较少的权限**，你 **将无法创建新用户** 来滥用其权限。\
然而，该用户具有 **`seImpersonate`** 权限，因此你可以使用[ **potato suite 来提升权限**](../roguepotato-and-printspoofer.md)。因此，在这种情况下，反向 shell 是比尝试创建用户更好的选择。
{% endhint %}

在撰写时，**任务调度程序** 服务以 **Nt AUTHORITY\SYSTEM** 运行。

在 **生成恶意 Dll** 后（_在我的情况下，我使用了 x64 反向 shell，并得到了一个 shell，但防御者杀死了它，因为它来自 msfvenom_），将其保存到可写的系统路径中，命名为 **WptsExtensions.dll**，然后 **重启** 计算机（或重启服务，或做任何需要的事情以重新运行受影响的服务/程序）。

当服务重新启动时，**dll 应该被加载和执行**（你可以 **重用** **procmon** 技巧来检查 **库是否按预期加载**）。

{% hint style="success" %}
学习和实践 AWS 黑客攻击：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客攻击：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **在 Twitter 上关注** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
