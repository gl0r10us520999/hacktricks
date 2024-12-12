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


# GUI uygulaması içindeki olası eylemleri kontrol edin

**Ortak Diyaloglar**, **bir dosyayı kaydetme**, **bir dosyayı açma**, bir yazı tipi, bir renk seçme gibi seçeneklerdir... Bunların çoğu **tam bir Gezgini işlevselliği sunacaktır**. Bu, bu seçeneklere erişebiliyorsanız Gezgini işlevselliğine erişebileceğiniz anlamına gelir:

* Kapat/Kapat olarak
* Aç/Aç ile
* Yazdır
* Dışa Aktar/Içe Aktar
* Ara
* Tara

Şunları kontrol etmelisiniz:

* Dosyaları değiştirme veya yeni dosyalar oluşturma
* Sembolik bağlantılar oluşturma
* Kısıtlı alanlara erişim sağlama
* Diğer uygulamaları çalıştırma

## Komut Yürütme

Belki **`Aç ile`** seçeneğini kullanarak bazı türde bir shell açabilir/çalıştırabilirsiniz.

### Windows

Örneğin _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ komutları yürütmek için kullanılabilecek daha fazla ikili dosyayı burada bulabilirsiniz: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

### \*NIX __

_bash, sh, zsh..._ Daha fazla bilgi burada: [https://gtfobins.github.io/](https://gtfobins.github.io)

# Windows

## Yol kısıtlamalarını aşma

* **Ortam değişkenleri**: Bazı yollara işaret eden birçok ortam değişkeni vardır
* **Diğer protokoller**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Sembolik bağlantılar**
* **Kısayollar**: CTRL+N (yeni oturum aç), CTRL+R (Komutları Yürüt), CTRL+SHIFT+ESC (Görev Yöneticisi), Windows+E (gezgini aç), CTRL-B, CTRL-I (Favoriler), CTRL-H (Geçmiş), CTRL-L, CTRL-O (Dosya/Aç Diyaloğu), CTRL-P (Yazdırma Diyaloğu), CTRL-S (Farklı Kaydet)
* Gizli Yönetici menüsü: CTRL-ALT-F8, CTRL-ESC-F9
* **Shell URI'leri**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC yolları**: Paylaşılan klasörlere bağlanmak için yollar. Yerel makinenin C$'sine bağlanmayı denemelisiniz ("\\\127.0.0.1\c$\Windows\System32")
* **Daha fazla UNC yolu:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

## İkili Dosyalarınızı İndirin

Konsol: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Gezgini: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Kayıt defteri düzenleyici: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

## Tarayıcıdan dosya sistemine erişim

| PATH                | PATH              | PATH               | PATH                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

## Kısayollar

* Yapışkan Tuşlar – SHIFT'e 5 kez basın
* Fare Tuşları – SHIFT+ALT+NUMLOCK
* Yüksek Kontrast – SHIFT+ALT+PRINTSCN
* Anahtarları Değiştir – NUMLOCK'a 5 saniye basılı tutun
* Filtre Tuşları – sağ SHIFT'e 12 saniye basılı tutun
* WINDOWS+F1 – Windows Arama
* WINDOWS+D – Masaüstünü Göster
* WINDOWS+E – Windows Gezgini'ni Başlat
* WINDOWS+R – Çalıştır
* WINDOWS+U – Erişim Kolaylığı Merkezi
* WINDOWS+F – Ara
* SHIFT+F10 – Bağlam Menüsü
* CTRL+SHIFT+ESC – Görev Yöneticisi
* CTRL+ALT+DEL – Daha yeni Windows sürümlerinde açılış ekranı
* F1 – Yardım F3 – Ara
* F6 – Adres Çubuğu
* F11 – Internet Explorer'da tam ekranı aç/kapat
* CTRL+H – Internet Explorer Geçmişi
* CTRL+T – Internet Explorer – Yeni Sekme
* CTRL+N – Internet Explorer – Yeni Sayfa
* CTRL+O – Dosya Aç
* CTRL+S – Kaydet CTRL+N – Yeni RDP / Citrix

## Kaydırmalar

* Sol taraftan sağa kaydırarak tüm açık pencereleri görün, KIOSK uygulamasını küçültün ve tüm işletim sistemine doğrudan erişin;
* Sağ taraftan sola kaydırarak Eylem Merkezi'ni açın, KIOSK uygulamasını küçültün ve tüm işletim sistemine doğrudan erişin;
* Üst kenardan kaydırarak tam ekran modunda açılan bir uygulamanın başlık çubuğunu görünür hale getirin;
* Aşağıdan yukarı kaydırarak tam ekran uygulamasında görev çubuğunu gösterin.

## Internet Explorer İpuçları

### 'Resim Araç Çubuğu'

Tıklandığında resmin sol üst kısmında beliren bir araç çubuğudur. Kaydetme, Yazdırma, Mailto, "Resimlerim"i Gezginde açma işlemlerini yapabileceksiniz. Kiosk'un Internet Explorer kullanması gerekir.

### Shell Protokolü

Bir Gezgini görünümü elde etmek için bu URL'leri yazın:

* `shell:Administrative Tools`
* `shell:DocumentsLibrary`
* `shell:Libraries`
* `shell:UserProfiles`
* `shell:Personal`
* `shell:SearchHomeFolder`
* `shell:NetworkPlacesFolder`
* `shell:SendTo`
* `shell:UserProfiles`
* `shell:Common Administrative Tools`
* `shell:MyComputerFolder`
* `shell:InternetFolder`
* `Shell:Profile`
* `Shell:ProgramFiles`
* `Shell:System`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Denetim Masası
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Bilgisayarım
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Ağ Yerlerim
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

## Dosya Uzantılarını Göster

Daha fazla bilgi için bu sayfayı kontrol edin: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

# Tarayıcı ipuçları

Yedek iKat sürümleri:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\

JavaScript kullanarak ortak bir diyalog oluşturun ve dosya gezgini erişin: `document.write('<input/type=file>')`
Kaynak: https://medium.com/@Rend_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

# iPad

## Hareketler ve düğmeler

* Dört (veya beş) parmakla yukarı kaydırın / Ana düğmeye çift dokunun: Çoklu görev görünümünü görmek ve Uygulamayı değiştirmek için

* Dört veya beş parmakla bir yöne kaydırın: Bir sonraki/son uygulamaya geçmek için

* Beş parmakla ekranı sıkıştırın / Ana düğmeye dokunun / Ekranın altından yukarı doğru hızlı bir hareketle 1 parmakla kaydırın: Ana ekrana erişmek için

* Ekranın altından 1-2 inç (yavaş) kaydırın: Dock görünecektir

* Ekranın üst kısmından 1 parmakla aşağı kaydırın: Bildirimlerinizi görüntülemek için

* Ekranın sağ üst köşesinden 1 parmakla aşağı kaydırın: iPad Pro'nun kontrol merkezini görmek için

* Ekranın sol tarafından 1-2 inç kaydırın: Bugün görünümünü görmek için

* Ekranın ortasından sağa veya sola hızlı bir şekilde 1 parmakla kaydırın: Bir sonraki/son uygulamaya geçmek için

* **iPad'in sağ üst köşesindeki Açma/Kapama/Uyku düğmesine basılı tutun** / Güç kapalı kaydırıcısını tamamen sağa kaydırın: Kapatmak için

* **iPad'in sağ üst köşesindeki Açma/Kapama/Uyku düğmesine ve Ana düğmeye birkaç saniye basın**: Zorla kapatma yapmak için

* **iPad'in sağ üst köşesindeki Açma/Kapama/Uyku düğmesine ve Ana düğmeye hızlıca basın**: Ekranın sol alt kısmında belirecek bir ekran görüntüsü almak için. Her iki düğmeye aynı anda çok kısa bir süre basın, birkaç saniye basılı tutarsanız zorla kapatma yapılır.

## Kısayollar

Bir iPad klavyesine veya bir USB klavye adaptörüne sahip olmalısınız. Uygulamadan kaçmaya yardımcı olabilecek yalnızca kısayollar burada gösterilecektir.

| Tuş | Adı         |
| --- | ------------ |
| ⌘   | Komut      |
| ⌥   | Seçenek (Alt) |
| ⇧   | Shift        |
| ↩   | Geri       |
| ⇥   | Sekme      |
| ^   | Kontrol      |
| ←   | Sol Ok     |
| →   | Sağ Ok     |
| ↑   | Yukarı Ok  |
| ↓   | Aşağı Ok   |

### Sistem kısayolları

Bu kısayollar, iPad'in kullanımına bağlı olarak görsel ayarlar ve ses ayarları içindir.

| Kısayol | Eylem                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Ekranı karart                                                                    |
| F2       | Ekranı aydınlat                                                                |
| F7       | Bir şarkıyı geri al                                                              |
| F8       | Oynat/durdur                                                                     |
| F9       | Şarkıyı at                                                                      |
| F10      | Ses kapalı                                                                       |
| F11      | Ses seviyesini azalt                                                            |
| F12      | Ses seviyesini artır                                                            |
| ⌘ Boşluk  | Mevcut dillerin listesini görüntüle; birini seçmek için boşluk tuşuna tekrar dokunun. |

### iPad navigasyonu

| Kısayol                                           | Eylem                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Ana ekrana git                                          |
| ⌘⇧H (Komut-Shift-H)                              | Ana ekrana git                                          |
| ⌘ (Boşluk)                                        | Spotlight'ı aç                                          |
| ⌘⇥ (Komut-Sekme)                                   | Son on kullanılan uygulamayı listele                    |
| ⌘\~                                                | Son uygulamaya git                                      |
| ⌘⇧3 (Komut-Shift-3)                              | Ekran görüntüsü (sol altta kaydetmek veya üzerinde işlem yapmak için) |
| ⌘⇧4                                                | Ekran görüntüsü al ve düzenleyicide aç                 |
| ⌘'e basılı tutun                                   | Uygulama için mevcut kısayolların listesini göster      |
| ⌘⌥D (Komut-Seçenek/Alt-D)                         | Dock'u açar                                            |
| ^⌥H (Kontrol-Seçenek-H)                             | Ana düğme                                             |
| ^⌥H H (Kontrol-Seçenek-H-H)                         | Çoklu görev çubuğunu göster                              |
| ^⌥I (Kontrol-Seçenek-i)                             | Öğe seçici                                            |
| Escape                                             | Geri düğmesi                                          |
| → (Sağ ok)                                        | Sonraki öğe                                           |
| ← (Sol ok)                                       | Önceki öğe                                           |
| ↑↓ (Yukarı ok, Aşağı ok)                          | Seçilen öğeye aynı anda dokunun                        |
| ⌥ ↓ (Seçenek-Aşağı ok)                            | Aşağı kaydır                                          |
| ⌥↑ (Seçenek-Yukarı ok)                           | Yukarı kaydır                                         |
| ⌥← veya ⌥→ (Seçenek-Sol ok veya Seçenek-Sağ ok) | Sola veya sağa kaydır                                  |
| ^⌥S (Kontrol-Seçenek-S)                           | VoiceOver konuşmasını aç veya kapat                    |
| ⌘⇧⇥ (Komut-Shift-Sekme)                            | Önceki uygulamaya geç                                   |
| ⌘⇥ (Komut-Sekme)                                   | Orijinal uygulamaya geri dön                            |
| ←+→, ardından Seçenek + ← veya Seçenek+→         | Dock'ta gezin                                         |

### Safari kısayolları

| Kısayol                | Eylem                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Komut-L)          | Konum Aç                                        |
| ⌘T                      | Yeni bir sekme aç                               |
| ⌘W                      | Mevcut sekmeyi kapat                            |
| ⌘R                      | Mevcut sekmeyi yenile                          |
| ⌘.                      | Mevcut sekmenin yüklenmesini durdur            |
| ^⇥                      | Sonraki sekmeye geç                             |
| ^⇧⇥ (Kontrol-Shift-Sekme) | Önceki sekmeye geç                             |
| ⌘L                      | Metin girişi/URL alanını seçip düzenlemek için  |
| ⌘⇧T (Komut-Shift-T)   | En son kapatılan sekmeyi aç (birkaç kez kullanılabilir) |
| ⌘\[                     | Tarayıcı geçmişinizde bir sayfaya geri dön      |
| ⌘]                      | Tarayıcı geçmişinizde bir sayfaya ileri git     |
| ⌘⇧R                     | Okuyucu Modunu etkinleştir                      |

### Mail kısayolları

| Kısayol                   | Eylem                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | Konum Aç                    |
| ⌘T                         | Yeni bir sekme aç           |
| ⌘W                         | Mevcut sekmeyi kapat        |
| ⌘R                         | Mevcut sekmeyi yenile      |
| ⌘.                         | Mevcut sekmenin yüklenmesini durdur |
| ⌘⌥F (Komut-Seçenek/Alt-F) | Posta kutunuzda arama yap  |

# Referanslar

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)


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
