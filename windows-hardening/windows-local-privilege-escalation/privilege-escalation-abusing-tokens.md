# Tokens का दुरुपयोग

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करके।

</details>
{% endhint %}

## Tokens

यदि आप **Windows Access Tokens क्या हैं** नहीं जानते हैं, तो आगे बढ़ने से पहले इस पृष्ठ को पढ़ें:

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

**शायद आप पहले से मौजूद टोकनों का दुरुपयोग करके विशेषाधिकार बढ़ाने में सक्षम हो सकते हैं**

### SeImpersonatePrivilege

यह विशेषाधिकार किसी भी प्रक्रिया द्वारा धारण किया जाता है जो किसी भी टोकन का अनुकरण (लेकिन निर्माण नहीं) करने की अनुमति देता है, बशर्ते कि इसके लिए एक हैंडल प्राप्त किया जा सके। एक विशेषाधिकार प्राप्त टोकन को Windows सेवा (DCOM) से NTLM प्रमाणीकरण को एक एक्सप्लॉइट के खिलाफ प्रेरित करके प्राप्त किया जा सकता है, जिसके बाद SYSTEM विशेषाधिकार के साथ एक प्रक्रिया के निष्पादन की अनुमति मिलती है। इस भेद्यता का दुरुपयोग विभिन्न उपकरणों का उपयोग करके किया जा सकता है, जैसे [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM](https://github.com/antonioCoco/RogueWinRM) (जिसके लिए winrm को अक्षम करना आवश्यक है), [SweetPotato](https://github.com/CCob/SweetPotato), और [PrintSpoofer](https://github.com/itm4n/PrintSpoofer).

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="juicypotato.md" %}
[juicypotato.md](juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege

यह **SeImpersonatePrivilege** के समान है, यह एक विशेषाधिकार प्राप्त टोकन प्राप्त करने के लिए **समान विधि** का उपयोग करेगा।\
फिर, यह विशेषाधिकार **एक नए/स्थगित प्रक्रिया** को एक प्राथमिक टोकन असाइन करने की अनुमति देता है। विशेषाधिकार प्राप्त अनुकरण टोकन के साथ आप एक प्राथमिक टोकन (DuplicateTokenEx) उत्पन्न कर सकते हैं।\
इस टोकन के साथ, आप 'CreateProcessAsUser' के साथ एक **नई प्रक्रिया** बना सकते हैं या एक प्रक्रिया को स्थगित कर सकते हैं और **टोकन सेट कर सकते हैं** (सामान्यतः, आप एक चल रही प्रक्रिया के प्राथमिक टोकन को संशोधित नहीं कर सकते)।

### SeTcbPrivilege

यदि आपने इस टोकन को सक्षम किया है, तो आप **KERB\_S4U\_LOGON** का उपयोग करके किसी अन्य उपयोगकर्ता के लिए **अनुकरण टोकन** प्राप्त कर सकते हैं बिना क्रेडेंशियल्स को जाने, **टोकन में एक मनमाना समूह** (admins) जोड़ सकते हैं, टोकन के **अखंडता स्तर** को "**मध्यम**" पर सेट कर सकते हैं, और इस टोकन को **वर्तमान थ्रेड** पर असाइन कर सकते हैं (SetThreadToken)।

### SeBackupPrivilege

यह विशेषाधिकार किसी भी फ़ाइल (पढ़ने के संचालन तक सीमित) को **सभी पढ़ने की पहुँच** नियंत्रण देने का कारण बनता है। इसका उपयोग **स्थानीय व्यवस्थापक** खातों के पासवर्ड हैश को रजिस्ट्री से पढ़ने के लिए किया जाता है, जिसके बाद, "**psexec**" या "**wmiexec**" जैसे उपकरणों का उपयोग हैश के साथ किया जा सकता है (Pass-the-Hash तकनीक)। हालाँकि, यह तकनीक दो स्थितियों में विफल होती है: जब स्थानीय व्यवस्थापक खाता अक्षम होता है, या जब एक नीति लागू होती है जो दूरस्थ रूप से कनेक्ट करने वाले स्थानीय व्यवस्थापकों से प्रशासनिक अधिकार हटा देती है।\
आप इस विशेषाधिकार का **दुरुपयोग** कर सकते हैं:

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec) में **IppSec** का पालन करें
* या जैसा कि **बैकअप ऑपरेटरों के साथ विशेषाधिकार बढ़ाने** के अनुभाग में समझाया गया है:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege

यह विशेषाधिकार किसी भी सिस्टम फ़ाइल के लिए **लिखने की पहुँच** की अनुमति देता है, चाहे फ़ाइल की एक्सेस कंट्रोल लिस्ट (ACL) कुछ भी हो। यह कई संभावनाओं को खोलता है, जिसमें **सेवाओं को संशोधित करना**, DLL Hijacking करना, और विभिन्न अन्य तकनीकों के बीच इमेज फ़ाइल निष्पादन विकल्पों के माध्यम से **डिबगर** सेट करना शामिल है।

### SeCreateTokenPrivilege

SeCreateTokenPrivilege एक शक्तिशाली अनुमति है, विशेष रूप से तब उपयोगी होती है जब एक उपयोगकर्ता के पास टोकनों का अनुकरण करने की क्षमता होती है, लेकिन SeImpersonatePrivilege की अनुपस्थिति में भी। यह क्षमता उस टोकन के अनुकरण की क्षमता पर निर्भर करती है जो उसी उपयोगकर्ता का प्रतिनिधित्व करता है और जिसका अखंडता स्तर वर्तमान प्रक्रिया के स्तर से अधिक नहीं होता है।

**मुख्य बिंदु:**

* **SeImpersonatePrivilege के बिना अनुकरण:** विशेष परिस्थितियों में टोकनों का अनुकरण करके EoP के लिए SeCreateTokenPrivilege का लाभ उठाना संभव है।
* **टोकन अनुकरण के लिए शर्तें:** सफल अनुकरण के लिए लक्षित टोकन को उसी उपयोगकर्ता का होना चाहिए और इसका अखंडता स्तर उस प्रक्रिया के अखंडता स्तर से कम या बराबर होना चाहिए जो अनुकरण करने का प्रयास कर रही है।
* **अनुकरण टोकनों का निर्माण और संशोधन:** उपयोगकर्ता एक अनुकरण टोकन बना सकते हैं और इसे एक विशेषाधिकार प्राप्त समूह के SID (सुरक्षा पहचानकर्ता) को जोड़कर बढ़ा सकते हैं।

### SeLoadDriverPrivilege

यह विशेषाधिकार **डिवाइस ड्राइवरों को लोड और अनलोड** करने की अनुमति देता है, जिसमें `ImagePath` और `Type` के लिए विशिष्ट मानों के साथ रजिस्ट्री प्रविष्टि का निर्माण शामिल है। चूंकि `HKLM` (HKEY\_LOCAL\_MACHINE) पर सीधे लिखने की पहुँच प्रतिबंधित है, इसलिए `HKCU` (HKEY\_CURRENT\_USER) का उपयोग किया जाना चाहिए। हालाँकि, ड्राइवर कॉन्फ़िगरेशन के लिए `HKCU` को कर्नेल द्वारा मान्यता प्राप्त करने के लिए, एक विशिष्ट पथ का पालन करना आवश्यक है।

यह पथ `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName` है, जहाँ `<RID>` वर्तमान उपयोगकर्ता का सापेक्ष पहचानकर्ता है। `HKCU` के अंदर, इस पूरे पथ को बनाना होगा, और दो मान सेट करने होंगे:

* `ImagePath`, जो निष्पादित होने वाले बाइनरी का पथ है
* `Type`, जिसका मान `SERVICE_KERNEL_DRIVER` (`0x00000001`) है।

**अनुसरण करने के चरण:**

1. प्रतिबंधित लिखने की पहुँच के कारण `HKLM` के बजाय `HKCU` तक पहुँचें।
2. `HKCU` के भीतर `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName` पथ बनाएं, जहाँ `<RID>` वर्तमान उपयोगकर्ता का सापेक्ष पहचानकर्ता है।
3. `ImagePath` को बाइनरी के निष्पादन पथ पर सेट करें।
4. `Type` को `SERVICE_KERNEL_DRIVER` (`0x00000001`) के रूप में असाइन करें।
```python
# Example Python code to set the registry values
import winreg as reg

# Define the path and values
path = r'Software\YourPath\System\CurrentControlSet\Services\DriverName' # Adjust 'YourPath' as needed
key = reg.OpenKey(reg.HKEY_CURRENT_USER, path, 0, reg.KEY_WRITE)
reg.SetValueEx(key, "ImagePath", 0, reg.REG_SZ, "path_to_binary")
reg.SetValueEx(key, "Type", 0, reg.REG_DWORD, 0x00000001)
reg.CloseKey(key)
```
More ways to abuse this privilege in [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege)

### SeTakeOwnershipPrivilege

यह **SeRestorePrivilege** के समान है। इसका मुख्य कार्य एक प्रक्रिया को **एक ऑब्जेक्ट का स्वामित्व ग्रहण करने** की अनुमति देना है, जो WRITE\_OWNER एक्सेस अधिकारों के प्रावधान के माध्यम से स्पष्ट विवेकाधीन पहुंच की आवश्यकता को दरकिनार करता है। प्रक्रिया में पहले लिखने के उद्देश्यों के लिए इच्छित रजिस्ट्री कुंजी का स्वामित्व सुरक्षित करना शामिल है, फिर लिखने के संचालन को सक्षम करने के लिए DACL को बदलना शामिल है।
```bash
takeown /f 'C:\some\file.txt' #Now the file is owned by you
icacls 'C:\some\file.txt' /grant <your_username>:F #Now you have full access
# Use this with files that might contain credentials such as
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software
%WINDIR%\repair\security
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
c:\inetpub\wwwwroot\web.config
```
### SeDebugPrivilege

यह विशेषाधिकार **अन्य प्रक्रियाओं को डिबग करने** की अनुमति देता है, जिसमें मेमोरी में पढ़ने और लिखने की क्षमता शामिल है। मेमोरी इंजेक्शन के लिए विभिन्न रणनीतियाँ, जो अधिकांश एंटीवायरस और होस्ट घुसपैठ रोकथाम समाधानों को चकमा देने में सक्षम हैं, इस विशेषाधिकार के साथ लागू की जा सकती हैं।

#### Dump memory

आप [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) का उपयोग [SysInternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) से **एक प्रक्रिया की मेमोरी कैप्चर करने** के लिए कर सकते हैं। विशेष रूप से, यह **स्थानीय सुरक्षा प्राधिकरण उपप्रणाली सेवा (**[**LSASS**](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)**)** प्रक्रिया पर लागू हो सकता है, जो एक बार उपयोगकर्ता के सफलतापूर्वक सिस्टम में लॉग इन करने के बाद उपयोगकर्ता क्रेडेंशियल्स को संग्रहीत करने के लिए जिम्मेदार है।

आप फिर इस डंप को mimikatz में लोड कर सकते हैं ताकि पासवर्ड प्राप्त कर सकें:
```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```
#### RCE

यदि आप `NT SYSTEM` शेल प्राप्त करना चाहते हैं तो आप उपयोग कर सकते हैं:

* [**SeDebugPrivilege-Exploit (C++)**](https://github.com/bruno-1337/SeDebugPrivilege-Exploit)
* [**SeDebugPrivilegePoC (C#)**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)
* [**psgetsys.ps1 (Powershell Script)**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)
```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```
## विशेषाधिकार जांचें
```
whoami /priv
```
The **tokens that appear as Disabled** को सक्षम किया जा सकता है, आप वास्तव में _Enabled_ और _Disabled_ tokens का दुरुपयोग कर सकते हैं।

### सभी tokens सक्षम करें

यदि आपके पास tokens निष्क्रिय हैं, तो आप सभी tokens को सक्षम करने के लिए स्क्रिप्ट [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) का उपयोग कर सकते हैं:
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
Or the **script** embed in this [**post**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/).

## Table

Full token privileges cheatsheet at [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin), summary below will only list direct ways to exploit the privilege to obtain an admin session or read sensitive files.

| Privilege                  | Impact      | Tool                    | Execution path                                                                                                                                                                                                                                                                                                                                     | Remarks                                                                                                                                                                                                                                                                                                                        |
| -------------------------- | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**Admin**_ | 3rd party tool          | _"यह एक उपयोगकर्ता को टोकन का अनुकरण करने और potato.exe, rottenpotato.exe और juicypotato.exe जैसे उपकरणों का उपयोग करके nt सिस्टम में प्रिवेस्क करने की अनुमति देगा"_                                                                                                                                                                                                      | Thank you [Aurélien Chalot](https://twitter.com/Defte\_) for the update. I will try to re-phrase it to something more recipe-like soon.                                                                                                                                                                                        |
| **`SeBackup`**             | **Threat**  | _**Built-in commands**_ | Read sensitve files with `robocopy /b`                                                                                                                                                                                                                                                                                                             | <p>- यदि आप %WINDIR%\MEMORY.DMP पढ़ सकते हैं तो यह अधिक दिलचस्प हो सकता है<br><br>- <code>SeBackupPrivilege</code> (और robocopy) खुली फ़ाइलों के मामले में सहायक नहीं है।<br><br>- Robocopy को /b पैरामीटर के साथ काम करने के लिए SeBackup और SeRestore दोनों की आवश्यकता होती है।</p>                                                                      |
| **`SeCreateToken`**        | _**Admin**_ | 3rd party tool          | Create arbitrary token including local admin rights with `NtCreateToken`.                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**              | _**Admin**_ | **PowerShell**          | Duplicate the `lsass.exe` token.                                                                                                                                                                                                                                                                                                                   | Script to be found at [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1)                                                                                                                                                                                                         |
| **`SeLoadDriver`**         | _**Admin**_ | 3rd party tool          | <p>1. Load buggy kernel driver such as <code>szkg64.sys</code><br>2. Exploit the driver vulnerability<br><br>Alternatively, the privilege may be used to unload security-related drivers with <code>ftlMC</code> builtin command. i.e.: <code>fltMC sysmondrv</code></p>                                                                           | <p>1. The <code>szkg64</code> vulnerability is listed as <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a><br>2. The <code>szkg64</code> <a href="https://www.greyhathacker.net/?p=1025">exploit code</a> was created by <a href="https://twitter.com/parvezghh">Parvez Anwar</a></p> |
| **`SeRestore`**            | _**Admin**_ | **PowerShell**          | <p>1. Launch PowerShell/ISE with the SeRestore privilege present.<br>2. Enable the privilege with <a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a>).<br>3. Rename utilman.exe to utilman.old<br>4. Rename cmd.exe to utilman.exe<br>5. Lock the console and press Win+U</p> | <p>Attack may be detected by some AV software.</p><p>Alternative method relies on replacing service binaries stored in "Program Files" using the same privilege</p>                                                                                                                                                            |
| **`SeTakeOwnership`**      | _**Admin**_ | _**Built-in commands**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. Rename cmd.exe to utilman.exe<br>4. Lock the console and press Win+U</p>                                                                                                                                       | <p>Attack may be detected by some AV software.</p><p>Alternative method relies on replacing service binaries stored in "Program Files" using the same privilege.</p>                                                                                                                                                           |
| **`SeTcb`**                | _**Admin**_ | 3rd party tool          | <p>Manipulate tokens to have local admin rights included. May require SeImpersonate.</p><p>To be verified.</p>                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                |

## Reference

* Take a look to this table defining Windows tokens: [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* Take a look to [**this paper**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) about privesc with tokens.

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
