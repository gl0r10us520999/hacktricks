{% hint style="success" %}
学习与实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习与实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}


# CBC

如果 **cookie** 仅仅是 **用户名**（或者 cookie 的第一部分是用户名），并且你想要冒充用户名 "**admin**"。那么，你可以创建用户名 **"bdmin"** 并 **暴力破解** **cookie 的第一个字节**。

# CBC-MAC

**密码块链接消息认证码**（**CBC-MAC**）是一种在密码学中使用的方法。它通过逐块加密消息来工作，每个块的加密与前一个块相连。这个过程创建了一个 **块链**，确保即使改变原始消息的一个比特，也会导致最后一个加密数据块的不可预测变化。要进行或逆转这样的变化，需要加密密钥，以确保安全性。

要计算消息 m 的 CBC-MAC，需在 CBC 模式下用零初始化向量加密 m，并保留最后一个块。下图勾勒了使用秘密密钥 k 和块密码 E 计算由块组成的消息的 CBC-MAC 的过程！[https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5)：

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# 漏洞

在 CBC-MAC 中，通常使用的 **IV 是 0**。\
这是一个问题，因为 2 个已知消息（`m1` 和 `m2`）独立生成 2 个签名（`s1` 和 `s2`）。所以：

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

然后由 m1 和 m2 连接而成的消息（m3）将生成 2 个签名（s31 和 s32）：

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**这可以在不知道加密密钥的情况下计算。**

想象一下你正在以 **8 字节** 块加密名称 **Administrator**：

* `Administ`
* `rator\00\00\00`

你可以创建一个名为 **Administ**（m1）的用户名并获取签名（s1）。\
然后，你可以创建一个名为 `rator\00\00\00 XOR s1` 的用户名。这将生成 `E(m2 XOR s1 XOR 0)`，即 s32。\
现在，你可以将 s32 作为完整名称 **Administrator** 的签名。

### 总结

1. 获取用户名 **Administ**（m1）的签名，即 s1
2. 获取用户名 **rator\x00\x00\x00 XOR s1 XOR 0** 的签名，即 s32**。**
3. 将 cookie 设置为 s32，它将是用户 **Administrator** 的有效 cookie。

# 攻击控制 IV

如果你可以控制使用的 IV，攻击可能会非常简单。\
如果 cookie 仅仅是加密的用户名，要冒充用户 "**administrator**"，你可以创建用户 "**Administrator**"，并获取它的 cookie。\
现在，如果你可以控制 IV，你可以改变 IV 的第一个字节，使得 **IV\[0] XOR "A" == IV'\[0] XOR "a"**，并为用户 **Administrator** 重新生成 cookie。这个 cookie 将有效地 **冒充** 用户 **administrator**，使用初始 **IV**。

## 参考

更多信息请见 [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


{% hint style="success" %}
学习与实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习与实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 分享黑客技巧。

</details>
{% endhint %}
