# Clés de Registre Windows Intéressantes

### Clés de Registre Windows Intéressantes

{% hint style="success" %}
Apprenez et pratiquez le Hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le Hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenir HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** nous sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PRs aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts github.

</details>
{% endhint %}


### **Informations sur la version de Windows et le propriétaire**
- Situé à **`Software\Microsoft\Windows NT\CurrentVersion`**, vous trouverez la version de Windows, le Service Pack, l'heure d'installation et le nom du propriétaire enregistré de manière simple.

### **Nom de l'ordinateur**
- Le nom d'hôte se trouve sous **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Paramètre de fuseau horaire**
- Le fuseau horaire du système est stocké dans **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Suivi du temps d'accès**
- Par défaut, le suivi du dernier temps d'accès est désactivé (**`NtfsDisableLastAccessUpdate=1`**). Pour l'activer, utilisez :
`fsutil behavior set disablelastaccess 0`

### Versions de Windows et Service Packs
- La **version de Windows** indique l'édition (par exemple, Home, Pro) et sa version (par exemple, Windows 10, Windows 11), tandis que les **Service Packs** sont des mises à jour qui incluent des corrections et, parfois, de nouvelles fonctionnalités.

### Activation du dernier temps d'accès
- Activer le suivi du dernier temps d'accès vous permet de voir quand les fichiers ont été ouverts pour la dernière fois, ce qui peut être crucial pour l'analyse judiciaire ou la surveillance du système.

### Détails des informations réseau
- Le registre contient des données étendues sur les configurations réseau, y compris **types de réseaux (sans fil, câble, 3G)** et **catégories de réseau (Public, Privé/Domicile, Domaine/Travail)**, qui sont essentielles pour comprendre les paramètres de sécurité réseau et les autorisations.

### Mise en cache côté client (CSC)
- **CSC** améliore l'accès aux fichiers hors ligne en mettant en cache des copies de fichiers partagés. Différents paramètres **CSCFlags** contrôlent comment et quels fichiers sont mis en cache, affectant les performances et l'expérience utilisateur, en particulier dans des environnements avec une connectivité intermittente.

### Programmes de démarrage automatique
- Les programmes listés dans diverses clés de registre `Run` et `RunOnce` sont lancés automatiquement au démarrage, affectant le temps de démarrage du système et pouvant être des points d'intérêt pour identifier des logiciels malveillants ou indésirables.

### Shellbags
- Les **Shellbags** non seulement stockent les préférences pour les vues de dossiers, mais fournissent également des preuves judiciaires d'accès aux dossiers même si le dossier n'existe plus. Ils sont inestimables pour les enquêtes, révélant l'activité des utilisateurs qui n'est pas évidente par d'autres moyens.

### Informations USB et analyses judiciaires
- Les détails stockés dans le registre concernant les appareils USB peuvent aider à retracer quels appareils ont été connectés à un ordinateur, liant potentiellement un appareil à des transferts de fichiers sensibles ou à des incidents d'accès non autorisé.

### Numéro de série de volume
- Le **numéro de série de volume** peut être crucial pour suivre l'instance spécifique d'un système de fichiers, utile dans des scénarios judiciaires où l'origine d'un fichier doit être établie à travers différents appareils.

### **Détails de l'arrêt**
- L'heure et le nombre d'arrêts (ce dernier uniquement pour XP) sont conservés dans **`System\ControlSet001\Control\Windows`** et **`System\ControlSet001\Control\Watchdog\Display`**.

### **Configuration réseau**
- Pour des informations détaillées sur l'interface réseau, consultez **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Les heures de première et de dernière connexion réseau, y compris les connexions VPN, sont enregistrées sous divers chemins dans **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Dossiers partagés**
- Les dossiers partagés et les paramètres se trouvent sous **`System\ControlSet001\Services\lanmanserver\Shares`**. Les paramètres de mise en cache côté client (CSC) dictent la disponibilité des fichiers hors ligne.

### **Programmes qui démarrent automatiquement**
- Des chemins comme **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** et des entrées similaires sous `Software\Microsoft\Windows\CurrentVersion` détaillent les programmes configurés pour s'exécuter au démarrage.

### **Recherches et chemins tapés**
- Les recherches de l'Explorateur et les chemins tapés sont suivis dans le registre sous **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** pour WordwheelQuery et TypedPaths, respectivement.

### **Documents récents et fichiers Office**
- Les documents récents et les fichiers Office accessibles sont notés dans `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` et des chemins spécifiques à la version d'Office.

### **Éléments les plus récemment utilisés (MRU)**
- Les listes MRU, indiquant les chemins de fichiers récents et les commandes, sont stockées dans diverses sous-clés `ComDlg32` et `Explorer` sous `NTUSER.DAT`.

### **Suivi de l'activité utilisateur**
- La fonctionnalité User Assist enregistre des statistiques détaillées sur l'utilisation des applications, y compris le nombre d'exécutions et le dernier temps d'exécution, à **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Analyse des Shellbags**
- Les Shellbags, révélant des détails d'accès aux dossiers, sont stockés dans `USRCLASS.DAT` et `NTUSER.DAT` sous `Software\Microsoft\Windows\Shell`. Utilisez **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** pour l'analyse.

### **Historique des appareils USB**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** et **`HKLM\SYSTEM\ControlSet001\Enum\USB`** contiennent des détails riches sur les appareils USB connectés, y compris le fabricant, le nom du produit et les horodatages de connexion.
- L'utilisateur associé à un appareil USB spécifique peut être identifié en recherchant les hives `NTUSER.DAT` pour le **{GUID}** de l'appareil.
- Le dernier appareil monté et son numéro de série de volume peuvent être retracés via `System\MountedDevices` et `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, respectivement.

Ce guide condense les chemins et méthodes cruciaux pour accéder à des informations détaillées sur le système, le réseau et l'activité utilisateur sur les systèmes Windows, visant la clarté et l'utilisabilité.



{% hint style="success" %}
Apprenez et pratiquez le Hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le Hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenir HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** nous sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PRs aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts github.

</details>
{% endhint %}
