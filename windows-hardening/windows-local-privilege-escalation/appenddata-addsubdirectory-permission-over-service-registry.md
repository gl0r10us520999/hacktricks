**Orijinal gönderi** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## Özet

Mevcut kullanıcı tarafından yazılabilir iki kayıt defteri anahtarı bulundu:

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

**RpcEptMapper** hizmetinin izinlerini **regedit GUI** kullanarak kontrol etmenin önerildiği belirtildi, özellikle **Gelişmiş Güvenlik Ayarları** penceresinin **Etkin İzinler** sekmesi. Bu yaklaşım, her Erişim Kontrol Girişi'ni (ACE) ayrı ayrı incelemeden belirli kullanıcılar veya gruplar için verilen izinlerin değerlendirilmesine olanak tanır.

Bir ekran görüntüsü, düşük ayrıcalıklı bir kullanıcıya atanan izinleri gösterdi; bunlar arasında **Alt Anahtar Oluştur** izni dikkat çekiciydi. Bu izin, **AppendData/AddSubdirectory** olarak da adlandırılmakta olup, scriptin bulgularıyla örtüşmektedir.

Belirli değerleri doğrudan değiştirme yeteneğinin olmaması, ancak yeni alt anahtarlar oluşturma yeteneğinin bulunması kaydedildi. Öne çıkan bir örnek, **ImagePath** değerini değiştirme girişimiydi; bu girişim, erişim reddedildi mesajıyla sonuçlandı.

Bu sınırlamalara rağmen, **RpcEptMapper** hizmetinin kayıt defteri yapısında varsayılan olarak bulunmayan **Performance** alt anahtarını kullanma olasılığı aracılığıyla ayrıcalık yükseltme potansiyeli belirlendi. Bu, DLL kaydı ve performans izleme olanağı sağlayabilirdi.

**Performance** alt anahtarı ve performans izleme için kullanımı hakkında belgeler incelendi ve bir kanıt konsepti DLL'si geliştirildi. Bu DLL, **OpenPerfData**, **CollectPerfData** ve **ClosePerfData** işlevlerinin uygulanmasını göstererek **rundll32** aracılığıyla test edildi ve başarılı bir şekilde çalıştığı doğrulandı.

Amaç, **RPC Endpoint Mapper hizmetini** oluşturulan Performans DLL'sini yüklemeye zorlamaktı. Gözlemler, PowerShell aracılığıyla Performans Verileri ile ilgili WMI sınıf sorgularının yürütülmesinin bir günlük dosyası oluşturduğunu ve böylece **LOCAL SYSTEM** bağlamında keyfi kod yürütülmesine olanak tanıdığını, bu durumun da yükseltilmiş ayrıcalıklar sağladığını ortaya koydu.

Bu güvenlik açığının kalıcılığı ve potansiyel etkileri vurgulandı; bu durum, istismar sonrası stratejiler, yan hareket ve antivirüs/EDR sistemlerinden kaçınma açısından önemini ortaya koydu.

Güvenlik açığının başlangıçta script aracılığıyla istemeden ifşa edildiği belirtilse de, istismarının yalnızca eski Windows sürümleriyle (örneğin, **Windows 7 / Server 2008 R2**) sınırlı olduğu ve yerel erişim gerektirdiği vurgulandı.
