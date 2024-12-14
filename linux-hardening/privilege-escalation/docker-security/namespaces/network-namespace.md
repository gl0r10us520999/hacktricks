# Network Namespace

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

## Basic Information

A network namespace is a Linux kernel feature that provides isolation of the network stack, allowing **kila network namespace kuwa na usanidi wake wa mtandao huru**, interfaces, IP addresses, routing tables, and firewall rules. This isolation is useful in various scenarios, such as containerization, where each container should have its own network configuration, independent of other containers and the host system.

### How it works:

1. When a new network namespace is created, it starts with a **completely isolated network stack**, with **no network interfaces** except for the loopback interface (lo). This means that processes running in the new network namespace cannot communicate with processes in other namespaces or the host system by default.
2. **Virtual network interfaces**, such as veth pairs, can be created and moved between network namespaces. This allows for establishing network connectivity between namespaces or between a namespace and the host system. For example, one end of a veth pair can be placed in a container's network namespace, and the other end can be connected to a **bridge** or another network interface in the host namespace, providing network connectivity to the container.
3. Network interfaces within a namespace can have their **own IP addresses, routing tables, and firewall rules**, independent of other namespaces. This allows processes in different network namespaces to have different network configurations and operate as if they are running on separate networked systems.
4. Processes can move between namespaces using the `setns()` system call, or create new namespaces using the `unshare()` or `clone()` system calls with the `CLONE_NEWNET` flag. When a process moves to a new namespace or creates one, it will start using the network configuration and interfaces associated with that namespace.

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```
Kwa kuunganisha mfano mpya wa mfumo wa `/proc` ikiwa unatumia param `--mount-proc`, unahakikisha kwamba nafasi mpya ya kuunganisha ina **mtazamo sahihi na uliojitegemea wa taarifa za mchakato maalum kwa nafasi hiyo**.

<details>

<summary>Kosa: bash: fork: Haiwezekani kugawa kumbukumbu</summary>

Wakati `unshare` inatekelezwa bila chaguo la `-f`, kosa linakutana kutokana na jinsi Linux inavyoshughulikia nafasi mpya za PID (Kitambulisho cha Mchakato). Maelezo muhimu na suluhisho yameelezwa hapa chini:

1. **Maelezo ya Tatizo**:
- Kernel ya Linux inaruhusu mchakato kuunda nafasi mpya kwa kutumia wito wa mfumo wa `unshare`. Hata hivyo, mchakato unaoanzisha uundaji wa nafasi mpya ya PID (inayojulikana kama mchakato wa "unshare") hauingii kwenye nafasi mpya; ni mchakato zake za watoto pekee ndizo zinaingia.
- Kuendesha `%unshare -p /bin/bash%` kunaanzisha `/bin/bash` katika mchakato sawa na `unshare`. Kwa hivyo, `/bin/bash` na mchakato zake za watoto ziko katika nafasi ya awali ya PID.
- Mchakato wa kwanza wa mtoto wa `/bin/bash` katika nafasi mpya inakuwa PID 1. Wakati mchakato huu unapoondoka, inasababisha kusafishwa kwa nafasi hiyo ikiwa hakuna mchakato mwingine, kwani PID 1 ina jukumu maalum la kupokea mchakato wa yatima. Kernel ya Linux itazima kisha ugawaji wa PID katika nafasi hiyo.

2. **Matokeo**:
- Kuondoka kwa PID 1 katika nafasi mpya kunasababisha kusafishwa kwa bendera ya `PIDNS_HASH_ADDING`. Hii inasababisha kazi ya `alloc_pid` kushindwa kugawa PID mpya wakati wa kuunda mchakato mpya, ikitoa kosa la "Haiwezekani kugawa kumbukumbu".

3. **Suluhisho**:
- Tatizo linaweza kutatuliwa kwa kutumia chaguo la `-f` pamoja na `unshare`. Chaguo hili linafanya `unshare` kuunda mchakato mpya baada ya kuunda nafasi mpya ya PID.
- Kutekeleza `%unshare -fp /bin/bash%` kunahakikisha kwamba amri ya `unshare` yenyewe inakuwa PID 1 katika nafasi mpya. `/bin/bash` na mchakato zake za watoto kisha zinahifadhiwa salama ndani ya nafasi hii mpya, kuzuia kuondoka mapema kwa PID 1 na kuruhusu ugawaji wa kawaida wa PID.

Kwa kuhakikisha kwamba `unshare` inatekelezwa na bendera ya `-f`, nafasi mpya ya PID inashikiliwa kwa usahihi, ikiruhusu `/bin/bash` na mchakato zake za chini kufanya kazi bila kukutana na kosa la ugawaji wa kumbukumbu.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
# Run ifconfig or ip -a
```
### &#x20;Angalia ni namespace ipi mchakato wako uko ndani yake
```bash
ls -l /proc/self/ns/net
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/net -> 'net:[4026531840]'
```
### Pata majina yote ya Mtandao

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name net -exec readlink {} \; 2>/dev/null | sort -u | grep "net:"
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name net -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### Ingia ndani ya Network namespace
```bash
nsenter -n TARGET_PID --pid /bin/bash
```
Pia, unaweza tu **kuingia katika nafasi nyingine ya mchakato ikiwa wewe ni root**. Na huwezi **kuingia** katika nafasi nyingine **bila desktopa** inayorejelea hiyo (kama `/proc/self/ns/net`).

## Marejeo
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

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
