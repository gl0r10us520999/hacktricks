# macOS System Extensions

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

## System Extensions / Cadre de Sécurité des Points de Terminaison

Contrairement aux extensions du noyau, **les extensions système s'exécutent dans l'espace utilisateur** plutôt que dans l'espace noyau, réduisant ainsi le risque d'un crash système dû à un dysfonctionnement de l'extension.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Il existe trois types d'extensions système : **DriverKit** Extensions, **Network** Extensions, et **Endpoint Security** Extensions.

### **DriverKit Extensions**

DriverKit est un remplacement pour les extensions du noyau qui **fournissent un support matériel**. Il permet aux pilotes de périphériques (comme les pilotes USB, Série, NIC, et HID) de s'exécuter dans l'espace utilisateur plutôt que dans l'espace noyau. Le cadre DriverKit inclut **des versions en espace utilisateur de certaines classes de l'I/O Kit**, et le noyau transmet les événements normaux de l'I/O Kit à l'espace utilisateur, offrant un environnement plus sûr pour l'exécution de ces pilotes.

### **Network Extensions**

Les extensions réseau offrent la possibilité de personnaliser les comportements réseau. Il existe plusieurs types d'extensions réseau :

* **App Proxy** : Cela est utilisé pour créer un client VPN qui implémente un protocole VPN personnalisé orienté flux. Cela signifie qu'il gère le trafic réseau en fonction des connexions (ou flux) plutôt qu'en fonction des paquets individuels.
* **Packet Tunnel** : Cela est utilisé pour créer un client VPN qui implémente un protocole VPN personnalisé orienté paquet. Cela signifie qu'il gère le trafic réseau en fonction des paquets individuels.
* **Filter Data** : Cela est utilisé pour filtrer les "flux" réseau. Il peut surveiller ou modifier les données réseau au niveau du flux.
* **Filter Packet** : Cela est utilisé pour filtrer les paquets réseau individuels. Il peut surveiller ou modifier les données réseau au niveau du paquet.
* **DNS Proxy** : Cela est utilisé pour créer un fournisseur DNS personnalisé. Il peut être utilisé pour surveiller ou modifier les requêtes et réponses DNS.

## Cadre de Sécurité des Points de Terminaison

La sécurité des points de terminaison est un cadre fourni par Apple dans macOS qui offre un ensemble d'APIs pour la sécurité système. Il est destiné à être utilisé par **des fournisseurs de sécurité et des développeurs pour créer des produits qui peuvent surveiller et contrôler l'activité système** afin d'identifier et de protéger contre les activités malveillantes.

Ce cadre fournit une **collection d'APIs pour surveiller et contrôler l'activité système**, telles que les exécutions de processus, les événements du système de fichiers, les événements réseau et noyau.

Le cœur de ce cadre est implémenté dans le noyau, en tant qu'extension du noyau (KEXT) située à **`/System/Library/Extensions/EndpointSecurity.kext`**. Ce KEXT est composé de plusieurs composants clés :

* **EndpointSecurityDriver** : Cela agit comme le "point d'entrée" pour l'extension du noyau. C'est le principal point d'interaction entre le système d'exploitation et le cadre de sécurité des points de terminaison.
* **EndpointSecurityEventManager** : Ce composant est responsable de l'implémentation des hooks du noyau. Les hooks du noyau permettent au cadre de surveiller les événements système en interceptant les appels système.
* **EndpointSecurityClientManager** : Cela gère la communication avec les clients en espace utilisateur, en gardant une trace des clients connectés et de ceux qui doivent recevoir des notifications d'événements.
* **EndpointSecurityMessageManager** : Cela envoie des messages et des notifications d'événements aux clients en espace utilisateur.

Les événements que le cadre de sécurité des points de terminaison peut surveiller sont classés en :

* Événements de fichiers
* Événements de processus
* Événements de socket
* Événements du noyau (comme le chargement/déchargement d'une extension du noyau ou l'ouverture d'un périphérique de l'I/O Kit)

### Architecture du Cadre de Sécurité des Points de Terminaison

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**La communication en espace utilisateur** avec le cadre de sécurité des points de terminaison se fait par l'intermédiaire de la classe IOUserClient. Deux sous-classes différentes sont utilisées, selon le type d'appelant :

* **EndpointSecurityDriverClient** : Cela nécessite l'autorisation `com.apple.private.endpoint-security.manager`, qui n'est détenue que par le processus système `endpointsecurityd`.
* **EndpointSecurityExternalClient** : Cela nécessite l'autorisation `com.apple.developer.endpoint-security.client`. Cela serait généralement utilisé par des logiciels de sécurité tiers qui ont besoin d'interagir avec le cadre de sécurité des points de terminaison.

Les extensions de sécurité des points de terminaison : **`libEndpointSecurity.dylib`** est la bibliothèque C que les extensions système utilisent pour communiquer avec le noyau. Cette bibliothèque utilise l'I/O Kit (`IOKit`) pour communiquer avec le KEXT de sécurité des points de terminaison.

**`endpointsecurityd`** est un démon système clé impliqué dans la gestion et le lancement des extensions système de sécurité des points de terminaison, en particulier pendant le processus de démarrage précoce. **Seules les extensions système** marquées avec **`NSEndpointSecurityEarlyBoot`** dans leur fichier `Info.plist` reçoivent ce traitement de démarrage précoce.

Un autre démon système, **`sysextd`**, **valide les extensions système** et les déplace dans les emplacements système appropriés. Il demande ensuite au démon pertinent de charger l'extension. Le **`SystemExtensions.framework`** est responsable de l'activation et de la désactivation des extensions système.

## Contournement de l'ESF

L'ESF est utilisé par des outils de sécurité qui essaieront de détecter un red teamer, donc toute information sur la façon dont cela pourrait être évité semble intéressante.

### CVE-2021-30965

Le fait est que l'application de sécurité doit avoir **des autorisations d'accès complet au disque**. Donc, si un attaquant pouvait supprimer cela, il pourrait empêcher le logiciel de fonctionner :
```bash
tccutil reset All
```
Pour **plus d'informations** sur ce contournement et ceux qui y sont liés, consultez la conférence [#OBTS v5.0 : "Le talon d'Achille de l'EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

À la fin, cela a été corrigé en donnant la nouvelle permission **`kTCCServiceEndpointSecurityClient`** à l'application de sécurité gérée par **`tccd`**, de sorte que `tccutil` ne supprimera pas ses permissions, empêchant son exécution.

## Références

* [**OBTS v3.0 : "Sécurité et insécurité des points de terminaison" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Apprenez et pratiquez le hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenir HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez-nous sur** **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PRs aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts github.

</details>
{% endhint %}
