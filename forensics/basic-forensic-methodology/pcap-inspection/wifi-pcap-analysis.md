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


# Proverite BSSID-e

Kada primite snimak čiji je glavni saobraćaj Wifi koristeći WireShark, možete početi da istražujete sve SSID-e snimka sa _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## Brute Force

Jedna od kolona na toj ekranu pokazuje da li je **bilo kakva autentifikacija pronađena unutar pcap-a**. Ako je to slučaj, možete pokušati da je brute force-ujete koristeći `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
For example it will retrieve the WPA passphrase protecting a PSK (pre shared-key), that will be required to decrypt the trafic later.

# Data in Beacons / Side Channel

If you suspect that **podaci se curi unutar beacon-a Wifi mreže** you can check the beacons of the network using a filter like the following one: `wlan contains <NAMEofNETWORK>`, or `wlan.ssid == "NAMEofNETWORK"` search inside the filtered packets for suspicious strings.

# Find Unknown MAC Addresses in A Wifi Network

The following link will be useful to find the **mašine koje šalju podatke unutar Wifi mreže**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

If you already know **MAC adrese možete ih ukloniti iz izlaza** adding checks like this one: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Once you have detected **nepoznate MAC** adrese communicating inside the network you can use **filters** like the following one: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)` to filter its traffic. Note that ftp/http/ssh/telnet filters are useful if you have decrypted the traffic.

# Decrypt Traffic

Edit --> Preferences --> Protocols --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (426).png>)





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
