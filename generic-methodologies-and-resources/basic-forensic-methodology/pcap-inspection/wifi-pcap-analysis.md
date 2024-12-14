# Wifi Pcap Analizi

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

## BSSID'leri Kontrol Edin

WireShark kullanarak Wifi'nın ana trafiğini içeren bir yakalama aldığınızda, _Wireless --> WLAN Traffic_ ile yakalamadaki tüm SSID'leri araştırmaya başlayabilirsiniz:

![](<../../../.gitbook/assets/image (106).png>)

![](<../../../.gitbook/assets/image (492).png>)

### Kaba Kuvvet

O ekranın sütunlarından biri, **pcap içinde herhangi bir kimlik doğrulamanın bulunup bulunmadığını** gösterir. Eğer durum buysa, `aircrack-ng` kullanarak kaba kuvvet denemesi yapabilirsiniz:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
For example it will retrieve the WPA passphrase protecting a PSK (pre shared-key), that will be required to decrypt the trafic later.

## Data in Beacons / Side Channel

If you suspect that **verilerin Wifi ağının beacon'larında sızdırıldığını** kontrol edebilirsiniz. Ağı kontrol etmek için aşağıdaki gibi bir filtre kullanabilirsiniz: `wlan contains <NAMEofNETWORK>`, veya `wlan.ssid == "NAMEofNETWORK"` filtrelenmiş paketler içinde şüpheli dizeleri arayın.

## Find Unknown MAC Addresses in A Wifi Network

The following link will be useful to find the **veri gönderen makineleri Wifi Ağında**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

If you already know **MAC adreslerini çıktıdan çıkarabilirsiniz** bu tür kontroller ekleyerek: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Once you have detected **bilinmeyen MAC** adresleri ağ içinde iletişim kurarken, **filtreler** kullanabilirsiniz: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)` trafiğini filtrelemek için. ftp/http/ssh/telnet filtrelerinin, trafiği şifrelediyseniz yararlı olduğunu unutmayın.

## Decrypt Traffic

Edit --> Preferences --> Protocols --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (499).png>)

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
