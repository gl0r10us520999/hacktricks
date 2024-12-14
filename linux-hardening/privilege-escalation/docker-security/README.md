# Docker Güvenliği

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) kullanarak dünyanın **en gelişmiş** topluluk araçlarıyla desteklenen **iş akışlarını** kolayca oluşturun ve **otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **Temel Docker Motoru Güvenliği**

**Docker motoru**, konteynerleri izole etmek için Linux çekirdeğinin **Namespaces** ve **Cgroups**'unu kullanarak temel bir güvenlik katmanı sunar. Ek koruma, **Capabilities dropping**, **Seccomp** ve **SELinux/AppArmor** ile sağlanarak konteyner izolasyonu artırılır. Bir **auth plugin** kullanıcı eylemlerini daha da kısıtlayabilir.

![Docker Güvenliği](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Docker Motoruna Güvenli Erişim

Docker motoruna, yerel olarak bir Unix soketi üzerinden veya uzaktan HTTP kullanarak erişilebilir. Uzaktan erişim için, gizlilik, bütünlük ve kimlik doğrulamasını sağlamak amacıyla HTTPS ve **TLS** kullanmak önemlidir.

Docker motoru, varsayılan olarak `unix:///var/run/docker.sock` adresindeki Unix soketinde dinler. Ubuntu sistemlerinde, Docker'ın başlangıç seçenekleri `/etc/default/docker` dosyasında tanımlanmıştır. Docker API'sine ve istemcisine uzaktan erişimi etkinleştirmek için, Docker daemon'unu bir HTTP soketi üzerinden açmak için aşağıdaki ayarları ekleyin:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
Ancak, Docker daemon'ını HTTP üzerinden açmak güvenlik endişeleri nedeniyle önerilmez. Bağlantıları HTTPS kullanarak güvence altına almak tavsiye edilir. Bağlantıyı güvence altına almanın iki ana yaklaşımı vardır:

1. İstemci, sunucunun kimliğini doğrular.
2. Hem istemci hem de sunucu, birbirlerinin kimliğini karşılıklı olarak doğrular.

Sunucunun kimliğini doğrulamak için sertifikalar kullanılır. Her iki yöntem için ayrıntılı örnekler için [**bu kılavuza**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/) bakın.

### Konteyner Görüntülerinin Güvenliği

Konteyner görüntüleri özel veya genel depolarda saklanabilir. Docker, konteyner görüntüleri için birkaç depolama seçeneği sunar:

* [**Docker Hub**](https://hub.docker.com): Docker'dan bir genel kayıt hizmeti.
* [**Docker Registry**](https://github.com/docker/distribution): Kullanıcıların kendi kayıtlarını barındırmalarına olanak tanıyan açık kaynaklı bir proje.
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): Rol tabanlı kullanıcı kimlik doğrulaması ve LDAP dizin hizmetleri ile entegrasyon sunan Docker'ın ticari kayıt teklifi.

### Görüntü Tarama

Konteynerler, ya temel görüntü nedeniyle ya da temel görüntünün üzerine kurulu yazılım nedeniyle **güvenlik açıklarına** sahip olabilir. Docker, konteynerlerin güvenlik taramasını yapan ve açıkları listeleyen **Nautilus** adlı bir proje üzerinde çalışıyor. Nautilus, her konteyner görüntü katmanını güvenlik açıkları deposu ile karşılaştırarak güvenlik açıklarını tanımlar.

Daha fazla [**bilgi için bunu okuyun**](https://docs.docker.com/engine/scan/).

* **`docker scan`**

**`docker scan`** komutu, mevcut Docker görüntülerini görüntü adı veya kimliği kullanarak taramanıza olanak tanır. Örneğin, hello-world görüntüsünü taramak için aşağıdaki komutu çalıştırın:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Docker Image Signing

Docker imaj imzalama, konteynerlerde kullanılan imajların güvenliğini ve bütünlüğünü sağlar. İşte kısaca bir açıklama:

* **Docker Content Trust**, imaj imzalamayı yönetmek için The Update Framework (TUF) tabanlı Notary projesini kullanır. Daha fazla bilgi için [Notary](https://github.com/docker/notary) ve [TUF](https://theupdateframework.github.io) sayfalarına bakın.
* Docker içerik güvenini etkinleştirmek için `export DOCKER_CONTENT_TRUST=1` ayarını yapın. Bu özellik, Docker sürüm 1.10 ve sonrasında varsayılan olarak kapalıdır.
* Bu özellik etkinleştirildiğinde, yalnızca imzalı imajlar indirilebilir. İlk imaj yüklemesi, kök ve etiketleme anahtarları için şifreler ayarlamayı gerektirir; Docker ayrıca artırılmış güvenlik için Yubikey'i de destekler. Daha fazla ayrıntı [burada](https://blog.docker.com/2015/11/docker-content-trust-yubikey/) bulunabilir.
* İçerik güveni etkinken imzasız bir imaj çekmeye çalışmak "No trust data for latest" hatası ile sonuçlanır.
* İlk imaj yüklemesinden sonraki imaj yüklemeleri için, Docker imajı imzalamak için depo anahtarının şifresini ister.

Özel anahtarlarınızı yedeklemek için şu komutu kullanın:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Docker ana bilgisayarlarını değiştirirken, işlemleri sürdürmek için kök ve depo anahtarlarını taşımak gereklidir.

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Dünyanın **en gelişmiş** topluluk araçlarıyla desteklenen **iş akışlarını** kolayca oluşturmak ve **otomatikleştirmek** için [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) kullanın.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## Konteyner Güvenlik Özellikleri

<details>

<summary>Konteyner Güvenlik Özelliklerinin Özeti</summary>

**Ana Süreç İzolasyon Özellikleri**

Konteynerleştirilmiş ortamlarda, projeleri ve süreçlerini izole etmek güvenlik ve kaynak yönetimi için çok önemlidir. İşte ana kavramların basitleştirilmiş bir açıklaması:

**İsim Alanları**

* **Amaç**: Süreçler, ağ ve dosya sistemleri gibi kaynakların izolasyonunu sağlamak. Özellikle Docker'da, isim alanları bir konteynerin süreçlerini ana bilgisayardan ve diğer konteynerlerden ayrı tutar.
* **`unshare` Kullanımı**: Yeni isim alanları oluşturmak için `unshare` komutu (veya temel syscall) kullanılır, bu da ek bir izolasyon katmanı sağlar. Ancak, Kubernetes bunu doğrudan engellemese de, Docker engeller.
* **Sınırlama**: Yeni isim alanları oluşturmak, bir sürecin ana bilgisayarın varsayılan isim alanlarına geri dönmesine izin vermez. Ana bilgisayar isim alanlarına girmek için genellikle ana bilgisayarın `/proc` dizinine erişim gereklidir ve giriş için `nsenter` kullanılır.

**Kontrol Grupları (CGroups)**

* **Fonksiyon**: Öncelikle süreçler arasında kaynak tahsis etmek için kullanılır.
* **Güvenlik Boyutu**: CGroups kendileri izolasyon güvenliği sunmaz, yalnızca yanlış yapılandırıldığında yetkisiz erişim için potansiyel olarak istismar edilebilecek `release_agent` özelliği vardır.

**Yetenek Düşürme**

* **Önemi**: Süreç izolasyonu için kritik bir güvenlik özelliğidir.
* **Fonksiyonu**: Belirli yetenekleri düşürerek bir kök sürecin gerçekleştirebileceği eylemleri kısıtlar. Bir süreç kök ayrıcalıklarıyla çalışsa bile, gerekli yeteneklere sahip olmaması, ayrıcalıklı eylemleri gerçekleştirmesini engeller, çünkü sistem çağrıları yetersiz izinler nedeniyle başarısız olur.

Bu, süreç diğerlerini düşürdükten sonra **kalan yeteneklerdir**:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

Docker'da varsayılan olarak etkinleştirilmiştir. Bu, **sistem çağrılarını** daha da **sınırlamaya** yardımcı olur.\
**Varsayılan Docker Seccomp profili** [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) adresinde bulunabilir.

**AppArmor**

Docker'ın etkinleştirebileceğiniz bir şablonu vardır: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Bu, yetenekleri, sistem çağrılarını, dosyalara ve klasörlere erişimi azaltmanıza olanak tanır...

</details>

### Namespaces

**Namespaces**, **çekirdek kaynaklarını** **bölümlere ayıran** Linux çekirdek özelliğidir; böylece bir **işlem** kümesi bir **kaynak** kümesini **görürken**, **diğer** bir **işlem** kümesi **farklı** bir kaynak kümesini görür. Bu özellik, bir kaynak ve işlem kümesi için aynı namespace'e sahip olarak çalışır, ancak bu namespace'ler farklı kaynaklara atıfta bulunur. Kaynaklar birden fazla alanda var olabilir.

Docker, konteyner izolasyonu sağlamak için aşağıdaki Linux çekirdek Namespaces'lerini kullanır:

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

**Namespaces hakkında daha fazla bilgi** için aşağıdaki sayfayı kontrol edin:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Linux çekirdek özelliği **cgroups**, bir dizi işlem arasında **cpu, bellek, io, ağ bant genişliği gibi kaynakları kısıtlama** yeteneği sağlar. Docker, belirli bir Konteyner için kaynak kontrolü sağlayan cgroup özelliğini kullanarak Konteynerler oluşturmanıza olanak tanır.\
Aşağıda, kullanıcı alanı belleği 500m ile sınırlı, çekirdek belleği 50m ile sınırlı, cpu payı 512, blkioweight 400 olan bir Konteyner oluşturulmuştur. CPU payı, Konteyner’in CPU kullanımını kontrol eden bir orandır. Varsayılan değeri 1024'tür ve 0 ile 1024 arasında bir aralığa sahiptir. Üç Konteyner 1024 CPU payına sahipse, her Konteyner CPU kaynak rekabeti durumunda %33'e kadar CPU alabilir. blkio-weight, Konteyner’in IO'sunu kontrol eden bir orandır. Varsayılan değeri 500'dür ve 10 ile 1000 arasında bir aralığa sahiptir.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Bir konteynerin cgroup'unu almak için şunu yapabilirsiniz:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
Daha fazla bilgi için kontrol edin:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Yetenekler

Yetenekler, **root kullanıcısı için izin verilebilecek yetenekler üzerinde daha ince bir kontrol sağlar**. Docker, **bir konteyner içinde yapılabilecek işlemleri sınırlamak için Linux çekirdek yetenek özelliğini** kullanır, kullanıcı türünden bağımsız olarak.

Bir docker konteyneri çalıştırıldığında, **işlem, izolasyondan kaçmak için kullanabileceği hassas yetenekleri düşürür**. Bu, işlemin hassas eylemleri gerçekleştiremeyeceğini ve kaçamayacağını sağlamaya çalışır:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Docker'da Seccomp

Bu, Docker'ın konteyner içinde kullanılabilecek **sistem çağrılarını sınırlamasına** olanak tanıyan bir güvenlik özelliğidir:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### Docker'da AppArmor

**AppArmor**, **konteynerleri** **sınırlı** bir **kaynak** setine **program başına profillerle** hapsetmek için bir çekirdek geliştirmesidir.:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### Docker'da SELinux

* **Etiketleme Sistemi**: SELinux, her işleme ve dosya sistemi nesnesine benzersiz bir etiket atar.
* **Politika Uygulaması**: Bir işlem etiketinin sistem içindeki diğer etiketler üzerinde hangi eylemleri gerçekleştirebileceğini tanımlayan güvenlik politikalarını uygular.
* **Konteyner İşlem Etiketleri**: Konteyner motorları konteyner işlemlerini başlattığında, genellikle sınırlı bir SELinux etiketi, yaygın olarak `container_t` atanır.
* **Konteyner İçindeki Dosya Etiketleme**: Konteyner içindeki dosyalar genellikle `container_file_t` olarak etiketlenir.
* **Politika Kuralları**: SELinux politikası esasen `container_t` etiketine sahip işlemlerin yalnızca `container_file_t` olarak etiketlenmiş dosyalarla etkileşimde bulunabileceğini garanti eder.

Bu mekanizma, bir konteyner içindeki bir işlem tehlikeye girse bile, yalnızca karşılık gelen etiketlere sahip nesnelerle etkileşimde bulunmakla sınırlı kalmasını sağlar ve bu tür tehlikelerin potansiyel zararını önemli ölçüde sınırlar.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

Docker'da, bir yetkilendirme eklentisi, Docker daemon'una yapılan istekleri kabul etme veya engelleme kararını vererek güvenlikte kritik bir rol oynar. Bu karar, iki ana bağlamı inceleyerek verilir:

* **Kimlik Doğrulama Bağlamı**: Bu, kullanıcının kim olduğu ve kendini nasıl kimlik doğruladığı gibi kapsamlı bilgileri içerir.
* **Komut Bağlamı**: Bu, yapılan istekle ilgili tüm ilgili verileri içerir.

Bu bağlamlar, yalnızca kimlik doğrulaması yapılmış kullanıcıların meşru isteklerinin işlenmesini sağlamaya yardımcı olur ve Docker işlemlerinin güvenliğini artırır.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Bir konteynerden DoS

Eğer bir konteynerin kullanabileceği kaynakları düzgün bir şekilde sınırlamazsanız, tehlikeye giren bir konteyner, çalıştığı ana makineyi DoS yapabilir.

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* Bant Genişliği DoS
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## İlginç Docker Bayrakları

### --privileged bayrağı

Aşağıdaki sayfada **`--privileged` bayrağının ne anlama geldiğini** öğrenebilirsiniz:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Eğer bir saldırganın düşük ayrıcalıklı bir kullanıcı olarak erişim sağladığı bir konteyner çalıştırıyorsanız. Eğer **yanlış yapılandırılmış bir suid ikili dosyanız** varsa, saldırgan bunu kötüye kullanabilir ve **konteyner içinde ayrıcalıkları artırabilir**. Bu, onun oradan kaçmasına izin verebilir.

Konteyneri **`no-new-privileges`** seçeneği etkinleştirilmiş olarak çalıştırmak, **bu tür ayrıcalık artışlarını önleyecektir**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Diğer
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
For more **`--security-opt`** options check: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## Diğer Güvenlik Dikkatleri

### Gizli Bilgilerin Yönetimi: En İyi Uygulamalar

Gizli bilgileri doğrudan Docker imajlarına gömmekten veya ortam değişkenleri kullanmaktan kaçınmak çok önemlidir, çünkü bu yöntemler hassas bilgilerinizi `docker inspect` veya `exec` gibi komutlar aracılığıyla konteynıra erişimi olan herkesin erişimine açar.

**Docker hacimleri**, hassas bilgilere erişim için önerilen daha güvenli bir alternatiftir. Bunlar, `docker inspect` ve günlükleme ile ilişkili riskleri azaltarak, bellek içinde geçici bir dosya sistemi olarak kullanılabilir. Ancak, kök kullanıcılar ve konteynıra `exec` erişimi olanlar yine de gizli bilgilere erişebilir.

**Docker gizli bilgileri**, hassas bilgileri yönetmek için daha güvenli bir yöntem sunar. İmaj oluşturma aşamasında gizli bilgilere ihtiyaç duyan örnekler için, **BuildKit** yapı zamanı gizli bilgileri destekleyerek, yapı hızını artıran ve ek özellikler sunan etkili bir çözüm sunar.

BuildKit'i kullanmak için üç şekilde etkinleştirilebilir:

1. Bir ortam değişkeni aracılığıyla: `export DOCKER_BUILDKIT=1`
2. Komutları ön ekleyerek: `DOCKER_BUILDKIT=1 docker build .`
3. Docker yapılandırmasında varsayılan olarak etkinleştirerek: `{ "features": { "buildkit": true } }`, ardından Docker'ı yeniden başlatarak.

BuildKit, `--secret` seçeneği ile yapı zamanı gizli bilgilerin kullanılmasına olanak tanır ve bu gizli bilgilerin imaj oluşturma önbelleğine veya nihai imaja dahil edilmemesini sağlar, şu komutla:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Çalışan bir konteynerde gereken gizli bilgiler için, **Docker Compose ve Kubernetes** sağlam çözümler sunar. Docker Compose, gizli dosyaları belirtmek için hizmet tanımında bir `secrets` anahtarı kullanır; bu, bir `docker-compose.yml` örneğinde gösterilmiştir:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
Bu yapılandırma, Docker Compose ile hizmetleri başlatırken gizli bilgilerin kullanılmasına olanak tanır.

Kubernetes ortamlarında, gizli bilgiler yerel olarak desteklenir ve [Helm-Secrets](https://github.com/futuresimple/helm-secrets) gibi araçlarla daha fazla yönetilebilir. Kubernetes'in Rol Tabanlı Erişim Kontrolleri (RBAC), gizli bilgi yönetimi güvenliğini artırır, bu Docker Enterprise'a benzer.

### gVisor

**gVisor**, Go dilinde yazılmış bir uygulama çekirdeğidir ve Linux sistem yüzeyinin önemli bir kısmını uygular. Uygulama ile ana makine çekirdeği arasında **izolasyon sınırı** sağlayan `runsc` adlı bir [Open Container Initiative (OCI)](https://www.opencontainers.org) çalışma zamanı içerir. `runsc` çalışma zamanı, Docker ve Kubernetes ile entegre olur ve kumanda edilmiş konteynerleri çalıştırmayı basit hale getirir.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers**, hafif sanal makinelerle güvenli bir konteyner çalışma zamanı oluşturmak için çalışan açık kaynak topluluğudur. Bu sanal makineler, konteynerler gibi hissettiren ve performans gösteren, ancak **donanım sanallaştırma** teknolojisini ikinci bir savunma katmanı olarak kullanarak **daha güçlü iş yükü izolasyonu** sağlayan bir yapıdır.

{% embed url="https://katacontainers.io/" %}

### Özet İpuçları

* **`--privileged` bayrağını kullanmayın veya** [**konteyner içinde Docker soketini monte etmeyin**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** Docker soketi, konteynerlerin başlatılmasına olanak tanır, bu nedenle başka bir konteyneri `--privileged` bayrağı ile çalıştırarak ana makinenin tam kontrolünü ele geçirmenin kolay bir yoludur.
* **Konteyner içinde root olarak çalışmayın.** [**Farklı bir kullanıcı**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) **kullanın ve** [**kullanıcı ad alanları**](https://docs.docker.com/engine/security/userns-remap/)**.** Konteynerdeki root, kullanıcı ad alanları ile yeniden haritalanmadıkça ana makinedeki ile aynıdır. Sadece, esasen, Linux ad alanları, yetenekler ve cgroups tarafından hafifçe kısıtlanmıştır.
* [**Tüm yetenekleri kaldırın**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) ve yalnızca gerekli olanları etkinleştirin** (`--cap-add=...`). Birçok iş yükü herhangi bir yetenek gerektirmez ve bunları eklemek, potansiyel bir saldırının kapsamını artırır.
* [**“no-new-privileges” güvenlik seçeneğini kullanın**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) **süreçlerin daha fazla yetki kazanmasını önlemek için, örneğin suid ikili dosyaları aracılığıyla.**
* [**Konteynere sunulan kaynakları sınırlayın**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Kaynak sınırları, makineyi hizmet reddi saldırılarından koruyabilir.
* **Kısıtlamaları en aza indirmek için** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(veya SELinux)** profillerini ayarlayın.
* **Resmi docker görüntülerini kullanın** [**ve imzaları gerektirin**](https://docs.docker.com/docker-hub/official_images/) veya bunlara dayanarak kendi görüntülerinizi oluşturun. [Arka kapılı](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) görüntüleri miras almayın veya kullanmayın. Ayrıca, root anahtarlarını ve şifreleri güvenli bir yerde saklayın. Docker, anahtarları UCP ile yönetmeyi planlıyor.
* **Düzenli olarak** **görüntülerinizi yeniden oluşturun** **güvenlik yamalarını ana makineye ve görüntülere uygulamak için.**
* **Gizli bilgilerinizi akıllıca yönetin** böylece saldırganın bunlara erişmesi zor olur.
* Eğer **docker daemon'u açığa çıkarıyorsanız HTTPS kullanın** istemci ve sunucu kimlik doğrulaması ile.
* Dockerfile'ınızda, **ADD yerine COPY'yi tercih edin**. ADD, otomatik olarak sıkıştırılmış dosyaları çıkarır ve URL'lerden dosya kopyalayabilir. COPY bu yeteneklere sahip değildir. Mümkün olduğunca, uzaktan URL'ler ve Zip dosyaları aracılığıyla saldırılara maruz kalmamak için ADD kullanmaktan kaçının.
* **Her mikro hizmet için ayrı konteynerler bulundurun.**
* **Konteyner içinde ssh bulundurmayın**, “docker exec” kullanılarak konteynere ssh yapılabilir.
* **Daha küçük** konteyner **görüntüleri** bulundurun.

## Docker Breakout / Yetki Yükseltme

Eğer **bir docker konteynerinin içindeyseniz** veya **docker grubunda** bir kullanıcıya erişiminiz varsa, **kaçmayı ve yetkileri yükseltmeyi** deneyebilirsiniz:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Docker Kimlik Doğrulama Eklentisi Atlatma

Eğer docker soketine erişiminiz varsa veya **docker grubunda** bir kullanıcıya erişiminiz varsa ancak eylemleriniz bir docker kimlik doğrulama eklentisi tarafından kısıtlanıyorsa, **bunu atlatıp atlatamayacağınıza bakın:**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Docker'ı Güçlendirme

* [**docker-bench-security**](https://github.com/docker/docker-bench-security) aracı, üretimde Docker konteynerlerini dağıtma ile ilgili birçok yaygın en iyi uygulamayı kontrol eden bir betiktir. Testler tamamen otomatik olup, [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/) temel alınarak yapılmaktadır.\
Aracı, docker'ı çalıştıran ana makineden veya yeterli yetkilere sahip bir konteynerden çalıştırmalısınız. **README'de nasıl çalıştırılacağını öğrenin:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## Referanslar

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux_namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins_authorization](https://docs.docker.com/engine/extend/plugins_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) kullanarak dünyanın **en gelişmiş** topluluk araçlarıyla **iş akışlarını** kolayca oluşturun ve **otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
