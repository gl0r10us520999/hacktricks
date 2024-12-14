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

## 基本情報

MIGは**Mach IPC**コード作成のプロセスを**簡素化するために作成されました**。基本的に、**サーバーとクライアントが特定の定義で通信するために必要なコードを生成します**。生成されたコードが醜い場合でも、開発者はそれをインポートするだけで、彼のコードは以前よりもはるかにシンプルになります。

定義は、`.defs`拡張子を使用してインターフェース定義言語（IDL）で指定されます。

これらの定義には5つのセクションがあります：

* **サブシステム宣言**：キーワードのサブシステムは、**名前**と**ID**を示すために使用されます。また、サーバーがカーネルで実行されるべき場合は、**`KernelServer`**としてマークすることも可能です。
* **インクルードとインポート**：MIGはCプリプロセッサを使用しているため、インポートを使用できます。さらに、ユーザーまたはサーバー生成コードのために`uimport`および`simport`を使用することも可能です。
* **型宣言**：データ型を定義することが可能ですが、通常は`mach_types.defs`および`std_types.defs`をインポートします。カスタムのものにはいくつかの構文を使用できます：
* \[i`n/out]tran`：受信メッセージまたは送信メッセージから翻訳する必要がある関数
* `c[user/server]type`：別のC型へのマッピング。
* `destructor`：型が解放されるときにこの関数を呼び出します。
* **操作**：これらはRPCメソッドの定義です。5つの異なるタイプがあります：
* `routine`：応答を期待
* `simpleroutine`：応答を期待しない
* `procedure`：応答を期待
* `simpleprocedure`：応答を期待しない
* `function`：応答を期待

### 例

定義ファイルを作成します。この場合、非常にシンプルな関数を持つファイルです：

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

最初の**引数はバインドするポート**であり、MIGは**応答ポートを自動的に処理します**（クライアントコードで`mig_get_reply_port()`を呼び出さない限り）。さらに、**操作のIDは**指定されたサブシステムIDから**順次**始まります（したがって、操作が非推奨の場合は削除され、`skip`が使用されてそのIDを引き続き使用します）。

次に、MIGを使用して、Subtract関数を呼び出すために互いに通信できるサーバーとクライアントコードを生成します：
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
現在のディレクトリにいくつかの新しいファイルが作成されます。

{% hint style="success" %}
システム内でより複雑な例を見つけるには、`mdfind mach_port.defs`を使用してください。\
また、ファイルと同じフォルダーからコンパイルするには、`mig -DLIBSYSCALL_INTERFACE mach_ports.defs`を使用してください。
{% endhint %}

ファイル**`myipcServer.c`**と**`myipcServer.h`**には、受信したメッセージIDに基づいて呼び出す関数を定義する構造体**`SERVERPREFmyipc_subsystem`**の宣言と定義があります（開始番号500を指定しました）。

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

前の構造に基づいて、関数 **`myipc_server_routine`** は **メッセージID** を取得し、呼び出すべき適切な関数を返します:
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
この例では、定義の中に1つの関数しか定義していませんが、もし他の関数を定義していた場合、それらは**`SERVERPREFmyipc_subsystem`**の配列の中にあり、最初のものはID **500**に、2番目のものはID **501**に割り当てられていました...

関数が**reply**を送信することが期待されている場合、関数`mig_internal kern_return_t __MIG_check__Reply__<name>`も存在します。

実際、この関係は**`myipcServer.h`**の構造体**`subsystem_to_name_map_myipc`**（他のファイルでは**`subsystem_to_name_map_***`**）で特定することが可能です：
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
最終的に、サーバーを機能させるためのもう一つの重要な関数は **`myipc_server`** であり、これは受信したIDに関連する関数を実際に **呼び出す** ものです：

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
/* 最小サイズ: routine() が異なる場合は更新します */
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

IDによって呼び出す関数にアクセスするために強調表示された行を確認してください。

以下は、クライアントがサーバーからSubtract関数を呼び出すことができるシンプルな **サーバー** と **クライアント** を作成するためのコードです：

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

NDR\_recordは`libsystem_kernel.dylib`によってエクスポートされる構造体で、MIGが**システムに依存しないデータを変換する**ことを可能にします。MIGは異なるシステム間で使用されることを意図して設計されているため（同じマシン内だけではなく）。

これは興味深いことで、バイナリ内に依存関係として`_NDR_record`が見つかると（`jtool2 -S <binary> | grep NDR`または`nm`）、そのバイナリがMIGクライアントまたはサーバーであることを意味します。

さらに、**MIGサーバー**は`__DATA.__const`（macOSカーネルでは`__CONST.__constdata`、他の\*OSカーネルでは`__DATA_CONST.__const`）にディスパッチテーブルを持っています。これは**`jtool2`**でダンプできます。

そして、**MIGクライアント**は`__mach_msg`を使用してサーバーに送信するために`__NDR_record`を使用します。

## バイナリ分析

### jtool

多くのバイナリが現在MIGを使用してマッチポートを公開しているため、**MIGが使用されたことを特定する方法**と**各メッセージIDでMIGが実行する関数**を知ることは興味深いです。

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2)は、Mach-OバイナリからMIG情報を解析し、メッセージIDを示し、実行する関数を特定できます：
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
さらに、MIG関数は実際に呼び出される関数のラッパーに過ぎないため、その逆アセンブルを取得し、BLをgrepすることで、実際に呼び出される関数を見つけることができるかもしれません：
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### Assembly

以前に、**受信したメッセージIDに応じて正しい関数を呼び出す**役割を果たす関数は`myipc_server`であると述べました。しかし、通常はバイナリのシンボル（関数名がない）を持っていないため、**デコンパイルされたときの見た目を確認することが興味深い**です。この関数のコードは、公開されている関数とは独立しています。

{% tabs %}
{% tab title="myipc_server decompiled 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// 正しい関数ポインタを見つけるための初期命令
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// sign_extend_64への呼び出しは、この関数を特定するのに役立ちます
// これにより、呼び出す必要がある呼び出しのポインタがraxに格納されます
// アドレス0x100004040（関数アドレス配列）の使用を確認します
// 0x1f4 = 500（開始ID）
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// if - else、ifがfalseを返す場合、elseは正しい関数を呼び出してtrueを返します
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// 2つの引数で正しい関数を呼び出す計算されたアドレス
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
これは、異なるHopperの無料版でデコンパイルされた同じ関数です：

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// 正しい関数ポインタを見つけるための初期命令
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
// 0x1f4 = 500（開始ID）
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
// 前のバージョンと同じif else
// アドレス0x100004040（関数アドレス配列）の使用を確認します
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// 関数がある計算されたアドレスへの呼び出し
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

実際、**`0x100004000`**に行くと、**`routine_descriptor`**構造体の配列が見つかります。構造体の最初の要素は、**関数**が実装されている**アドレス**であり、**構造体は0x28バイト**を占めるため、0バイトから始まる各0x28バイトごとに8バイトを取得すると、それが呼び出される**関数のアドレス**になります。

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

このデータは、[**このHopperスクリプトを使用して**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py)抽出できます。

### Debug

MIGによって生成されたコードは、`kernel_debug`も呼び出して、エントリとエグジットの操作に関するログを生成します。これらは**`trace`**または**`kdv`**を使用して確認できます：`kdv all | grep MIG`

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
