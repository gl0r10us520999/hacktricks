# macOS MIG - Mach Interface Generator

{% hint style="success" %}
AWSハッキングの学習と練習:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と練習: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に**参加**するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **HackTricks**と**HackTricks Cloud**のGitHubリポジトリにPRを提出して、ハッキングトリックを共有してください。

</details>
{% endhint %}

## 基本情報

MIGは**Mach IPCのコード作成プロセスを簡素化**するために作成されました。基本的には、サーバーとクライアントが指定された定義と通信するために必要なコードを**生成**します。生成されたコードが見た目が悪くても、開発者はそれをインポートするだけで、以前よりもコードがはるかにシンプルになります。

定義は、`.defs`拡張子を使用してインターフェース定義言語（IDL）で指定されます。

これらの定義には5つのセクションがあります:

* **サブシステム宣言**: キーワードのサブシステムは**名前**と**ID**を示すために使用されます。サーバーがカーネルで実行される必要がある場合は、**`KernelServer`**としてマークすることも可能です。
* **インクルージョンとインポート**: MIGはCプリプロセッサを使用しているため、インポートを使用することができます。さらに、ユーザーまたはサーバー生成コードに対して`uimport`および`simport`を使用することも可能です。
* **型の宣言**: データ型を定義することができますが、通常は`mach_types.defs`および`std_types.defs`をインポートします。カスタムの場合は、いくつかの構文を使用できます:
* \[i`n/out]tran`: 受信または送信メッセージから変換する必要がある関数
* `c[user/server]type`: 別のC型にマッピング
* `destructor`: 型が解放されるときにこの関数を呼び出す
* **操作**: これらはRPCメソッドの定義です。5つの異なるタイプがあります:
* `routine`: 返信を期待
* `simpleroutine`: 返信を期待しない
* `procedure`: 返信を期待
* `simpleprocedure`: 返信を期待しない
* `function`: 返信を期待

### 例

非常にシンプルな関数を持つ定義ファイルを作成します:

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

最初の**引数はバインドするポート**であり、MIGは**自動的に応答ポートを処理**します（クライアントコードで`mig_get_reply_port()`を呼び出さない限り）。さらに、**操作のID**は、指定されたサブシステムIDから始まる**連続した**ものになります（したがって、操作が非推奨になると削除され、そのIDを引き続き使用するために`skip`が使用されます）。

今、MIGを使用して、互いに通信できるサーバーおよびクライアントコードを生成し、Subtract関数を呼び出すようにします：
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
現在のディレクトリにいくつかの新しいファイルが作成されます。

{% hint style="success" %}
システム内でより複雑な例を見つけることができます: `mdfind mach_port.defs`\
そして、同じフォルダからファイルをコンパイルすることができます: `mig -DLIBSYSCALL_INTERFACE mach_ports.defs`
{% endhint %}

**`myipcServer.c`** と **`myipcServer.h`** のファイルには、受信したメッセージIDに基づいて呼び出す関数を基本的に定義する **`SERVERPREFmyipc_subsystem`** 構造体の宣言と定義が含まれています（開始番号は500と指定しました）:

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

{% tab title="myipcServer.h" %}次に、`myipcServer.h`ファイルを作成します。このファイルには、MIG関数のプロトタイプが含まれます。{% endtab %}
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

前の構造に基づいて、関数**`myipc_server_routine`**は**メッセージID**を取得し、適切な呼び出すべき関数を返します：
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
この例では、定義で関数を1つだけ定義していますが、複数の関数を定義した場合、それらは**`SERVERPREFmyipc_subsystem`**の配列内にあり、最初の関数はID **500**に割り当てられ、2番目の関数はID **501**に割り当てられるでしょう...

関数が**reply**を送信することが期待される場合、関数`mig_internal kern_return_t __MIG_check__Reply__<name>`も存在します。

実際には、この関係を**`myipcServer.h`**の**`subsystem_to_name_map_myipc`**構造体（他のファイルでは**`subsystem_to_name_map_***`**）から特定することが可能です。
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
最後に、サーバーを動作させるための重要な関数は**`myipc_server`**になります。これは実際に受信したIDに関連する関数を**呼び出す**ものです：

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
/* 最小サイズ：異なる場合はroutine()が更新します */
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

以前に強調された行をチェックして、IDによって呼び出す関数にアクセスします。

以下は、クライアントがサーバーから関数を呼び出すことができる単純な**サーバー**と**クライアント**を作成するコードです：

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

### macOS IPC: Inter-Process Communication

#### macOS MIG: Mach Interface Generator

Mach Interface Generator (MIG) is a tool used to define inter-process communication (IPC) for macOS. It generates client-side and server-side code for message-based IPC. MIG is commonly used in macOS kernel programming for defining system calls and handling IPC between user-space and kernel-space.

To use MIG, you need to define an interface definition file (.defs) that specifies the messages and data structures exchanged between processes. This file is then processed by MIG to generate the necessary C code for IPC.

Here is a basic example of using MIG for IPC in macOS:

1. Define the interface in a .defs file.
2. Run MIG on the .defs file to generate C code.
3. Include the generated C code in your client and server applications.
4. Use the generated functions to send and receive messages between processes.

By leveraging MIG, developers can easily implement secure and efficient IPC mechanisms in macOS applications. 

For more information on MIG and IPC in macOS, refer to the official Apple documentation. 

```c
#include <stdio.h>
#include <mach/mach.h>
#include "myipc.h"

int main() {
    mach_port_t server_port;
    kern_return_t kr;

    kr = bootstrap_look_up(bootstrap_port, "com.example.myipcserver", &server_port);
    if (kr != KERN_SUCCESS) {
        printf("Failed to look up server port\n");
        return 1;
    }

    myipc_server_routine(server_port);

    return 0;
}
```

{% endtab %}
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

NDR\_recordは`libsystem_kernel.dylib`によってエクスポートされ、MIGが異なるシステム間で使用されることを想定していたため、**データをシステムに対して不可知に変換**するための構造体です。

これは興味深いことです。なぜなら、バイナリ中に`_NDR_record`が依存関係として見つかる場合（`jtool2 -S <binary> | grep NDR`または`nm`を使用）、そのバイナリがMIGクライアントまたはサーバーであることを意味します。

さらに、**MIGサーバー**はディスパッチテーブルを`__DATA.__const`（またはmacOSカーネルでは`__CONST.__constdata`、他の\*OSカーネルでは`__DATA_CONST.__const`）に持っています。これは**`jtool2`**でダンプできます。

そして、**MIGクライアント**は`__mach_msg`を使用して`__NDR_record`をサーバーに送信します。

## バイナリ解析

### jtool

多くのバイナリが現在MIGを使用してMachポートを公開しているため、MIGが使用されたことを**特定**し、各メッセージIDごとにMIGが実行する**関数**を特定する方法を知ることは興味深いです。

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2)は、Mach-OバイナリからMIG情報を解析し、メッセージIDを示し、実行する関数を特定することができます。
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
さらに、MIG関数は実際に呼び出される関数のラッパーであるため、その逆アセンブリを取得し、BLを検索することで、呼び出される実際の関数を見つけることができるかもしれません。
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### アセンブリ

以前に、**受信したメッセージIDに応じて正しい関数を呼び出す関数**は`myipc_server`であると述べられていました。ただし、通常はバイナリのシンボル（関数名なし）を持っていないため、**デコンパイルしたものを確認するのが興味深い**です。なぜなら、（この関数のコードは公開された関数に独立しているため）常に非常に似ているからです：

{% tabs %}
{% tab title="myipc_serverのデコンパイル 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// 適切な関数ポインタを見つけるための初期命令
*(int32_t *)var_18 = *(int32_t *)var_10 & 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) <= 0x1f4 && *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// 呼び出す必要がある呼び出しポインタをraxに格納するsign_extend_64の呼び出し
// これにより、呼び出す必要のあるポインタがraxに格納されます
// アドレス0x100004040（関数アドレス配列の使用を確認）
// 0x1f4 = 500（開始ID）
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// if - else文。ifがfalseを返し、elseが正しい関数を呼び出してtrueを返す
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// 2つの引数を使用して適切な関数を呼び出す計算されたアドレス
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

{% tab title="myipc_serverのデコンパイル 2" %}
これは、異なるHopper無料バージョンでデコンパイルされた同じ関数です：

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// 適切な関数ポインタを見つけるための初期命令
*(int32_t *)var_18 = *(int32_t *)var_10 & 0x1f | 0x0;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 > 0x0) {
if (CPU_FLAGS & G) {
r8 = 0x1;
}
}
if ((r8 & 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 < 0x0) {
if (CPU_FLAGS & L) {
r8 = 0x1;
}
}
if ((r8 & 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
// 0x1f4 = 500（開始ID）
<strong>                    r8 = r8 - 0x1f4;
</strong>                    asm { smaddl     x8, w8, w9, x10 };
r8 = *(r8 + 0x8);
var_20 = r8;
r8 = r8 - 0x0;
if (r8 != 0x0) {
if (CPU_FLAGS & NE) {
r8 = 0x1;
}
}
// 前のバージョンと同じif else
// アドレス0x100004040（関数アドレス配列の使用を確認）
<strong>                    if ((r8 & 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// 関数があるべき計算されたアドレスに呼び出し
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

実際には、**`0x100004000`**の関数に移動すると、**`routine_descriptor`**構造体の配列が見つかります。構造体の最初の要素は**関数が実装されているアドレス**であり、**構造体は0x28バイト**を取るため、0バイトから始まる各0x28バイトで8バイトを取得し、それが**呼び出される関数のアドレス**になります：

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

このデータは、[**このHopperスクリプト**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py)を使用して抽出できます。
### デバッグ

MIGによって生成されたコードは、`kernel_debug`を呼び出してエントリと終了時の操作に関するログを生成します。これらを**`trace`**または**`kdv`**を使用してチェックすることが可能です：`kdv all | grep MIG`

## 参考文献

* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

{% hint style="success" %}
AWSハッキングの学習と実践:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **HackTricks**と**HackTricks Cloud**のGitHubリポジトリにPRを提出して、ハッキングトリックを共有してください。

</details>
{% endhint %}
