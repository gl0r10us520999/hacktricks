{% hint style="success" %}
学习与实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习与实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}


# ECB

(ECB) 电子密码本 - 对称加密方案，**将每个明文块**替换为**密文块**。它是**最简单**的加密方案。主要思想是将明文分割成**N位块**（取决于输入数据块的大小和加密算法），然后使用唯一的密钥对每个明文块进行加密（解密）。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

使用 ECB 有多种安全隐患：

* **可以删除加密消息中的块**
* **可以移动加密消息中的块**

# 漏洞检测

想象一下，你多次登录一个应用程序，并且**总是获得相同的 cookie**。这是因为该应用程序的 cookie 是**`<username>|<password>`**。\
然后，你生成两个新用户，他们都有**相同的长密码**和**几乎相同的** **用户名**。\
你发现**8B 的块**中**两个用户的信息**是**相等**的。然后，你想象这可能是因为**使用了 ECB**。

如以下示例所示。观察这**两个解码的 cookie**中有多次出现块**`\x23U\xE45K\xCB\x21\xC8`**。
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
这是因为这些 cookie 的 **用户名和密码包含了多次字母 "a"**（例如）。**不同的块**是包含 **至少 1 个不同字符** 的块（可能是分隔符 "|" 或用户名中的某些必要差异）。

现在，攻击者只需发现格式是 `<username><delimiter><password>` 还是 `<password><delimiter><username>`。为此，他可以 **生成多个相似且较长的用户名和密码，直到找到格式和分隔符的长度：**

| 用户名长度: | 密码长度: | 用户名+密码长度: | Cookie 的长度（解码后）: |
| ------------ | ---------- | ----------------- | ------------------------- |
| 2            | 2          | 4                 | 8                         |
| 3            | 3          | 6                 | 8                         |
| 3            | 4          | 7                 | 8                         |
| 4            | 4          | 8                 | 16                        |
| 7            | 7          | 14                | 16                        |

# 漏洞利用

## 移除整个块

知道 cookie 的格式（`<username>|<password>`），为了冒充用户名 `admin`，创建一个名为 `aaaaaaaaadmin` 的新用户并获取 cookie 并解码它：
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
我们可以看到之前用仅包含 `a` 的用户名创建的模式 `\x23U\xE45K\xCB\x21\xC8`。\
然后，您可以删除第一个 8B 块，您将获得用户名 `admin` 的有效 cookie：
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## 移动块

在许多数据库中，搜索 `WHERE username='admin';` 或 `WHERE username='admin    ';` 是一样的 _(注意额外的空格)_

因此，冒充用户 `admin` 的另一种方法是：

* 生成一个用户名：`len(<username>) + len(<delimiter) % len(block)`。使用 `8B` 的块大小，可以生成一个名为 `username       ` 的用户名，使用分隔符 `|`，块 `<username><delimiter>` 将生成 2 个 8B 的块。
* 然后，生成一个密码，填充包含我们想要冒充的用户名和空格的确切块数，例如：`admin   `

该用户的 cookie 将由 3 个块组成：前 2 个是用户名 + 分隔符的块，第三个是密码（伪装成用户名）：`username       |admin   `

**然后，只需用最后一个块替换第一个块，就可以冒充用户 `admin`：`admin          |username`**

## 参考

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


{% hint style="success" %}
学习和实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习和实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **在** **Twitter** 🐦 **上关注我们** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
