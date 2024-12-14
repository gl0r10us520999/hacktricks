# macOS Bundles

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}

## 基本信息

macOS 中的 Bundles 作为各种资源的容器，包括应用程序、库和其他必要文件，使它们在 Finder 中看起来像单一对象，例如熟悉的 `*.app` 文件。最常见的 bundle 是 `.app` bundle，尽管 `.framework`、`.systemextension` 和 `.kext` 等其他类型也很普遍。

### Bundle 的基本组件

在 bundle 中，特别是在 `<application>.app/Contents/` 目录下，存放着各种重要资源：

* **\_CodeSignature**：此目录存储代码签名详细信息，对于验证应用程序的完整性至关重要。您可以使用以下命令检查代码签名信息： %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**：包含在用户交互时运行的应用程序的可执行二进制文件。
* **Resources**：应用程序用户界面组件的存储库，包括图像、文档和界面描述（nib/xib 文件）。
* **Info.plist**：作为应用程序的主要配置文件，对于系统正确识别和与应用程序交互至关重要。

#### Info.plist 中的重要键

`Info.plist` 文件是应用程序配置的基石，包含以下键：

* **CFBundleExecutable**：指定位于 `Contents/MacOS` 目录中的主可执行文件的名称。
* **CFBundleIdentifier**：为应用程序提供全局标识符，macOS 在应用程序管理中广泛使用。
* **LSMinimumSystemVersion**：指示运行应用程序所需的最低 macOS 版本。

### 探索 Bundles

要探索 bundle 的内容，例如 `Safari.app`，可以使用以下命令：`bash ls -lR /Applications/Safari.app/Contents`

此探索揭示了像 `_CodeSignature`、`MacOS`、`Resources` 这样的目录，以及像 `Info.plist` 这样的文件，每个文件都在保护应用程序、定义其用户界面和操作参数方面发挥独特作用。

#### 其他 Bundle 目录

除了常见目录，bundles 还可能包括：

* **Frameworks**：包含应用程序使用的捆绑框架。框架类似于带有额外资源的 dylibs。
* **PlugIns**：用于增强应用程序功能的插件和扩展的目录。
* **XPCServices**：保存应用程序用于进程间通信的 XPC 服务。

这种结构确保所有必要组件都封装在 bundle 内，促进模块化和安全的应用程序环境。

有关 `Info.plist` 键及其含义的更多详细信息，Apple 开发者文档提供了广泛的资源：[Apple Info.plist 键参考](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html)。

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** 上关注我们。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
