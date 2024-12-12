# Extensions Système macOS

{% hint style="success" %}
Apprenez et pratiquez le piratage AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Formation HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le piratage GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Formation HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenez HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop)!
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
{% endhint %}

## Extensions Système / Cadre de Sécurité des Points d'Extrémité

Contrairement aux Extensions Noyau, les **Extensions Système s'exécutent dans l'espace utilisateur** au lieu de l'espace noyau, réduisant ainsi le risque de plantage du système en cas de dysfonctionnement de l'extension.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Il existe trois types d'extensions système : les Extensions **DriverKit**, les Extensions **Réseau** et les Extensions de **Sécurité des Points d'Extrémité**.

### **Extensions DriverKit**

DriverKit est un remplacement des extensions noyau qui **fournissent un support matériel**. Il permet aux pilotes de périphériques (comme les pilotes USB, série, NIC et HID) de s'exécuter dans l'espace utilisateur plutôt que dans l'espace noyau. Le framework DriverKit inclut des **versions en espace utilisateur de certaines classes I/O Kit**, et le noyau transfère les événements normaux de l'I/O Kit vers l'espace utilisateur, offrant ainsi un environnement plus sûr pour l'exécution de ces pilotes.

### **Extensions Réseau**

Les Extensions Réseau offrent la possibilité de personnaliser les comportements réseau. Il existe plusieurs types d'Extensions Réseau :

* **Proxy d'Application** : Cela est utilisé pour créer un client VPN qui implémente un protocole VPN personnalisé orienté flux. Cela signifie qu'il gère le trafic réseau en fonction des connexions (ou flux) plutôt que des paquets individuels.
* **Tunnel de Paquets** : Cela est utilisé pour créer un client VPN qui implémente un protocole VPN personnalisé orienté paquet. Cela signifie qu'il gère le trafic réseau en fonction des paquets individuels.
* **Filtrer les Données** : Cela est utilisé pour filtrer les "flux" réseau. Il peut surveiller ou modifier les données réseau au niveau du flux.
* **Filtrer les Paquets** : Cela est utilisé pour filtrer les paquets réseau individuels. Il peut surveiller ou modifier les données réseau au niveau du paquet.
* **Proxy DNS** : Cela est utilisé pour créer un fournisseur DNS personnalisé. Il peut être utilisé pour surveiller ou modifier les requêtes et réponses DNS.

## Cadre de Sécurité des Points d'Extrémité

La Sécurité des Points d'Extrémité est un cadre fourni par Apple dans macOS qui propose un ensemble d'API pour la sécurité du système. Il est destiné à être utilisé par les **fournisseurs de sécurité et les développeurs pour construire des produits capables de surveiller et contrôler l'activité du système** afin d'identifier et de se protéger contre les activités malveillantes.

Ce cadre fournit une **collection d'API pour surveiller et contrôler l'activité du système**, telle que l'exécution des processus, les événements du système de fichiers, les événements réseau et noyau.

Le cœur de ce cadre est implémenté dans le noyau, en tant qu'Extension Noyau (KEXT) située à **`/System/Library/Extensions/EndpointSecurity.kext`**. Cette KEXT est composée de plusieurs composants clés :

* **EndpointSecurityDriver** : Il agit comme le "point d'entrée" de l'extension noyau. C'est le principal point d'interaction entre le système d'exploitation et le cadre de Sécurité des Points d'Extrémité.
* **EndpointSecurityEventManager** : Ce composant est responsable de la mise en œuvre des crochets noyau. Les crochets noyau permettent au cadre de surveiller les événements système en interceptant les appels système.
* **EndpointSecurityClientManager** : Il gère la communication avec les clients en espace utilisateur, en suivant les clients connectés et nécessitant de recevoir des notifications d'événements.
* **EndpointSecurityMessageManager** : Il envoie des messages et des notifications d'événements aux clients en espace utilisateur.

Les événements que le cadre de Sécurité des Points d'Extrémité peut surveiller sont catégorisés en :

* Événements de fichiers
* Événements de processus
* Événements de socket
* Événements noyau (comme le chargement/déchargement d'une extension noyau ou l'ouverture d'un périphérique I/O Kit)

### Architecture du Cadre de Sécurité des Points d'Extrémité

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

La **communication en espace utilisateur** avec le cadre de Sécurité des Points d'Extrémité se fait via la classe IOUserClient. Deux sous-classes différentes sont utilisées, en fonction du type d'appelant :

* **EndpointSecurityDriverClient** : Cela nécessite l'autorisation `com.apple.private.endpoint-security.manager`, détenue uniquement par le processus système `endpointsecurityd`.
* **EndpointSecurityExternalClient** : Cela nécessite l'autorisation `com.apple.developer.endpoint-security.client`. Cela serait généralement utilisé par des logiciels de sécurité tiers qui doivent interagir avec le cadre de Sécurité des Points d'Extrémité.

Les Extensions de Sécurité des Points d'Extrémité : **`libEndpointSecurity.dylib`** est la bibliothèque C que les extensions système utilisent pour communiquer avec le noyau. Cette bibliothèque utilise l'I/O Kit (`IOKit`) pour communiquer avec la KEXT de Sécurité des Points d'Extrémité.

**`endpointsecurityd`** est un démon système clé impliqué dans la gestion et le lancement des extensions système de sécurité des points d'extrémité, en particulier pendant le processus de démarrage initial. Seules les extensions système marquées avec **`NSEndpointSecurityEarlyBoot`** dans leur fichier `Info.plist` reçoivent ce traitement de démarrage anticipé.

Un autre démon système, **`sysextd`**, **valide les extensions système** et les déplace dans les emplacements système appropriés. Il demande ensuite au démon pertinent de charger l'extension. Le **`SystemExtensions.framework`** est responsable de l'activation et de la désactivation des extensions système.

## Contourner l'ESF

L'ESF est utilisé par des outils de sécurité qui tenteront de détecter un membre de l'équipe rouge, donc toute information sur la manière dont cela pourrait être évité semble intéressante.

### CVE-2021-30965

Le fait est que l'application de sécurité doit avoir les **autorisations d'Accès Complet au Disque**. Donc, si un attaquant pouvait les supprimer, il pourrait empêcher le logiciel de s'exécuter :
```bash
tccutil reset All
```
Pour **plus d'informations** sur ce contournement et les contournements associés, consultez la présentation [#OBTS v5.0 : "Le talon d'Achille de la sécurité des points de terminaison" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

À la fin, cela a été corrigé en donnant la nouvelle autorisation **`kTCCServiceEndpointSecurityClient`** à l'application de sécurité gérée par **`tccd`** afin que `tccutil` ne supprime pas ses autorisations, l'empêchant ainsi de s'exécuter.

## Références

* [**OBTS v3.0 : "Sécurité et insécurité des points de terminaison" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Apprenez et pratiquez le piratage AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Formation HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le piratage GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Formation HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenez HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop)!
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** nous sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
