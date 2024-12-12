# Windows Credentials Protections

## Credentials Protections

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

## WDigest

[WDigest](https://technet.microsoft.com/pt-pt/library/cc778868\(v=ws.10\).aspx?f=255\&MSPPError=-2147217396) प्रोटोकॉल, जो Windows XP के साथ पेश किया गया था, HTTP प्रोटोकॉल के माध्यम से प्रमाणीकरण के लिए डिज़ाइन किया गया है और **Windows XP से Windows 8.0 और Windows Server 2003 से Windows Server 2012 तक डिफ़ॉल्ट रूप से सक्षम है**। यह डिफ़ॉल्ट सेटिंग **LSASS (स्थानीय सुरक्षा प्राधिकरण उपप्रणाली सेवा) में स्पष्ट पाठ पासवर्ड भंडारण** का परिणाम देती है। एक हमलावर Mimikatz का उपयोग करके **इन क्रेडेंशियल्स को निकाल सकता है**:
```bash
sekurlsa::wdigest
```
इस फ़ीचर को **बंद या चालू करने के लिए**, _**UseLogonCredential**_ और _**Negotiate**_ रजिस्ट्री कुंजी _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_ के भीतर "1" पर सेट की जानी चाहिए। यदि ये कुंजी **गायब हैं या "0" पर सेट हैं**, तो WDigest **अक्षम** है:
```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
## LSA सुरक्षा

**Windows 8.1** से शुरू होकर, Microsoft ने LSA की सुरक्षा को **अविश्वसनीय प्रक्रियाओं द्वारा अनधिकृत मेमोरी पढ़ने या कोड इंजेक्शन को रोकने** के लिए बढ़ाया। यह सुधार `mimikatz.exe sekurlsa:logonpasswords` जैसे आदेशों के सामान्य कार्य को बाधित करता है। इस **सुधारित सुरक्षा** को **सक्षम** करने के लिए, _**RunAsPPL**_ मान को _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ में 1 पर समायोजित किया जाना चाहिए:
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### Bypass

इस सुरक्षा को Mimikatz ड्राइवर mimidrv.sys का उपयोग करके बायपास करना संभव है:

![](../../.gitbook/assets/mimidrv.png)

## Credential Guard

**Credential Guard**, **Windows 10 (Enterprise और Education editions)** के लिए विशेष एक फीचर, मशीन क्रेडेंशियल्स की सुरक्षा को **Virtual Secure Mode (VSM)** और **Virtualization Based Security (VBS)** का उपयोग करके बढ़ाता है। यह CPU वर्चुअलाइजेशन एक्सटेंशन का लाभ उठाकर महत्वपूर्ण प्रक्रियाओं को एक सुरक्षित मेमोरी स्पेस में अलग करता है, जो मुख्य ऑपरेटिंग सिस्टम की पहुंच से दूर है। यह अलगाव सुनिश्चित करता है कि यहां तक कि कर्नेल भी VSM में मेमोरी तक पहुंच नहीं सकता, प्रभावी रूप से क्रेडेंशियल्स को **pass-the-hash** जैसे हमलों से सुरक्षित रखता है। **Local Security Authority (LSA)** इस सुरक्षित वातावरण में एक ट्रस्टलेट के रूप में कार्य करता है, जबकि मुख्य OS में **LSASS** प्रक्रिया केवल VSM के LSA के साथ संवाद करने के रूप में कार्य करती है।

डिफ़ॉल्ट रूप से, **Credential Guard** सक्रिय नहीं है और इसे एक संगठन के भीतर मैन्युअल रूप से सक्रिय करने की आवश्यकता है। यह **Mimikatz** जैसे उपकरणों के खिलाफ सुरक्षा बढ़ाने के लिए महत्वपूर्ण है, जो क्रेडेंशियल्स को निकालने की अपनी क्षमता में बाधित होते हैं। हालाँकि, कस्टम **Security Support Providers (SSP)** को जोड़कर कमजोरियों का लाभ उठाया जा सकता है ताकि लॉगिन प्रयासों के दौरान क्रेडेंशियल्स को स्पष्ट पाठ में कैप्चर किया जा सके।

**Credential Guard** की सक्रियण स्थिति की पुष्टि करने के लिए, _**HKLM\System\CurrentControlSet\Control\LSA**_ के तहत रजिस्ट्री कुंजी _**LsaCfgFlags**_ की जांच की जा सकती है। "**1**" का मान **UEFI लॉक** के साथ सक्रियण को दर्शाता है, "**2**" बिना लॉक के, और "**0**" यह दर्शाता है कि यह सक्षम नहीं है। यह रजिस्ट्री जांच, जबकि एक मजबूत संकेतक है, Credential Guard को सक्षम करने के लिए एकमात्र कदम नहीं है। इस फीचर को सक्षम करने के लिए विस्तृत मार्गदर्शन और एक PowerShell स्क्रिप्ट ऑनलाइन उपलब्ध है।
```powershell
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
For a comprehensive understanding and instructions on enabling **Credential Guard** in Windows 10 and its automatic activation in compatible systems of **Windows 11 Enterprise and Education (version 22H2)**, visit [Microsoft's documentation](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage).

Further details on implementing custom SSPs for credential capture are provided in [this guide](../active-directory-methodology/custom-ssp.md).

## RDP RestrictedAdmin Mode

**Windows 8.1 और Windows Server 2012 R2** ने कई नए सुरक्षा सुविधाएँ पेश की, जिसमें _**RDP के लिए Restricted Admin मोड**_ शामिल है। इस मोड को [**pass the hash**](https://blog.ahasayen.com/pass-the-hash/) हमलों से जुड़े जोखिमों को कम करने के लिए सुरक्षा बढ़ाने के लिए डिज़ाइन किया गया था।

परंपरागत रूप से, जब आप RDP के माध्यम से एक दूरस्थ कंप्यूटर से कनेक्ट करते हैं, तो आपके क्रेडेंशियल्स लक्ष्य मशीन पर संग्रहीत होते हैं। यह एक महत्वपूर्ण सुरक्षा जोखिम प्रस्तुत करता है, विशेष रूप से उन खातों का उपयोग करते समय जिनके पास उच्चाधिकार होते हैं। हालाँकि, _**Restricted Admin मोड**_ के परिचय के साथ, यह जोखिम काफी हद तक कम हो गया है।

जब आप **mstsc.exe /RestrictedAdmin** कमांड का उपयोग करके RDP कनेक्शन शुरू करते हैं, तो दूरस्थ कंप्यूटर पर आपके क्रेडेंशियल्स को संग्रहीत किए बिना प्रमाणीकरण किया जाता है। यह दृष्टिकोण सुनिश्चित करता है कि, यदि किसी मैलवेयर संक्रमण या यदि एक दुर्भावनापूर्ण उपयोगकर्ता दूरस्थ सर्वर तक पहुँच प्राप्त करता है, तो आपके क्रेडेंशियल्स से समझौता नहीं किया जाएगा, क्योंकि वे सर्वर पर संग्रहीत नहीं होते हैं।

यह ध्यान रखना महत्वपूर्ण है कि **Restricted Admin मोड** में, RDP सत्र से नेटवर्क संसाधनों तक पहुँचने के प्रयास आपके व्यक्तिगत क्रेडेंशियल्स का उपयोग नहीं करेंगे; इसके बजाय, **मशीन की पहचान** का उपयोग किया जाता है।

यह सुविधा दूरस्थ डेस्कटॉप कनेक्शनों को सुरक्षित करने और सुरक्षा उल्लंघन की स्थिति में संवेदनशील जानकारी को उजागर होने से बचाने में एक महत्वपूर्ण कदम है।

![](../../.gitbook/assets/RAM.png)

For more detailed information on visit [this resource](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/).

## Cached Credentials

Windows **डोमेन क्रेडेंशियल्स** को **स्थानीय सुरक्षा प्राधिकरण (LSA)** के माध्यम से सुरक्षित करता है, जो **Kerberos** और **NTLM** जैसे सुरक्षा प्रोटोकॉल के साथ लॉगिन प्रक्रियाओं का समर्थन करता है। Windows की एक प्रमुख विशेषता यह है कि यह **अंतिम दस डोमेन लॉगिन्स** को कैश करने की क्षमता रखता है ताकि उपयोगकर्ता अपने कंप्यूटरों तक पहुँच प्राप्त कर सकें, भले ही **डोमेन नियंत्रक ऑफ़लाइन** हो—यह उन लैपटॉप उपयोगकर्ताओं के लिए एक वरदान है जो अक्सर अपनी कंपनी के नेटवर्क से दूर होते हैं।

कैश किए गए लॉगिन की संख्या को एक विशिष्ट **रजिस्ट्री कुंजी या समूह नीति** के माध्यम से समायोजित किया जा सकता है। इस सेटिंग को देखने या बदलने के लिए, निम्नलिखित कमांड का उपयोग किया जाता है:
```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
Access to these cached credentials is tightly controlled, with only the **SYSTEM** account having the necessary permissions to view them. Administrators needing to access this information must do so with SYSTEM user privileges. The credentials are stored at: `HKEY_LOCAL_MACHINE\SECURITY\Cache`

**Mimikatz** can be employed to extract these cached credentials using the command `lsadump::cache`.

For further details, the original [source](http://juggernaut.wikidot.com/cached-credentials) provides comprehensive information.

## Protected Users

**Protected Users group** में सदस्यता उपयोगकर्ताओं के लिए कई सुरक्षा सुधार लाती है, जो क्रेडेंशियल चोरी और दुरुपयोग के खिलाफ उच्च स्तर की सुरक्षा सुनिश्चित करती है:

* **Credential Delegation (CredSSP)**: भले ही **Allow delegating default credentials** के लिए Group Policy सेटिंग सक्षम हो, Protected Users के स्पष्ट पाठ क्रेडेंशियल्स को कैश नहीं किया जाएगा।
* **Windows Digest**: **Windows 8.1 और Windows Server 2012 R2** से शुरू होकर, सिस्टम Protected Users के स्पष्ट पाठ क्रेडेंशियल्स को कैश नहीं करेगा, चाहे Windows Digest स्थिति कुछ भी हो।
* **NTLM**: सिस्टम Protected Users के स्पष्ट पाठ क्रेडेंशियल्स या NT एक-तरफा कार्यों (NTOWF) को कैश नहीं करेगा।
* **Kerberos**: Protected Users के लिए, Kerberos प्रमाणीकरण **DES** या **RC4 keys** उत्पन्न नहीं करेगा, न ही यह स्पष्ट पाठ क्रेडेंशियल्स या प्रारंभिक Ticket-Granting Ticket (TGT) अधिग्रहण के बाद दीर्घकालिक कुंजियों को कैश करेगा।
* **Offline Sign-In**: Protected Users के लिए साइन-इन या अनलॉक पर कोई कैश किया गया वेरिफायर नहीं बनाया जाएगा, जिसका अर्थ है कि इन खातों के लिए ऑफ़लाइन साइन-इन समर्थित नहीं है।

ये सुरक्षा उपाय तब सक्रिय होते हैं जब **Protected Users group** का सदस्य डिवाइस में साइन इन करता है। यह विभिन्न क्रेडेंशियल समझौता विधियों के खिलाफ सुरक्षा सुनिश्चित करने के लिए महत्वपूर्ण सुरक्षा उपायों को लागू करता है।

अधिक विस्तृत जानकारी के लिए, आधिकारिक [documentation](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group) देखें।

**Table from** [**the docs**](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory)**.**

| Windows Server 2003 RTM | Windows Server 2003 SP1+ | <p>Windows Server 2012,<br>Windows Server 2008 R2,<br>Windows Server 2008</p> | Windows Server 2016          |
| ----------------------- | ------------------------ | ----------------------------------------------------------------------------- | ---------------------------- |
| Account Operators       | Account Operators        | Account Operators                                                             | Account Operators            |
| Administrator           | Administrator            | Administrator                                                                 | Administrator                |
| Administrators          | Administrators           | Administrators                                                                | Administrators               |
| Backup Operators        | Backup Operators         | Backup Operators                                                              | Backup Operators             |
| Cert Publishers         |                          |                                                                               |                              |
| Domain Admins           | Domain Admins            | Domain Admins                                                                 | Domain Admins                |
| Domain Controllers      | Domain Controllers       | Domain Controllers                                                            | Domain Controllers           |
| Enterprise Admins       | Enterprise Admins        | Enterprise Admins                                                             | Enterprise Admins            |
|                         |                          |                                                                               | Enterprise Key Admins        |
|                         |                          |                                                                               | Key Admins                   |
| Krbtgt                  | Krbtgt                   | Krbtgt                                                                        | Krbtgt                       |
| Print Operators         | Print Operators          | Print Operators                                                               | Print Operators              |
|                         |                          | Read-only Domain Controllers                                                  | Read-only Domain Controllers |
| Replicator              | Replicator               | Replicator                                                                    | Replicator                   |
| Schema Admins           | Schema Admins            | Schema Admins                                                                 | Schema Admins                |
| Server Operators        | Server Operators         | Server Operators                                                              | Server Operators             |

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
