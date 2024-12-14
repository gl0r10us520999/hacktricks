# SmbExec/ScExec

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** bizi **takip edin** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**Web uygulamalarınız, ağınız ve bulutunuz hakkında bir hacker perspektifi edinin**

**Gerçek iş etkisi olan kritik, istismar edilebilir güvenlik açıklarını bulun ve raporlayın.** Saldırı yüzeyini haritalamak, ayrıcalıkları artırmanıza izin veren güvenlik sorunlarını bulmak ve temel kanıtları toplamak için otomatik istismarları kullanmak için 20'den fazla özel aracımızı kullanın, böylece sıkı çalışmanızı ikna edici raporlara dönüştürün.

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

## Nasıl Çalışır

**Smbexec**, Windows sistemlerinde uzaktan komut yürütmek için kullanılan bir araçtır, **Psexec**'e benzer, ancak hedef sistemde herhangi bir kötü amaçlı dosya bırakmaktan kaçınır.

### **SMBExec** Hakkında Ana Noktalar

- Hedef makinede cmd.exe (%COMSPEC%) aracılığıyla komutları yürütmek için geçici bir hizmet (örneğin, "BTOBTO") oluşturarak çalışır, herhangi bir ikili dosya bırakmaz.
- Gizli yaklaşımına rağmen, yürütülen her komut için olay günlükleri oluşturur ve etkileşimsiz bir "shell" biçimi sunar.
- **Smbexec** kullanarak bağlanma komutu şu şekildedir:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Binariesiz Komut Çalıştırma

- **Smbexec**, hedefte fiziksel ikili dosyalar olmaksızın hizmet binPath'leri aracılığıyla doğrudan komut yürütmeyi sağlar.
- Bu yöntem, bir Windows hedefinde tek seferlik komutlar yürütmek için faydalıdır. Örneğin, Metasploit'in `web_delivery` modülü ile birleştirildiğinde, PowerShell hedefli ters Meterpreter yükünün yürütülmesini sağlar.
- Saldırganın makinesinde binPath'i cmd.exe aracılığıyla sağlanan komutu çalıştıracak şekilde ayarlayarak uzaktan bir hizmet oluşturmak, yükün başarılı bir şekilde yürütülmesini sağlamakta, Metasploit dinleyicisi ile geri çağırma ve yük yürütme gerçekleştirmekte, hizmet yanıt hataları olsa bile mümkündür.

### Komut Örneği

Hizmeti oluşturmak ve başlatmak aşağıdaki komutlarla gerçekleştirilebilir:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
FOr further details check [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## References
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<figure><img src="/.gitbook/assets/pentest-tools.svg" alt=""><figcaption></figcaption></figure>

**Web uygulamalarınız, ağınız ve bulutunuz hakkında bir hacker perspektifi edinin**

**Gerçek iş etkisi olan kritik, istismar edilebilir güvenlik açıklarını bulun ve raporlayın.** Saldırı yüzeyini haritalamak, ayrıcalıkları artırmanıza izin veren güvenlik sorunlarını bulmak ve temel kanıtları toplamak için otomatik istismarları kullanmak için 20'den fazla özel aracımızı kullanın, sıkı çalışmanızı ikna edici raporlara dönüştürün.

{% embed url="https://pentest-tools.com/?utm_term=jul2024&utm_medium=link&utm_source=hacktricks&utm_campaign=spons" %}

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** bizi takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
