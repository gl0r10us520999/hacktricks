# macOS Defensive Apps

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
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}
{% endhint %}

## Firewalls

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): Pratiće svaku vezu koju uspostavi svaki proces. U zavisnosti od režima (tiho dozvoliti veze, tiho odbiti vezu i upozoriti) **pokazaće vam upozorenje** svaki put kada se uspostavi nova veza. Takođe ima veoma lepu GUI za pregled svih ovih informacija.
* [**LuLu**](https://objective-see.org/products/lulu.html): Objective-See vatrozid. Ovo je osnovni vatrozid koji će vas upozoriti na sumnjive veze (ima GUI, ali nije tako sofisticiran kao onaj kod Little Snitch).

## Persistence detection

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Objective-See aplikacija koja će pretraživati na nekoliko lokacija gde **malver može da se zadrži** (to je alat za jednokratnu upotrebu, nije servis za praćenje).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): Kao KnockKnock, prati procese koji generišu postojanost.

## Keyloggers detection

* [**ReiKey**](https://objective-see.org/products/reikey.html): Objective-See aplikacija za pronalaženje **keylogger-a** koji instaliraju "event taps" za tastaturu.
