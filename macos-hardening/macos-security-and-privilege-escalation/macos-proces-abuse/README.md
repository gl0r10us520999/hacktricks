# macOS Process Abuse

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

## Informations de base sur les processus

Un processus est une instance d'un exécutable en cours d'exécution, cependant les processus n'exécutent pas de code, ce sont des threads. Par conséquent, **les processus ne sont que des conteneurs pour des threads en cours d'exécution** fournissant la mémoire, des descripteurs, des ports, des permissions...

Traditionnellement, les processus étaient lancés dans d'autres processus (sauf le PID 1) en appelant **`fork`** qui créerait une copie exacte du processus actuel et ensuite le **processus enfant** appellerait généralement **`execve`** pour charger le nouvel exécutable et l'exécuter. Ensuite, **`vfork`** a été introduit pour rendre ce processus plus rapide sans aucune copie de mémoire.\
Ensuite, **`posix_spawn`** a été introduit combinant **`vfork`** et **`execve`** en un seul appel et acceptant des drapeaux :

* `POSIX_SPAWN_RESETIDS` : Réinitialiser les identifiants effectifs aux identifiants réels
* `POSIX_SPAWN_SETPGROUP` : Définir l'affiliation au groupe de processus
* `POSUX_SPAWN_SETSIGDEF` : Définir le comportement par défaut des signaux
* `POSIX_SPAWN_SETSIGMASK` : Définir le masque de signal
* `POSIX_SPAWN_SETEXEC` : Exécuter dans le même processus (comme `execve` avec plus d'options)
* `POSIX_SPAWN_START_SUSPENDED` : Démarrer suspendu
* `_POSIX_SPAWN_DISABLE_ASLR` : Démarrer sans ASLR
* `_POSIX_SPAWN_NANO_ALLOCATOR:` Utiliser l'allocateur Nano de libmalloc
* `_POSIX_SPAWN_ALLOW_DATA_EXEC:` Autoriser `rwx` sur les segments de données
* `POSIX_SPAWN_CLOEXEC_DEFAULT` : Fermer toutes les descriptions de fichiers sur exec(2) par défaut
* `_POSIX_SPAWN_HIGH_BITS_ASLR:` Randomiser les bits élevés du glissement ASLR

De plus, `posix_spawn` permet de spécifier un tableau de **`posix_spawnattr`** qui contrôle certains aspects du processus créé, et **`posix_spawn_file_actions`** pour modifier l'état des descripteurs.

Lorsqu'un processus meurt, il envoie le **code de retour au processus parent** (si le parent est mort, le nouveau parent est le PID 1) avec le signal `SIGCHLD`. Le parent doit obtenir cette valeur en appelant `wait4()` ou `waitid()` et jusqu'à ce que cela se produise, l'enfant reste dans un état zombie où il est toujours listé mais ne consomme pas de ressources.

### PIDs

Les PIDs, identifiants de processus, identifient un processus unique. Dans XNU, les **PIDs** sont de **64 bits** augmentant de manière monotone et **ne se réinitialisent jamais** (pour éviter les abus).

### Groupes de processus, sessions et coalitions

**Les processus** peuvent être insérés dans des **groupes** pour faciliter leur gestion. Par exemple, les commandes dans un script shell seront dans le même groupe de processus, il est donc possible de **les signaler ensemble** en utilisant kill par exemple.\
Il est également possible de **grouper des processus en sessions**. Lorsqu'un processus démarre une session (`setsid(2)`), les processus enfants sont placés à l'intérieur de la session, sauf s'ils démarrent leur propre session.

La coalition est une autre façon de regrouper des processus dans Darwin. Un processus rejoignant une coalition lui permet d'accéder à des ressources partagées, de partager un registre ou de faire face à Jetsam. Les coalitions ont différents rôles : Leader, service XPC, Extension.

### Identifiants et personnes

Chaque processus détient des **identifiants** qui **identifient ses privilèges** dans le système. Chaque processus aura un `uid` principal et un `gid` principal (bien qu'il puisse appartenir à plusieurs groupes).\
Il est également possible de changer l'identifiant utilisateur et l'identifiant de groupe si le binaire a le bit `setuid/setgid`.\
Il existe plusieurs fonctions pour **définir de nouveaux uids/gids**.

L'appel système **`persona`** fournit un ensemble **alternatif** d'**identifiants**. Adopter une personne suppose son uid, gid et les appartenances de groupe **en une seule fois**. Dans le [**code source**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h), il est possible de trouver la structure :
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## Informations de base sur les threads

1. **Threads POSIX (pthreads) :** macOS prend en charge les threads POSIX (`pthreads`), qui font partie d'une API de threading standard pour C/C++. L'implémentation de pthreads dans macOS se trouve dans `/usr/lib/system/libsystem_pthread.dylib`, qui provient du projet `libpthread` disponible publiquement. Cette bibliothèque fournit les fonctions nécessaires pour créer et gérer des threads.
2. **Création de threads :** La fonction `pthread_create()` est utilisée pour créer de nouveaux threads. En interne, cette fonction appelle `bsdthread_create()`, qui est un appel système de niveau inférieur spécifique au noyau XNU (le noyau sur lequel macOS est basé). Cet appel système prend divers drapeaux dérivés de `pthread_attr` (attributs) qui spécifient le comportement des threads, y compris les politiques de planification et la taille de la pile.
* **Taille de pile par défaut :** La taille de pile par défaut pour les nouveaux threads est de 512 Ko, ce qui est suffisant pour des opérations typiques mais peut être ajusté via les attributs de thread si plus ou moins d'espace est nécessaire.
3. **Initialisation des threads :** La fonction `__pthread_init()` est cruciale lors de la configuration des threads, utilisant l'argument `env[]` pour analyser les variables d'environnement qui peuvent inclure des détails sur l'emplacement et la taille de la pile.

#### Terminaison des threads dans macOS

1. **Sortie des threads :** Les threads sont généralement terminés en appelant `pthread_exit()`. Cette fonction permet à un thread de sortir proprement, effectuant le nettoyage nécessaire et permettant au thread d'envoyer une valeur de retour à tout rejoignant.
2. **Nettoyage des threads :** Lors de l'appel de `pthread_exit()`, la fonction `pthread_terminate()` est invoquée, qui gère la suppression de toutes les structures de thread associées. Elle désalloue les ports de thread Mach (Mach est le sous-système de communication dans le noyau XNU) et appelle `bsdthread_terminate`, un appel système qui supprime les structures de niveau noyau associées au thread.

#### Mécanismes de synchronisation

Pour gérer l'accès aux ressources partagées et éviter les conditions de course, macOS fournit plusieurs primitives de synchronisation. Celles-ci sont critiques dans les environnements multi-threading pour garantir l'intégrité des données et la stabilité du système :

1. **Mutex :**
* **Mutex régulier (Signature : 0x4D555458) :** Mutex standard avec une empreinte mémoire de 60 octets (56 octets pour le mutex et 4 octets pour la signature).
* **Mutex rapide (Signature : 0x4d55545A) :** Semblable à un mutex régulier mais optimisé pour des opérations plus rapides, également de 60 octets de taille.
2. **Variables de condition :**
* Utilisées pour attendre que certaines conditions se produisent, avec une taille de 44 octets (40 octets plus une signature de 4 octets).
* **Attributs de variable de condition (Signature : 0x434e4441) :** Attributs de configuration pour les variables de condition, d'une taille de 12 octets.
3. **Variable Once (Signature : 0x4f4e4345) :**
* Assure qu'un morceau de code d'initialisation est exécuté une seule fois. Sa taille est de 12 octets.
4. **Verrous de lecture-écriture :**
* Permet plusieurs lecteurs ou un écrivain à la fois, facilitant l'accès efficace aux données partagées.
* **Verrou de lecture-écriture (Signature : 0x52574c4b) :** Taille de 196 octets.
* **Attributs de verrou de lecture-écriture (Signature : 0x52574c41) :** Attributs pour les verrous de lecture-écriture, de 20 octets de taille.

{% hint style="success" %}
Les 4 derniers octets de ces objets sont utilisés pour détecter les débordements.
{% endhint %}

### Variables locales aux threads (TLV)

**Variables locales aux threads (TLV)** dans le contexte des fichiers Mach-O (le format pour les exécutables dans macOS) sont utilisées pour déclarer des variables qui sont spécifiques à **chaque thread** dans une application multi-threadée. Cela garantit que chaque thread a sa propre instance séparée d'une variable, fournissant un moyen d'éviter les conflits et de maintenir l'intégrité des données sans avoir besoin de mécanismes de synchronisation explicites comme les mutex.

En C et dans les langages connexes, vous pouvez déclarer une variable locale au thread en utilisant le mot-clé **`__thread`**. Voici comment cela fonctionne dans votre exemple :
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
Ce snippet définit `tlv_var` comme une variable locale à un thread. Chaque thread exécutant ce code aura son propre `tlv_var`, et les modifications qu'un thread apporte à `tlv_var` n'affecteront pas `tlv_var` dans un autre thread.

Dans le binaire Mach-O, les données liées aux variables locales à un thread sont organisées en sections spécifiques :

* **`__DATA.__thread_vars`** : Cette section contient les métadonnées sur les variables locales à un thread, comme leurs types et leur état d'initialisation.
* **`__DATA.__thread_bss`** : Cette section est utilisée pour les variables locales à un thread qui ne sont pas explicitement initialisées. C'est une partie de la mémoire réservée pour les données initialisées à zéro.

Mach-O fournit également une API spécifique appelée **`tlv_atexit`** pour gérer les variables locales à un thread lors de la sortie d'un thread. Cette API vous permet de **enregistrer des destructeurs**—des fonctions spéciales qui nettoient les données locales à un thread lorsque celui-ci se termine.

### Priorités de Thread

Comprendre les priorités des threads implique d'examiner comment le système d'exploitation décide quels threads exécuter et quand. Cette décision est influencée par le niveau de priorité attribué à chaque thread. Dans macOS et les systèmes de type Unix, cela est géré à l'aide de concepts tels que `nice`, `renice`, et les classes de Qualité de Service (QoS).

#### Nice et Renice

1. **Nice :**
* La valeur `nice` d'un processus est un nombre qui affecte sa priorité. Chaque processus a une valeur nice allant de -20 (la plus haute priorité) à 19 (la plus basse priorité). La valeur nice par défaut lors de la création d'un processus est généralement 0.
* Une valeur nice plus basse (plus proche de -20) rend un processus plus "égoïste", lui donnant plus de temps CPU par rapport à d'autres processus avec des valeurs nice plus élevées.
2. **Renice :**
* `renice` est une commande utilisée pour changer la valeur nice d'un processus déjà en cours d'exécution. Cela peut être utilisé pour ajuster dynamiquement la priorité des processus, soit en augmentant soit en diminuant leur allocation de temps CPU en fonction de nouvelles valeurs nice.
* Par exemple, si un processus a besoin de plus de ressources CPU temporairement, vous pourriez abaisser sa valeur nice en utilisant `renice`.

#### Classes de Qualité de Service (QoS)

Les classes QoS sont une approche plus moderne pour gérer les priorités des threads, en particulier dans des systèmes comme macOS qui supportent **Grand Central Dispatch (GCD)**. Les classes QoS permettent aux développeurs de **catégoriser** le travail en différents niveaux en fonction de leur importance ou urgence. macOS gère la priorisation des threads automatiquement en fonction de ces classes QoS :

1. **Interactif Utilisateur :**
* Cette classe est pour les tâches qui interagissent actuellement avec l'utilisateur ou nécessitent des résultats immédiats pour offrir une bonne expérience utilisateur. Ces tâches se voient attribuer la plus haute priorité pour maintenir l'interface réactive (par exemple, animations ou gestion d'événements).
2. **Initié par l'Utilisateur :**
* Tâches que l'utilisateur initie et attend des résultats immédiats, comme ouvrir un document ou cliquer sur un bouton nécessitant des calculs. Celles-ci ont une haute priorité mais sont en dessous de l'interactif utilisateur.
3. **Utilitaire :**
* Ces tâches sont de longue durée et montrent généralement un indicateur de progression (par exemple, téléchargement de fichiers, importation de données). Elles ont une priorité inférieure à celle des tâches initiées par l'utilisateur et n'ont pas besoin de se terminer immédiatement.
4. **Arrière-plan :**
* Cette classe est pour les tâches qui fonctionnent en arrière-plan et ne sont pas visibles pour l'utilisateur. Cela peut être des tâches comme l'indexation, la synchronisation ou les sauvegardes. Elles ont la plus basse priorité et un impact minimal sur les performances du système.

En utilisant les classes QoS, les développeurs n'ont pas besoin de gérer les numéros de priorité exacts mais plutôt de se concentrer sur la nature de la tâche, et le système optimise les ressources CPU en conséquence.

De plus, il existe différentes **politiques de planification des threads** qui spécifient un ensemble de paramètres de planification que le planificateur prendra en compte. Cela peut être fait en utilisant `thread_policy_[set/get]`. Cela pourrait être utile dans les attaques par condition de course.

## Abus de Processus MacOS

MacOS, comme tout autre système d'exploitation, fournit une variété de méthodes et de mécanismes pour **interagir, communiquer et partager des données** entre les processus. Bien que ces techniques soient essentielles pour le bon fonctionnement du système, elles peuvent également être abusées par des acteurs malveillants pour **effectuer des activités malveillantes**.

### Injection de Bibliothèque

L'injection de bibliothèque est une technique par laquelle un attaquant **force un processus à charger une bibliothèque malveillante**. Une fois injectée, la bibliothèque s'exécute dans le contexte du processus cible, fournissant à l'attaquant les mêmes permissions et accès que le processus.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Accrochage de Fonction

L'accrochage de fonction implique **d'intercepter les appels de fonction** ou les messages au sein d'un code logiciel. En accrochant des fonctions, un attaquant peut **modifier le comportement** d'un processus, observer des données sensibles, ou même prendre le contrôle du flux d'exécution.

{% content-ref url="macos-function-hooking.md" %}
[macos-function-hooking.md](macos-function-hooking.md)
{% endcontent-ref %}

### Communication Inter-Processus

La communication inter-processus (IPC) fait référence à différentes méthodes par lesquelles des processus séparés **partagent et échangent des données**. Bien que l'IPC soit fondamental pour de nombreuses applications légitimes, il peut également être mal utilisé pour subvertir l'isolation des processus, fuir des informations sensibles, ou effectuer des actions non autorisées.

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Injection d'Applications Electron

Les applications Electron exécutées avec des variables d'environnement spécifiques pourraient être vulnérables à l'injection de processus :

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Injection de Chromium

Il est possible d'utiliser les drapeaux `--load-extension` et `--use-fake-ui-for-media-stream` pour effectuer une **attaque de l'homme dans le navigateur** permettant de voler des frappes, du trafic, des cookies, d'injecter des scripts dans des pages... :

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### NIB Sale

Les fichiers NIB **définissent les éléments de l'interface utilisateur (UI)** et leurs interactions au sein d'une application. Cependant, ils peuvent **exécuter des commandes arbitraires** et **Gatekeeper ne bloque pas** une application déjà exécutée si un **fichier NIB est modifié**. Par conséquent, ils pourraient être utilisés pour faire exécuter des programmes arbitraires des commandes arbitraires :

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Injection d'Applications Java

Il est possible d'abuser de certaines capacités java (comme la variable d'environnement **`_JAVA_OPTS`**) pour faire exécuter à une application java **du code/commandes arbitraires**.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### Injection d'Applications .Net

Il est possible d'injecter du code dans des applications .Net en **abusant de la fonctionnalité de débogage .Net** (non protégée par les protections macOS telles que le durcissement à l'exécution).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Injection Perl

Vérifiez différentes options pour faire exécuter un script Perl du code arbitraire dans :

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Injection Ruby

Il est également possible d'abuser des variables d'environnement ruby pour faire exécuter des scripts arbitraires du code arbitraire :

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Injection Python

Si la variable d'environnement **`PYTHONINSPECT`** est définie, le processus python passera à un cli python une fois terminé. Il est également possible d'utiliser **`PYTHONSTARTUP`** pour indiquer un script python à exécuter au début d'une session interactive.\
Cependant, notez que le script **`PYTHONSTARTUP`** ne sera pas exécuté lorsque **`PYTHONINSPECT`** crée la session interactive.

D'autres variables d'environnement telles que **`PYTHONPATH`** et **`PYTHONHOME`** pourraient également être utiles pour faire exécuter une commande python du code arbitraire.

Notez que les exécutables compilés avec **`pyinstaller`** n'utiliseront pas ces variables environnementales même s'ils s'exécutent en utilisant un python intégré.

{% hint style="danger" %}
Dans l'ensemble, je n'ai pas trouvé de moyen de faire exécuter du code arbitraire à python en abusant des variables d'environnement.\
Cependant, la plupart des gens installent python en utilisant **Homebrew**, ce qui installera python dans un **emplacement écrivable** pour l'utilisateur admin par défaut. Vous pouvez le détourner avec quelque chose comme :
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Même **root** exécutera ce code lors de l'exécution de python.  
{% endhint %}

## Détection

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) est une application open source qui peut **détecter et bloquer les actions d'injection de processus** :

* Utilisation des **variables d'environnement** : Elle surveillera la présence de l'une des variables d'environnement suivantes : **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** et **`ELECTRON_RUN_AS_NODE`**
* Utilisation des appels **`task_for_pid`** : Pour trouver quand un processus veut obtenir le **port de tâche d'un autre** ce qui permet d'injecter du code dans le processus.
* **Paramètres des applications Electron** : Quelqu'un peut utiliser l'argument de ligne de commande **`--inspect`**, **`--inspect-brk`** et **`--remote-debugging-port`** pour démarrer une application Electron en mode débogage, et ainsi y injecter du code.
* Utilisation de **symlinks** ou **hardlinks** : Typiquement, l'abus le plus courant est de **placer un lien avec nos privilèges d'utilisateur**, et **de le pointer vers un emplacement de privilège supérieur**. La détection est très simple pour les hardlinks et les symlinks. Si le processus créant le lien a un **niveau de privilège différent** de celui du fichier cible, nous créons une **alerte**. Malheureusement, dans le cas des symlinks, le blocage n'est pas possible, car nous n'avons pas d'informations sur la destination du lien avant sa création. C'est une limitation du framework EndpointSecuriy d'Apple.

### Appels effectués par d'autres processus

Dans [**cet article de blog**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html), vous pouvez trouver comment il est possible d'utiliser la fonction **`task_name_for_pid`** pour obtenir des informations sur d'autres **processus injectant du code dans un processus** et ensuite obtenir des informations sur cet autre processus.

Notez que pour appeler cette fonction, vous devez avoir le **même uid** que celui exécutant le processus ou être **root** (et cela retourne des informations sur le processus, pas un moyen d'injecter du code).

## Références

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

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
