# IPC Namespace

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
{% endhint %}

## Basic Information

एक IPC (Inter-Process Communication) नामस्थान एक Linux कर्नेल विशेषता है जो **System V IPC वस्तुओं** जैसे संदेश कतारों, साझा मेमोरी खंडों और सेमाफोर का **अलगाव** प्रदान करती है। यह अलगाव सुनिश्चित करता है कि **विभिन्न IPC नामस्थानों में प्रक्रियाएँ एक-दूसरे की IPC वस्तुओं को सीधे एक्सेस या संशोधित नहीं कर सकतीं**, जो प्रक्रिया समूहों के बीच सुरक्षा और गोपनीयता की एक अतिरिक्त परत प्रदान करता है।

### How it works:

1. जब एक नया IPC नामस्थान बनाया जाता है, तो यह **System V IPC वस्तुओं के पूरी तरह से अलग सेट** के साथ शुरू होता है। इसका मतलब है कि नए IPC नामस्थान में चल रही प्रक्रियाएँ डिफ़ॉल्ट रूप से अन्य नामस्थानों या होस्ट सिस्टम में IPC वस्तुओं को एक्सेस या हस्तक्षेप नहीं कर सकतीं।
2. एक नामस्थान के भीतर बनाई गई IPC वस्तुएँ केवल **उस नामस्थान के भीतर की प्रक्रियाओं के लिए दृश्य और सुलभ होती हैं**। प्रत्येक IPC वस्तु को अपने नामस्थान के भीतर एक अद्वितीय कुंजी द्वारा पहचाना जाता है। हालांकि कुंजी विभिन्न नामस्थानों में समान हो सकती है, वस्तुएँ स्वयं अलग होती हैं और नामस्थानों के बीच एक्सेस नहीं की जा सकतीं।
3. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नामस्थानों के बीच स्थानांतरित हो सकती हैं या `CLONE_NEWIPC` ध्वज के साथ `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नामस्थान बना सकती हैं। जब एक प्रक्रिया नए नामस्थान में जाती है या एक बनाती है, तो यह उस नामस्थान से संबंधित IPC वस्तुओं का उपयोग करना शुरू कर देगी।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
By mounting a new instance of the `/proc` filesystem if you use the param `--mount-proc`, you ensure that the new mount namespace has an **सटीक और अलग दृष्टिकोण उस namespace के लिए विशिष्ट प्रक्रिया जानकारी**.

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

When `unshare` is executed without the `-f` option, an error is encountered due to the way Linux handles new PID (Process ID) namespaces. The key details and the solution are outlined below:

1. **समस्या का विवरण**:
- The Linux kernel allows a process to create new namespaces using the `unshare` system call. However, the process that initiates the creation of a new PID namespace (referred to as the "unshare" process) does not enter the new namespace; only its child processes do.
- Running `%unshare -p /bin/bash%` starts `/bin/bash` in the same process as `unshare`. Consequently, `/bin/bash` and its child processes are in the original PID namespace.
- The first child process of `/bin/bash` in the new namespace becomes PID 1. When this process exits, it triggers the cleanup of the namespace if there are no other processes, as PID 1 has the special role of adopting orphan processes. The Linux kernel will then disable PID allocation in that namespace.

2. **परिणाम**:
- The exit of PID 1 in a new namespace leads to the cleaning of the `PIDNS_HASH_ADDING` flag. This results in the `alloc_pid` function failing to allocate a new PID when creating a new process, producing the "Cannot allocate memory" error.

3. **समाधान**:
- The issue can be resolved by using the `-f` option with `unshare`. This option makes `unshare` fork a new process after creating the new PID namespace.
- Executing `%unshare -fp /bin/bash%` ensures that the `unshare` command itself becomes PID 1 in the new namespace. `/bin/bash` and its child processes are then safely contained within this new namespace, preventing the premature exit of PID 1 and allowing normal PID allocation.

By ensuring that `unshare` runs with the `-f` flag, the new PID namespace is correctly maintained, allowing `/bin/bash` and its sub-processes to operate without encountering the memory allocation error.

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपका प्रोसेस किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### सभी IPC नामस्थान खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### IPC नामस्थान के अंदर प्रवेश करें
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
आप केवल **दूसरे प्रक्रिया नामस्थान में प्रवेश कर सकते हैं यदि आप रूट हैं**। और आप **दूसरे नामस्थान में प्रवेश नहीं कर सकते** **बिना एक वर्णनकर्ता** जो इसे इंगित करता है (जैसे `/proc/self/ns/net`)।

### IPC ऑब्जेक्ट बनाएं
```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
```
## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
</details>
{% endhint %}
