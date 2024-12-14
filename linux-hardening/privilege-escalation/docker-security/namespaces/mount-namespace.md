# Mount Namespace

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

एक माउंट नेमस्पेस एक लिनक्स कर्नेल फीचर है जो प्रक्रियाओं के एक समूह द्वारा देखे जाने वाले फ़ाइल प्रणाली माउंट बिंदुओं का पृथक्करण प्रदान करता है। प्रत्येक माउंट नेमस्पेस के पास अपने स्वयं के फ़ाइल प्रणाली माउंट बिंदुओं का सेट होता है, और **एक नेमस्पेस में माउंट बिंदुओं में परिवर्तन अन्य नेमस्पेस को प्रभावित नहीं करते**। इसका मतलब है कि विभिन्न माउंट नेमस्पेस में चलने वाली प्रक्रियाएँ फ़ाइल प्रणाली पदानुक्रम का अलग-अलग दृश्य रख सकती हैं।

माउंट नेमस्पेस विशेष रूप से कंटेनराइजेशन में उपयोगी होते हैं, जहाँ प्रत्येक कंटेनर को अन्य कंटेनरों और होस्ट सिस्टम से पृथक अपने स्वयं के फ़ाइल प्रणाली और कॉन्फ़िगरेशन होना चाहिए।

### How it works:

1. जब एक नया माउंट नेमस्पेस बनाया जाता है, तो इसे **अपने माता-पिता के नेमस्पेस से माउंट बिंदुओं की एक प्रति के साथ प्रारंभ किया जाता है**। इसका मतलब है कि, निर्माण के समय, नया नेमस्पेस अपने माता-पिता के समान फ़ाइल प्रणाली का दृश्य साझा करता है। हालाँकि, नेमस्पेस के भीतर माउंट बिंदुओं में किसी भी बाद के परिवर्तन का माता-पिता या अन्य नेमस्पेस पर कोई प्रभाव नहीं पड़ेगा।
2. जब एक प्रक्रिया अपने नेमस्पेस के भीतर एक माउंट बिंदु को संशोधित करती है, जैसे कि फ़ाइल प्रणाली को माउंट या अनमाउंट करना, तो **परिवर्तन उस नेमस्पेस के लिए स्थानीय होता है** और अन्य नेमस्पेस को प्रभावित नहीं करता। यह प्रत्येक नेमस्पेस को अपनी स्वतंत्र फ़ाइल प्रणाली पदानुक्रम रखने की अनुमति देता है।
3. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच स्थानांतरित हो सकती हैं, या `CLONE_NEWNS` ध्वज के साथ `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बना सकती हैं। जब एक प्रक्रिया एक नए नेमस्पेस में जाती है या एक बनाती है, तो यह उस नेमस्पेस से जुड़े माउंट बिंदुओं का उपयोग करना शुरू कर देगी।
4. **फ़ाइल डिस्क्रिप्टर्स और इनोड्स नेमस्पेस के बीच साझा किए जाते हैं**, जिसका अर्थ है कि यदि एक नेमस्पेस में एक प्रक्रिया के पास एक फ़ाइल की ओर इशारा करने वाला एक खुला फ़ाइल डिस्क्रिप्टर है, तो यह **उस फ़ाइल डिस्क्रिप्टर को** दूसरे नेमस्पेस में एक प्रक्रिया को **दे सकती है**, और **दोनों प्रक्रियाएँ उसी फ़ाइल तक पहुँचेंगी**। हालाँकि, फ़ाइल का पथ दोनों नेमस्पेस में माउंट बिंदुओं में भिन्नता के कारण समान नहीं हो सकता है।

## Lab:

### Create different Namespaces

#### CLI
```bash
sudo unshare -m [--mount-proc] /bin/bash
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
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```
### सभी माउंट नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

{% code overflow="wrap" %}
```bash
findmnt
```
{% endcode %}

### एक माउंट नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -m TARGET_PID --pid /bin/bash
```
आप केवल **दूसरे प्रक्रिया नामस्थान में प्रवेश कर सकते हैं यदि आप रूट हैं**। और आप **दूसरे नामस्थान में प्रवेश नहीं कर सकते** **बिना एक वर्णनकर्ता** जो इसे इंगित करता है (जैसे `/proc/self/ns/mnt`)।

क्योंकि नए माउंट केवल नामस्थान के भीतर ही सुलभ होते हैं, यह संभव है कि एक नामस्थान में संवेदनशील जानकारी हो जो केवल उसी से सुलभ हो।

### कुछ माउंट करें
```bash
# Generate new mount ns
unshare -m /bin/bash
mkdir /tmp/mount_ns_example
mount -t tmpfs tmpfs /tmp/mount_ns_example
mount | grep tmpfs # "tmpfs on /tmp/mount_ns_example"
echo test > /tmp/mount_ns_example/test
ls /tmp/mount_ns_example/test # Exists

# From the host
mount | grep tmpfs # Cannot see "tmpfs on /tmp/mount_ns_example"
ls /tmp/mount_ns_example/test # Doesn't exist
```

```
# findmnt # List existing mounts
TARGET                                SOURCE                                                                                                           FSTYPE     OPTIONS
/                                     /dev/mapper/web05--vg-root

# unshare --mount  # run a shell in a new mount namespace
# mount --bind /usr/bin/ /mnt/
# ls /mnt/cp
/mnt/cp
# exit  # exit the shell, and hence the mount namespace
# ls /mnt/cp
ls: cannot access '/mnt/cp': No such file or directory

## Notice there's different files in /tmp
# ls /tmp
revshell.elf

# ls /mnt/tmp
krb5cc_75401103_X5yEyy
systemd-private-3d87c249e8a84451994ad692609cd4b6-apache2.service-77w9dT
systemd-private-3d87c249e8a84451994ad692609cd4b6-systemd-resolved.service-RnMUhT
systemd-private-3d87c249e8a84451994ad692609cd4b6-systemd-timesyncd.service-FAnDql
vmware-root_662-2689143848

```
## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)
* [https://unix.stackexchange.com/questions/464033/understanding-how-mount-namespaces-work-in-linux](https://unix.stackexchange.com/questions/464033/understanding-how-mount-namespaces-work-in-linux)


{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **हमारे** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर हमें **फॉलो करें** [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) गिटहब रिपोजिटरी में PR सबमिट करें।

</details>
{% endhint %}
