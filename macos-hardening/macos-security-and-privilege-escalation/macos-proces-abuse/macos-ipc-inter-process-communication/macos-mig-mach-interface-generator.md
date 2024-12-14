# macOS MIG - Mach Interface Generator

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

## 基本信息

MIG的创建是为了**简化Mach IPC**代码的生成过程。它基本上**生成所需的代码**，使服务器和客户端能够根据给定的定义进行通信。即使生成的代码不够优雅，开发者只需导入它，他的代码将比之前简单得多。

定义使用接口定义语言（IDL）以`.defs`扩展名指定。

这些定义有5个部分：

* **子系统声明**：关键字subsystem用于指示**名称**和**id**。如果服务器应该在内核中运行，也可以将其标记为**`KernelServer`**。
* **包含和导入**：MIG使用C预处理器，因此能够使用导入。此外，可以使用`uimport`和`simport`来处理用户或服务器生成的代码。
* **类型声明**：可以定义数据类型，尽管通常会导入`mach_types.defs`和`std_types.defs`。对于自定义类型，可以使用一些语法：
* \[i`n/out]tran`：需要从传入消息或到传出消息进行转换的函数
* `c[user/server]type`：映射到另一个C类型。
* `destructor`：当类型被释放时调用此函数。
* **操作**：这些是RPC方法的定义。有5种不同类型：
* `routine`：期望回复
* `simpleroutine`：不期望回复
* `procedure`：期望回复
* `simpleprocedure`：不期望回复
* `function`：期望回复

### 示例

创建一个定义文件，在这种情况下是一个非常简单的函数：

{% code title="myipc.defs" %}
```cpp
subsystem myipc 500; // Arbitrary name and id

userprefix USERPREF;        // Prefix for created functions in the client
serverprefix SERVERPREF;    // Prefix for created functions in the server

#include <mach/mach_types.defs>
#include <mach/std_types.defs>

simpleroutine Subtract(
server_port :  mach_port_t;
n1          :  uint32_t;
n2          :  uint32_t);
```
{% endcode %}

请注意，第一个 **参数是要绑定的端口**，MIG 将 **自动处理回复端口**（除非在客户端代码中调用 `mig_get_reply_port()`）。此外，**操作的 ID** 将是 **顺序的**，从指定的子系统 ID 开始（因此，如果某个操作被弃用，它将被删除，并且使用 `skip` 仍然使用其 ID）。

现在使用 MIG 生成能够相互通信以调用 Subtract 函数的服务器和客户端代码：
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
在当前目录中将创建几个新文件。

{% hint style="success" %}
您可以在系统中找到一个更复杂的示例，使用：`mdfind mach_port.defs`\
并且您可以从与文件相同的文件夹中编译它，使用：`mig -DLIBSYSCALL_INTERFACE mach_ports.defs`
{% endhint %}

在文件 **`myipcServer.c`** 和 **`myipcServer.h`** 中，您可以找到结构 **`SERVERPREFmyipc_subsystem`** 的声明和定义，该结构基本上根据接收到的消息 ID 定义要调用的函数（我们指定了起始编号为 500）：

{% tabs %}
{% tab title="myipcServer.c" %}
```c
/* Description of this subsystem, for use in direct RPC */
const struct SERVERPREFmyipc_subsystem SERVERPREFmyipc_subsystem = {
myipc_server_routine,
500, // start ID
501, // end ID
(mach_msg_size_t)sizeof(union __ReplyUnion__SERVERPREFmyipc_subsystem),
(vm_address_t)0,
{
{ (mig_impl_routine_t) 0,
// Function to call
(mig_stub_routine_t) _XSubtract, 3, 0, (routine_arg_descriptor_t)0, (mach_msg_size_t)sizeof(__Reply__Subtract_t)},
}
};
```
{% endtab %}

{% tab title="myipcServer.h" %}
```c
/* Description of this subsystem, for use in direct RPC */
extern const struct SERVERPREFmyipc_subsystem {
mig_server_routine_t	server;	/* Server routine */
mach_msg_id_t	start;	/* Min routine number */
mach_msg_id_t	end;	/* Max routine number + 1 */
unsigned int	maxsize;	/* Max msg size */
vm_address_t	reserved;	/* Reserved */
struct routine_descriptor	/* Array of routine descriptors */
routine[1];
} SERVERPREFmyipc_subsystem;
```
{% endtab %}
{% endtabs %}

基于之前的结构，函数 **`myipc_server_routine`** 将获取 **消息 ID** 并返回适当的调用函数：
```c
mig_external mig_routine_t myipc_server_routine
(mach_msg_header_t *InHeadP)
{
int msgh_id;

msgh_id = InHeadP->msgh_id - 500;

if ((msgh_id > 0) || (msgh_id < 0))
return 0;

return SERVERPREFmyipc_subsystem.routine[msgh_id].stub_routine;
}
```
在这个例子中，我们只在定义中定义了 1 个函数，但如果我们定义了更多函数，它们将位于 **`SERVERPREFmyipc_subsystem`** 数组中，第一个将被分配给 ID **500**，第二个将被分配给 ID **501**...

如果该函数预期发送一个 **reply**，则函数 `mig_internal kern_return_t __MIG_check__Reply__<name>` 也会存在。

实际上，可以在 **`myipcServer.h`** 中的结构 **`subsystem_to_name_map_myipc`** 中识别这种关系（在其他文件中为 **`subsystem_to_name_map_***`）：
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
最后，另一个使服务器正常工作的关键功能是 **`myipc_server`**，它实际上会 **调用与接收到的 id 相关的函数**：

<pre class="language-c"><code class="lang-c">mig_external boolean_t myipc_server
(mach_msg_header_t *InHeadP, mach_msg_header_t *OutHeadP)
{
/*
* typedef struct {
* 	mach_msg_header_t Head;
* 	NDR_record_t NDR;
* 	kern_return_t RetCode;
* } mig_reply_error_t;
*/

mig_routine_t routine;

OutHeadP->msgh_bits = MACH_MSGH_BITS(MACH_MSGH_BITS_REPLY(InHeadP->msgh_bits), 0);
OutHeadP->msgh_remote_port = InHeadP->msgh_reply_port;
/* 最小大小：routine() 如果不同会更新它 */
OutHeadP->msgh_size = (mach_msg_size_t)sizeof(mig_reply_error_t);
OutHeadP->msgh_local_port = MACH_PORT_NULL;
OutHeadP->msgh_id = InHeadP->msgh_id + 100;
OutHeadP->msgh_reserved = 0;

if ((InHeadP->msgh_id > 500) || (InHeadP->msgh_id &#x3C; 500) ||
<strong>	    ((routine = SERVERPREFmyipc_subsystem.routine[InHeadP->msgh_id - 500].stub_routine) == 0)) {
</strong>		((mig_reply_error_t *)OutHeadP)->NDR = NDR_record;
((mig_reply_error_t *)OutHeadP)->RetCode = MIG_BAD_ID;
return FALSE;
}
<strong>	(*routine) (InHeadP, OutHeadP);
</strong>	return TRUE;
}
</code></pre>

检查之前突出显示的行以通过 ID 访问要调用的函数。

以下是创建一个简单的 **服务器** 和 **客户端** 的代码，其中客户端可以调用服务器的 Subtract 函数：

{% tabs %}
{% tab title="myipc_server.c" %}
```c
// gcc myipc_server.c myipcServer.c -o myipc_server

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcServer.h"

kern_return_t SERVERPREFSubtract(mach_port_t server_port, uint32_t n1, uint32_t n2)
{
printf("Received: %d - %d = %d\n", n1, n2, n1 - n2);
return KERN_SUCCESS;
}

int main() {

mach_port_t port;
kern_return_t kr;

// Register the mach service
kr = bootstrap_check_in(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_check_in() failed with code 0x%x\n", kr);
return 1;
}

// myipc_server is the function that handles incoming messages (check previous exlpanation)
mach_msg_server(myipc_server, sizeof(union __RequestUnion__SERVERPREFmyipc_subsystem), port, MACH_MSG_TIMEOUT_NONE);
}
```
{% endtab %}

{% tab title="myipc_client.c" %}
```c
// gcc myipc_client.c myipcUser.c -o myipc_client

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcUser.h"

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("Port right name %d\n", port);
USERPREFSubtract(port, 40, 2);
}
```
{% endtab %}
{% endtabs %}

### NDR\_record

NDR\_record 是由 `libsystem_kernel.dylib` 导出的，它是一个结构体，允许 MIG **转换数据，使其与所使用的系统无关**，因为 MIG 被认为是用于不同系统之间的（而不仅仅是在同一台机器上）。

这很有趣，因为如果在二进制文件中找到 `_NDR_record` 作为依赖项（`jtool2 -S <binary> | grep NDR` 或 `nm`），这意味着该二进制文件是一个 MIG 客户端或服务器。

此外，**MIG 服务器**在 `__DATA.__const` 中有调度表（在 macOS 内核中为 `__CONST.__constdata`，在其他 \*OS 内核中为 `__DATA_CONST.__const`）。这可以通过 **`jtool2`** 转储。

而 **MIG 客户端** 将使用 `__NDR_record` 通过 `__mach_msg` 发送给服务器。

## 二进制分析

### jtool

由于许多二进制文件现在使用 MIG 来暴露 mach 端口，因此了解如何 **识别 MIG 的使用** 以及 **MIG 执行的每个消息 ID 的函数** 是很有趣的。

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) 可以解析 Mach-O 二进制文件中的 MIG 信息，指示消息 ID 并识别要执行的函数：
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
此外，MIG 函数只是实际被调用函数的包装，这意味着通过获取其反汇编并搜索 BL，您可能能够找到实际被调用的函数：
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### Assembly

之前提到过，负责**根据接收到的消息 ID 调用正确函数**的函数是 `myipc_server`。然而，通常你不会拥有二进制文件的符号（没有函数名），因此检查**反编译后的样子**是很有趣的，因为它总是非常相似（该函数的代码与暴露的函数无关）：

{% tabs %}
{% tab title="myipc_server decompiled 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// 初始指令以找到正确的函数指针
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// 调用 sign_extend_64，可以帮助识别该函数
// 这将指针存储在 rax 中，以便调用需要调用的函数
// 检查地址 0x100004040 的使用（函数地址数组）
// 0x1f4 = 500（起始 ID）
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// 如果 - else，if 返回 false，而 else 调用正确的函数并返回 true
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// 计算的地址调用带有 2 个参数的正确函数
<strong>                    (var_20)(var_10, var_18);
</strong>                    var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
rax = var_4;
return rax;
}
</code></pre>
{% endtab %}

{% tab title="myipc_server decompiled 2" %}
这是在不同的 Hopper 免费版本中反编译的相同函数：

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// 初始指令以找到正确的函数指针
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f | 0x0;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 > 0x0) {
if (CPU_FLAGS &#x26; G) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 &#x3C; 0x0) {
if (CPU_FLAGS &#x26; L) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
// 0x1f4 = 500（起始 ID）
<strong>                    r8 = r8 - 0x1f4;
</strong>                    asm { smaddl     x8, w8, w9, x10 };
r8 = *(r8 + 0x8);
var_20 = r8;
r8 = r8 - 0x0;
if (r8 != 0x0) {
if (CPU_FLAGS &#x26; NE) {
r8 = 0x1;
}
}
// 与之前版本相同的 if else
// 检查地址 0x100004040 的使用（函数地址数组）
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// 调用计算的地址，函数应该在这里
<strong>                            (var_20)(var_10, var_18);
</strong>                            var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
r0 = var_4;
return r0;
}

</code></pre>
{% endtab %}
{% endtabs %}

实际上，如果你去到函数**`0x100004000`**，你会发现**`routine_descriptor`** 结构的数组。结构的第一个元素是**函数**实现的**地址**，并且**结构占用 0x28 字节**，因此从字节 0 开始每 0x28 字节你可以获取 8 字节，这将是**将被调用的函数的地址**：

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

这些数据可以通过[**使用这个 Hopper 脚本**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py)提取。

### Debug

MIG 生成的代码还调用 `kernel_debug` 以生成有关进入和退出操作的日志。可以使用**`trace`**或**`kdv`**检查它们：`kdv all | grep MIG`

## References

* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

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
