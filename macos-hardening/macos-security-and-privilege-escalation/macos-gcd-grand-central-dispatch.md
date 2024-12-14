# macOS GCD - Grand Central Dispatch

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

**Grand Central Dispatch (GCD),** également connu sous le nom de **libdispatch** (`libdispatch.dyld`), est disponible à la fois sur macOS et iOS. C'est une technologie développée par Apple pour optimiser le support des applications pour l'exécution concurrente (multithread) sur du matériel multicœur.

**GCD** fournit et gère des **queues FIFO** auxquelles votre application peut **soumettre des tâches** sous forme d'**objets bloc**. Les blocs soumis aux queues de dispatch sont **exécutés sur un pool de threads** entièrement géré par le système. GCD crée automatiquement des threads pour exécuter les tâches dans les queues de dispatch et planifie ces tâches pour s'exécuter sur les cœurs disponibles.

{% hint style="success" %}
En résumé, pour exécuter du code en **parallèle**, les processus peuvent envoyer des **blocs de code à GCD**, qui se chargera de leur exécution. Par conséquent, les processus ne créent pas de nouveaux threads ; **GCD exécute le code donné avec son propre pool de threads** (qui peut augmenter ou diminuer si nécessaire).
{% endhint %}

Ceci est très utile pour gérer l'exécution parallèle avec succès, réduisant considérablement le nombre de threads que les processus créent et optimisant l'exécution parallèle. C'est idéal pour les tâches qui nécessitent un **grand parallélisme** (brute-forcing ?) ou pour les tâches qui ne devraient pas bloquer le thread principal : Par exemple, le thread principal sur iOS gère les interactions UI, donc toute autre fonctionnalité qui pourrait faire planter l'application (recherche, accès à un web, lecture d'un fichier...) est gérée de cette manière.

### Blocks

Un bloc est une **section de code autonome** (comme une fonction avec des arguments retournant une valeur) et peut également spécifier des variables liées.\
Cependant, au niveau du compilateur, les blocs n'existent pas, ce sont des `os_object`s. Chacun de ces objets est formé par deux structures :

* **littéral de bloc** :&#x20;
* Il commence par le champ **`isa`**, pointant vers la classe du bloc :
* `NSConcreteGlobalBlock` (blocs de `__DATA.__const`)
* `NSConcreteMallocBlock` (blocs dans le tas)
* `NSConcreateStackBlock` (blocs dans la pile)
* Il a des **`flags`** (indiquant les champs présents dans le descripteur de bloc) et quelques octets réservés
* Le pointeur de fonction à appeler
* Un pointeur vers le descripteur de bloc
* Variables importées du bloc (le cas échéant)
* **descripteur de bloc** : Sa taille dépend des données présentes (comme indiqué dans les flags précédents)
* Il a quelques octets réservés
* Sa taille
* Il aura généralement un pointeur vers une signature de style Objective-C pour savoir combien d'espace est nécessaire pour les paramètres (flag `BLOCK_HAS_SIGNATURE`)
* Si des variables sont référencées, ce bloc aura également des pointeurs vers un helper de copie (copiant la valeur au début) et un helper de suppression (libérant).

### Queues

Une queue de dispatch est un objet nommé fournissant un ordre FIFO des blocs pour les exécutions.

Les blocs sont mis dans des queues pour être exécutés, et celles-ci supportent 2 modes : `DISPATCH_QUEUE_SERIAL` et `DISPATCH_QUEUE_CONCURRENT`. Bien sûr, la **série** ne **posera pas de problèmes de conditions de course** car un bloc ne sera pas exécuté tant que le précédent n'a pas terminé. Mais **l'autre type de queue pourrait en avoir**.

Queues par défaut :

* `.main-thread`: De `dispatch_get_main_queue()`
* `.libdispatch-manager`: Gestionnaire de queue de GCD
* `.root.libdispatch-manager`: Gestionnaire de queue de GCD
* `.root.maintenance-qos`: Tâches de priorité la plus basse
* `.root.maintenance-qos.overcommit`
* `.root.background-qos`: Disponible en tant que `DISPATCH_QUEUE_PRIORITY_BACKGROUND`
* `.root.background-qos.overcommit`
* `.root.utility-qos`: Disponible en tant que `DISPATCH_QUEUE_PRIORITY_NON_INTERACTIVE`
* `.root.utility-qos.overcommit`
* `.root.default-qos`: Disponible en tant que `DISPATCH_QUEUE_PRIORITY_DEFAULT`
* `.root.background-qos.overcommit`
* `.root.user-initiated-qos`: Disponible en tant que `DISPATCH_QUEUE_PRIORITY_HIGH`
* `.root.background-qos.overcommit`
* `.root.user-interactive-qos`: Priorité la plus élevée
* `.root.background-qos.overcommit`

Remarquez que c'est le système qui décide **quels threads gèrent quelles queues à chaque instant** (plusieurs threads peuvent travailler dans la même queue ou le même thread peut travailler dans différentes queues à un moment donné)

#### Attributs

Lors de la création d'une queue avec **`dispatch_queue_create`**, le troisième argument est un `dispatch_queue_attr_t`, qui est généralement soit `DISPATCH_QUEUE_SERIAL` (qui est en fait NULL) ou `DISPATCH_QUEUE_CONCURRENT`, qui est un pointeur vers une structure `dispatch_queue_attr_t` permettant de contrôler certains paramètres de la queue.

### Objets de dispatch

Il existe plusieurs objets que libdispatch utilise et les queues et blocs ne sont que 2 d'entre eux. Il est possible de créer ces objets avec `dispatch_object_create` :

* `block`
* `data`: Blocs de données
* `group`: Groupe de blocs
* `io`: Requêtes I/O asynchrones
* `mach`: Ports Mach
* `mach_msg`: Messages Mach
* `pthread_root_queue`: Une queue avec un pool de threads pthread et non des workqueues
* `queue`
* `semaphore`
* `source`: Source d'événements

## Objective-C

En Objective-C, il existe différentes fonctions pour envoyer un bloc à exécuter en parallèle :

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Soumet un bloc pour une exécution asynchrone sur une queue de dispatch et retourne immédiatement.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Soumet un objet bloc pour exécution et retourne après que ce bloc ait fini d'exécuter.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Exécute un objet bloc une seule fois pour la durée de vie d'une application.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Soumet un élément de travail pour exécution et retourne uniquement après qu'il ait fini d'exécuter. Contrairement à [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), cette fonction respecte tous les attributs de la queue lorsqu'elle exécute le bloc.

Ces fonctions attendent ces paramètres : [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Ceci est la **structure d'un Bloc** :
```c
struct Block {
void *isa; // NSConcreteStackBlock,...
int flags;
int reserved;
void *invoke;
struct BlockDescriptor *descriptor;
// captured variables go here
};
```
Et voici un exemple d'utilisation du **parallélisme** avec **`dispatch_async`** :
```objectivec
#import <Foundation/Foundation.h>

// Define a block
void (^backgroundTask)(void) = ^{
// Code to be executed in the background
for (int i = 0; i < 10; i++) {
NSLog(@"Background task %d", i);
sleep(1);  // Simulate a long-running task
}
};

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Create a dispatch queue
dispatch_queue_t backgroundQueue = dispatch_queue_create("com.example.backgroundQueue", NULL);

// Submit the block to the queue for asynchronous execution
dispatch_async(backgroundQueue, backgroundTask);

// Continue with other work on the main queue or thread
for (int i = 0; i < 10; i++) {
NSLog(@"Main task %d", i);
sleep(1);  // Simulate a long-running task
}
}
return 0;
}
```
## Swift

**`libswiftDispatch`** est une bibliothèque qui fournit des **liaisons Swift** au framework Grand Central Dispatch (GCD) qui est à l'origine écrit en C.\
La bibliothèque **`libswiftDispatch`** enveloppe les API C GCD dans une interface plus conviviale pour Swift, facilitant ainsi le travail des développeurs Swift avec GCD.

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**Exemple de code**:
```swift
import Foundation

// Define a closure (the Swift equivalent of a block)
let backgroundTask: () -> Void = {
for i in 0..<10 {
print("Background task \(i)")
sleep(1)  // Simulate a long-running task
}
}

// Entry point
autoreleasepool {
// Create a dispatch queue
let backgroundQueue = DispatchQueue(label: "com.example.backgroundQueue")

// Submit the closure to the queue for asynchronous execution
backgroundQueue.async(execute: backgroundTask)

// Continue with other work on the main queue
for i in 0..<10 {
print("Main task \(i)")
sleep(1)  // Simulate a long-running task
}
}
```
## Frida

Le script Frida suivant peut être utilisé pour **s'accrocher à plusieurs fonctions `dispatch`** et extraire le nom de la file d'attente, la trace de retour et le bloc : [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
```bash
frida -U <prog_name> -l libdispatch.js

dispatch_sync
Calling queue: com.apple.UIKit._UIReusePool.reuseSetAccess
Callback function: 0x19e3a6488 UIKitCore!__26-[_UIReusePool addObject:]_block_invoke
Backtrace:
0x19e3a6460 UIKitCore!-[_UIReusePool addObject:]
0x19e3a5db8 UIKitCore!-[UIGraphicsRenderer _enqueueContextForReuse:]
0x19e3a57fc UIKitCore!+[UIGraphicsRenderer _destroyCGContext:withRenderer:]
[...]
```
## Ghidra

Actuellement, Ghidra ne comprend ni la structure ObjectiveC **`dispatch_block_t`**, ni celle de **`swift_dispatch_block`**.

Donc, si vous voulez qu'il les comprenne, vous pouvez simplement **les déclarer** :

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

Ensuite, trouvez un endroit dans le code où ils sont **utilisés** :

{% hint style="success" %}
Notez toutes les références faites à "block" pour comprendre comment vous pourriez déterminer que la structure est utilisée.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1164).png" alt="" width="563"><figcaption></figcaption></figure>

Cliquez avec le bouton droit sur la variable -> Retaper la variable et sélectionnez dans ce cas **`swift_dispatch_block`** :

<figure><img src="../../.gitbook/assets/image (1165).png" alt="" width="563"><figcaption></figcaption></figure>

Ghidra réécrira automatiquement tout :

<figure><img src="../../.gitbook/assets/image (1166).png" alt="" width="563"><figcaption></figcaption></figure>

## Références

* [**\*OS Internals, Volume I: User Mode. Par Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

{% hint style="success" %}
Apprenez et pratiquez le hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenir HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop) !
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** nous sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
{% endhint %}
