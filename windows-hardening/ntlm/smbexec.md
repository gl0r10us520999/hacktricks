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

## Kako to funkcioniše

**Smbexec** je alat koji se koristi za daljinsko izvršavanje komandi na Windows sistemima, sličan **Psexec**, ali izbegava postavljanje bilo kojih zlonamernih datoteka na ciljni sistem.

### Ključne tačke o **SMBExec**

- Radi tako što kreira privremenu uslugu (na primer, "BTOBTO") na ciljnim mašinama kako bi izvršio komande putem cmd.exe (%COMSPEC%), bez ispuštanja bilo kakvih binarnih datoteka.
- I pored svog diskretnog pristupa, generiše logove događaja za svaku izvršenu komandu, nudeći oblik neinteraktivnog "shell-a".
- Komanda za povezivanje koristeći **Smbexec** izgleda ovako:
```bash
smbexec.py WORKGROUP/genericuser:genericpassword@10.10.10.10
```
### Izvršavanje Komandi Bez Binarnih Fajlova

- **Smbexec** omogućava direktno izvršavanje komandi kroz binPaths servisa, eliminišući potrebu za fizičkim binarnim fajlovima na meti.
- Ova metoda je korisna za izvršavanje jednokratnih komandi na Windows meti. Na primer, kombinovanjem sa Metasploit-ovim `web_delivery` modulom omogućava se izvršavanje PowerShell-targetiranog reverznog Meterpreter payload-a.
- Kreiranjem udaljenog servisa na napadačevoj mašini sa binPath postavljenim da pokrene pruženu komandu kroz cmd.exe, moguće je uspešno izvršiti payload, ostvarujući povratne informacije i izvršavanje payload-a sa Metasploit slušateljem, čak i ako dođe do grešaka u odgovoru servisa.

### Primer Komandi

Kreiranje i pokretanje servisa može se ostvariti sledećim komandama:
```bash
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
FOr further details check [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## References
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

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
