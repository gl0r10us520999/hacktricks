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

## Basic Information

MIG foi criado para **simplificar o processo de criação de código Mach IPC**. Ele basicamente **gera o código necessário** para que o servidor e o cliente se comuniquem com uma definição dada. Mesmo que o código gerado seja feio, um desenvolvedor só precisará importá-lo e seu código será muito mais simples do que antes.

A definição é especificada na Linguagem de Definição de Interface (IDL) usando a extensão `.defs`.

Essas definições têm 5 seções:

* **Declaração de subsistema**: A palavra-chave subsistema é usada para indicar o **nome** e o **id**. Também é possível marcá-lo como **`KernelServer`** se o servidor deve ser executado no kernel.
* **Inclusões e imports**: MIG usa o pré-processador C, então é capaz de usar imports. Além disso, é possível usar `uimport` e `simport` para código gerado pelo usuário ou servidor.
* **Declarações de tipo**: É possível definir tipos de dados, embora geralmente ele importe `mach_types.defs` e `std_types.defs`. Para tipos personalizados, pode-se usar alguma sintaxe:
* \[i`n/out]tran`: Função que precisa ser traduzida de uma mensagem de entrada ou para uma mensagem de saída
* `c[user/server]type`: Mapeamento para outro tipo C.
* `destructor`: Chame esta função quando o tipo for liberado.
* **Operações**: Estas são as definições dos métodos RPC. Existem 5 tipos diferentes:
* `routine`: Espera resposta
* `simpleroutine`: Não espera resposta
* `procedure`: Espera resposta
* `simpleprocedure`: Não espera resposta
* `function`: Espera resposta

### Example

Crie um arquivo de definição, neste caso com uma função muito simples:

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

Observe que o primeiro **argumento é a porta a ser vinculada** e o MIG **manipulará automaticamente a porta de resposta** (a menos que `mig_get_reply_port()` seja chamado no código do cliente). Além disso, o **ID das operações** será **sequenial** começando pelo ID do subsistema indicado (então, se uma operação for obsoleta, ela é excluída e `skip` é usado para ainda usar seu ID).

Agora use o MIG para gerar o código do servidor e do cliente que será capaz de se comunicar entre si para chamar a função Subtract:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
Vários novos arquivos serão criados no diretório atual.

{% hint style="success" %}
Você pode encontrar um exemplo mais complexo em seu sistema com: `mdfind mach_port.defs`\
E você pode compilá-lo a partir da mesma pasta que o arquivo com: `mig -DLIBSYSCALL_INTERFACE mach_ports.defs`
{% endhint %}

Nos arquivos **`myipcServer.c`** e **`myipcServer.h`** você pode encontrar a declaração e definição da struct **`SERVERPREFmyipc_subsystem`**, que basicamente define a função a ser chamada com base no ID da mensagem recebida (indicamos um número inicial de 500):

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

Com base na estrutura anterior, a função **`myipc_server_routine`** obterá o **ID da mensagem** e retornará a função apropriada a ser chamada:
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
Neste exemplo, definimos apenas 1 função nas definições, mas se tivéssemos definido mais funções, elas estariam dentro do array de **`SERVERPREFmyipc_subsystem`** e a primeira teria sido atribuída ao ID **500**, a segunda ao ID **501**...

Se a função fosse esperada para enviar uma **reply**, a função `mig_internal kern_return_t __MIG_check__Reply__<name>` também existiria.

Na verdade, é possível identificar essa relação na struct **`subsystem_to_name_map_myipc`** de **`myipcServer.h`** (**`subsystem_to_name_map_***`** em outros arquivos):
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
Finalmente, outra função importante para fazer o servidor funcionar será **`myipc_server`**, que é a que realmente **chamará a função** relacionada ao id recebido:

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
/* Tamanho mínimo: routine() irá atualizá-lo se diferente */
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

Verifique as linhas destacadas anteriormente acessando a função a ser chamada pelo ID.

O seguinte é o código para criar um **servidor** e **cliente** simples onde o cliente pode chamar as funções Subtrair do servidor:

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

### O NDR\_record

O NDR\_record é exportado por `libsystem_kernel.dylib`, e é uma struct que permite que o MIG **transforme dados para que sejam agnósticos ao sistema** em que está sendo usado, já que o MIG foi pensado para ser usado entre diferentes sistemas (e não apenas na mesma máquina).

Isso é interessante porque se `_NDR_record` for encontrado em um binário como uma dependência (`jtool2 -S <binary> | grep NDR` ou `nm`), isso significa que o binário é um cliente ou servidor MIG.

Além disso, **servidores MIG** têm a tabela de despacho em `__DATA.__const` (ou em `__CONST.__constdata` no kernel do macOS e `__DATA_CONST.__const` em outros kernels \*OS). Isso pode ser despejado com **`jtool2`**.

E **clientes MIG** usarão o `__NDR_record` para enviar com `__mach_msg` para os servidores.

## Análise de Binários

### jtool

Como muitos binários agora usam MIG para expor portas mach, é interessante saber como **identificar que o MIG foi usado** e as **funções que o MIG executa** com cada ID de mensagem.

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) pode analisar informações do MIG a partir de um binário Mach-O, indicando o ID da mensagem e identificando a função a ser executada:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
Além disso, as funções MIG são apenas envoltórios da função real que é chamada, o que significa que, ao obter sua desassemblagem e procurar por BL, você pode ser capaz de encontrar a função real que está sendo chamada:
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### Assembly

Foi mencionado anteriormente que a função que se encarregará de **chamar a função correta dependendo do ID da mensagem recebida** era `myipc_server`. No entanto, você geralmente não terá os símbolos do binário (sem nomes de funções), então é interessante **ver como ela se parece decompilada**, pois será sempre muito semelhante (o código desta função é independente das funções expostas):

{% tabs %}
{% tab title="myipc_server decompiled 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// Instruções iniciais para encontrar os ponteiros de função apropriados
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// Chamada para sign_extend_64 que pode ajudar a identificar esta função
// Isso armazena em rax o ponteiro para a chamada que precisa ser chamada
// Verifique o uso do endereço 0x100004040 (array de endereços de funções)
// 0x1f4 = 500 (o ID inicial)
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// Se - else, o if retorna falso, enquanto o else chama a função correta e retorna verdadeiro
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// Endereço calculado que chama a função apropriada com 2 argumentos
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
Esta é a mesma função decompilada em uma versão diferente do Hopper free:

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// Instruções iniciais para encontrar os ponteiros de função apropriados
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
// 0x1f4 = 500 (o ID inicial)
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
// Mesmo if else que na versão anterior
// Verifique o uso do endereço 0x100004040 (array de endereços de funções)
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// Chamada para o endereço calculado onde a função deve estar
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

Na verdade, se você for para a função **`0x100004000`**, encontrará o array de **`routine_descriptor`** structs. O primeiro elemento da struct é o **endereço** onde a **função** está implementada, e a **struct ocupa 0x28 bytes**, então a cada 0x28 bytes (começando do byte 0) você pode obter 8 bytes e esse será o **endereço da função** que será chamada:

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Esses dados podem ser extraídos [**usando este script do Hopper**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py).

### Debug

O código gerado pelo MIG também chama `kernel_debug` para gerar logs sobre operações de entrada e saída. É possível verificá-los usando **`trace`** ou **`kdv`**: `kdv all | grep MIG`

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
