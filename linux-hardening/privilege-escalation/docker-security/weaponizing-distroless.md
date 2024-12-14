# Weaponizing Distroless

{% hint style="success" %}
Leer & oefen AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Leer & oefen GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Kyk na die [**subskripsie planne**](https://github.com/sponsors/carlospolop)!
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel hacking truuks deur PRs in te dien na die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Wat is Distroless

'n Distroless-container is 'n tipe container wat **slegs die nodige afhanklikhede bevat om 'n spesifieke toepassing te laat loop**, sonder enige addisionele sagteware of gereedskap wat nie benodig word nie. Hierdie houers is ontwerp om so **liggewig** en **veilig** as moontlik te wees, en hulle poog om die **aanvaloppervlak te minimaliseer** deur enige onnodige komponente te verwyder.

Distroless-houers word dikwels in **produksie-omgewings gebruik waar sekuriteit en betroubaarheid van die grootste belang is**.

Sommige **voorbeelde** van **distroless-houers** is:

* Verskaf deur **Google**: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* Verskaf deur **Chainguard**: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## Weaponizing Distroless

Die doel van die wapen van 'n distroless-container is om in staat te wees om **arbitraire binaire en payloads uit te voer selfs met die beperkings** wat deur **distroless** impliseer (gebrek aan algemene binaire in die stelsel) en ook beskermings wat algemeen in houers voorkom soos **lees-slegs** of **geen-uitvoering** in `/dev/shm`.

### Deur geheue

Kom op 'n sekere punt in 2023...

### Via Bestaande binaire

#### openssl

****[**In hierdie pos,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) word verduidelik dat die binaire **`openssl`** gereeld in hierdie houers gevind word, moontlik omdat dit **benodig** word deur die sagteware wat binne die houer gaan loop.
