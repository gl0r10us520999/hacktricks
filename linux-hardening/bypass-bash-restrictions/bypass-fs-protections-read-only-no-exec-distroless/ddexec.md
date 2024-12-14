# DDexec / EverythingExec

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

## 上下文

在 Linux 中，为了运行一个程序，它必须作为一个文件存在，必须通过文件系统层次以某种方式可访问（这就是 `execve()` 的工作原理）。这个文件可以存储在磁盘上或在内存中（tmpfs, memfd），但你需要一个文件路径。这使得控制在 Linux 系统上运行的内容变得非常简单，容易检测威胁和攻击者的工具，或者防止他们尝试执行任何他们的程序（例如，不允许无权限用户在任何地方放置可执行文件）。

但这个技术将改变这一切。如果你无法启动你想要的进程... **那么你就劫持一个已经存在的进程**。

这个技术允许你 **绕过常见的保护技术，如只读、noexec、文件名白名单、哈希白名单...**

## 依赖关系

最终脚本依赖于以下工具才能工作，它们需要在你攻击的系统中可访问（默认情况下，你会在任何地方找到它们）：
```
dd
bash | zsh | ash (busybox)
head
tail
cut
grep
od
readlink
wc
tr
base64
```
## 技术

如果您能够任意修改进程的内存，那么您就可以接管它。这可以用来劫持一个已经存在的进程并用另一个程序替换它。我们可以通过使用 `ptrace()` 系统调用（这要求您能够执行系统调用或在系统上有 gdb 可用）来实现这一点，或者更有趣的是，写入 `/proc/$pid/mem`。

文件 `/proc/$pid/mem` 是进程整个地址空间的一对一映射（_例如_ 从 `0x0000000000000000` 到 `0x7ffffffffffff000` 在 x86-64 中）。这意味着在偏移量 `x` 处读取或写入此文件与在虚拟地址 `x` 处读取或修改内容是相同的。

现在，我们面临四个基本问题：

* 通常，只有 root 和文件的程序所有者可以修改它。
* ASLR。
* 如果我们尝试读取或写入未映射在程序地址空间中的地址，我们将得到 I/O 错误。

这些问题有解决方案，尽管它们并不完美，但效果不错：

* 大多数 shell 解释器允许创建文件描述符，这些描述符将被子进程继承。我们可以创建一个指向 shell 的 `mem` 文件的 fd，并赋予写权限……因此使用该 fd 的子进程将能够修改 shell 的内存。
* ASLR 甚至不是问题，我们可以检查 shell 的 `maps` 文件或 procfs 中的任何其他文件，以获取有关进程地址空间的信息。
* 所以我们需要在文件上使用 `lseek()`。从 shell 中，这不能完成，除非使用臭名昭著的 `dd`。

### 更详细地

这些步骤相对简单，不需要任何专业知识来理解：

* 解析我们想要运行的二进制文件和加载器，以找出它们需要什么映射。然后制作一个“shell”代码，广义上讲，执行内核在每次调用 `execve()` 时所做的相同步骤：
* 创建上述映射。
* 将二进制文件读入其中。
* 设置权限。
* 最后用程序的参数初始化堆栈，并放置辅助向量（加载器所需）。
* 跳转到加载器，让它完成其余工作（加载程序所需的库）。
* 从 `syscall` 文件中获取进程在执行的系统调用后将返回的地址。
* 用我们的 shellcode（通过 `mem` 我们可以修改不可写页面）覆盖该可执行位置。
* 将我们想要运行的程序传递给进程的 stdin（将被上述“shell”代码 `read()`）。
* 此时，加载器将负责加载我们程序所需的库并跳转到它。

**查看工具在** [**https://github.com/arget13/DDexec**](https://github.com/arget13/DDexec)

## EverythingExec

有几种替代 `dd` 的方法，其中之一 `tail`，目前是用于通过 `mem` 文件进行 `lseek()` 的默认程序（这就是使用 `dd` 的唯一目的）。这些替代方案是：
```bash
tail
hexdump
cmp
xxd
```
设置变量 `SEEKER` 可以更改使用的搜索器，例如：
```bash
SEEKER=cmp bash ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
如果您找到另一个在脚本中未实现的有效查找器，您仍然可以通过设置 `SEEKER_ARGS` 变量来使用它：
```bash
SEEKER=xxd SEEKER_ARGS='-s $offset' zsh ddexec.sh ls -l <<< $(base64 -w0 /bin/ls)
```
Block this, EDRs.

## References
* [https://github.com/arget13/DDexec](https://github.com/arget13/DDexec)

{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
