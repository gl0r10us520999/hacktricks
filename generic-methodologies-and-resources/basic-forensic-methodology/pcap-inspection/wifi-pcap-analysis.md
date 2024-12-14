# Análise de Pcap Wifi

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}

## Verificar BSSIDs

Quando você recebe uma captura cujo tráfego principal é Wifi usando WireShark, você pode começar a investigar todos os SSIDs da captura com _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (106).png>)

![](<../../../.gitbook/assets/image (492).png>)

### Força Bruta

Uma das colunas daquela tela indica se **alguma autenticação foi encontrada dentro do pcap**. Se esse for o caso, você pode tentar forçar a autenticação usando `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
Por exemplo, ele irá recuperar a senha WPA que protege uma PSK (chave pré-compartilhada), que será necessária para descriptografar o tráfego mais tarde.

## Dados em Beacons / Canal Lateral

Se você suspeitar que **dados estão sendo vazados dentro dos beacons de uma rede Wifi**, você pode verificar os beacons da rede usando um filtro como o seguinte: `wlan contains <NAMEofNETWORK>`, ou `wlan.ssid == "NAMEofNETWORK"` e procurar dentro dos pacotes filtrados por strings suspeitas.

## Encontrar Endereços MAC Desconhecidos em uma Rede Wifi

O seguinte link será útil para encontrar as **máquinas enviando dados dentro de uma Rede Wifi**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Se você já conhece **endereços MAC, pode removê-los da saída** adicionando verificações como esta: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Uma vez que você tenha detectado **endereços MAC desconhecidos** se comunicando dentro da rede, você pode usar **filtros** como o seguinte: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)` para filtrar seu tráfego. Note que os filtros ftp/http/ssh/telnet são úteis se você tiver descriptografado o tráfego.

## Descriptografar Tráfego

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
