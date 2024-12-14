{% hint style="success" %}
学习与实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习与实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}


# 切割工具

## Autopsy

在取证中提取图像中的文件最常用的工具是 [**Autopsy**](https://www.autopsy.com/download/)。下载并安装它，然后让它处理文件以查找“隐藏”文件。请注意，Autopsy 是为支持磁盘映像和其他类型的映像而构建的，但不支持简单文件。

## Binwalk <a id="binwalk"></a>

**Binwalk** 是一个用于搜索二进制文件（如图像和音频文件）中嵌入文件和数据的工具。它可以通过 `apt` 安装，但 [源代码](https://github.com/ReFirmLabs/binwalk) 可以在 github 上找到。  
**有用的命令**：
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

另一个常用的工具来查找隐藏文件是 **foremost**。您可以在 `/etc/foremost.conf` 中找到 foremost 的配置文件。如果您只想搜索某些特定文件，请取消注释它们。如果您不取消注释任何内容，foremost 将搜索其默认配置的文件类型。
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** 是另一个可以用来查找和提取 **嵌入在文件中的文件** 的工具。在这种情况下，您需要从配置文件 \(_/etc/scalpel/scalpel.conf_\) 中取消注释您希望提取的文件类型。
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

这个工具包含在kali中，但你可以在这里找到它: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

这个工具可以扫描一个镜像并**提取其中的pcaps**，**网络信息（URLs、域名、IPs、MACs、邮件）**以及更多**文件**。你只需执行：
```text
bulk_extractor memory.img -o out_folder
```
导航通过**工具收集的所有信息**（密码？），**分析** **数据包**（阅读[**Pcaps分析**](../pcap-inspection/)），搜索**奇怪的域名**（与**恶意软件**或**不存在的**域名相关）。

## PhotoRec

您可以在[https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download)找到它。

它提供GUI和CLI版本。您可以选择希望PhotoRec搜索的**文件类型**。

![](../../../.gitbook/assets/image%20%28524%29.png)

# 特定数据雕刻工具

## FindAES

通过搜索其密钥调度来搜索AES密钥。能够找到128、192和256位密钥，例如TrueCrypt和BitLocker使用的密钥。

在[这里下载](https://sourceforge.net/projects/findaes/)。

# 补充工具

您可以使用[**viu**](https://github.com/atanunq/viu)从终端查看图像。  
您可以使用Linux命令行工具**pdftotext**将PDF转换为文本并进行阅读。



{% hint style="success" %}
学习和实践AWS黑客攻击：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践GCP黑客攻击：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f)或[**电报群组**](https://t.me/peass)或**关注**我们在**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub库提交PR分享黑客技巧。

</details>
{% endhint %}
