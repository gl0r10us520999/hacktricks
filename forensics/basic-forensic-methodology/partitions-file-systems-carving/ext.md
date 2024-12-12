<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>


# Ext - Système de fichiers étendu

**Ext2** est le système de fichiers le plus courant pour les partitions **sans journalisation** (partitions qui ne changent pas beaucoup) comme la partition de démarrage. **Ext3/4** sont **journalisés** et sont généralement utilisés pour les **autres partitions**.

Tous les groupes de blocs dans le système de fichiers ont la même taille et sont stockés séquentiellement. Cela permet au noyau de déterminer facilement l'emplacement d'un groupe de blocs sur un disque à partir de son index entier.

Chaque groupe de blocs contient les éléments d'information suivants :

* Une copie du superbloc du système de fichiers
* Une copie des descripteurs de groupe de blocs
* Une carte de bits de blocs de données qui est utilisée pour identifier les blocs libres à l'intérieur du groupe
* Une carte de bits d'inodes, qui est utilisée pour identifier les inodes libres à l'intérieur du groupe
* Table d'inodes : elle se compose d'une série de blocs consécutifs, chacun contenant un nombre prédéfini d'inodes de la Figure 1 Ext2. Toutes les inodes ont la même taille : 128 octets. Un bloc de 1 024 octets contient 8 inodes, tandis qu'un bloc de 4 096 octets contient 32 inodes. Notez qu'en Ext2, il n'est pas nécessaire de stocker sur le disque une correspondance entre un numéro d'inode et le numéro de bloc correspondant car cette valeur peut être déduite du numéro de groupe de blocs et de la position relative à l'intérieur de la table d'inodes. Par exemple, supposons que chaque groupe de blocs contient 4 096 inodes et que nous voulons connaître l'adresse sur le disque de l'inode 13 021. Dans ce cas, l'inode appartient au troisième groupe de blocs et son adresse sur le disque est stockée dans la 733e entrée de la table d'inodes correspondante. Comme vous pouvez le voir, le numéro d'inode est simplement une clé utilisée par les routines Ext2 pour récupérer rapidement le descripteur d'inode approprié sur le disque
* blocs de données, contenant des fichiers. Tout bloc qui ne contient aucune information significative est dit être libre.

![](<../../../.gitbook/assets/image (406).png>)

## Fonctionnalités optionnelles d'Ext

Les **fonctionnalités affectent où** les données sont situées, **comment** les données sont stockées dans les inodes et certaines d'entre elles peuvent fournir des **métadonnées supplémentaires** pour l'analyse, donc les fonctionnalités sont importantes dans Ext.

Ext dispose de fonctionnalités optionnelles que votre système d'exploitation peut ou non prendre en charge, il existe 3 possibilités :

* Compatible
* Incompatible
* Compatible en lecture seule : Il peut être monté mais pas pour l'écriture

S'il y a des fonctionnalités **incompatibles**, vous ne pourrez pas monter le système de fichiers car le système d'exploitation ne saura pas comment accéder aux données.

{% hint style="info" %}
Un attaquant suspecté pourrait avoir des extensions non standard
{% endhint %}

**Tout utilitaire** qui lit le **superbloc** pourra indiquer les **fonctionnalités** d'un **système de fichiers Ext**, mais vous pourriez également utiliser `file -sL /dev/sd*`

## Superbloc

Le superbloc est les premiers 1024 octets depuis le début et il est répété dans le premier bloc de chaque groupe et contient :

* Taille de bloc
* Blocs totaux
* Blocs par groupe de blocs
* Blocs réservés avant le premier groupe de blocs
* Inodes totaux
* Inodes par groupe de blocs
* Nom du volume
* Dernière heure d'écriture
* Dernière heure de montage
* Chemin où le système de fichiers a été monté pour la dernière fois
* État du système de fichiers (propre ?)

Il est possible d'obtenir ces informations à partir d'un fichier système de fichiers Ext en utilisant :
```bash
fsstat -o <offsetstart> /pat/to/filesystem-file.ext
#You can get the <offsetstart> with the "p" command inside fdisk
```
Vous pouvez également utiliser l'application GUI gratuite : [https://www.disk-editor.org/index.html](https://www.disk-editor.org/index.html)\
Ou vous pouvez également utiliser **python** pour obtenir les informations du superbloc : [https://pypi.org/project/superblock/](https://pypi.org/project/superblock/)

## inodes

Les **inodes** contiennent la liste des **blocs** qui **contiennent** les données réelles d'un **fichier**.\
Si le fichier est volumineux, un inode peut contenir des pointeurs vers d'autres inodes qui pointent vers les blocs/d'autres inodes contenant les données du fichier.

![](<../../../.gitbook/assets/image (416).png>)

Dans les inodes **Ext2** et **Ext3** ont une taille de **128B**, **Ext4** utilise actuellement **156B** mais alloue **256B** sur le disque pour permettre une expansion future.

Structure de l'inode :

| Décalage | Taille | Nom               | Description                                      |
| -------- | ------ | ----------------- | ------------------------------------------------ |
| 0x0      | 2      | Mode de fichier   | Mode de fichier et type                          |
| 0x2      | 2      | UID               | 16 bits inférieurs de l'ID propriétaire          |
| 0x4      | 4      | Taille Il         | 32 bits inférieurs de la taille du fichier       |
| 0x8      | 4      | Atime             | Heure d'accès en secondes depuis l'époque        |
| 0xC      | 4      | Ctime             | Heure de modification en secondes depuis l'époque|
| 0x10     | 4      | Mtime             | Heure de modification en secondes depuis l'époque|
| 0x14     | 4      | Dtime             | Heure de suppression en secondes depuis l'époque|
| 0x18     | 2      | GID               | 16 bits inférieurs de l'ID de groupe             |
| 0x1A     | 2      | Compteur de liens | Nombre de liens rigides                          |
| 0xC      | 4      | Blocs Io          | 32 bits inférieurs du nombre de blocs            |
| 0x20     | 4      | Drapeaux          | Drapeaux                                         |
| 0x24     | 4      | Union osd1        | Linux : version I                                |
| 0x28     | 69     | Bloc\[15]         | 15 points vers un bloc de données                |
| 0x64     | 4      | Version           | Version du fichier pour NFS                      |
| 0x68     | 4      | ACL de fichier bas | 32 bits inférieurs des attributs étendus (ACL, etc)|
| 0x6C     | 4      | Taille de fichier hi | 32 bits supérieurs de la taille du fichier (ext4 uniquement)|
| 0x70     | 4      | Fragment obsolète | Adresse de fragment obsolète                    |
| 0x74     | 12     | Osd 2             | Deuxième union dépendante du système d'exploitation|
| 0x74     | 2      | Blocs hi          | 16 bits supérieurs du nombre de blocs           |
| 0x76     | 2      | ACL de fichier hi | 16 bits supérieurs des attributs étendus (ACL, etc)|
| 0x78     | 2      | UID hi            | 16 bits supérieurs de l'ID propriétaire          |
| 0x7A     | 2      | GID hi            | 16 bits supérieurs de l'ID de groupe             |
| 0x7C     | 2      | Vérification Io   | 16 bits inférieurs de la somme de contrôle de l'inode|

"Modifier" est l'horodatage de la dernière fois que le contenu du fichier a été modifié. C'est souvent appelé "_mtime_".\
"Changer" est l'horodatage de la dernière fois que l'_inode_ du fichier a été modifié, par exemple en modifiant les autorisations, la propriété, le nom du fichier et le nombre de liens physiques. C'est souvent appelé "_ctime_".

Structure étendue de l'inode (Ext4) :

| Décalage | Taille | Nom         | Description                                 |
| -------- | ------ | ----------- | ------------------------------------------- |
| 0x80     | 2      | Taille supplémentaire | Combien d'octets au-delà des 128 standard sont utilisés |
| 0x82     | 2      | Vérification hi | 16 bits supérieurs de la somme de contrôle de l'inode |
| 0x84     | 4      | Ctime supplémentaire | Bits supplémentaires de l'heure de modification |
| 0x88     | 4      | Mtime supplémentaire | Bits supplémentaires de l'heure de modification |
| 0x8C     | 4      | Atime supplémentaire | Bits supplémentaires de l'heure d'accès       |
| 0x90     | 4      | Crtime       | Heure de création du fichier (secondes depuis l'époque) |
| 0x94     | 4      | Crtime supplémentaire | Bits supplémentaires de l'heure de création du fichier |
| 0x98     | 4      | Version hi   | 32 bits supérieurs de la version             |
| 0x9C     |        | Inutilisé    | Espace réservé pour les expansions futures    |

Inodes spéciaux :

| Inode | Objectif spécial                                     |
| ----- | ---------------------------------------------------- |
| 0     | Aucun inode de ce type, la numérotation commence à 1 |
| 1     | Liste de blocs défectueux                            |
| 2     | Répertoire racine                                    |
| 3     | Quotas utilisateur                                   |
| 4     | Quotas de groupe                                     |
| 5     | Chargeur de démarrage                                |
| 6     | Répertoire de récupération                            |
| 7     | Descripteurs de groupe réservés (pour redimensionner le système de fichiers) |
| 8     | Journal                                              |
| 9     | Exclure l'inode (pour les instantanés)               |
| 10    | Inode de réplica                                     |
| 11    | Premier inode non réservé (souvent perdu + trouvé)   |

{% hint style="info" %}
Notez que l'heure de création n'apparaît que dans Ext4.
{% endhint %}

En connaissant le numéro d'inode, vous pouvez facilement trouver son index :

* **Groupe de blocs** auquel appartient un inode : (Numéro d'inode - 1) / (Inodes par groupe)
* **Index à l'intérieur de son groupe** : (Numéro d'inode - 1) mod (Inodes/groupes)
* **Décalage** dans la **table d'inodes** : Numéro d'inode \* (Taille de l'inode)
* Le "-1" est dû au fait que l'inode 0 est indéfini (non utilisé)
```bash
ls -ali /bin | sort -n #Get all inode numbers and sort by them
stat /bin/ls #Get the inode information of a file
istat -o <start offset> /path/to/image.ext 657103 #Get information of that inode inside the given ext file
icat -o <start offset> /path/to/image.ext 657103 #Cat the file
```
```markdown
File Mode

| Numéro | Description                                                                                         |
| ------ | --------------------------------------------------------------------------------------------------- |
| **15** | **Reg/Slink-13/Socket-14**                                                                          |
| **14** | **Répertoire/Block Bit 13**                                                                         |
| **13** | **Périphérique de caractère/Block Bit 14**                                                         |
| **12** | **FIFO**                                                                                            |
| 11     | Set UID                                                                                             |
| 10     | Set GID                                                                                             |
| 9      | Bit collant (sans cela, toute personne avec des autorisations d'écriture et d'exécution sur un répertoire peut supprimer et renommer des fichiers) |
| 8      | Lecture propriétaire                                                                                |
| 7      | Écriture propriétaire                                                                               |
| 6      | Exécution propriétaire                                                                              |
| 5      | Lecture de groupe                                                                                   |
| 4      | Écriture de groupe                                                                                  |
| 3      | Exécution de groupe                                                                                 |
| 2      | Lecture des autres                                                                                  |
| 1      | Écriture des autres                                                                                 |
| 0      | Exécution des autres                                                                                |

Les bits en gras (12, 13, 14, 15) indiquent le type de fichier (un répertoire, une prise...) seule l'une des options en gras peut exister.

Répertoires

| Décalage | Taille | Nom       | Description                                                                                                                                                  |
| -------- | ------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0x0      | 4      | Inode     |                                                                                                                                                            |
| 0x4      | 2      | Longueur d'enregistrement | Longueur de l'enregistrement                                                                                                                               |
| 0x6      | 1      | Longueur du nom | Longueur du nom                                                                                                                                             |
| 0x7      | 1      | Type de fichier | <p>0x00 Inconnu<br>0x01 Régulier</p><p>0x02 Répertoire</p><p>0x03 Périphérique de caractère</p><p>0x04 Périphérique de bloc</p><p>0x05 FIFO</p><p>0x06 Prise</p><p>0x07 Lien symbolique</p> |
| 0x8      |        | Nom       | Chaîne de caractères du nom (jusqu'à 255 caractères)                                                                                                       |

**Pour améliorer les performances, les blocs de répertoire de hachage racine peuvent être utilisés.**

**Attributs étendus**

Peuvent être stockés dans

* Espace supplémentaire entre les inodes (256 - taille de l'inode, généralement = 100)
* Un bloc de données pointé par file_acl dans l'inode

Peut être utilisé pour stocker n'importe quoi en tant qu'attribut d'utilisateur si le nom commence par "user". Ainsi, les données peuvent être cachées de cette manière.

Entrées d'attributs étendus

| Décalage | Taille | Nom           | Description                                                                                                                                                                                                        |
| -------- | ------ | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0x0      | 1      | Longueur du nom | Longueur du nom de l'attribut                                                                                                                                                                                      |
| 0x1      | 1      | Index du nom  | <p>0x0 = pas de préfixe</p><p>0x1 = préfixe user.</p><p>0x2 = system.posix_acl_access</p><p>0x3 = system.posix_acl_default</p><p>0x4 = trusted.</p><p>0x6 = security.</p><p>0x7 = system.</p><p>0x8 = system.richacl</p> |
| 0x2      | 2      | Décalage de la valeur | Décalage à partir de la première entrée inode ou du début du bloc                                                                                                                                                  |
| 0x4      | 4      | Blocs de valeur | Bloc de disque où la valeur est stockée ou zéro pour ce bloc                                                                                                                                                        |
| 0x8      | 4      | Taille de la valeur | Longueur de la valeur                                                                                                                                                                                              |
| 0xC      | 4      | Hachage       | Hachage pour les attributs dans le bloc ou zéro s'ils sont dans l'inode                                                                                                                                             |
| 0x10     |        | Nom           | Nom de l'attribut sans le caractère NULL final                                                                                                                                                                      |
```
```bash
setfattr -n 'user.secret' -v 'This is a secret' file.txt #Save a secret using extended attributes
getfattr file.txt #Get extended attribute names of a file
getdattr -n 'user.secret' file.txt #Get extended attribute called "user.secret"
```
## Vue du système de fichiers

Pour voir le contenu du système de fichiers, vous pouvez **utiliser l'outil gratuit**: [https://www.disk-editor.org/index.html](https://www.disk-editor.org/index.html)\
Ou vous pouvez le monter dans votre linux en utilisant la commande `mount`.

[https://piazza.com/class\_profile/get\_resource/il71xfllx3l16f/inz4wsb2m0w2oz#:\~:text=Le%20système%20de%20fichiers%20Ext2%20divise,temps%20de%20recherche%20de%20disque%20moyen%20inférieur.](https://piazza.com/class\_profile/get\_resource/il71xfllx3l16f/inz4wsb2m0w2oz#:\~:text=Le%20système%20de%20fichiers%20Ext2%20divise,temps%20de%20recherche%20de%20disque%20moyen%20inférieur.)
