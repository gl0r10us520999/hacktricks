# Applications Défensives macOS

{% hint style="success" %}
Apprenez et pratiquez le hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Formation HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Formation HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenir HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** nous sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

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

## Pare-feu

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html) : Il surveillera chaque connexion effectuée par chaque processus. Selon le mode (autoriser silencieusement les connexions, refuser silencieusement la connexion et alerter), il vous **affichera une alerte** chaque fois qu'une nouvelle connexion est établie. Il dispose également d'une très belle interface graphique pour voir toutes ces informations.
* [**LuLu**](https://objective-see.org/products/lulu.html) : Pare-feu d'Objective-See. C'est un pare-feu de base qui vous alertera pour des connexions suspectes (il a une interface graphique mais elle n'est pas aussi élégante que celle de Little Snitch).

## Détection de persistance

* [**KnockKnock**](https://objective-see.org/products/knockknock.html) : Application d'Objective-See qui recherchera à plusieurs endroits où **le malware pourrait persister** (c'est un outil à usage unique, pas un service de surveillance).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html) : Comme KnockKnock en surveillant les processus qui génèrent de la persistance.

## Détection de keyloggers

* [**ReiKey**](https://objective-see.org/products/reikey.html) : Application d'Objective-See pour trouver des **keyloggers** qui installent des "taps" d'événements clavier.
