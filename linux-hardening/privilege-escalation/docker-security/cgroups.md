# CGroups

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

**Linux Control Groups**, au **cgroups**, ni kipengele cha kernel ya Linux kinachoruhusu ugawaji, mipaka, na kipaumbele cha rasilimali za mfumo kama CPU, kumbukumbu, na disk I/O kati ya vikundi vya michakato. Wanatoa mekanizma ya **kusimamia na kutenga matumizi ya rasilimali** za makundi ya michakato, ambayo ni muhimu kwa madhumuni kama vile mipaka ya rasilimali, kutengwa kwa mzigo, na kipaumbele cha rasilimali kati ya vikundi tofauti vya michakato.

Kuna **matoleo mawili ya cgroups**: toleo la 1 na toleo la 2. Zote zinaweza kutumika kwa pamoja kwenye mfumo. Tofauti kuu ni kwamba **cgroups toleo la 2** linaanzisha **muundo wa hierarchal, kama mti**, unaowezesha usambazaji wa rasilimali kwa undani zaidi kati ya vikundi vya michakato. Zaidi ya hayo, toleo la 2 linakuja na maboresho mbalimbali, ikiwa ni pamoja na:

Mbali na shirika jipya la hierarchal, cgroups toleo la 2 pia limeanzisha **mabadiliko na maboresho mengine kadhaa**, kama vile msaada wa **wasimamizi wapya wa rasilimali**, msaada bora kwa programu za zamani, na utendaji bora.

Kwa ujumla, cgroups **toleo la 2 linatoa vipengele vingi na utendaji bora** kuliko toleo la 1, lakini la mwisho linaweza bado kutumika katika hali fulani ambapo ulinganifu na mifumo ya zamani ni wasiwasi.

Unaweza kuorodhesha cgroups za v1 na v2 kwa ajili ya mchakato wowote kwa kuangalia faili yake ya cgroup katika /proc/\<pid>. Unaweza kuanza kwa kuangalia cgroups za shell yako kwa amri hii:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
The output structure is as follows:

* **Numbers 2–12**: cgroups v1, with each line representing a different cgroup. Controllers for these are specified adjacent to the number.
* **Number 1**: Pia cgroups v1, lakini kwa madhumuni ya usimamizi pekee (iliyowekwa na, e.g., systemd), na haina kiongozi.
* **Number 0**: Inawakilisha cgroups v2. Hakuna viongozi waliotajwa, na mstari huu ni wa kipekee kwenye mifumo inayotumia cgroups v2 pekee.
* The **names are hierarchical**, resembling file paths, indicating the structure and relationship between different cgroups.
* **Names like /user.slice or /system.slice** specify the categorization of cgroups, with user.slice typically for login sessions managed by systemd and system.slice for system services.

### Viewing cgroups

The filesystem is typically utilized for accessing **cgroups**, diverging from the Unix system call interface traditionally used for kernel interactions. To investigate a shell's cgroup configuration, one should examine the **/proc/self/cgroup** file, which reveals the shell's cgroup. Then, by navigating to the **/sys/fs/cgroup** (or **`/sys/fs/cgroup/unified`**) directory and locating a directory that shares the cgroup's name, one can observe various settings and resource usage information pertinent to the cgroup.

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

The key interface files for cgroups are prefixed with **cgroup**. The **cgroup.procs** file, which can be viewed with standard commands like cat, lists the processes within the cgroup. Another file, **cgroup.threads**, includes thread information.

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Cgroups managing shells typically encompass two controllers that regulate memory usage and process count. To interact with a controller, files bearing the controller's prefix should be consulted. For instance, **pids.current** would be referenced to ascertain the count of threads in the cgroup.

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

The indication of **max** in a value suggests the absence of a specific limit for the cgroup. However, due to the hierarchical nature of cgroups, limits might be imposed by a cgroup at a lower level in the directory hierarchy.

### Manipulating and Creating cgroups

Processes are assigned to cgroups by **writing their Process ID (PID) to the `cgroup.procs` file**. This requires root privileges. For instance, to add a process:
```bash
echo [pid] > cgroup.procs
```
Vivyo hivyo, **kubadilisha sifa za cgroup, kama kuweka kikomo cha PID**, hufanywa kwa kuandika thamani inayotakiwa kwenye faili husika. Ili kuweka kiwango cha juu cha PIDs 3,000 kwa cgroup:
```bash
echo 3000 > pids.max
```
**Kuunda cgroups mpya** kunahusisha kuunda subdirectory mpya ndani ya hiyerararkia ya cgroup, ambayo inasababisha kernel kuunda kiotomatiki faili za interface zinazohitajika. Ingawa cgroups bila michakato haiwezi kuondolewa kwa `rmdir`, kuwa makini na vizuizi fulani:

* **Michakato inaweza kuwekwa tu katika cgroups za majani** (yaani, zile zilizozungukwa zaidi katika hiyerararkia).
* **Cgroup haiwezi kuwa na kiongozi asiye katika mzazi wake**.
* **Viongozi wa cgroups za watoto lazima watangazwe wazi** katika faili ya `cgroup.subtree_control`. Kwa mfano, ili kuwezesha viongozi wa CPU na PID katika cgroup ya mtoto:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
The **root cgroup** ni kivyatu katika sheria hizi, ikiruhusu kuweka mchakato moja kwa moja. Hii inaweza kutumika kuondoa michakato kutoka usimamizi wa systemd.

**Kufuatilia matumizi ya CPU** ndani ya cgroup inawezekana kupitia faili ya `cpu.stat`, inayoonyesha jumla ya muda wa CPU ulio tumika, muhimu kwa kufuatilia matumizi kati ya michakato ya huduma:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>Takwimu za matumizi ya CPU kama zinavyoonyeshwa katika faili ya cpu.stat</p></figcaption></figure>

## References

* **Kitabu: Jinsi Linux Inavyofanya Kazi, Toleo la 3: Kila Mtumiaji wa Juu Anapaswa Kujua Na Brian Ward**

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
