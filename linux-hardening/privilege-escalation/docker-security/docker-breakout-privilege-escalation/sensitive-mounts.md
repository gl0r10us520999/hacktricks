# Hassas Bağlantı Noktaları

{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) katılın veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* Hacking püf noktalarını paylaşarak PR'ler göndererek [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

Doğru namespace izolasyonu olmadan `/proc` ve `/sys`'in maruz kalması, saldırı yüzeyinin genişlemesi ve bilgi sızdırma gibi ciddi güvenlik risklerini beraberinde getirir. Bu dizinler, yanlış yapılandırılmış veya yetkisiz bir kullanıcı tarafından erişilen hassas dosyaları içerir ve bu da konteyner kaçışına, ana bilgisayarın değiştirilmesine veya daha fazla saldırıya yardımcı olacak bilgilerin sağlanmasına yol açabilir. Örneğin, `-v /proc:/host/proc` şeklinde yanlış bağlama yapılması, yol tabanlı doğası nedeniyle AppArmor korumasını atlayabilir ve `/host/proc`'u korumasız bırakabilir.

**Her potansiyel zafiyetin daha fazla ayrıntısını** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)** adresinde bulabilirsiniz.**

## procfs Zafiyetleri

### `/proc/sys`

Bu dizin, genellikle `sysctl(2)` aracılığıyla çekirdek değişkenlerini değiştirme izni verir ve endişe kaynağı olan birkaç alt dizini içerir:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) adresinde açıklanmıştır.
* İlk 128 baytı argümanlar olarak alan bir programın çekirdek dosyası oluşturulduğunda çalıştırılmasına izin verir. Dosya bir boru `|` ile başlıyorsa kod yürütme olabilir.
*   **Test ve Sömürü Örneği**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # Yazma erişimini test et
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Özel işleyiciyi ayarla
sleep 5 && ./crash & # İşleyiciyi tetikle
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde detaylı olarak açıklanmıştır.
* Çekirdek modül yükleyicisinin yolunu içerir ve çekirdek modüllerini yüklemek için çağrılır.
*   **Erişimi Kontrol Etme Örneği**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe erişimini kontrol et
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde referans gösterilmiştir.
* Bir OOM durumu meydana geldiğinde çekirdeğin çökmesini veya OOM öldürücüyü çağırmasını kontrol eden genel bir bayrak.

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde belirtildiği gibi, dosya sistemi hakkında seçenekler ve bilgiler içerir.
* Yazma erişimi, ana bilgisayara karşı çeşitli hizmet reddi saldırılarına olanak tanır.

#### **`/proc/sys/fs/binfmt_misc`**

* Sihirli sayılarına dayalı olmayan ikili biçimler için yorumlayıcıları kaydetmeye olanak tanır.
* `/proc/sys/fs/binfmt_misc/register` yazılabilirse ayrıcalık yükseltmesine veya kök kabuk erişimine yol açabilir.
* İlgili saldırı ve açıklama:
* [binfmt\_misc ile yoksul adamın kök kiti](https://github.com/toffan/binfmt\_misc)
* Detaylı öğretici: [Video bağlantısı](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Diğerleri `/proc` içinde

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC` etkinse çekirdek yapılandırmasını ortaya çıkarabilir.
* Çalışan çekirdekteki zafiyetleri belirlemek için saldırganlar için faydalıdır.

#### **`/proc/sysrq-trigger`**

* Sysrq komutlarını çağırmaya izin verir ve potansiyel olarak anında sistem yeniden başlatmalar veya diğer kritik işlemlere neden olabilir.
*   **Ana Bilgisayarı Yeniden Başlatma Örneği**:

```bash
echo b > /proc/sysrq-trigger # Ana bilgisayarı yeniden başlatır
```

#### **`/proc/kmsg`**

* Çekirdek halka tamponu mesajlarını açığa çıkarır.
* Çekirdek saldırılarına, adres sızıntılarına ve hassas sistem bilgilerinin sağlanmasına yardımcı olabilir.

#### **`/proc/kallsyms`**

* Çekirdek dışa aktarılan sembolleri ve adreslerini listeler.
* Özellikle KASLR'yi aşmak için çekirdek saldırı geliştirme için temel öneme sahiptir.
* Adres bilgileri `kptr_restrict`'in `1` veya `2` olarak ayarlanmasıyla sınırlıdır.
* Detaylar [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresinde bulunabilir.

#### **`/proc/[pid]/mem`**

* Çekirdek bellek cihazı `/dev/mem` ile etkileşim sağlar.
* Tarihsel olarak ayrıcalık yükseltme saldırılarına karşı savunmasızdır.
* Daha fazlası için [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) adresine bakın.

#### **`/proc/kcore`**

* Sistemin fiziksel belleğini ELF çekirdek biçiminde temsil eder.
* Okuma, ana bilgisayar sistemi ve diğer konteynerlerin bellek içeriğini sızdırabilir.
* Büyük dosya boyutu okuma sorunlarına veya yazılım çökmelerine yol açabilir.
* Ayrıntılı kullanım [2019'da /proc/kcore Dökme](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) adresinde bulunabilir.

#### **`/proc/kmem`**

* Çekirdek sanal belleği temsil eden `/dev/kmem` için alternatif bir arayüz.
* Okuma ve yazma izni verir, dolayısıyla çekirdek belleğinin doğrudan değiştirilmesine olanak tanır.

#### **`/proc/mem`**

* Fiziksel belleği temsil eden `/dev/mem` için alternatif bir arayüz.
* Okuma ve yazma izni verir, tüm belleğin değiştirilmesi sanal adresleri fiziksel adreslere çözümlemeyi gerektirir.

#### **`/proc/sched_debug`**

* PID ad alanı korumalarını atlayarak işlem zamanlama bilgilerini döndürür.
* İşlem adlarını, kimlikleri ve cgroup tanımlayıcılarını açığa çıkarır.

#### **`/proc/[pid]/mountinfo`**

* İşlem bağlantı noktaları hakkında bilgi sağlar.
* Konteynerin `rootfs` veya görüntünün konumunu açığa çıkarır.

### `/sys` Zafiyetleri

#### **`/sys/kernel/uevent_helper`**

* Çekirdek cihaz `uevent`'leri işlemek için kullanılır.
* `/sys/kernel/uevent_helper`'a yazmak, `uevent` tetikleyicileri üzerine keyfi komut dosyalarını yürütebilir.
*   **Sömürü için Örnek**: %%%bash

#### Bir yük oluşturur

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### OverlayFS bağlantı noktasından ana bilgisayar yolunu bulur

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### uevent\_helper'ı kötü amaçlı yardımcıya ayarlar

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Bir uevent tetikler

echo change > /sys/class/mem/null/uevent

#### Çıktıyı okur

cat /output %%%
#### **`/sys/class/thermal`**

* Sıcaklık ayarlarını kontrol eder, olası DoS saldırılarına veya fiziksel hasara neden olabilir.

#### **`/sys/kernel/vmcoreinfo`**

* Kernel adreslerini sızdırır, KASLR'ı tehlikeye atabilir.

#### **`/sys/kernel/security`**

* `securityfs` arayüzünü barındırır, AppArmor gibi Linux Güvenlik Modülleri'nin yapılandırılmasına izin verir.
* Erişim, bir konteynerin MAC sistemini devre dışı bırakmasına olanak tanıyabilir.

#### **`/sys/firmware/efi/vars` ve `/sys/firmware/efi/efivars`**

* NVRAM'daki EFI değişkenleriyle etkileşim için arayüzler sunar.
* Yanlış yapılandırma veya kötüye kullanım, tuğla gibi olan dizüstü bilgisayarlar veya başlatılamayan ana bilgisayar makinelerine yol açabilir.

#### **`/sys/kernel/debug`**

* `debugfs`, çekirdeğe "kurallar olmadan" hata ayıklama arayüzü sunar.
* Sınırsız doğası nedeniyle güvenlik sorunları geçmişi bulunmaktadır.

### Referanslar

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
AWS Hacking'ı öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'ı öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi Twitter'da takip edin 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* Hacking püf noktalarını paylaşarak PR'ler göndererek **HackTricks** ve **HackTricks Cloud** github depolarına katkıda bulunun.

</details>
{% endhint %}
