# CGroups

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

## Basic Information

**Linux 控制组**，或称 **cgroups**，是 Linux 内核的一个特性，允许在进程组之间分配、限制和优先处理系统资源，如 CPU、内存和磁盘 I/O。它们提供了一种**管理和隔离进程集合资源使用**的机制，有利于资源限制、工作负载隔离和不同进程组之间的资源优先级管理等目的。

有 **两个版本的 cgroups**：版本 1 和版本 2。两者可以在系统上同时使用。主要区别在于 **cgroups 版本 2** 引入了 **层次化的树状结构**，使得在进程组之间进行更细致和详细的资源分配成为可能。此外，版本 2 还带来了各种增强功能，包括：

除了新的层次化组织，cgroups 版本 2 还引入了 **其他几个变化和改进**，例如对 **新资源控制器**的支持、更好的对遗留应用程序的支持以及性能的提升。

总体而言，cgroups **版本 2 提供了比版本 1 更多的功能和更好的性能**，但后者在某些需要与旧系统兼容的场景中仍然可以使用。

您可以通过查看 /proc/\<pid> 中的 cgroup 文件来列出任何进程的 v1 和 v2 cgroups。您可以通过以下命令开始查看您 shell 的 cgroups：
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
* **数字 2–12**：cgroups v1，每行代表一个不同的 cgroup。控制器在数字旁边指定。
* **数字 1**：也是 cgroups v1，但仅用于管理目的（由例如 systemd 设置），并且没有控制器。
* **数字 0**：表示 cgroups v2。没有列出控制器，这一行仅在仅运行 cgroups v2 的系统上存在。
* **名称是层次结构的**，类似于文件路径，指示不同 cgroups 之间的结构和关系。
* **像 /user.slice 或 /system.slice 的名称** 指定 cgroups 的分类，其中 user.slice 通常用于由 systemd 管理的登录会话，而 system.slice 用于系统服务。

### 查看 cgroups

文件系统通常用于访问 **cgroups**，与传统用于内核交互的 Unix 系统调用接口不同。要调查 shell 的 cgroup 配置，应检查 **/proc/self/cgroup** 文件，该文件显示 shell 的 cgroup。然后，通过导航到 **/sys/fs/cgroup**（或 **`/sys/fs/cgroup/unified`**）目录并找到一个与 cgroup 名称相同的目录，可以观察与 cgroup 相关的各种设置和资源使用信息。

![Cgroup 文件系统](<../../../.gitbook/assets/image (1128).png>)

cgroups 的关键接口文件以 **cgroup** 为前缀。**cgroup.procs** 文件可以使用标准命令（如 cat）查看，列出 cgroup 中的进程。另一个文件 **cgroup.threads** 包含线程信息。

![Cgroup 进程](<../../../.gitbook/assets/image (281).png>)

管理 shell 的 cgroups 通常包含两个控制器，分别调节内存使用和进程数量。要与控制器交互，应参考带有控制器前缀的文件。例如，**pids.current** 将被引用以确定 cgroup 中的线程数量。

![Cgroup 内存](<../../../.gitbook/assets/image (677).png>)

值中 **max** 的指示表明 cgroup 没有特定限制。然而，由于 cgroups 的层次结构，限制可能由目录层次结构中较低级别的 cgroup 强加。

### 操作和创建 cgroups

通过 **将其进程 ID (PID) 写入 `cgroup.procs` 文件** 将进程分配给 cgroups。这需要 root 权限。例如，要添加一个进程：
```bash
echo [pid] > cgroup.procs
```
同样，**修改 cgroup 属性，例如设置 PID 限制**，是通过将所需值写入相关文件来完成的。要为 cgroup 设置最多 3,000 个 PIDs：
```bash
echo 3000 > pids.max
```
**创建新的 cgroups** 涉及在 cgroup 层次结构中创建一个新的子目录，这会提示内核自动生成必要的接口文件。尽管没有活动进程的 cgroups 可以使用 `rmdir` 删除，但要注意某些限制：

* **进程只能放置在叶子 cgroups 中**（即层次结构中最嵌套的那些）。
* **一个 cgroup 不能拥有其父级中缺失的控制器**。
* **子 cgroups 的控制器必须在 `cgroup.subtree_control` 文件中显式声明**。例如，要在子 cgroup 中启用 CPU 和 PID 控制器：
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
The **root cgroup** 是这些规则的例外，允许直接放置进程。这可以用来将进程从 systemd 管理中移除。

**监控 cgroup 中的 CPU 使用情况** 可以通过 `cpu.stat` 文件实现，该文件显示了消耗的总 CPU 时间，有助于跟踪服务的子进程的使用情况：

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>cpu.stat 文件中显示的 CPU 使用统计信息</p></figcaption></figure>

## References

* **书籍：How Linux Works, 3rd Edition: What Every Superuser Should Know By Brian Ward**

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
