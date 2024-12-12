# SmbExec/ScExec

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

## Nasıl Çalışır

**Smbexec**, Windows sistemlerinde uzaktan komut yürütmek için kullanılan bir araçtır, **Psexec**'e benzer, ancak hedef sistemde herhangi bir kötü amaçlı dosya bırakmaktan kaçınır.

### **SMBExec** Hakkında Ana Noktalar

- Hedef makinede cmd.exe (%COMSPEC%) aracılığıyla komutları yürütmek için geçici bir hizmet (örneğin, "BTOBTO") oluşturarak çalışır, herhangi bir ikili dosya bırakmaz.
- Gizli yaklaşımına rağmen, yürütülen her komut için olay günlükleri oluşturur ve etkileşimsiz bir "shell" biçimi sunar.
- **Smbexec** kullanarak bağlanma komutu şu şekildedir:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Binaries Olmadan Komut Çalıştırma

- **Smbexec**, hedefte fiziksel binary'lere ihtiyaç duymadan, hizmet binPath'leri aracılığıyla doğrudan komut yürütmeyi sağlar.
- Bu yöntem, bir Windows hedefinde tek seferlik komutlar yürütmek için faydalıdır. Örneğin, Metasploit'in `web_delivery` modülü ile eşleştirildiğinde, PowerShell hedefli ters Meterpreter yükünü çalıştırmak mümkündür.
- Saldırganın makinesinde binPath'i cmd.exe aracılığıyla sağlanan komutu çalıştıracak şekilde ayarlayarak uzaktan bir hizmet oluşturmak, yükü başarıyla çalıştırmak ve Metasploit dinleyicisi ile geri çağırma ve yük yürütme sağlamak mümkündür; bu, hizmet yanıt hataları olsa bile geçerlidir.

### Komutlar Örneği

Hizmeti oluşturmak ve başlatmak aşağıdaki komutlarla gerçekleştirilebilir:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
Daha fazla detay için [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/) adresini kontrol edin.


## Referanslar
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

{% hint style="success" %}
AWS Hacking öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
