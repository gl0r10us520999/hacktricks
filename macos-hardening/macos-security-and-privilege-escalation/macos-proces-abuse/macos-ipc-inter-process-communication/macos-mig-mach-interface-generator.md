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

## Grundinformationen

MIG wurde entwickelt, um den **Prozess der Erstellung von Mach IPC**-Code zu **vereinfachen**. Es **generiert den benötigten Code** für Server und Client, um mit einer gegebenen Definition zu kommunizieren. Auch wenn der generierte Code unansehnlich ist, muss ein Entwickler ihn nur importieren, und sein Code wird viel einfacher sein als zuvor.

Die Definition wird in der Interface Definition Language (IDL) mit der Erweiterung `.defs` angegeben.

Diese Definitionen haben 5 Abschnitte:

* **Subsystemdeklaration**: Das Schlüsselwort subsystem wird verwendet, um den **Namen** und die **ID** anzugeben. Es ist auch möglich, es als **`KernelServer`** zu kennzeichnen, wenn der Server im Kernel ausgeführt werden soll.
* **Inklusionen und Importe**: MIG verwendet den C-Präprozessor, sodass es in der Lage ist, Importe zu verwenden. Darüber hinaus ist es möglich, `uimport` und `simport` für benutzer- oder servergenerierten Code zu verwenden.
* **Typdeklarationen**: Es ist möglich, Datentypen zu definieren, obwohl normalerweise `mach_types.defs` und `std_types.defs` importiert werden. Für benutzerdefinierte Typen kann eine bestimmte Syntax verwendet werden:
* \[i`n/out]tran`: Funktion, die von einer eingehenden oder zu einer ausgehenden Nachricht übersetzt werden muss
* `c[user/server]type`: Zuordnung zu einem anderen C-Typ.
* `destructor`: Rufen Sie diese Funktion auf, wenn der Typ freigegeben wird.
* **Operationen**: Dies sind die Definitionen der RPC-Methoden. Es gibt 5 verschiedene Typen:
* `routine`: Erwartet eine Antwort
* `simpleroutine`: Erwartet keine Antwort
* `procedure`: Erwartet eine Antwort
* `simpleprocedure`: Erwartet keine Antwort
* `function`: Erwartet eine Antwort

### Beispiel

Erstellen Sie eine Definitionsdatei, in diesem Fall mit einer sehr einfachen Funktion:

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

Beachten Sie, dass das erste **Argument der Port ist, an den gebunden werden soll** und MIG **automatisch den Antwortport verwaltet** (es sei denn, `mig_get_reply_port()` wird im Client-Code aufgerufen). Darüber hinaus wird die **ID der Operationen** **sequentiell** beginnend mit der angegebenen Subsystem-ID sein (wenn eine Operation veraltet ist, wird sie gelöscht und `skip` wird verwendet, um ihre ID weiterhin zu verwenden).

Verwenden Sie nun MIG, um den Server- und Client-Code zu generieren, der in der Lage ist, miteinander zu kommunizieren, um die Subtract-Funktion aufzurufen:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
Mehrere neue Dateien werden im aktuellen Verzeichnis erstellt.

{% hint style="success" %}
Sie können ein komplexeres Beispiel in Ihrem System mit: `mdfind mach_port.defs` finden.\
Und Sie können es aus demselben Ordner wie die Datei mit: `mig -DLIBSYSCALL_INTERFACE mach_ports.defs` kompilieren.
{% endhint %}

In den Dateien **`myipcServer.c`** und **`myipcServer.h`** finden Sie die Deklaration und Definition der Struktur **`SERVERPREFmyipc_subsystem`**, die im Grunde die Funktion definiert, die basierend auf der empfangenen Nachrichten-ID aufgerufen werden soll (wir haben eine Startnummer von 500 angegeben):

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

Basierend auf der vorherigen Struktur wird die Funktion **`myipc_server_routine`** die **Nachrichten-ID** erhalten und die entsprechende Funktion zurückgeben, die aufgerufen werden soll:
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
In diesem Beispiel haben wir nur 1 Funktion in den Definitionen definiert, aber wenn wir mehr Funktionen definiert hätten, wären sie im Array von **`SERVERPREFmyipc_subsystem`** enthalten gewesen und die erste wäre der ID **500** zugewiesen worden, die zweite der ID **501**...

Wenn die Funktion erwartet wurde, eine **Antwort** zu senden, würde die Funktion `mig_internal kern_return_t __MIG_check__Reply__<name>` ebenfalls existieren.

Tatsächlich ist es möglich, diese Beziehung in der Struktur **`subsystem_to_name_map_myipc`** aus **`myipcServer.h`** (**`subsystem_to_name_map_***`** in anderen Dateien) zu identifizieren:
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
Schließlich ist eine weitere wichtige Funktion, um den Server zum Laufen zu bringen, **`myipc_server`**, die tatsächlich **die Funktion** aufruft, die mit der empfangenen ID verbunden ist:

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
/* Minimale Größe: routine() wird sie aktualisieren, wenn sie unterschiedlich ist */
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

Überprüfen Sie die zuvor hervorgehobenen Zeilen, die auf die Funktion zugreifen, die nach ID aufgerufen werden soll.

Der folgende Code erstellt einen einfachen **Server** und **Client**, bei dem der Client die Funktionen Subtract vom Server aufrufen kann:

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

### Der NDR\_record

Der NDR\_record wird von `libsystem_kernel.dylib` exportiert und ist eine Struktur, die es MIG ermöglicht, **Daten so zu transformieren, dass sie systemunabhängig sind**, da MIG ursprünglich für die Verwendung zwischen verschiedenen Systemen (und nicht nur auf derselben Maschine) gedacht war.

Dies ist interessant, da das Vorhandensein von `_NDR_record` in einer Binärdatei als Abhängigkeit (`jtool2 -S <binary> | grep NDR` oder `nm`) bedeutet, dass die Binärdatei ein MIG-Client oder -Server ist.

Darüber hinaus haben **MIG-Server** die Dispatch-Tabelle in `__DATA.__const` (oder in `__CONST.__constdata` im macOS-Kernel und `__DATA_CONST.__const` in anderen \*OS-Kernen). Dies kann mit **`jtool2`** ausgegeben werden.

Und **MIG-Clients** verwenden den `__NDR_record`, um mit `__mach_msg` an die Server zu senden.

## Binäranalyse

### jtool

Da viele Binärdateien jetzt MIG verwenden, um Mach-Ports bereitzustellen, ist es interessant zu wissen, wie man **erkennt, dass MIG verwendet wurde** und die **Funktionen, die MIG mit jeder Nachrichten-ID ausführt**.

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) kann MIG-Informationen aus einer Mach-O-Binärdatei analysieren, die die Nachrichten-ID angibt und die auszuführende Funktion identifiziert:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
Darüber hinaus sind MIG-Funktionen nur Wrapper der tatsächlichen Funktion, die aufgerufen wird, was bedeutet, dass Sie durch das Abrufen ihrer Disassemblierung und das Durchsuchen nach BL möglicherweise die tatsächlich aufgerufene Funktion finden können:
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### Assembly

Es wurde zuvor erwähnt, dass die Funktion, die sich um **den Aufruf der richtigen Funktion je nach empfangenem Nachrichten-ID** kümmert, `myipc_server` war. Allerdings haben Sie normalerweise nicht die Symbole der Binärdatei (keine Funktionsnamen), daher ist es interessant zu **überprüfen, wie es dekompiliert aussieht**, da der Code dieser Funktion immer sehr ähnlich sein wird (der Code dieser Funktion ist unabhängig von den exponierten Funktionen):

{% tabs %}
{% tab title="myipc_server decompiled 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// Anfangsinstruktionen, um die richtigen Funktionszeiger zu finden
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// Aufruf von sign_extend_64, der helfen kann, diese Funktion zu identifizieren
// Dies speichert in rax den Zeiger auf den Aufruf, der aufgerufen werden muss
// Überprüfen Sie die Verwendung der Adresse 0x100004040 (Funktionsadressenarray)
// 0x1f4 = 500 (die Start-ID)
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// Wenn - sonst, gibt die if false zurück, während else die richtige Funktion aufruft und true zurückgibt
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// Berechnete Adresse, die die richtige Funktion mit 2 Argumenten aufruft
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
Dies ist die gleiche Funktion, die in einer anderen kostenlosen Hopper-Version dekompiliert wurde:

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// Anfangsinstruktionen, um die richtigen Funktionszeiger zu finden
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
// 0x1f4 = 500 (die Start-ID)
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
// Gleiches if else wie in der vorherigen Version
// Überprüfen Sie die Verwendung der Adresse 0x100004040 (Funktionsadressenarray)
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// Aufruf der berechneten Adresse, wo die Funktion sein sollte
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

Tatsächlich, wenn Sie zur Funktion **`0x100004000`** gehen, finden Sie das Array von **`routine_descriptor`** Strukturen. Das erste Element der Struktur ist die **Adresse**, an der die **Funktion** implementiert ist, und die **Struktur benötigt 0x28 Bytes**, sodass Sie alle 0x28 Bytes (beginnend bei Byte 0) 8 Bytes erhalten können, und das wird die **Adresse der Funktion** sein, die aufgerufen wird:

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Diese Daten können [**mit diesem Hopper-Skript extrahiert werden**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py).

### Debug

Der von MIG generierte Code ruft auch `kernel_debug` auf, um Protokolle über Operationen beim Eintritt und Austritt zu generieren. Es ist möglich, sie mit **`trace`** oder **`kdv`** zu überprüfen: `kdv all | grep MIG`

## References

* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichen.

</details>
{% endhint %}
