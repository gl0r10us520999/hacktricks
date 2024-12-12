# Algorithmes cryptographiques/de compression

## Algorithmes cryptographiques/de compression

{% hint style="success" %}
Apprenez et pratiquez le piratage AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Formation HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le piratage GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Formation HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenez HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
{% endhint %}

## Identification des algorithmes

Si vous vous retrouvez avec un code **utilisant des décalages à droite et à gauche, des XOR et plusieurs opérations arithmétiques**, il est très probable qu'il s'agisse de la mise en œuvre d'un **algorithme cryptographique**. Voici quelques façons d'**identifier l'algorithme utilisé sans avoir besoin de déchiffrer chaque étape**.

### Fonctions API

**CryptDeriveKey**

Si cette fonction est utilisée, vous pouvez trouver quel **algorithme est utilisé** en vérifiant la valeur du deuxième paramètre :

![](<../../.gitbook/assets/image (156).png>)

Consultez ici le tableau des algorithmes possibles et de leurs valeurs attribuées : [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

Compresse et décompresse un tampon de données donné.

**CryptAcquireContext**

D'après [la documentation](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta) : La fonction **CryptAcquireContext** est utilisée pour acquérir une poignée sur un conteneur de clés particulier dans un fournisseur de services cryptographiques (CSP) particulier. **Cette poignée retournée est utilisée dans les appels aux fonctions CryptoAPI** qui utilisent le CSP sélectionné.

**CryptCreateHash**

Initie le hachage d'un flux de données. Si cette fonction est utilisée, vous pouvez trouver quel **algorithme est utilisé** en vérifiant la valeur du deuxième paramètre :

![](<../../.gitbook/assets/image (549).png>)

\
Consultez ici le tableau des algorithmes possibles et de leurs valeurs attribuées : [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### Constantes de code

Parfois, il est vraiment facile d'identifier un algorithme grâce au fait qu'il doit utiliser une valeur spéciale et unique.

![](<../../.gitbook/assets/image (833).png>)

Si vous recherchez la première constante sur Google, voici ce que vous obtenez :

![](<../../.gitbook/assets/image (529).png>)

Par conséquent, vous pouvez supposer que la fonction décompilée est un **calculateur sha256**.\
Vous pouvez rechercher n'importe laquelle des autres constantes et vous obtiendrez (probablement) le même résultat.

### Informations sur les données

Si le code ne contient aucune constante significative, il peut être en train de **charger des informations de la section .data**.\
Vous pouvez accéder à ces données, **regrouper le premier dword** et rechercher sur Google comme nous l'avons fait dans la section précédente :

![](<../../.gitbook/assets/image (531).png>)

Dans ce cas, si vous recherchez **0xA56363C6**, vous pouvez trouver que cela est lié aux **tables de l'algorithme AES**.

## RC4 **(Cryptage symétrique)**

### Caractéristiques

Il est composé de 3 parties principales :

* **Étape d'initialisation/** : Crée une **table de valeurs de 0x00 à 0xFF** (256 octets au total, 0x100). Cette table est communément appelée **Boîte de substitution** (ou SBox).
* **Étape de brouillage** : Va **parcourir la table** créée précédemment (boucle de 0x100 itérations, encore une fois) en modifiant chaque valeur avec des octets **semi-aléatoires**. Pour créer ces octets semi-aléatoires, la clé RC4 est utilisée. Les clés RC4 peuvent être **de 1 à 256 octets de longueur**, cependant il est généralement recommandé qu'elles dépassent 5 octets. Généralement, les clés RC4 font 16 octets de longueur.
* **Étape XOR** : Enfin, le texte en clair ou le texte chiffré est **XORé avec les valeurs créées précédemment**. La fonction pour chiffrer et déchiffrer est la même. Pour cela, une **boucle à travers les 256 octets créés** sera effectuée autant de fois que nécessaire. Cela est généralement reconnu dans un code décompilé avec un **%256 (mod 256)**.

{% hint style="info" %}
**Pour identifier un RC4 dans un code de désassemblage/décompilé, vous pouvez rechercher 2 boucles de taille 0x100 (avec l'utilisation d'une clé) et ensuite un XOR des données d'entrée avec les 256 valeurs créées précédemment dans les 2 boucles en utilisant probablement un %256 (mod 256)**
{% endhint %}

### **Étape d'initialisation/Boîte de substitution :** (Notez le nombre 256 utilisé comme compteur et comment un 0 est écrit à chaque emplacement des 256 caractères)

![](<../../.gitbook/assets/image (584).png>)

### **Étape de brouillage :**

![](<../../.gitbook/assets/image (835).png>)

### **Étape XOR :**

![](<../../.gitbook/assets/image (904).png>)

## **AES (Cryptage symétrique)**

### **Caractéristiques**

* Utilisation de **boîtes de substitution et de tables de recherche**
* Il est possible de **distinguer AES grâce à l'utilisation de valeurs spécifiques de tables de recherche** (constantes). _Notez que la **constante** peut être **stockée** dans le binaire **ou créée**_ _**dynamiquement**._
* La **clé de chiffrement** doit être **divisible** par **16** (généralement 32 octets) et généralement un **IV** de 16 octets est utilisé.

### Constantes SBox

![](<../../.gitbook/assets/image (208).png>)

## Serpent **(Cryptage symétrique)**

### Caractéristiques

* Il est rare de trouver des logiciels malveillants l'utilisant mais il existe des exemples (Ursnif)
* Facile à déterminer si un algorithme est Serpent ou non en fonction de sa longueur (fonction extrêmement longue)

### Identification

Sur l'image suivante, remarquez comment la constante **0x9E3779B9** est utilisée (notez que cette constante est également utilisée par d'autres algorithmes de cryptographie comme **TEA** - Tiny Encryption Algorithm).\
Notez également la **taille de la boucle** (**132**) et le **nombre d'opérations XOR** dans les instructions de **désassemblage** et dans l'exemple de **code** :

![](<../../.gitbook/assets/image (547).png>)

Comme mentionné précédemment, ce code peut être visualisé à l'intérieur de n'importe quel décompilateur comme une **fonction très longue** car il n'y a **pas de sauts** à l'intérieur. Le code décompilé peut ressembler à ce qui suit :

![](<../../.gitbook/assets/image (513).png>)

Par conséquent, il est possible d'identifier cet algorithme en vérifiant le **nombre magique** et les **XOR initiaux**, en voyant une **fonction très longue** et en **comparant** certaines **instructions** de la longue fonction **avec une implémentation** (comme le décalage à gauche de 7 et la rotation à gauche de 22).
## RSA **(Cryptographie Asymétrique)**

### Caractéristiques

* Plus complexe que les algorithmes symétriques
* Il n'y a pas de constantes ! (les implémentations personnalisées sont difficiles à déterminer)
* KANAL (un analyseur crypto) échoue à montrer des indices sur RSA car il repose sur des constantes.

### Identification par comparaisons

![](<../../.gitbook/assets/image (1113).png>)

* À la ligne 11 (gauche), il y a un `+7) >> 3` qui est le même qu'à la ligne 35 (droite) : `+7) / 8`
* La ligne 12 (gauche) vérifie si `modulus_len < 0x040` et à la ligne 36 (droite) elle vérifie si `inputLen+11 > modulusLen`

## MD5 & SHA (hachage)

### Caractéristiques

* 3 fonctions : Init, Update, Final
* Fonctions d'initialisation similaires

### Identification

**Init**

Vous pouvez les identifier tous les deux en vérifiant les constantes. Notez que sha\_init a 1 constante que MD5 n'a pas :

![](<../../.gitbook/assets/image (406).png>)

**Transformation MD5**

Notez l'utilisation de plus de constantes

![](<../../.gitbook/assets/image (253) (1) (1).png>)

## CRC (hachage)

* Plus petit et plus efficace car sa fonction est de trouver des changements accidentels dans les données
* Utilise des tables de recherche (vous pouvez donc identifier des constantes)

### Identification

Vérifiez les **constantes de la table de recherche** :

![](<../../.gitbook/assets/image (508).png>)

Un algorithme de hachage CRC ressemble à :

![](<../../.gitbook/assets/image (391).png>)

## APLib (Compression)

### Caractéristiques

* Constantes non reconnaissables
* Vous pouvez essayer d'écrire l'algorithme en python et rechercher des choses similaires en ligne

### Identification

Le graphique est assez grand :

![](<../../.gitbook/assets/image (207) (2) (1).png>)

Vérifiez **3 comparaisons pour le reconnaître** :

![](<../../.gitbook/assets/image (430).png>)
