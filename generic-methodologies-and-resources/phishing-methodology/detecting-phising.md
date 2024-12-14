# 检测钓鱼

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

## 介绍

要检测钓鱼尝试，重要的是 **了解当前使用的钓鱼技术**。在此帖子的父页面上，您可以找到这些信息，因此如果您不知道今天使用了哪些技术，我建议您访问父页面并至少阅读该部分。

这篇文章基于这样的想法：**攻击者会试图以某种方式模仿或使用受害者的域名**。如果您的域名是 `example.com`，而您被钓鱼使用了一个完全不同的域名，例如 `youwonthelottery.com`，这些技术将无法揭示它。

## 域名变体

揭露那些在电子邮件中使用 **相似域名** 的 **钓鱼** 尝试是相对 **简单** 的。\
只需 **生成一份攻击者可能使用的最可能的钓鱼名称列表**，并 **检查** 它是否 **已注册**，或者检查是否有任何 **IP** 使用它。

### 查找可疑域名

为此，您可以使用以下任何工具。请注意，这些工具还会自动执行 DNS 请求，以检查域名是否分配了任何 IP：

* [**dnstwist**](https://github.com/elceef/dnstwist)
* [**urlcrazy**](https://github.com/urbanadventurer/urlcrazy)

### 位翻转

**您可以在父页面找到此技术的简短解释。或者阅读原始研究** [**https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/**](https://www.bleepingcomputer.com/news/security/hijacking-traffic-to-microsoft-s-windowscom-with-bitflipping/)

例如，对域名 microsoft.com 进行 1 位修改可以将其转换为 _windnws.com._\
**攻击者可能会注册尽可能多的与受害者相关的位翻转域名，以将合法用户重定向到他们的基础设施**。

**所有可能的位翻转域名也应进行监控。**

### 基本检查

一旦您拥有潜在可疑域名的列表，您应该 **检查** 它们（主要是 HTTP 和 HTTPS 端口），以 **查看它们是否使用与受害者域名相似的登录表单**。\
您还可以检查端口 3333，以查看它是否开放并运行 `gophish` 实例。\
了解 **每个发现的可疑域名的年龄** 也很有趣，越年轻的风险越大。\
您还可以获取可疑网页的 **截图**，以查看它是否可疑，如果是，则 **访问它以进行更深入的查看**。

### 高级检查

如果您想更进一步，我建议您 **监控这些可疑域名，并不时搜索更多**（每天？只需几秒钟/分钟）。您还应该 **检查** 相关 IP 的开放 **端口**，并 **搜索 `gophish` 或类似工具的实例**（是的，攻击者也会犯错），并 **监控可疑域名和子域名的 HTTP 和 HTTPS 网页**，以查看它们是否复制了受害者网页的任何登录表单。\
为了 **自动化这一过程**，我建议您拥有受害者域名的登录表单列表，爬取可疑网页，并使用类似 `ssdeep` 的工具比较可疑域名中的每个登录表单与受害者域名的每个登录表单。\
如果您找到了可疑域名的登录表单，您可以尝试 **发送垃圾凭据**，并 **检查它是否将您重定向到受害者的域名**。

## 使用关键字的域名

父页面还提到了一种域名变体技术，即将 **受害者的域名放入更大的域名中**（例如 paypal-financial.com 对于 paypal.com）。

### 证书透明度

无法采用之前的“暴力破解”方法，但实际上 **也可以通过证书透明度揭露此类钓鱼尝试**。每当 CA 发出证书时，详细信息会公开。这意味着通过阅读证书透明度或甚至监控它，**可以找到在其名称中使用关键字的域名**。例如，如果攻击者生成了 [https://paypal-financial.com](https://paypal-financial.com) 的证书，通过查看证书可以找到关键字“paypal”，并知道正在使用可疑电子邮件。

帖子 [https://0xpatrik.com/phishing-domains/](https://0xpatrik.com/phishing-domains/) 建议您可以使用 Censys 搜索影响特定关键字的证书，并按日期（仅“新”证书）和 CA 发行者“Let's Encrypt”进行过滤：

![https://0xpatrik.com/content/images/2018/07/cert\_listing.png](<../../.gitbook/assets/image (1115).png>)

然而，您可以使用免费的网页 [**crt.sh**](https://crt.sh) 做“同样的事情”。您可以 **搜索关键字**，并 **按日期和 CA 过滤** 结果（如果您愿意）。

![](<../../.gitbook/assets/image (519).png>)

使用最后一个选项，您甚至可以使用匹配身份字段查看真实域名的任何身份是否与任何可疑域名匹配（请注意，可疑域名可能是误报）。

**另一个替代方案** 是一个名为 [**CertStream**](https://medium.com/cali-dog-security/introducing-certstream-3fc13bb98067) 的精彩项目。CertStream 提供新生成证书的实时流，您可以使用它来实时检测指定关键字。实际上，有一个名为 [**phishing\_catcher**](https://github.com/x0rz/phishing\_catcher) 的项目就是这样做的。

### **新域名**

**最后一个替代方案** 是收集某些 TLD 的 **新注册域名** 列表（[Whoxy](https://www.whoxy.com/newly-registered-domains/) 提供此服务），并 **检查这些域名中的关键字**。然而，长域名通常使用一个或多个子域，因此关键字不会出现在 FLD 中，您将无法找到钓鱼子域。

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
