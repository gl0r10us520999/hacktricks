# Orodha ya Ukaguzi - Kuinua Mamlaka ya Windows ya Mitaa

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za udukuzi kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}

### **Zana bora ya kutafuta njia za kuinua mamlaka ya Windows ya ndani:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [Taarifa ya Mfumo](windows-local-privilege-escalation/#system-info)

* [ ] Pata [**taarifa za mfumo**](windows-local-privilege-escalation/#system-info)
* [ ] Tafuta **kernel** [**exploits kwa kutumia scripts**](windows-local-privilege-escalation/#version-exploits)
* [ ] Tumia **Google kutafuta** **exploits** za kernel
* [ ] Tumia **searchsploit kutafuta** **exploits** za kernel
* [ ] Taarifa ya kuvutia katika [**env vars**](windows-local-privilege-escalation/#environment)?
* [ ] Nywila katika [**histori ya PowerShell**](windows-local-privilege-escalation/#powershell-history)?
* [ ] Taarifa ya kuvutia katika [**mipangilio ya Internet**](windows-local-privilege-escalation/#internet-settings)?
* [ ] [**Diski**](windows-local-privilege-escalation/#drives)?
* [ ] [**WSUS exploit**](windows-local-privilege-escalation/#wsus)?
* [ ] [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)?

### [Kuhesabu/AV kuorodhesha](windows-local-privilege-escalation/#enumeration)

* [ ] Angalia [**ukaguzi**](windows-local-privilege-escalation/#audit-settings) na [**WEF**](windows-local-privilege-escalation/#wef) mipangilio
* [ ] Angalia [**LAPS**](windows-local-privilege-escalation/#laps)
* [ ] Angalia kama [**WDigest**](windows-local-privilege-escalation/#wdigest) inafanya kazi
* [ ] [**Ulinzi wa LSA**](windows-local-privilege-escalation/#lsa-protection)?
* [ ] [**Credentials Guard**](windows-local-privilege-escalation/#credentials-guard)[?](windows-local-privilege-escalation/#cached-credentials)
* [ ] [**Cached Credentials**](windows-local-privilege-escalation/#cached-credentials)?
* [ ] Angalia kama kuna [**AV**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/windows-av-bypass/README.md)
* [ ] [**Sera ya AppLocker**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/README.md#applocker-policy)?
* [ ] [**UAC**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control/README.md)
* [ ] [**Mamlaka ya Mtumiaji**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Angalia [**mamlaka za mtumiaji wa sasa**](windows-local-privilege-escalation/#users-and-groups)
* [ ] Je, wewe ni [**mwanachama wa kikundi chochote chenye mamlaka**](windows-local-privilege-escalation/#privileged-groups)?
* [ ] Angalia kama una [miongoni mwa hizi tokens zilizowekwa](windows-local-privilege-escalation/#token-manipulation): **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege** ?
* [ ] [**Sessions za Watumiaji**](windows-local-privilege-escalation/#logged-users-sessions)?
* [ ] Angalia [**nyumba za watumiaji**](windows-local-privilege-escalation/#home-folders) (ufikiaji?)
* [ ] Angalia [**Sera ya Nywila**](windows-local-privilege-escalation/#password-policy)
* [ ] Nini kiko [**ndani ya Clipboard**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)?

### [Mtandao](windows-local-privilege-escalation/#network)

* [ ] Angalia **taarifa za sasa za** [**mtandao**](windows-local-privilege-escalation/#network)
* [ ] Angalia **huduma za ndani zilizofichwa** zilizozuiliwa kwa nje

### [Mchakato unaoendelea](windows-local-privilege-escalation/#running-processes)

* [ ] Mchakato wa binaries [**file na ruhusa za folda**](windows-local-privilege-escalation/#file-and-folder-permissions)
* [ ] [**Kuchimba nywila za Kumbukumbu**](windows-local-privilege-escalation/#memory-password-mining)
* [ ] [**Programu za GUI zisizo salama**](windows-local-privilege-escalation/#insecure-gui-apps)
* [ ] Pora nywila na **michakato ya kuvutia** kupitia `ProcDump.exe` ? (firefox, chrome, nk ...)

### [Huduma](windows-local-privilege-escalation/#services)

* [ ] [Je, unaweza **kubadilisha huduma yoyote**?](windows-local-privilege-escalation/#permissions)
* [ ] [Je, unaweza **kubadilisha** **binary** inayotekelezwa na **huduma yoyote**?](windows-local-privilege-escalation/#modify-service-binary-path)
* [ ] [Je, unaweza **kubadilisha** **registry** ya **huduma yoyote**?](windows-local-privilege-escalation/#services-registry-modify-permissions)
* [ ] [Je, unaweza kunufaika na **path** ya **binary** ya **huduma isiyo na nukuu**?](windows-local-privilege-escalation/#unquoted-service-paths)

### [**Programu**](windows-local-privilege-escalation/#applications)

* [ ] **Andika** [**ruhusa kwenye programu zilizowekwa**](windows-local-privilege-escalation/#write-permissions)
* [ ] [**Programu za Kuanzisha**](windows-local-privilege-escalation/#run-at-startup)
* [ ] **Zana** [**zilizo hatarini**](windows-local-privilege-escalation/#drivers)

### [DLL Hijacking](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] Je, unaweza **kuandika katika folda yoyote ndani ya PATH**?
* [ ] Je, kuna binary ya huduma inayojulikana ambayo **inajaribu kupakia DLL isiyokuwepo**?
* [ ] Je, unaweza **kuandika** katika **folda za binaries**?

### [Mtandao](windows-local-privilege-escalation/#network)

* [ ] Orodhesha mtandao (shiriki, interfaces, njia, majirani, ...)
* [ ] Angalia kwa makini huduma za mtandao zinazokisikiliza kwenye localhost (127.0.0.1)

### [Nywila za Windows](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Winlogon**](windows-local-privilege-escalation/#winlogon-credentials) nywila
* [ ] [**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault) nywila ambazo unaweza kutumia?
* [ ] Taarifa ya kuvutia [**DPAPI credentials**](windows-local-privilege-escalation/#dpapi)?
* [ ] Nywila za mitandao ya [**Wifi zilizohifadhiwa**](windows-local-privilege-escalation/#wifi)?
* [ ] Taarifa ya kuvutia katika [**Mawasiliano ya RDP yaliyohifadhiwa**](windows-local-privilege-escalation/#saved-rdp-connections)?
* [ ] Nywila katika [**amri zilizokimbizwa hivi karibuni**](windows-local-privilege-escalation/#recently-run-commands)?
* [ ] [**Meneja wa Nywila za Desktop ya KijRemote**](windows-local-privilege-escalation/#remote-desktop-credential-manager) nywila?
* [ ] [**AppCmd.exe** ipo](windows-local-privilege-escalation/#appcmd-exe)? Nywila?
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)? DLL Side Loading?

### [Faili na Registry (Nywila)](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**Creds**](windows-local-privilege-escalation/#putty-creds) **na** [**funguo za mwenyeji wa SSH**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] [**Funguo za SSH katika registry**](windows-local-privilege-escalation/#ssh-keys-in-registry)?
* [ ] Nywila katika [**faili zisizofuatiliwa**](windows-local-privilege-escalation/#unattended-files)?
* [ ] Backup yoyote ya [**SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups)?
* [ ] [**Nywila za Cloud**](windows-local-privilege-escalation/#cloud-credentials)?
* [ ] Faili ya [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml)?
* [ ] [**Cached GPP Password**](windows-local-privilege-escalation/#cached-gpp-pasword)?
* [ ] Nywila katika [**faili ya usanidi wa IIS Web**](windows-local-privilege-escalation/#iis-web-config)?
* [ ] Taarifa ya kuvutia katika [**maktaba** **za**](windows-local-privilege-escalation/#logs)?
* [ ] Je, unataka [**kuomba nywila**](windows-local-privilege-escalation/#ask-for-credentials) kwa mtumiaji?
* [ ] Taarifa ya kuvutia [**katika Kijalala**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)?
* [ ] Registry nyingine inayohifadhi nywila [**zimehifadhiwa**](windows-local-privilege-escalation/#inside-the-registry)?
* [ ] Ndani ya [**data za kivinjari**](windows-local-privilege-escalation/#browsers-history) (dbs, historia, alama, ...)?
* [ ] [**Utafutaji wa nywila wa jumla**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry) katika faili na registry
* [ ] [**Zana**](windows-local-privilege-escalation/#tools-that-search-for-passwords) za kutafuta nywila kiotomatiki

### [Wakandarasi Waliovuja](windows-local-privilege-escalation/#leaked-handlers)

* [ ] Je, una ufikiaji wa handler yoyote ya mchakato unaoendeshwa na msimamizi?

### [Kujifanya kwa Mteja wa Pipe](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] Angalia kama unaweza kuitumia vibaya

{% hint style="success" %}
Jifunze na fanya mazoezi ya AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Jifunze na fanya mazoezi ya GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Angalia [**mpango wa usajili**](https://github.com/sponsors/carlospolop)!
* **Jiunge na** 💬 [**kikundi cha Discord**](https://discord.gg/hRep4RUj7f) au [**kikundi cha telegram**](https://t.me/peass) au **tufuatilie** kwenye **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Shiriki mbinu za udukuzi kwa kuwasilisha PRs kwa** [**HackTricks**](https://github.com/carlospolop/hacktricks) na [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos za github.

</details>
{% endhint %}
