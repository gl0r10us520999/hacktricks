# Docker 安全

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

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
使用 [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) 轻松构建和 **自动化工作流**，由世界上 **最先进** 的社区工具提供支持。\
今天就获取访问权限：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **基本 Docker 引擎安全**

**Docker 引擎** 利用 Linux 内核的 **Namespaces** 和 **Cgroups** 来隔离容器，提供基本的安全层。通过 **能力丢弃**、**Seccomp** 和 **SELinux/AppArmor** 提供额外保护，增强容器隔离。一个 **auth 插件** 可以进一步限制用户操作。

![Docker 安全](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### 安全访问 Docker 引擎

Docker 引擎可以通过 Unix 套接字本地访问或通过 HTTP 远程访问。对于远程访问，使用 HTTPS 和 **TLS** 确保机密性、完整性和身份验证至关重要。

Docker 引擎默认在 `unix:///var/run/docker.sock` 上监听 Unix 套接字。在 Ubuntu 系统上，Docker 的启动选项在 `/etc/default/docker` 中定义。要启用对 Docker API 和客户端的远程访问，通过添加以下设置来通过 HTTP 套接字公开 Docker 守护进程：
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
然而，由于安全问题，不建议通过HTTP暴露Docker守护进程。建议使用HTTPS来保护连接。保护连接的主要方法有两种：

1. 客户端验证服务器的身份。
2. 客户端和服务器相互验证对方的身份。

证书用于确认服务器的身份。有关这两种方法的详细示例，请参阅[**本指南**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)。

### 容器镜像的安全性

容器镜像可以存储在私有或公共仓库中。Docker为容器镜像提供了几种存储选项：

* [**Docker Hub**](https://hub.docker.com): Docker的公共注册服务。
* [**Docker Registry**](https://github.com/docker/distribution): 一个开源项目，允许用户托管自己的注册表。
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Docker的商业注册表产品，具有基于角色的用户身份验证和与LDAP目录服务的集成。

### 镜像扫描

容器可能存在**安全漏洞**，这可能是由于基础镜像或在基础镜像上安装的软件造成的。Docker正在进行一个名为**Nautilus**的项目，该项目对容器进行安全扫描并列出漏洞。Nautilus通过将每个容器镜像层与漏洞库进行比较来识别安全漏洞。

有关更多[**信息，请阅读此文**](https://docs.docker.com/engine/scan/)。

* **`docker scan`**

**`docker scan`**命令允许您使用镜像名称或ID扫描现有的Docker镜像。例如，运行以下命令以扫描hello-world镜像：
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Docker 镜像签名

Docker 镜像签名确保了在容器中使用的镜像的安全性和完整性。以下是简要说明：

* **Docker 内容信任** 利用基于更新框架 (TUF) 的 Notary 项目来管理镜像签名。有关更多信息，请参见 [Notary](https://github.com/docker/notary) 和 [TUF](https://theupdateframework.github.io)。
* 要激活 Docker 内容信任，请设置 `export DOCKER_CONTENT_TRUST=1`。在 Docker 版本 1.10 及更高版本中，此功能默认关闭。
* 启用此功能后，仅可以下载签名的镜像。初始镜像推送需要为根密钥和标记密钥设置密码短语，Docker 还支持 Yubikey 以增强安全性。更多详细信息可以在 [这里](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) 找到。
* 尝试在启用内容信任的情况下拉取未签名的镜像会导致 "No trust data for latest" 错误。
* 在第一次之后的镜像推送中，Docker 会要求输入存储库密钥的密码短语以签署镜像。

要备份您的私钥，请使用以下命令：
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
当切换 Docker 主机时，必须移动根和存储库密钥以维持操作。

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
使用 [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) 轻松构建和 **自动化工作流**，由世界上 **最先进** 的社区工具提供支持。\
立即获取访问权限：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## 容器安全特性

<details>

<summary>容器安全特性摘要</summary>

**主要进程隔离特性**

在容器化环境中，隔离项目及其进程对于安全和资源管理至关重要。以下是关键概念的简化解释：

**命名空间**

* **目的**：确保进程、网络和文件系统等资源的隔离。特别是在 Docker 中，命名空间使容器的进程与主机和其他容器分开。
* **`unshare` 的使用**：`unshare` 命令（或底层系统调用）用于创建新的命名空间，提供额外的隔离层。然而，虽然 Kubernetes 本身并不阻止这一点，但 Docker 是会的。
* **限制**：创建新命名空间并不允许进程恢复到主机的默认命名空间。要进入主机命名空间，通常需要访问主机的 `/proc` 目录，并使用 `nsenter` 进行进入。

**控制组 (CGroups)**

* **功能**：主要用于在进程之间分配资源。
* **安全方面**：CGroups 本身不提供隔离安全，除了 `release_agent` 特性，如果配置错误，可能会被利用进行未经授权的访问。

**能力丢弃**

* **重要性**：这是进程隔离的重要安全特性。
* **功能**：通过丢弃某些能力来限制根进程可以执行的操作。即使进程以根权限运行，缺乏必要的能力也会阻止其执行特权操作，因为系统调用将因权限不足而失败。

这些是进程丢弃其他能力后的 **剩余能力**：

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

它在 Docker 中默认启用。它有助于**进一步限制进程可以调用的系统调用**。\
**默认的 Docker Seccomp 配置文件**可以在 [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) 找到。

**AppArmor**

Docker 有一个可以激活的模板：[https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

这将允许减少能力、系统调用、对文件和文件夹的访问...

</details>

### Namespaces

**Namespaces** 是 Linux 内核的一个特性，它**将内核资源进行分区**，使得一组**进程****看到**一组**资源**，而**另一**组**进程**看到**不同**的资源。该特性通过为一组资源和进程使用相同的命名空间来工作，但这些命名空间指向不同的资源。资源可以存在于多个空间中。

Docker 利用以下 Linux 内核命名空间来实现容器隔离：

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

有关**命名空间的更多信息**，请查看以下页面：

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Linux 内核特性**cgroups**提供了**限制一组进程的资源，如 CPU、内存、IO、网络带宽**的能力。Docker 允许使用 cgroup 特性创建容器，从而实现对特定容器的资源控制。\
以下是一个用户空间内存限制为 500m，内核内存限制为 50m，CPU 共享为 512，blkio-weight 为 400 的容器。CPU 共享是控制容器 CPU 使用的比例。它的默认值为 1024，范围在 0 到 1024 之间。如果三个容器的 CPU 共享均为 1024，则在 CPU 资源争用的情况下，每个容器最多可以占用 33% 的 CPU。blkio-weight 是控制容器 IO 的比例。它的默认值为 500，范围在 10 到 1000 之间。
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
要获取容器的 cgroup，您可以执行：
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
有关更多信息，请查看：

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### 能力

能力允许**对可以允许的根用户能力进行更细粒度的控制**。Docker使用Linux内核能力特性来**限制可以在容器内执行的操作**，无论用户类型如何。

当运行docker容器时，**进程会放弃敏感能力，以防止进程逃离隔离**。这试图确保进程无法执行敏感操作并逃逸：

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Docker中的Seccomp

这是一种安全特性，允许Docker**限制可以在容器内使用的系统调用**：

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### Docker中的AppArmor

**AppArmor**是一个内核增强，用于将**容器**限制在**有限**的**资源**集内，并具有**每个程序的配置文件**：

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### Docker中的SELinux

* **标记系统**：SELinux为每个进程和文件系统对象分配一个唯一的标签。
* **策略执行**：它执行定义进程标签可以对系统内其他标签执行哪些操作的安全策略。
* **容器进程标签**：当容器引擎启动容器进程时，通常会分配一个受限的SELinux标签，通常为`container_t`。
* **容器内的文件标记**：容器内的文件通常标记为`container_file_t`。
* **策略规则**：SELinux策略主要确保具有`container_t`标签的进程只能与标记为`container_file_t`的文件进行交互（读取、写入、执行）。

该机制确保即使容器内的进程被攻陷，它也仅限于与具有相应标签的对象进行交互，从而显著限制此类攻陷可能造成的损害。

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

在Docker中，授权插件在安全性方面发挥着关键作用，通过决定是否允许或阻止对Docker守护进程的请求来实现。这一决定是通过检查两个关键上下文来做出的：

* **身份验证上下文**：这包括有关用户的全面信息，例如他们是谁以及他们如何进行身份验证。
* **命令上下文**：这包括与所发出请求相关的所有相关数据。

这些上下文有助于确保只有经过身份验证的用户的合法请求被处理，从而增强Docker操作的安全性。

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## 来自容器的DoS

如果您没有正确限制容器可以使用的资源，则被攻陷的容器可能会对其运行的主机造成DoS。

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* 带宽 DoS
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## 有趣的 Docker 标志

### --privileged 标志

在以下页面中，您可以了解 **`--privileged` 标志的含义**：

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

如果您正在运行一个容器，攻击者设法以低权限用户身份获得访问权限。如果您有一个 **配置错误的 suid 二进制文件**，攻击者可能会利用它并 **在容器内提升权限**。这可能允许他逃离容器。

启用 **`no-new-privileges`** 选项运行容器将 **防止这种权限提升**。
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### 其他
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
对于更多 **`--security-opt`** 选项，请查看: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## 其他安全考虑

### 管理机密：最佳实践

避免直接在 Docker 镜像中嵌入机密或使用环境变量至关重要，因为这些方法会通过 `docker inspect` 或 `exec` 等命令将您的敏感信息暴露给任何可以访问容器的人。

**Docker 卷** 是一种更安全的替代方案，推荐用于访问敏感信息。它们可以作为内存中的临时文件系统使用，从而降低与 `docker inspect` 和日志记录相关的风险。然而，根用户和具有 `exec` 访问权限的用户仍然可能访问这些机密。

**Docker secrets** 提供了一种更安全的方法来处理敏感信息。对于在镜像构建阶段需要机密的实例，**BuildKit** 提供了一种高效的解决方案，支持构建时机密，提升构建速度并提供额外功能。

要利用 BuildKit，可以通过三种方式激活：

1. 通过环境变量: `export DOCKER_BUILDKIT=1`
2. 通过命令前缀: `DOCKER_BUILDKIT=1 docker build .`
3. 通过在 Docker 配置中默认启用: `{ "features": { "buildkit": true } }`，然后重启 Docker。

BuildKit 允许使用 `--secret` 选项来使用构建时机密，确保这些机密不包含在镜像构建缓存或最终镜像中，使用命令如下:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
对于运行中的容器所需的秘密，**Docker Compose 和 Kubernetes** 提供了强大的解决方案。Docker Compose 在服务定义中使用 `secrets` 键来指定秘密文件，如 `docker-compose.yml` 示例所示：
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
此配置允许在使用 Docker Compose 启动服务时使用秘密。

在 Kubernetes 环境中，秘密是原生支持的，并且可以通过像 [Helm-Secrets](https://github.com/futuresimple/helm-secrets) 这样的工具进一步管理。Kubernetes 的基于角色的访问控制 (RBAC) 增强了秘密管理的安全性，类似于 Docker Enterprise。

### gVisor

**gVisor** 是一个应用内核，使用 Go 编写，实施了 Linux 系统表面的相当大一部分。它包括一个名为 `runsc` 的 [开放容器倡议 (OCI)](https://www.opencontainers.org) 运行时，提供了 **应用程序与主机内核之间的隔离边界**。`runsc` 运行时与 Docker 和 Kubernetes 集成，使得运行沙箱容器变得简单。

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** 是一个开源社区，致力于构建一个安全的容器运行时，使用轻量级虚拟机，感觉和表现像容器，但提供 **使用硬件虚拟化技术的更强工作负载隔离** 作为第二道防线。

{% embed url="https://katacontainers.io/" %}

### 总结提示

* **不要使用 `--privileged` 标志或在容器内挂载** [**Docker 套接字**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**。** Docker 套接字允许生成容器，因此这是完全控制主机的简单方法，例如，通过使用 `--privileged` 标志运行另一个容器。
* **不要在容器内以 root 身份运行。使用** [**不同用户**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) **和** [**用户命名空间**](https://docs.docker.com/engine/security/userns-remap/)**。** 容器中的 root 与主机上的 root 是相同的，除非通过用户命名空间重新映射。它仅受到 Linux 命名空间、能力和 cgroups 的轻微限制。
* [**丢弃所有能力**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`)，仅启用所需的能力** (`--cap-add=...`)。许多工作负载不需要任何能力，添加它们会增加潜在攻击的范围。
* [**使用“no-new-privileges”安全选项**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) 防止进程获得更多权限，例如通过 suid 二进制文件。
* [**限制容器可用的资源**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**。** 资源限制可以保护机器免受拒绝服务攻击。
* **调整** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**、** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **（或 SELinux）** 配置文件，以将容器可用的操作和系统调用限制到最低要求。
* **使用** [**官方 Docker 镜像**](https://docs.docker.com/docker-hub/official_images/) **并要求签名**，或基于它们构建自己的镜像。不要继承或使用 [后门](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) 镜像。还要将 root 密钥、密码短语存放在安全的地方。Docker 计划通过 UCP 管理密钥。
* **定期** **重建** 镜像以 **应用安全补丁到主机和镜像。**
* 明智地管理您的 **秘密**，使攻击者难以访问它们。
* 如果您 **暴露 Docker 守护进程，请使用 HTTPS**，并进行客户端和服务器身份验证。
* 在您的 Dockerfile 中，**优先使用 COPY 而不是 ADD**。ADD 会自动提取压缩文件，并可以从 URL 复制文件。COPY 没有这些功能。尽可能避免使用 ADD，以免受到通过远程 URL 和 Zip 文件的攻击。
* 为每个微服务 **使用单独的容器**
* **不要在容器内放置 ssh**，可以使用 “docker exec” 连接到容器。
* 拥有 **更小的** 容器 **镜像**

## Docker 突破 / 权限提升

如果您 **在 Docker 容器内** 或者您有权访问 **docker 组中的用户**，您可以尝试 **逃逸并提升权限**：

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Docker 身份验证插件绕过

如果您可以访问 Docker 套接字或有权访问 **docker 组中的用户，但您的操作受到 Docker 身份验证插件的限制**，请检查您是否可以 **绕过它：**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## 加固 Docker

* 工具 [**docker-bench-security**](https://github.com/docker/docker-bench-security) 是一个脚本，检查在生产中部署 Docker 容器的数十个常见最佳实践。所有测试都是自动化的，基于 [CIS Docker 基准 v1.3.1](https://www.cisecurity.org/benchmark/docker/)。\
您需要从运行 Docker 的主机或具有足够权限的容器中运行该工具。了解 **如何在 README 中运行它：** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security)。

## 参考

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux_namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins_authorization](https://docs.docker.com/engine/extend/plugins_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
使用 [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) 轻松构建和 **自动化工作流**，由世界上 **最先进** 的社区工具提供支持。\
立即获取访问权限：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

{% hint style="success" %}
学习和实践 AWS 黑客攻击：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客攻击： <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **在 Twitter 上关注** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
