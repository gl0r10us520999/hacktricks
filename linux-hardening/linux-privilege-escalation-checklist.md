# Kontrol Listesi - Linux Yetki Yükseltme

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Deneyimli hackerlar ve bug bounty avcıları ile iletişim kurmak için [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) sunucusuna katılın!

**Hacking İçgörüleri**\
Hacking'in heyecanı ve zorluklarına dalan içeriklerle etkileşimde bulunun

**Gerçek Zamanlı Hack Haberleri**\
Gerçek zamanlı haberler ve içgörülerle hızlı tempolu hacking dünyasında güncel kalın

**Son Duyurular**\
Yeni başlayan bug bounty'ler ve kritik platform güncellemeleri hakkında bilgi sahibi olun

**Bugün [**Discord**](https://discord.com/invite/N3FrSbmwdy) üzerinden bize katılın ve en iyi hackerlarla işbirliği yapmaya başlayın!**

### **Linux yerel yetki yükseltme vektörlerini aramak için en iyi araç:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [Sistem Bilgisi](privilege-escalation/#system-information)

* [ ] **OS bilgilerini** alın
* [ ] [**PATH**](privilege-escalation/#path)'i kontrol edin, herhangi bir **yazılabilir klasör** var mı?
* [ ] [**env değişkenlerini**](privilege-escalation/#env-info) kontrol edin, herhangi bir hassas detay var mı?
* [ ] [**kernel exploit'lerini**](privilege-escalation/#kernel-exploits) **script kullanarak** arayın (DirtyCow?)
* [ ] [**sudo versiyonunun**](privilege-escalation/#sudo-version) **güvenli olup olmadığını** kontrol edin
* [ ] [**Dmesg** imza doğrulaması başarısız](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] Daha fazla sistem enum ([tarih, sistem istatistikleri, cpu bilgisi, yazıcılar](privilege-escalation/#more-system-enumeration))
* [ ] [Daha fazla savunma enumerate edin](privilege-escalation/#enumerate-possible-defenses)

### [Sürücüler](privilege-escalation/#drives)

* [ ] **Bağlı** sürücüleri listeleyin
* [ ] **Herhangi bir bağlı olmayan sürücü var mı?**
* [ ] **fstab'da herhangi bir kimlik bilgisi var mı?**

### [**Yüklenmiş Yazılım**](privilege-escalation/#installed-software)

* [ ] **Yüklenmiş** [**yararlı yazılımları**](privilege-escalation/#useful-software) kontrol edin
* [ ] **Yüklenmiş** [**güvenlik açığı olan yazılımları**](privilege-escalation/#vulnerable-software-installed) kontrol edin

### [Süreçler](privilege-escalation/#processes)

* [ ] Herhangi bir **bilinmeyen yazılım çalışıyor mu**?
* [ ] Herhangi bir yazılım **olması gerektiğinden daha fazla yetkiyle** mi çalışıyor?
* [ ] **Çalışan süreçlerin exploit'lerini** arayın (özellikle çalışan versiyonu).
* [ ] Herhangi bir çalışan sürecin **ikili dosyasını** **değiştirebilir misiniz**?
* [ ] **Süreçleri izleyin** ve ilginç bir sürecin sıkça çalışıp çalışmadığını kontrol edin.
* [ ] Bazı ilginç **süreç belleğini** (şifrelerin kaydedilebileceği yer) **okuyabilir misiniz**?

### [Zamanlanmış/Cron görevleri?](privilege-escalation/#scheduled-jobs)

* [ ] [**PATH**](privilege-escalation/#cron-path) bazı cron tarafından değiştiriliyor mu ve siz **yazabilir misiniz**?
* [ ] Bir cron görevinde herhangi bir [**wildcard**](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection) var mı?
* [ ] **Değiştirilebilir bir script** (cron script'inin üzerine yazma ve symlink) **çalıştırılıyor mu** veya **değiştirilebilir klasör** içinde mi?
* [ ] Bazı **script'lerin** [**çok sık**](privilege-escalation/#frequent-cron-jobs) **çalıştırıldığını** tespit ettiniz mi? (her 1, 2 veya 5 dakikada bir)

### [Hizmetler](privilege-escalation/#services)

* [ ] Herhangi bir **yazılabilir .service** dosyası var mı?
* [ ] Herhangi bir **hizmet tarafından yürütülen yazılabilir ikili** var mı?
* [ ] **systemd PATH** içinde herhangi bir **yazılabilir klasör** var mı?

### [Zamanlayıcılar](privilege-escalation/#timers)

* [ ] Herhangi bir **yazılabilir zamanlayıcı** var mı?

### [Socket'ler](privilege-escalation/#sockets)

* [ ] Herhangi bir **yazılabilir .socket** dosyası var mı?
* [ ] Herhangi bir socket ile **iletişim kurabilir misiniz**?
* [ ] **İlginç bilgiler içeren HTTP socket'leri** var mı?

### [D-Bus](privilege-escalation/#d-bus)

* [ ] Herhangi bir **D-Bus ile iletişim kurabilir misiniz**?

### [Ağ](privilege-escalation/#network)

* Ağda nerede olduğunuzu bilmek için enumerate edin
* **Makineye shell alana kadar erişemediğiniz açık portlar var mı?**
* `tcpdump` kullanarak **trafik dinleyebilir misiniz**?

### [Kullanıcılar](privilege-escalation/#users)

* Genel kullanıcılar/gruplar **enumerasyonu**
* **Çok büyük bir UID'niz var mı? Makine **güvenlik açığı** mı?**
* **Ait olduğunuz bir grup sayesinde** [**yetki yükseltebilir misiniz**](privilege-escalation/interesting-groups-linux-pe/)?
* **Pano** verileri?
* Şifre Politikası?
* Daha önce keşfettiğiniz her **bilinen şifreyi** kullanarak **her bir** olası **kullanıcıyla** giriş yapmayı deneyin. Şifre olmadan da giriş yapmayı deneyin.

### [Yazılabilir PATH](privilege-escalation/#writable-path-abuses)

* Eğer **PATH'teki bazı klasörlerde yazma yetkiniz varsa** yetki yükseltebilirsiniz

### [SUDO ve SUID komutları](privilege-escalation/#sudo-and-suid)

* **Herhangi bir komutu sudo ile çalıştırabilir misiniz**? Root olarak herhangi bir şeyi OKUMAK, YAZMAK veya ÇALIŞTIRMAK için kullanabilir misiniz? ([**GTFOBins**](https://gtfobins.github.io))
* Herhangi bir **istismar edilebilir SUID ikilisi** var mı? ([**GTFOBins**](https://gtfobins.github.io))
* [**sudo** komutları **path** ile **sınırlı mı**? kısıtlamaları **aşabilir misiniz**](privilege-escalation/#sudo-execution-bypassing-paths)?
* [**Path belirtilmeden Sudo/SUID ikilisi**](privilege-escalation/#sudo-command-suid-binary-without-command-path)?
* [**Path belirten SUID ikilisi**](privilege-escalation/#suid-binary-with-command-path)? Aşma
* [**LD\_PRELOAD açığı**](privilege-escalation/#ld_preload)
* Yazılabilir bir klasörden [**SUID ikilisinde .so kütüphanesi eksikliği**](privilege-escalation/#suid-binary-so-injection)?
* [**SUDO token'leri mevcut**](privilege-escalation/#reusing-sudo-tokens)? [**Bir SUDO token'i oluşturabilir misiniz**](privilege-escalation/#var-run-sudo-ts-less-than-username-greater-than)?
* [**sudoers dosyalarını okuyabilir veya değiştirebilir misiniz**](privilege-escalation/#etc-sudoers-etc-sudoers-d)?
* [**/etc/ld.so.conf.d/**](privilege-escalation/#etc-ld-so-conf-d) dosyasını **değiştirebilir misiniz**?
* [**OpenBSD DOAS**](privilege-escalation/#doas) komutu

### [Yetenekler](privilege-escalation/#capabilities)

* Herhangi bir ikilinin herhangi bir **beklenmedik yeteneği** var mı?

### [ACL'ler](privilege-escalation/#acls)

* Herhangi bir dosyanın herhangi bir **beklenmedik ACL'si** var mı?

### [Açık Shell oturumları](privilege-escalation/#open-shell-sessions)

* [ ] **screen**
* [ ] **tmux**

### [SSH](privilege-escalation/#ssh)

* [ ] **Debian** [**OpenSSL Tahmin Edilebilir PRNG - CVE-2008-0166**](privilege-escalation/#debian-openssl-predictable-prng-cve-2008-0166)
* [ ] [**SSH İlginç yapılandırma değerleri**](privilege-escalation/#ssh-interesting-configuration-values)

### [İlginç Dosyalar](privilege-escalation/#interesting-files)

* [ ] **Profil dosyaları** - Hassas verileri okuyun? Privesc için yazın?
* [ ] **passwd/shadow dosyaları** - Hassas verileri okuyun? Privesc için yazın?
* [ ] Hassas veriler için **yaygın ilginç klasörleri** kontrol edin
* [ ] **Garip Konum/Sahip dosyalar,** erişiminiz olabileceği veya yürütülebilir dosyaları değiştirebileceğiniz dosyalar
* [ ] Son dakikalarda **değiştirildi**
* [ ] **Sqlite DB dosyaları**
* [ ] **Gizli dosyalar**
* [ ] **PATH'teki Script/İkili dosyalar**
* [ ] **Web dosyaları** (şifreler?)
* [ ] **Yedekler**?
* [ ] **Şifreleri içeren bilinen dosyalar**: **Linpeas** ve **LaZagne** kullanın
* [ ] **Genel arama**

### [**Yazılabilir Dosyalar**](privilege-escalation/#writable-files)

* **Rasgele komutlar çalıştırmak için python kütüphanesini** değiştirebilir misiniz?
* **Log dosyalarını değiştirebilir misiniz**? **Logtotten** açığı
* **/etc/sysconfig/network-scripts/** dosyasını değiştirebilir misiniz? Centos/Redhat açığı
* [**ini, int.d, systemd veya rc.d dosyalarına yazabilir misiniz**](privilege-escalation/#init-init-d-systemd-and-rc-d)?

### [**Diğer ipuçları**](privilege-escalation/#other-tricks)

* [**NFS'i yetki yükseltmek için kötüye kullanabilir misiniz**](privilege-escalation/#nfs-privilege-escalation)?
* [**Kısıtlayıcı bir shell'den kaçmak için**](privilege-escalation/#escaping-from-restricted-shells) **kaçış yapmanız gerekiyor mu**?

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Deneyimli hackerlar ve bug bounty avcıları ile iletişim kurmak için [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) sunucusuna katılın!

**Hacking İçgörüleri**\
Hacking'in heyecanı ve zorluklarına dalan içeriklerle etkileşimde bulunun

**Gerçek Zamanlı Hack Haberleri**\
Gerçek zamanlı haberler ve içgörülerle hızlı tempolu hacking dünyasında güncel kalın

**Son Duyurular**\
Yeni başlayan bug bounty'ler ve kritik platform güncellemeleri hakkında bilgi sahibi olun

**Bugün [**Discord**](https://discord.com/invite/N3FrSbmwdy) üzerinden bize katılın ve en iyi hackerlarla işbirliği yapmaya başlayın!**

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="../.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="../.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="../.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="../.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
