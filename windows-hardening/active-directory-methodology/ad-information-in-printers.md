<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>!</strong></a> <strong>qaStaHvIS</strong> <strong>AWS hacking</strong> <strong>ghItlhvam</strong></summary>

<strong>HackTricks</strong> <strong>poH</strong> <strong>support</strong> <strong>ways</strong>:

* <strong>HackTricks</strong> <strong>advertised</strong> <strong>company</strong> <strong>see</strong> <strong>want</strong> <strong>**SUBSCRIPTION PLANS**</strong> <strong>[**Check**](https://github.com/sponsors/carlospolop)</strong> <strong>**HackTricks**</strong> <strong>**download**</strong> <strong>**PDF**</strong>
* <strong>PEASS</strong> <strong>&amp;</strong> <strong>HackTricks</strong> <strong>swag</strong> <strong>**official**</strong> <strong>[**Get**](https://peass.creator-spring.com)</strong>
* <strong>PEASS</strong> <strong>Family</strong> <strong>**The**</strong> <strong>[**Discover**](https://opensea.io/collection/the-peass-family)</strong> <strong>NFTs</strong> <strong>**exclusive**</strong> <strong>**collection**</strong> <strong>**our**</strong>
* <strong>**Join**</strong> 💬 <strong>**group**</strong> <strong>**Discord**</strong> <strong>[**The**](https://discord.gg/hRep4RUj7f)</strong> <strong>**group**</strong> <strong>**telegram**</strong> <strong>[**or**](https://t.me/peass)</strong> <strong>**us**</strong> <strong>**follow**</strong> 🐦 <strong>**Twitter**</strong> [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* <strong>**hacking**</strong> <strong>**tricks**</strong> <strong>**Share**</strong> <strong>**submitting**</strong> <strong>**by**</strong> <strong>**tricks**</strong> <strong>**hacking**</strong> <strong>**your**</strong> <strong>**Share**</strong> <strong>**repos**</strong> <strong>**github**</strong> <strong>**and**</strong> <strong>**HackTricks**</strong> <strong>**Cloud**</strong> <strong>**HackTricks**</strong> [**PRs**](https://github.com/carlospolop/hacktricks) <strong>**the**</strong> <strong>**to**</strong> <strong>**by**</strong> <strong>**submitting**</strong> <strong>**tricks**</strong> <strong>**hacking**</strong> <strong>**your**</strong> <strong>**Share**</strong> <strong>**github**</strong> <strong>**and**</strong> <strong>**HackTricks**</strong> <strong>**Cloud**</strong> <strong>**HackTricks**</strong>

</details>


There are several blogs in the Internet which **highlight the dangers of leaving printers configured with LDAP with default/weak** logon credentials.\
This is because an attacker could **trick the printer to authenticate against a rouge LDAP server** (typically a `nc -vv -l -p 444` is enough) and to capture the printer **credentials on clear-text**.

Also, several printers will contains **logs with usernames** or could even be able to **download all usernames** from the Domain Controller.

All this **sensitive information** and the common **lack of security** makes printers very interesting for attackers.

Some blogs about the topic:

* [https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/](https://www.ceos3c.com/hacking/obtaining-domain-credentials-printer-netcat/)
* [https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856](https://medium.com/@nickvangilder/exploiting-multifunction-printers-during-a-penetration-test-engagement-28d3840d8856)

## Printer Configuration
- **Location**: The LDAP server list is found at: `Network > LDAP Setting > Setting Up LDAP`.
- **Behavior**: The interface allows LDAP server modifications without re-entering credentials, aiming for user convenience but posing security risks.
- **Exploit**: The exploit involves redirecting the LDAP server address to a controlled machine and leveraging the "Test Connection" feature to capture credentials.

## Capturing Credentials

**For more detailed steps, refer to the original [source](https://grimhacker.com/2018/03/09/just-a-printer/).**

### Method 1: Netcat Listener
A simple netcat listener might suffice:
```bash
sudo nc -k -v -l -p 386
```
### Method 2: Full LDAP Server with Slapd
A more reliable approach involves setting up a full LDAP server because the printer performs a null bind followed by a query before attempting credential binding.

1. **LDAP Server Setup**: The guide follows steps from [this source](https://www.server-world.info/en/note?os=Fedora_26&p=openldap).
2. **Key Steps**:
- Install OpenLDAP.
- Configure admin password.
- Import basic schemas.
- Set domain name on LDAP DB.
- Configure LDAP TLS.
3. **LDAP Service Execution**: Once set up, the LDAP service can be run using:

### Method 2: Slapd jIHDaq LDAP Server
ghItlhlaHbe'chugh, 'ej printer null bind followed by a query before attempting credential binding. 

1. **LDAP Server Setup**: [This source](https://www.server-world.info/en/note?os=Fedora_26&p=openldap) laH follow steps.
2. **Key Steps**:
- OpenLDAP qay'be'.
- admin password configure.
- basic schemas Import.
- LDAP DB domain name set.
- LDAP TLS configure.
3. **LDAP Service Execution**: ghItlhlaHbe'chugh, LDAP service run can be.
```bash
slapd -d 2
```
## References
* [https://grimhacker.com/2018/03/09/just-a-printer/](https://grimhacker.com/2018/03/09/just-a-printer/)


<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>!</strong></a> <strong>qaStaHvIS AWS hacking vItlhutlh</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
