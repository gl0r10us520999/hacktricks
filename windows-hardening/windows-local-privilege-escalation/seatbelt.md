<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF**, consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sigue**me en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de github** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>




# Comienzo

[Necesitas compilarlo](https://github.com/GhostPack/Seatbelt) o [usar binarios precompilados \(por mí\)](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)
```text
SeatbeltNet3.5x64.exe all
SeatbeltNet3.5x64.exe all full #Without filtering
```
Me gusta mucho el filtrado realizado.

# Verificar

Esta herramienta está más orientada a la recopilación de información que a privesc, pero tiene algunas verificaciones bastante buenas y busca algunas contraseñas.

**SeatBelt.exe system** recopila los siguientes datos del sistema:
```text
BasicOSInfo           -   Basic OS info (i.e. architecture, OS version, etc.)
RebootSchedule        -   Reboot schedule (last 15 days) based on event IDs 12 and 13
TokenGroupPrivs       -   Current process/token privileges (e.g. SeDebugPrivilege/etc.)
UACSystemPolicies     -   UAC system policies via the registry
PowerShellSettings    -   PowerShell versions and security settings
AuditSettings         -   Audit settings via the registry
WEFSettings           -   Windows Event Forwarding (WEF) settings via the registry
LSASettings           -   LSA settings (including auth packages)
UserEnvVariables      -   Current user environment variables
SystemEnvVariables    -   Current system environment variables
UserFolders           -   Folders in C:\Users\
NonstandardServices   -   Services with file info company names that don't contain 'Microsoft'
InternetSettings      -   Internet settings including proxy configs
LapsSettings          -   LAPS settings, if installed
LocalGroupMembers     -   Members of local admins, RDP, and DCOM
MappedDrives          -   Mapped drives
RDPSessions           -   Current incoming RDP sessions
WMIMappedDrives       -   Mapped drives via WMI
NetworkShares         -   Network shares
FirewallRules         -   Deny firewall rules, "full" dumps all
AntiVirusWMI          -   Registered antivirus (via WMI)
InterestingProcesses  -   "Interesting" processes- defensive products and admin tools
RegistryAutoRuns      -   Registry autoruns
RegistryAutoLogon     -   Registry autologon information
DNSCache              -   DNS cache entries (via WMI)
ARPTable              -   Lists the current ARP table and adapter information (equivalent to arp -a)
AllTcpConnections     -   Lists current TCP connections and associated processes
AllUdpConnections     -   Lists current UDP connections and associated processes
NonstandardProcesses  -   Running processeswith file info company names that don't contain 'Microsoft'
*  If the user is in high integrity, the following additional actions are run:
SysmonConfig          -   Sysmon configuration from the registry
```
**SeatBelt.exe user** recopila los siguientes datos del usuario:
```text
SavedRDPConnections   -   Saved RDP connections
TriageIE              -   Internet Explorer bookmarks and history (last 7 days)
DumpVault             -   Dump saved credentials in Windows Vault (i.e. logins from Internet Explorer and Edge), from SharpWeb
RecentRunCommands     -   Recent "run" commands
PuttySessions         -   Interesting settings from any saved Putty configurations
PuttySSHHostKeys      -   Saved putty SSH host keys
CloudCreds            -   AWS/Google/Azure cloud credential files (SharpCloud)
RecentFiles           -   Parsed "recent files" shortcuts (last 7 days)
MasterKeys            -   List DPAPI master keys
CredFiles             -   List Windows credential DPAPI blobs
RDCManFiles           -   List Windows Remote Desktop Connection Manager settings files
*  If the user is in high integrity, this data is collected for ALL users instead of just the current user
```
Opciones de recolección no predeterminadas:
```text
CurrentDomainGroups   -   The current user's local and domain groups
Patches               -   Installed patches via WMI (takes a bit on some systems)
LogonSessions         -   User logon session data
KerberosTGTData       -   ALL TEH TGTZ!
InterestingFiles      -   "Interesting" files matching various patterns in the user's folder
IETabs                -   Open Internet Explorer tabs
TriageChrome          -   Chrome bookmarks and history
TriageFirefox         -   Firefox history (no bookmarks)
RecycleBin            -   Items in the Recycle Bin deleted in the last 30 days - only works from a user context!
4624Events            -   4624 logon events from the security event log
4648Events            -   4648 explicit logon events from the security event log
KerberosTickets       -   List Kerberos tickets. If elevated, grouped by all logon sessions.
```
<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** revisa los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sigue** a **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de github** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
