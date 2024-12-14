# CGroups

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

## Basic Information

**Les groupes de contrôle Linux**, ou **cgroups**, sont une fonctionnalité du noyau Linux qui permet l'allocation, la limitation et la priorisation des ressources système telles que le CPU, la mémoire et les entrées/sorties disque parmi des groupes de processus. Ils offrent un mécanisme pour **gérer et isoler l'utilisation des ressources** des collections de processus, bénéfique pour des objectifs tels que la limitation des ressources, l'isolement des charges de travail et la priorisation des ressources parmi différents groupes de processus.

Il existe **deux versions de cgroups** : la version 1 et la version 2. Les deux peuvent être utilisées simultanément sur un système. La principale distinction est que **la version 2 des cgroups** introduit une **structure hiérarchique en arbre**, permettant une distribution des ressources plus nuancée et détaillée parmi les groupes de processus. De plus, la version 2 apporte diverses améliorations, notamment :

En plus de la nouvelle organisation hiérarchique, la version 2 des cgroups a également introduit **plusieurs autres changements et améliorations**, tels que le support de **nouveaux contrôleurs de ressources**, un meilleur support pour les applications héritées et des performances améliorées.

Dans l'ensemble, les cgroups **version 2 offrent plus de fonctionnalités et de meilleures performances** que la version 1, mais cette dernière peut encore être utilisée dans certains scénarios où la compatibilité avec les anciens systèmes est une préoccupation.

Vous pouvez lister les cgroups v1 et v2 pour n'importe quel processus en consultant son fichier cgroup dans /proc/\<pid>. Vous pouvez commencer par examiner les cgroups de votre shell avec cette commande :
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
La structure de sortie est la suivante :

* **Numéros 2–12** : cgroups v1, chaque ligne représentant un cgroup différent. Les contrôleurs pour ceux-ci sont spécifiés à côté du numéro.
* **Numéro 1** : Également cgroups v1, mais uniquement à des fins de gestion (défini par, par exemple, systemd), et n'a pas de contrôleur.
* **Numéro 0** : Représente les cgroups v2. Aucun contrôleur n'est listé, et cette ligne est exclusive aux systèmes ne fonctionnant qu'avec des cgroups v2.
* Les **noms sont hiérarchiques**, ressemblant à des chemins de fichiers, indiquant la structure et la relation entre différents cgroups.
* **Des noms comme /user.slice ou /system.slice** spécifient la catégorisation des cgroups, avec user.slice généralement pour les sessions de connexion gérées par systemd et system.slice pour les services système.

### Visualiser les cgroups

Le système de fichiers est généralement utilisé pour accéder aux **cgroups**, divergeant de l'interface d'appel système Unix traditionnellement utilisée pour les interactions avec le noyau. Pour examiner la configuration du cgroup d'un shell, il faut consulter le fichier **/proc/self/cgroup**, qui révèle le cgroup du shell. Ensuite, en naviguant vers le répertoire **/sys/fs/cgroup** (ou **`/sys/fs/cgroup/unified`**) et en localisant un répertoire partageant le nom du cgroup, on peut observer divers paramètres et informations sur l'utilisation des ressources pertinentes au cgroup.

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

Les fichiers d'interface clés pour les cgroups sont préfixés par **cgroup**. Le fichier **cgroup.procs**, qui peut être consulté avec des commandes standard comme cat, liste les processus au sein du cgroup. Un autre fichier, **cgroup.threads**, inclut des informations sur les threads.

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

Les cgroups gérant les shells englobent généralement deux contrôleurs qui régulent l'utilisation de la mémoire et le nombre de processus. Pour interagir avec un contrôleur, les fichiers portant le préfixe du contrôleur doivent être consultés. Par exemple, **pids.current** serait référencé pour déterminer le nombre de threads dans le cgroup.

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

L'indication de **max** dans une valeur suggère l'absence de limite spécifique pour le cgroup. Cependant, en raison de la nature hiérarchique des cgroups, des limites peuvent être imposées par un cgroup à un niveau inférieur dans la hiérarchie des répertoires.

### Manipuler et créer des cgroups

Les processus sont assignés aux cgroups en **écrivant leur identifiant de processus (PID) dans le fichier `cgroup.procs`**. Cela nécessite des privilèges root. Par exemple, pour ajouter un processus :
```bash
echo [pid] > cgroup.procs
```
De même, **modifier les attributs de cgroup, comme définir une limite de PID**, se fait en écrivant la valeur souhaitée dans le fichier pertinent. Pour définir un maximum de 3 000 PIDs pour un cgroup :
```bash
echo 3000 > pids.max
```
**Créer de nouveaux cgroups** implique de créer un nouveau sous-répertoire dans la hiérarchie des cgroups, ce qui incite le noyau à générer automatiquement les fichiers d'interface nécessaires. Bien que les cgroups sans processus actifs puissent être supprimés avec `rmdir`, soyez conscient de certaines contraintes :

* **Les processus ne peuvent être placés que dans des cgroups feuilles** (c'est-à-dire, les plus imbriqués dans une hiérarchie).
* **Un cgroup ne peut posséder un contrôleur absent dans son parent**.
* **Les contrôleurs pour les cgroups enfants doivent être explicitement déclarés** dans le fichier `cgroup.subtree_control`. Par exemple, pour activer les contrôleurs CPU et PID dans un cgroup enfant :
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
Le **cgroup root** est une exception à ces règles, permettant un placement direct des processus. Cela peut être utilisé pour retirer des processus de la gestion de systemd.

**La surveillance de l'utilisation du CPU** au sein d'un cgroup est possible via le fichier `cpu.stat`, affichant le temps total de CPU consommé, utile pour suivre l'utilisation à travers les sous-processus d'un service :

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>Statistiques d'utilisation du CPU telles qu'affichées dans le fichier cpu.stat</p></figcaption></figure>

## Références

* **Livre : Comment Linux fonctionne, 3ème édition : Ce que chaque superutilisateur devrait savoir par Brian Ward**

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
