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

**Grand Central Dispatch (GCD),** também conhecido como **libdispatch** (`libdispatch.dyld`), está disponível tanto no macOS quanto no iOS. É uma tecnologia desenvolvida pela Apple para otimizar o suporte a aplicativos para execução concorrente (multithreaded) em hardware multicore.

**GCD** fornece e gerencia **filas FIFO** às quais seu aplicativo pode **enviar tarefas** na forma de **objetos de bloco**. Blocos enviados para filas de dispatch são **executados em um pool de threads** totalmente gerenciado pelo sistema. O GCD cria automaticamente threads para executar as tarefas nas filas de dispatch e agenda essas tarefas para serem executadas nos núcleos disponíveis.

{% hint style="success" %}
Em resumo, para executar código em **paralelo**, processos podem enviar **blocos de código para o GCD**, que cuidará de sua execução. Portanto, os processos não criam novas threads; **o GCD executa o código fornecido com seu próprio pool de threads** (que pode aumentar ou diminuir conforme necessário).
{% endhint %}

Isso é muito útil para gerenciar a execução paralela com sucesso, reduzindo significativamente o número de threads que os processos criam e otimizando a execução paralela. Isso é ideal para tarefas que requerem **grande paralelismo** (força bruta?) ou para tarefas que não devem bloquear a thread principal: Por exemplo, a thread principal no iOS lida com interações de UI, então qualquer outra funcionalidade que poderia fazer o aplicativo travar (buscando, acessando um site, lendo um arquivo...) é gerenciada dessa forma.

### Blocks

Um bloco é uma **seção de código autocontida** (como uma função com argumentos retornando um valor) e também pode especificar variáveis vinculadas.\
No entanto, no nível do compilador, os blocos não existem, eles são `os_object`s. Cada um desses objetos é formado por duas estruturas:

* **literal de bloco**:&#x20;
* Começa pelo campo **`isa`**, apontando para a classe do bloco:
* `NSConcreteGlobalBlock` (blocos de `__DATA.__const`)
* `NSConcreteMallocBlock` (blocos na heap)
* `NSConcreateStackBlock` (blocos na pilha)
* Tem **`flags`** (indicando campos presentes no descritor do bloco) e alguns bytes reservados
* O ponteiro da função a ser chamada
* Um ponteiro para o descritor do bloco
* Variáveis importadas do bloco (se houver)
* **descritor de bloco**: Seu tamanho depende dos dados que estão presentes (como indicado nas flags anteriores)
* Tem alguns bytes reservados
* O tamanho dele
* Geralmente terá um ponteiro para uma assinatura no estilo Objective-C para saber quanto espaço é necessário para os parâmetros (flag `BLOCK_HAS_SIGNATURE`)
* Se variáveis forem referenciadas, este bloco também terá ponteiros para um helper de cópia (copiando o valor no início) e um helper de descarte (liberando-o).

### Queues

Uma fila de dispatch é um objeto nomeado que fornece ordenação FIFO de blocos para execuções.

Os blocos são definidos em filas para serem executados, e essas suportam 2 modos: `DISPATCH_QUEUE_SERIAL` e `DISPATCH_QUEUE_CONCURRENT`. Claro que a **serial** não terá problemas de condição de corrida, pois um bloco não será executado até que o anterior tenha terminado. Mas **o outro tipo de fila pode ter**.

Filas padrão:

* `.main-thread`: De `dispatch_get_main_queue()`
* `.libdispatch-manager`: Gerenciador de filas do GCD
* `.root.libdispatch-manager`: Gerenciador de filas do GCD
* `.root.maintenance-qos`: Tarefas de prioridade mais baixa
* `.root.maintenance-qos.overcommit`
* `.root.background-qos`: Disponível como `DISPATCH_QUEUE_PRIORITY_BACKGROUND`
* `.root.background-qos.overcommit`
* `.root.utility-qos`: Disponível como `DISPATCH_QUEUE_PRIORITY_NON_INTERACTIVE`
* `.root.utility-qos.overcommit`
* `.root.default-qos`: Disponível como `DISPATCH_QUEUE_PRIORITY_DEFAULT`
* `.root.background-qos.overcommit`
* `.root.user-initiated-qos`: Disponível como `DISPATCH_QUEUE_PRIORITY_HIGH`
* `.root.background-qos.overcommit`
* `.root.user-interactive-qos`: Maior prioridade
* `.root.background-qos.overcommit`

Observe que será o sistema quem decidirá **quais threads lidam com quais filas a cada momento** (múltiplas threads podem trabalhar na mesma fila ou a mesma thread pode trabalhar em diferentes filas em algum momento)

#### Atributos

Ao criar uma fila com **`dispatch_queue_create`**, o terceiro argumento é um `dispatch_queue_attr_t`, que geralmente é `DISPATCH_QUEUE_SERIAL` (que na verdade é NULL) ou `DISPATCH_QUEUE_CONCURRENT`, que é um ponteiro para uma estrutura `dispatch_queue_attr_t` que permite controlar alguns parâmetros da fila.

### Objetos de Dispatch

Existem vários objetos que a libdispatch usa e filas e blocos são apenas 2 deles. É possível criar esses objetos com `dispatch_object_create`:

* `block`
* `data`: Blocos de dados
* `group`: Grupo de blocos
* `io`: Solicitações de I/O assíncronas
* `mach`: Portas Mach
* `mach_msg`: Mensagens Mach
* `pthread_root_queue`: Uma fila com um pool de threads pthread e não workqueues
* `queue`
* `semaphore`
* `source`: Fonte de evento

## Objective-C

Em Objective-C, existem diferentes funções para enviar um bloco para ser executado em paralelo:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Envia um bloco para execução assíncrona em uma fila de dispatch e retorna imediatamente.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Envia um objeto de bloco para execução e retorna após esse bloco terminar de executar.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Executa um objeto de bloco apenas uma vez durante a vida útil de um aplicativo.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Envia um item de trabalho para execução e retorna apenas após ele terminar de executar. Ao contrário de [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), esta função respeita todos os atributos da fila ao executar o bloco.

Essas funções esperam esses parâmetros: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Esta é a **estrutura de um Bloco**:
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
E este é um exemplo de usar **parallelism** com **`dispatch_async`**:
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

**`libswiftDispatch`** é uma biblioteca que fornece **bindings Swift** para o framework Grand Central Dispatch (GCD), que foi originalmente escrito em C.\
A biblioteca **`libswiftDispatch`** envolve as APIs C do GCD em uma interface mais amigável ao Swift, facilitando e tornando mais intuitivo para os desenvolvedores Swift trabalharem com o GCD.

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**Exemplo de código**:
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

O seguinte script Frida pode ser usado para **interceptar várias funções `dispatch`** e extrair o nome da fila, o backtrace e o bloco: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
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

Atualmente, o Ghidra não entende nem a estrutura **`dispatch_block_t`** do ObjectiveC, nem a **`swift_dispatch_block`**.

Então, se você quiser que ele as entenda, você pode **declará-las**:

<figure><img src="../../.gitbook/assets/image (1160).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1162).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1163).png" alt="" width="563"><figcaption></figcaption></figure>

Em seguida, encontre um lugar no código onde elas são **usadas**:

{% hint style="success" %}
Observe todas as referências feitas a "block" para entender como você poderia descobrir que a struct está sendo usada.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1164).png" alt="" width="563"><figcaption></figcaption></figure>

Clique com o botão direito na variável -> Retype Variable e selecione, neste caso, **`swift_dispatch_block`**:

<figure><img src="../../.gitbook/assets/image (1165).png" alt="" width="563"><figcaption></figcaption></figure>

O Ghidra reescreverá automaticamente tudo:

<figure><img src="../../.gitbook/assets/image (1166).png" alt="" width="563"><figcaption></figcaption></figure>

## References

* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
