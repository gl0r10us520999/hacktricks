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

## Podstawowe informacje

MIG został stworzony, aby **uprościć proces tworzenia kodu Mach IPC**. W zasadzie **generuje potrzebny kod** dla serwera i klienta, aby mogły komunikować się zgodnie z daną definicją. Nawet jeśli wygenerowany kod jest brzydki, programista będzie musiał go tylko zaimportować, a jego kod będzie znacznie prostszy niż wcześniej.

Definicja jest określona w języku definicji interfejsu (IDL) przy użyciu rozszerzenia `.defs`.

Te definicje mają 5 sekcji:

* **Deklaracja podsystemu**: Słowo kluczowe subsystem jest używane do wskazania **nazwa** i **id**. Możliwe jest również oznaczenie go jako **`KernelServer`**, jeśli serwer ma działać w jądrze.
* **Inkluzje i importy**: MIG używa preprocesora C, więc może korzystać z importów. Ponadto możliwe jest użycie `uimport` i `simport` dla kodu generowanego przez użytkownika lub serwer.
* **Deklaracje typów**: Możliwe jest zdefiniowanie typów danych, chociaż zazwyczaj zaimportuje `mach_types.defs` i `std_types.defs`. Dla niestandardowych można użyć pewnej składni:
* \[i`n/out]tran`: Funkcja, która musi być przetłumaczona z wiadomości przychodzącej lub do wiadomości wychodzącej
* `c[user/server]type`: Mapowanie na inny typ C.
* `destructor`: Wywołaj tę funkcję, gdy typ zostanie zwolniony.
* **Operacje**: To są definicje metod RPC. Istnieje 5 różnych typów:
* `routine`: Oczekuje odpowiedzi
* `simpleroutine`: Nie oczekuje odpowiedzi
* `procedure`: Oczekuje odpowiedzi
* `simpleprocedure`: Nie oczekuje odpowiedzi
* `function`: Oczekuje odpowiedzi

### Przykład

Utwórz plik definicji, w tym przypadku z bardzo prostą funkcją:

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

Zauważ, że pierwszy **argument to port do powiązania** a MIG **automatycznie obsłuży port odpowiedzi** (chyba że wywołasz `mig_get_reply_port()` w kodzie klienta). Ponadto, **ID operacji** będzie **sekwencyjne**, zaczynając od wskazanego ID podsystemu (więc jeśli operacja jest przestarzała, jest usuwana, a `skip` jest używane, aby nadal używać jej ID).

Teraz użyj MIG, aby wygenerować kod serwera i klienta, który będzie w stanie komunikować się ze sobą, aby wywołać funkcję Subtract:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
W bieżącym katalogu zostanie utworzonych kilka nowych plików.

{% hint style="success" %}
Możesz znaleźć bardziej złożony przykład w swoim systemie za pomocą: `mdfind mach_port.defs`\
A możesz go skompilować z tego samego folderu co plik za pomocą: `mig -DLIBSYSCALL_INTERFACE mach_ports.defs`
{% endhint %}

W plikach **`myipcServer.c`** i **`myipcServer.h`** znajdziesz deklarację i definicję struktury **`SERVERPREFmyipc_subsystem`**, która zasadniczo definiuje funkcję do wywołania na podstawie otrzymanego identyfikatora wiadomości (wskazaliśmy początkowy numer 500):

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

Na podstawie poprzedniej struktury funkcja **`myipc_server_routine`** otrzyma **ID wiadomości** i zwróci odpowiednią funkcję do wywołania:
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
W tym przykładzie zdefiniowaliśmy tylko 1 funkcję w definicjach, ale gdybyśmy zdefiniowali więcej funkcji, byłyby one wewnątrz tablicy **`SERVERPREFmyipc_subsystem`**, a pierwsza zostałaby przypisana do ID **500**, druga do ID **501**...

Jeśli oczekiwano, że funkcja wyśle **reply**, funkcja `mig_internal kern_return_t __MIG_check__Reply__<name>` również by istniała.

W rzeczywistości możliwe jest zidentyfikowanie tej relacji w strukturze **`subsystem_to_name_map_myipc`** z **`myipcServer.h`** (**`subsystem_to_name_map_***`** w innych plikach):
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
Finally, another important function to make the server work will be **`myipc_server`**, which is the one that will actually **call the function** related to the received id:

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
/* Minimal size: routine() will update it if different */
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

Check the previously highlighted lines accessing the function to call by ID.

The following is the code to create a simple **server** and **client** where the client can call the functions Subtract from the server:

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

NDR\_record jest eksportowany przez `libsystem_kernel.dylib` i jest to struktura, która pozwala MIG na **transformację danych, aby były niezależne od systemu**, w którym są używane, ponieważ MIG był zaprojektowany do użycia między różnymi systemami (a nie tylko na tej samej maszynie).

To jest interesujące, ponieważ jeśli `_NDR_record` zostanie znaleziony w binarnym pliku jako zależność (`jtool2 -S <binary> | grep NDR` lub `nm`), oznacza to, że binarny plik jest klientem lub serwerem MIG.

Ponadto **serwery MIG** mają tabelę dyspozycyjną w `__DATA.__const` (lub w `__CONST.__constdata` w jądrze macOS i `__DATA_CONST.__const` w innych jądrze \*OS). Można to zrzucić za pomocą **`jtool2`**.

A **klienci MIG** będą używać `__NDR_record`, aby wysłać z `__mach_msg` do serwerów.

## Analiza binarna

### jtool

Ponieważ wiele binarnych plików teraz używa MIG do udostępniania portów mach, interesujące jest wiedzieć, jak **zidentyfikować, że MIG był używany** oraz **funkcje, które MIG wykonuje** z każdym identyfikatorem wiadomości.

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) może analizować informacje MIG z binarnego pliku Mach-O, wskazując identyfikator wiadomości i identyfikując funkcję do wykonania:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
Ponadto, funkcje MIG są jedynie opakowaniami rzeczywistej funkcji, która jest wywoływana, co oznacza, że uzyskując jej dezasemblację i przeszukując pod kątem BL, możesz być w stanie znaleźć rzeczywistą funkcję, która jest wywoływana:
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### Assembly

Wcześniej wspomniano, że funkcja, która zajmie się **wywoływaniem odpowiedniej funkcji w zależności od otrzymanego identyfikatora wiadomości**, to `myipc_server`. Jednak zazwyczaj nie będziesz miał symboli binarnych (brak nazw funkcji), więc interesujące jest **sprawdzić, jak wygląda dekompilacja**, ponieważ zawsze będzie bardzo podobna (kod tej funkcji jest niezależny od funkcji eksponowanych):

{% tabs %}
{% tab title="myipc_server decompiled 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// Wstępne instrukcje do znalezienia odpowiednich wskaźników funkcji
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// Wywołanie sign_extend_64, które może pomóc w identyfikacji tej funkcji
// To przechowuje w rax wskaźnik do wywołania, które musi być wywołane
// Sprawdź użycie adresu 0x100004040 (tablica adresów funkcji)
// 0x1f4 = 500 (początkowy ID)
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// Jeśli - else, if zwraca fałsz, podczas gdy else wywołuje odpowiednią funkcję i zwraca prawdę
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// Obliczony adres, który wywołuje odpowiednią funkcję z 2 argumentami
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
To ta sama funkcja dekompilowana w innej wersji Hopper free:

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// Wstępne instrukcje do znalezienia odpowiednich wskaźników funkcji
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
// 0x1f4 = 500 (początkowy ID)
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
// To samo if else jak w poprzedniej wersji
// Sprawdź użycie adresu 0x100004040 (tablica adresów funkcji)
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// Wywołanie obliczonego adresu, gdzie powinna być funkcja
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

W rzeczywistości, jeśli przejdziesz do funkcji **`0x100004000`**, znajdziesz tablicę struktur **`routine_descriptor`**. Pierwszym elementem struktury jest **adres**, w którym **funkcja** jest zaimplementowana, a **struktura zajmuje 0x28 bajtów**, więc co 0x28 bajtów (zaczynając od bajtu 0) możesz uzyskać 8 bajtów, a to będzie **adres funkcji**, która zostanie wywołana:

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Te dane można wyodrębnić [**używając tego skryptu Hopper**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py).

### Debug

Kod generowany przez MIG również wywołuje `kernel_debug`, aby generować logi dotyczące operacji przy wejściu i wyjściu. Można je sprawdzić, używając **`trace`** lub **`kdv`**: `kdv all | grep MIG`

## References

* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

{% hint style="success" %}
Ucz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Ucz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wsparcie dla HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się sztuczkami hackingowymi, przesyłając PR do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
