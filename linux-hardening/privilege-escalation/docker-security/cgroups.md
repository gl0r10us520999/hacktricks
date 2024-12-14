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

## Temel Bilgiler

**Linux Kontrol Grupları**, veya **cgroups**, sistem kaynaklarının (CPU, bellek ve disk I/O gibi) süreç grupları arasında tahsis edilmesini, sınırlanmasını ve önceliklendirilmesini sağlayan Linux çekirdeğinin bir özelliğidir. **Süreç koleksiyonlarının kaynak kullanımını yönetme ve izole etme** mekanizması sunarak, kaynak sınırlaması, iş yükü izolasyonu ve farklı süreç grupları arasında kaynak önceliklendirmesi gibi amaçlar için faydalıdır.

**Cgroups'ın iki versiyonu** vardır: versiyon 1 ve versiyon 2. Her ikisi de bir sistemde eşzamanlı olarak kullanılabilir. Ana ayrım, **cgroups versiyon 2'nin** daha ayrıntılı ve ince kaynak dağıtımını sağlayan **hiyerarşik, ağaç benzeri bir yapı** sunmasıdır. Ayrıca, versiyon 2 çeşitli iyileştirmeler de getirir, bunlar arasında:

Yeni hiyerarşik organizasyona ek olarak, cgroups versiyon 2 ayrıca **birçok başka değişiklik ve iyileştirme** tanıtmıştır; bunlar arasında **yeni kaynak denetleyicileri** için destek, eski uygulamalar için daha iyi destek ve geliştirilmiş performans bulunmaktadır.

Genel olarak, cgroups **versiyon 2, versiyon 1'den daha fazla özellik ve daha iyi performans** sunar, ancak eski sistemlerle uyumluluğun önemli olduğu belirli senaryolarda hala kullanılabilir.

Herhangi bir sürecin v1 ve v2 cgroups'ını, /proc/\<pid> içindeki cgroup dosyasına bakarak listeleyebilirsiniz. Bu komutla shell'inizin cgroups'ına bakarak başlayabilirsiniz:
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

* **Numbers 2–12**: cgroups v1, her bir satır farklı bir cgroup'u temsil eder. Bunların denetleyicileri numaranın yanında belirtilmiştir.
* **Number 1**: Ayrıca cgroups v1, ancak yalnızca yönetim amaçları için (örneğin, systemd tarafından ayarlanmış) ve bir denetleyiciye sahip değildir.
* **Number 0**: cgroups v2'yi temsil eder. Denetleyiciler listelenmez ve bu satır yalnızca cgroups v2 çalışan sistemlerde özeldir.
* **İsimler hiyerarşiktir**, dosya yollarına benzer, farklı cgroup'lar arasındaki yapı ve ilişkiyi gösterir.
* **/user.slice veya /system.slice** gibi isimler, cgroup'ların kategorisini belirtir; user.slice genellikle systemd tarafından yönetilen oturumlar için ve system.slice sistem hizmetleri içindir.

### Cgroup'ları Görüntüleme

Dosya sistemi genellikle **cgroup'lara** erişim için kullanılır, bu da geleneksel olarak çekirdek etkileşimleri için kullanılan Unix sistem çağrısı arayüzünden farklıdır. Bir shell'in cgroup yapılandırmasını incelemek için, shell'in cgroup'unu gösteren **/proc/self/cgroup** dosyasına bakılmalıdır. Ardından, **/sys/fs/cgroup** (veya **`/sys/fs/cgroup/unified`**) dizinine giderek cgroup'un adıyla aynı isme sahip bir dizin bulduğunuzda, cgroup ile ilgili çeşitli ayarları ve kaynak kullanım bilgilerini gözlemleyebilirsiniz.

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

Cgroup'lar için ana arayüz dosyaları **cgroup** ile başlar. **cgroup.procs** dosyası, cat gibi standart komutlarla görüntülenebilir ve cgroup içindeki süreçleri listeler. Diğer bir dosya, **cgroup.threads**, thread bilgilerini içerir.

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Shell'leri yöneten cgroup'lar genellikle bellek kullanımını ve süreç sayısını düzenleyen iki denetleyici içerir. Bir denetleyici ile etkileşimde bulunmak için, denetleyicinin ön eki ile başlayan dosyalar incelenmelidir. Örneğin, **pids.current** cgroup'taki thread sayısını belirlemek için referans alınır.

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

Bir değerde **max** ifadesinin bulunması, cgroup için belirli bir sınırın olmadığını gösterir. Ancak, cgroup'ların hiyerarşik doğası nedeniyle, sınırlamalar dizin hiyerarşisinde daha alt seviyedeki bir cgroup tarafından uygulanabilir.

### Cgroup'ları Manipüle Etme ve Oluşturma

Süreçler, **Process ID (PID) değerini `cgroup.procs` dosyasına yazarak** cgroup'lara atanır. Bu, root ayrıcalıkları gerektirir. Örneğin, bir süreci eklemek için:
```bash
echo [pid] > cgroup.procs
```
Benzer şekilde, **cgroup özelliklerini değiştirmek, örneğin bir PID sınırı ayarlamak**, istenen değeri ilgili dosyaya yazarak yapılır. Bir cgroup için maksimum 3,000 PID ayarlamak için:
```bash
echo 3000 > pids.max
```
**Yeni cgroup'lar oluşturmak**, cgroup hiyerarşisinde yeni bir alt dizin oluşturmayı içerir; bu, çekirdeğin gerekli arayüz dosyalarını otomatik olarak oluşturmasını sağlar. Aktif süreçleri olmayan cgroup'lar `rmdir` ile kaldırılabilir, ancak belirli kısıtlamaların farkında olun:

* **Süreçler yalnızca yaprak cgroup'lara yerleştirilebilir** (yani, bir hiyerarşide en çok iç içe geçmiş olanlar).
* **Bir cgroup, ebeveyninde bulunmayan bir kontrolöre sahip olamaz**.
* **Çocuk cgroup'lar için kontrolörler, `cgroup.subtree_control` dosyasında açıkça belirtilmelidir**. Örneğin, bir çocuk cgroup'ta CPU ve PID kontrolörlerini etkinleştirmek için:
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
**root cgroup**, bu kuralların bir istisnasıdır ve doğrudan işlem yerleştirmeye izin verir. Bu, sistemd yönetiminden işlemleri kaldırmak için kullanılabilir.

Bir cgroup içindeki **CPU kullanımını** izlemek, toplam tüketilen CPU zamanını gösteren `cpu.stat` dosyası aracılığıyla mümkündür; bu, bir hizmetin alt süreçleri arasındaki kullanımı takip etmek için faydalıdır:

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>cpu.stat dosyasında gösterilen CPU kullanım istatistikleri</p></figcaption></figure>

## Referanslar

* **Kitap: How Linux Works, 3rd Edition: What Every Superuser Should Know By Brian Ward**

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
