{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

# Mwongozo wa Decompilation ya Wasm na Uundaji wa Wat

Katika ulimwengu wa **WebAssembly**, zana za **decompiling** na **compiling** ni muhimu kwa waendelezaji. Mwongo huu unawasilisha baadhi ya rasilimali za mtandaoni na programu za kushughulikia **Wasm (WebAssembly binary)** na **Wat (WebAssembly text)** files.

## Zana za Mtandaoni

- Ili **decompile** Wasm hadi Wat, zana inayopatikana kwenye [Wabt's wasm2wat demo](https://webassembly.github.io/wabt/demo/wasm2wat/index.html) inasaidia.
- Kwa **compiling** Wat kurudi kwa Wasm, [Wabt's wat2wasm demo](https://webassembly.github.io/wabt/demo/wat2wasm/) inatumika.
- Chaguo kingine cha decompilation kinaweza kupatikana kwenye [web-wasmdec](https://wwwg.github.io/web-wasmdec/).

## Suluhisho za Programu

- Kwa suluhisho thabiti zaidi, [JEB by PNF Software](https://www.pnfsoftware.com/jeb/demo) inatoa vipengele vingi.
- Mradi wa chanzo wazi [wasmdec](https://github.com/wwwg/wasmdec) pia unapatikana kwa kazi za decompilation.

# Rasilimali za Decompilation ya .Net

Decompiling .Net assemblies inaweza kufanywa kwa zana kama:

- [ILSpy](https://github.com/icsharpcode/ILSpy), ambayo pia inatoa [plugin kwa Visual Studio Code](https://github.com/icsharpcode/ilspy-vscode), ikiruhusu matumizi ya cross-platform.
- Kwa kazi zinazohusisha **decompilation**, **modification**, na **recompilation**, [dnSpy](https://github.com/0xd4d/dnSpy/releases) inapendekezwa sana. **Kubonyeza-kulia** kwenye njia na kuchagua **Modify Method** kunaruhusu mabadiliko ya msimbo.
- [JetBrains' dotPeek](https://www.jetbrains.com/es-es/decompiler/) ni chaguo jingine kwa decompiling .Net assemblies.

## Kuimarisha Debugging na Logging na DNSpy

### DNSpy Logging
Ili kuandika taarifa kwenye faili kwa kutumia DNSpy, jumuisha kipande hiki cha msimbo wa .Net:

%%%cpp
using System.IO;
path = "C:\\inetpub\\temp\\MyTest2.txt";
File.AppendAllText(path, "Password: " + password + "\n");
%%%

### DNSpy Debugging
Kwa debugging bora na DNSpy, mfululizo wa hatua unashauriwa kubadilisha **Assembly attributes** kwa ajili ya debugging, kuhakikisha kuwa optimizations ambazo zinaweza kuzuia debugging zimezimwa. Mchakato huu unajumuisha kubadilisha mipangilio ya `DebuggableAttribute`, kuunda upya assembly, na kuokoa mabadiliko.

Zaidi ya hayo, ili kudebug programu ya .Net inayotumiwa na **IIS**, kutekeleza `iisreset /noforce` kunaanzisha upya IIS. Ili kuunganisha DNSpy kwenye mchakato wa IIS kwa ajili ya debugging, mwongozo unashauri kuchagua mchakato wa **w3wp.exe** ndani ya DNSpy na kuanza kikao cha debugging.

Kwa mtazamo wa kina wa moduli zilizopakiwa wakati wa debugging, kufikia dirisha la **Modules** ndani ya DNSpy kunashauriwa, kisha kufungua moduli zote na kupanga assemblies kwa urahisi wa urambazaji na debugging.

Mwongo huu unajumuisha kiini cha WebAssembly na decompilation ya .Net, ukitoa njia kwa waendelezaji kushughulikia kazi hizi kwa urahisi.

## **Java Decompiler**
Ili decompile Java bytecode, zana hizi zinaweza kuwa na msaada mkubwa:
- [jadx](https://github.com/skylot/jadx)
- [JD-GUI](https://github.com/java-decompiler/jd-gui/releases)

## **Debugging DLLs**
### Kutumia IDA
- **Rundll32** inaloadiwa kutoka njia maalum za toleo la 64-bit na 32-bit.
- **Windbg** inachaguliwa kama debugger na chaguo la kusimamisha kwenye upakuaji/kuondoa maktaba limewezeshwa.
- Mipangilio ya utekelezaji inajumuisha njia ya DLL na jina la kazi. Mpangilio huu unasimamisha utekelezaji kila wakati DLL inapoingizwa.

### Kutumia x64dbg/x32dbg
- Kama IDA, **rundll32** inaloadiwa na marekebisho ya mistari ya amri ili kubainisha DLL na kazi.
- Mipangilio inarekebishwa ili kuvunja kwenye kuingia kwa DLL, ikiruhusu kuweka alama ya kuvunja kwenye kiingilio kinachotakiwa cha DLL.

### Picha
- Mipangilio ya kusimamisha utekelezaji na mipangilio inaonyeshwa kupitia picha za skrini.

## **ARM & MIPS**
- Kwa emulation, [arm_now](https://github.com/nongiach/arm_now) ni rasilimali muhimu.

## **Shellcodes**
### Mbinu za Debugging
- **Blobrunner** na **jmp2it** ni zana za kugawa shellcodes kwenye kumbukumbu na kuzi-debug na Ida au x64dbg.
- Blobrunner [toleo](https://github.com/OALabs/BlobRunner/releases/tag/v0.0.5)
- jmp2it [toleo lililotengenezwa](https://github.com/adamkramer/jmp2it/releases/)
- **Cutter** inatoa emulation na ukaguzi wa shellcode kwa kutumia GUI, ikionyesha tofauti katika kushughulikia shellcode kama faili dhidi ya shellcode moja kwa moja.

### Deobfuscation na Uchambuzi
- **scdbg** inatoa maarifa kuhusu kazi za shellcode na uwezo wa deobfuscation.
%%%bash
scdbg.exe -f shellcode # Taarifa za msingi
scdbg.exe -f shellcode -r # Ripoti ya uchambuzi
scdbg.exe -f shellcode -i -r # Vidokezo vya mwingiliano
scdbg.exe -f shellcode -d # Dump shellcode iliyotafsiriwa
scdbg.exe -f shellcode /findsc # Pata ofset ya kuanzia
scdbg.exe -f shellcode /foff 0x0000004D # Tekeleza kutoka ofset
%%%

- **CyberChef** kwa ajili ya kuondoa shellcode: [CyberChef recipe](https://gchq.github.io/CyberChef/#recipe=To_Hex%28'Space',0%29Disassemble_x86%28'32','Full%20x86%20architecture',16,0,true,true%29)

## **Movfuscator**
- Obfuscator inayobadilisha maagizo yote na `mov`.
- Rasilimali muhimu ni pamoja na [ufafanuzi wa YouTube](https://www.youtube.com/watch?v=2VF_wPkiBJY) na [slides za PDF](https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas_2015_the_movfuscator.pdf).
- **demovfuscator** inaweza kubadilisha obfuscation ya movfuscator, ikihitaji utegemezi kama `libcapstone-dev` na `libz3-dev`, na kufunga [keystone](https://github.com/keystone-engine/keystone/blob/master/docs/COMPILE-NIX.md).

## **Delphi**
- Kwa binaries za Delphi, [IDR](https://github.com/crypto2011/IDR) inapendekezwa.


# Kozi

* [https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering](https://github.com/0xZ0F/Z0FCourse_ReverseEngineering)
* [https://github.com/malrev/ABD](https://github.com/malrev/ABD) \(Uondoaji wa binary\)



{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za hacking kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
