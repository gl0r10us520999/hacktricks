# AD Certificates

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

### 证书的组成部分

- 证书的 **主题** 表示其所有者。
- **公钥** 与私有密钥配对，将证书与其合法所有者关联。
- **有效期** 由 **NotBefore** 和 **NotAfter** 日期定义，标记证书的有效持续时间。
- 由证书颁发机构 (CA) 提供的唯一 **序列号** 标识每个证书。
- **颁发者** 指的是颁发证书的 CA。
- **SubjectAlternativeName** 允许为主题提供额外名称，增强识别灵活性。
- **基本约束** 确定证书是用于 CA 还是终端实体，并定义使用限制。
- **扩展密钥用途 (EKUs)** 通过对象标识符 (OIDs) 划定证书的特定用途，如代码签名或电子邮件加密。
- **签名算法** 指定签署证书的方法。
- **签名** 由颁发者的私钥创建，保证证书的真实性。

### 特殊考虑

- **主题备用名称 (SANs)** 扩展证书的适用性到多个身份，对于具有多个域的服务器至关重要。安全的颁发流程对于避免攻击者操纵 SAN 规范而导致的冒充风险至关重要。

### Active Directory (AD) 中的证书颁发机构 (CAs)

AD CS 通过指定的容器在 AD 林中承认 CA 证书，每个容器承担独特角色：

- **Certification Authorities** 容器保存受信任的根 CA 证书。
- **Enrolment Services** 容器详细说明企业 CA 及其证书模板。
- **NTAuthCertificates** 对象包括被授权用于 AD 认证的 CA 证书。
- **AIA (Authority Information Access)** 容器通过中间和交叉 CA 证书促进证书链验证。

### 证书获取：客户端证书请求流程

1. 请求过程从客户端查找企业 CA 开始。
2. 在生成公私钥对后，创建包含公钥和其他详细信息的 CSR。
3. CA 根据可用证书模板评估 CSR，并根据模板的权限颁发证书。
4. 经批准后，CA 用其私钥签署证书并将其返回给客户端。

### 证书模板

在 AD 中定义，这些模板概述了颁发证书的设置和权限，包括允许的 EKUs 和注册或修改权限，对于管理证书服务的访问至关重要。

## 证书注册

证书的注册过程由管理员 **创建证书模板** 开始，然后由企业证书颁发机构 (CA) **发布**。这使得模板可用于客户端注册，此步骤通过将模板名称添加到 Active Directory 对象的 `certificatetemplates` 字段来实现。

为了让客户端请求证书，必须授予 **注册权限**。这些权限由证书模板和企业 CA 本身的安全描述符定义。必须在两个位置授予权限，才能成功请求。

### 模板注册权限

这些权限通过访问控制条目 (ACEs) 指定，详细说明权限，如：
- **Certificate-Enrollment** 和 **Certificate-AutoEnrollment** 权限，每个权限与特定 GUID 相关联。
- **ExtendedRights**，允许所有扩展权限。
- **FullControl/GenericAll**，提供对模板的完全控制。

### 企业 CA 注册权限

CA 的权限在其安全描述符中列出，可以通过证书颁发机构管理控制台访问。有些设置甚至允许低权限用户远程访问，这可能是一个安全隐患。

### 额外的颁发控制

某些控制可能适用，例如：
- **经理批准**：将请求置于待处理状态，直到由证书经理批准。
- **注册代理和授权签名**：指定 CSR 上所需的签名数量和必要的应用程序策略 OIDs。

### 请求证书的方法

可以通过以下方式请求证书：
1. **Windows 客户端证书注册协议** (MS-WCCE)，使用 DCOM 接口。
2. **ICertPassage 远程协议** (MS-ICPR)，通过命名管道或 TCP/IP。
3. **证书注册 Web 界面**，安装了证书颁发机构 Web 注册角色。
4. **证书注册服务** (CES)，与证书注册策略 (CEP) 服务结合使用。
5. **网络设备注册服务** (NDES) 用于网络设备，使用简单证书注册协议 (SCEP)。

Windows 用户还可以通过 GUI (`certmgr.msc` 或 `certlm.msc`) 或命令行工具 (`certreq.exe` 或 PowerShell 的 `Get-Certificate` 命令) 请求证书。
```powershell
# Example of requesting a certificate using PowerShell
Get-Certificate -Template "User" -CertStoreLocation "cert:\\CurrentUser\\My"
```
## 证书认证

Active Directory (AD) 支持证书认证，主要利用 **Kerberos** 和 **安全通道 (Schannel)** 协议。

### Kerberos 认证过程

在 Kerberos 认证过程中，用户请求票证授予票证 (TGT) 的请求使用用户证书的 **私钥** 进行签名。该请求经过域控制器的多个验证，包括证书的 **有效性**、**路径** 和 **撤销状态**。验证还包括确认证书来自受信任的来源，并确认发行者在 **NTAUTH 证书存储** 中的存在。成功的验证将导致 TGT 的发放。AD 中的 **`NTAuthCertificates`** 对象位于：
```bash
CN=NTAuthCertificates,CN=Public Key Services,CN=Services,CN=Configuration,DC=<domain>,DC=<com>
```
is central to establishing trust for certificate authentication.

### Secure Channel (Schannel) Authentication

Schannel 促进安全的 TLS/SSL 连接，在握手过程中，客户端提供一个证书，如果成功验证，则授权访问。将证书映射到 AD 账户可能涉及 Kerberos 的 **S4U2Self** 函数或证书的 **Subject Alternative Name (SAN)**，以及其他方法。

### AD Certificate Services Enumeration

AD 的证书服务可以通过 LDAP 查询进行枚举，揭示有关 **Enterprise Certificate Authorities (CAs)** 及其配置的信息。这对任何经过域身份验证的用户都是可访问的，无需特殊权限。工具如 **[Certify](https://github.com/GhostPack/Certify)** 和 **[Certipy](https://github.com/ly4k/Certipy)** 用于在 AD CS 环境中进行枚举和漏洞评估。

Commands for using these tools include:
```bash
# Enumerate trusted root CA certificates and Enterprise CAs with Certify
Certify.exe cas
# Identify vulnerable certificate templates with Certify
Certify.exe find /vulnerable

# Use Certipy for enumeration and identifying vulnerable templates
certipy find -vulnerable -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128

# Enumerate Enterprise CAs and certificate templates with certutil
certutil.exe -TCAInfo
certutil -v -dstemplate
```
## 参考文献

* [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)
* [https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html](https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html)

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
