# macOS System Extensions

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

## System Extensions / Endpoint Security Framework

Ao contrário das Kernel Extensions, **as System Extensions são executadas no espaço do usuário** em vez do espaço do kernel, reduzindo o risco de uma falha do sistema devido a mau funcionamento da extensão.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Existem três tipos de extensões de sistema: **DriverKit** Extensions, **Network** Extensions e **Endpoint Security** Extensions.

### **DriverKit Extensions**

DriverKit é um substituto para extensões de kernel que **fornecem suporte de hardware**. Ele permite que drivers de dispositivo (como drivers USB, Serial, NIC e HID) sejam executados no espaço do usuário em vez do espaço do kernel. O framework DriverKit inclui **versões de espaço do usuário de certas classes do I/O Kit**, e o kernel encaminha eventos normais do I/O Kit para o espaço do usuário, oferecendo um ambiente mais seguro para esses drivers.

### **Network Extensions**

As Network Extensions fornecem a capacidade de personalizar comportamentos de rede. Existem vários tipos de Network Extensions:

* **App Proxy**: Isso é usado para criar um cliente VPN que implementa um protocolo VPN personalizado orientado a fluxo. Isso significa que ele lida com o tráfego de rede com base em conexões (ou fluxos) em vez de pacotes individuais.
* **Packet Tunnel**: Isso é usado para criar um cliente VPN que implementa um protocolo VPN personalizado orientado a pacotes. Isso significa que ele lida com o tráfego de rede com base em pacotes individuais.
* **Filter Data**: Isso é usado para filtrar "fluxos" de rede. Ele pode monitorar ou modificar dados de rede no nível do fluxo.
* **Filter Packet**: Isso é usado para filtrar pacotes individuais de rede. Ele pode monitorar ou modificar dados de rede no nível do pacote.
* **DNS Proxy**: Isso é usado para criar um provedor DNS personalizado. Ele pode ser usado para monitorar ou modificar solicitações e respostas DNS.

## Endpoint Security Framework

O Endpoint Security é um framework fornecido pela Apple no macOS que oferece um conjunto de APIs para segurança do sistema. É destinado ao uso por **fornecedores de segurança e desenvolvedores para construir produtos que podem monitorar e controlar a atividade do sistema** para identificar e proteger contra atividades maliciosas.

Este framework fornece uma **coleção de APIs para monitorar e controlar a atividade do sistema**, como execuções de processos, eventos do sistema de arquivos, eventos de rede e do kernel.

O núcleo deste framework é implementado no kernel, como uma Kernel Extension (KEXT) localizada em **`/System/Library/Extensions/EndpointSecurity.kext`**. Este KEXT é composto por vários componentes-chave:

* **EndpointSecurityDriver**: Isso atua como o "ponto de entrada" para a extensão do kernel. É o principal ponto de interação entre o OS e o framework de Endpoint Security.
* **EndpointSecurityEventManager**: Este componente é responsável por implementar hooks do kernel. Hooks do kernel permitem que o framework monitore eventos do sistema interceptando chamadas de sistema.
* **EndpointSecurityClientManager**: Isso gerencia a comunicação com clientes do espaço do usuário, mantendo o controle de quais clientes estão conectados e precisam receber notificações de eventos.
* **EndpointSecurityMessageManager**: Isso envia mensagens e notificações de eventos para clientes do espaço do usuário.

Os eventos que o framework de Endpoint Security pode monitorar são categorizados em:

* Eventos de arquivo
* Eventos de processo
* Eventos de socket
* Eventos do kernel (como carregar/descarregar uma extensão de kernel ou abrir um dispositivo do I/O Kit)

### Arquitetura do Endpoint Security Framework

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

A **comunicação no espaço do usuário** com o framework de Endpoint Security acontece através da classe IOUserClient. Duas subclasses diferentes são usadas, dependendo do tipo de chamador:

* **EndpointSecurityDriverClient**: Isso requer a permissão `com.apple.private.endpoint-security.manager`, que é mantida apenas pelo processo do sistema `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Isso requer a permissão `com.apple.developer.endpoint-security.client`. Isso seria tipicamente usado por software de segurança de terceiros que precisa interagir com o framework de Endpoint Security.

As Endpoint Security Extensions:**`libEndpointSecurity.dylib`** é a biblioteca C que as extensões de sistema usam para se comunicar com o kernel. Esta biblioteca usa o I/O Kit (`IOKit`) para se comunicar com o KEXT de Endpoint Security.

**`endpointsecurityd`** é um daemon do sistema chave envolvido na gestão e lançamento de extensões de sistema de segurança de endpoint, particularmente durante o processo de inicialização inicial. **Apenas extensões de sistema** marcadas com **`NSEndpointSecurityEarlyBoot`** em seu arquivo `Info.plist` recebem esse tratamento de inicialização antecipada.

Outro daemon do sistema, **`sysextd`**, **valida extensões de sistema** e as move para os locais apropriados do sistema. Em seguida, ele pede ao daemon relevante para carregar a extensão. O **`SystemExtensions.framework`** é responsável por ativar e desativar extensões de sistema.

## Bypassing ESF

ESF é usado por ferramentas de segurança que tentarão detectar um red teamer, então qualquer informação sobre como isso poderia ser evitado soa interessante.

### CVE-2021-30965

A questão é que o aplicativo de segurança precisa ter **permissões de Acesso Completo ao Disco**. Portanto, se um atacante puder remover isso, ele poderá impedir que o software seja executado:
```bash
tccutil reset All
```
Para **mais informações** sobre este bypass e relacionados, confira a palestra [#OBTS v5.0: "O Calcanhar de Aquiles do EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

No final, isso foi corrigido ao dar a nova permissão **`kTCCServiceEndpointSecurityClient`** ao aplicativo de segurança gerenciado por **`tccd`**, para que `tccutil` não limpe suas permissões, impedindo-o de ser executado.

## Referências

* [**OBTS v3.0: "Segurança e Insegurança do Endpoint" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

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
