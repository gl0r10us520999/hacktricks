{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}


**原始帖子是** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## 摘要

发现当前用户可以写入两个注册表项：

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

建议使用 **regedit GUI** 检查 **RpcEptMapper** 服务的权限，特别是 **高级安全设置** 窗口的 **有效权限** 选项卡。此方法可以评估特定用户或组的授予权限，而无需逐个检查每个访问控制条目 (ACE)。

一张截图显示了分配给低权限用户的权限，其中 **创建子项** 权限尤为显著。该权限也称为 **AppendData/AddSubdirectory**，与脚本的发现相符。

注意到无法直接修改某些值，但可以创建新的子项。一个例子是尝试更改 **ImagePath** 值，结果显示访问被拒绝的消息。

尽管存在这些限制，但通过利用 **RpcEptMapper** 服务的注册表结构中的 **Performance** 子项，识别出潜在的权限提升机会，该子项默认情况下不存在。这可能使 DLL 注册和性能监控成为可能。

查阅了有关 **Performance** 子项及其在性能监控中的使用文档，开发了一个概念验证 DLL。该 DLL 演示了 **OpenPerfData**、**CollectPerfData** 和 **ClosePerfData** 函数的实现，通过 **rundll32** 测试，确认其成功运行。

目标是强迫 **RPC Endpoint Mapper 服务** 加载构造的 Performance DLL。观察发现，通过 PowerShell 执行与性能数据相关的 WMI 类查询会创建一个日志文件，从而使任意代码在 **LOCAL SYSTEM** 上下文中执行，从而授予提升的权限。

强调了此漏洞的持久性和潜在影响，突显其在后期利用策略、横向移动和规避 antivirus/EDR 系统中的相关性。

尽管该漏洞最初是通过脚本无意中披露的，但强调其利用仅限于过时的 Windows 版本（例如 **Windows 7 / Server 2008 R2**），并且需要本地访问。

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}
