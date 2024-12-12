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


# Identificando binários empacotados

* **falta de strings**: É comum encontrar que binários empacotados não têm quase nenhuma string.
* Muitas **strings não utilizadas**: Além disso, quando um malware usa algum tipo de empacotador comercial, é comum encontrar muitas strings sem referências cruzadas. Mesmo que essas strings existam, isso não significa que o binário não esteja empacotado.
* Você também pode usar algumas ferramentas para tentar descobrir qual empacotador foi usado para empacotar um binário:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# Recomendações Básicas

* **Comece** a analisar o binário empacotado **de baixo para cima no IDA**. Desempacotadores saem uma vez que o código desempacotado sai, então é improvável que o desempacotador passe a execução para o código desempacotado no início.
* Procure por **JMP's** ou **CALLs** para **registradores** ou **regiões** de **memória**. Também procure por **funções que empurram argumentos e uma direção de endereço e depois chamam `retn`**, porque o retorno da função, nesse caso, pode chamar o endereço que foi apenas empurrado para a pilha antes de chamá-lo.
* Coloque um **breakpoint** em `VirtualAlloc`, pois isso aloca espaço na memória onde o programa pode escrever código desempacotado. "executar até o código do usuário" ou use F8 para **obter o valor dentro de EAX** após executar a função e "**seguir aquele endereço no dump**". Você nunca sabe se essa é a região onde o código desempacotado será salvo.
* **`VirtualAlloc`** com o valor "**40**" como argumento significa Leitura+Gravação+Execução (algum código que precisa de execução será copiado aqui).
* **Enquanto desempacota** o código, é normal encontrar **várias chamadas** para **operações aritméticas** e funções como **`memcopy`** ou **`Virtual`**`Alloc`. Se você se encontrar em uma função que aparentemente apenas realiza operações aritméticas e talvez algum `memcopy`, a recomendação é tentar **encontrar o final da função** (talvez um JMP ou chamada para algum registrador) **ou** pelo menos a **chamada para a última função** e executar até lá, pois o código não é interessante.
* Enquanto desempacota o código, **note** sempre que você **muda a região de memória**, pois uma mudança de região de memória pode indicar o **início do código desempacotado**. Você pode facilmente despejar uma região de memória usando o Process Hacker (processo --> propriedades --> memória).
* Enquanto tenta desempacotar o código, uma boa maneira de **saber se você já está trabalhando com o código desempacotado** (para que você possa apenas despejá-lo) é **verificar as strings do binário**. Se em algum momento você realizar um salto (talvez mudando a região de memória) e notar que **muitas mais strings foram adicionadas**, então você pode saber **que está trabalhando com o código desempacotado**.\
No entanto, se o empacotador já contém muitas strings, você pode ver quantas strings contêm a palavra "http" e verificar se esse número aumenta.
* Quando você despeja um executável de uma região de memória, pode corrigir alguns cabeçalhos usando [PE-bear](https://github.com/hasherezade/pe-bear-releases/releases).


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
