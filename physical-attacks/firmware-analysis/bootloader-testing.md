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

Aşağıdaki adımlar, U-boot gibi cihaz başlangıç yapılandırmalarını ve bootloader'ları değiştirmek için önerilmektedir:

1. **Bootloader'ın Yorumlayıcı Shell'ine Erişim**:
- Başlangıç sırasında "0", boşluk veya diğer tanımlanmış "sihirli kodlar"ı basarak bootloader'ın yorumlayıcı shell'ine erişin.

2. **Boot Argümanlarını Değiştirin**:
- Aşağıdaki komutları çalıştırarak '`init=/bin/sh`' ifadesini boot argümanlarına ekleyin, bu shell komutunun çalıştırılmasına izin verir:
%%%
#printenv
#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh
#saveenv
#boot
%%%

3. **TFTP Sunucusu Kurun**:
- Yerel bir ağ üzerinden görüntüleri yüklemek için bir TFTP sunucusu yapılandırın:
%%%
#setenv ipaddr 192.168.2.2 #cihazın yerel IP'si
#setenv serverip 192.168.2.1 #TFTP sunucu IP'si
#saveenv
#reset
#ping 192.168.2.1 #ağ erişimini kontrol et
#tftp ${loadaddr} uImage-3.6.35 #loadaddr, dosyanın yükleneceği adresi ve TFTP sunucusundaki görüntü dosyasının adını alır
%%%

4. **`ubootwrite.py` Kullanımı**:
- Root erişimi kazanmak için U-boot görüntüsünü yazmak ve değiştirilmiş bir firmware yüklemek için `ubootwrite.py` kullanın.

5. **Debug Özelliklerini Kontrol Edin**:
- Ayrıntılı günlükleme, rastgele çekirdek yükleme veya güvenilmeyen kaynaklardan başlatma gibi debug özelliklerinin etkin olup olmadığını doğrulayın.

6. **Dikkatli Donanım Müdahalesi**:
- Cihazın başlatma sırası sırasında bir pini toprağa bağlarken ve SPI veya NAND flash yongalarıyla etkileşimde bulunurken dikkatli olun, özellikle çekirdek açılmadan önce. Pinleri kısaltmadan önce NAND flash yongasının veri sayfasını kontrol edin.

7. **Sahte DHCP Sunucusu Yapılandırın**:
- PXE başlatma sırasında bir cihazın alması için kötü niyetli parametrelerle sahte bir DHCP sunucusu kurun. Metasploit'in (MSF) DHCP yardımcı sunucusu gibi araçları kullanın. 'FILENAME' parametresini `'a";/bin/sh;#'` gibi komut enjeksiyon komutlarıyla değiştirerek cihaz başlangıç prosedürleri için giriş doğrulamasını test edin.

**Not**: Cihaz pinleriyle fiziksel etkileşim içeren adımlar (*yıldızlarla işaretlenmiş) cihazın zarar görmesini önlemek için son derece dikkatli bir şekilde yaklaşılmalıdır.


## References
* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)


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
