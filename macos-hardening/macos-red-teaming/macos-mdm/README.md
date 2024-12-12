# macOS MDM

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Treinamento HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Treinamento HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

**Para aprender sobre macOS MDMs, confira:**

* [https://www.youtube.com/watch?v=ku8jZe-MHUU](https://www.youtube.com/watch?v=ku8jZe-MHUU)
* [https://duo.com/labs/research/mdm-me-maybe](https://duo.com/labs/research/mdm-me-maybe)

## Conceitos Básicos

### **Visão Geral do MDM (Gerenciamento de Dispositivos Móveis)**

[Gerenciamento de Dispositivos Móveis](https://en.wikipedia.org/wiki/Mobile\_device\_management) (MDM) é utilizado para gerenciar vários dispositivos de usuários finais, como smartphones, laptops e tablets. Especificamente para as plataformas da Apple (iOS, macOS, tvOS), envolve um conjunto de recursos especializados, APIs e práticas. A operação do MDM depende de um servidor MDM compatível, que pode ser comercial ou de código aberto, e deve suportar o [Protocolo MDM](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Pontos-chave incluem:

* Controle centralizado sobre os dispositivos.
* Dependência de um servidor MDM que siga o protocolo MDM.
* Capacidade do servidor MDM de enviar vários comandos para os dispositivos, como apagamento remoto de dados ou instalação de configurações.

### **Fundamentos do DEP (Programa de Inscrição de Dispositivos)**

O [Programa de Inscrição de Dispositivos](https://www.apple.com/business/site/docs/DEP\_Guide.pdf) (DEP) oferecido pela Apple simplifica a integração do Gerenciamento de Dispositivos Móveis (MDM) ao facilitar a configuração sem intervenção para dispositivos iOS, macOS e tvOS. O DEP automatiza o processo de inscrição, permitindo que os dispositivos estejam operacionais imediatamente, com intervenção mínima do usuário ou administrativa. Aspectos essenciais incluem:

* Permite que os dispositivos se registrem autonomamente em um servidor MDM pré-definido durante a ativação inicial.
* Principalmente benéfico para dispositivos novos, mas também aplicável a dispositivos em reconfiguração.
* Facilita uma configuração simples, tornando os dispositivos prontos para uso organizacional rapidamente.

### **Consideração de Segurança**

É crucial observar que a facilidade de inscrição fornecida pelo DEP, embora benéfica, também pode representar riscos de segurança. Se as medidas de proteção não forem adequadamente aplicadas para a inscrição no MDM, os atacantes podem explorar esse processo simplificado para registrar seu dispositivo no servidor MDM da organização, se passando por um dispositivo corporativo.

{% hint style="danger" %}
**Alerta de Segurança**: A inscrição simplificada no DEP pode potencialmente permitir o registro de dispositivos não autorizados no servidor MDM da organização se as salvaguardas adequadas não estiverem em vigor.
{% endhint %}

### O que é SCEP (Protocolo Simples de Inscrição de Certificado)?

* Um protocolo relativamente antigo, criado antes da disseminação do TLS e HTTPS.
* Fornece aos clientes uma maneira padronizada de enviar uma **Solicitação de Assinatura de Certificado** (CSR) com o objetivo de obter um certificado. O cliente solicitará ao servidor um certificado assinado.

### O que são Perfis de Configuração (também conhecidos como mobileconfigs)?

* Forma oficial da Apple de **definir/impor configurações do sistema.**
* Formato de arquivo que pode conter vários payloads.
* Baseado em listas de propriedades (do tipo XML).
* "podem ser assinados e criptografados para validar sua origem, garantir sua integridade e proteger seu conteúdo." Conceitos Básicos — Página 70, Guia de Segurança do iOS, Janeiro de 2018.

## Protocolos

### MDM

* Combinação de APNs (**servidores da Apple**) + API RESTful (**servidores de fornecedores de MDM**)
* A **comunicação** ocorre entre um **dispositivo** e um servidor associado a um **produto de gerenciamento de dispositivos**
* **Comandos** entregues do MDM para o dispositivo em **dicionários codificados em plist**
* Tudo via **HTTPS**. Servidores MDM podem ser (e geralmente são) fixados.
* A Apple concede ao fornecedor de MDM um **certificado APNs** para autenticação

### DEP

* **3 APIs**: 1 para revendedores, 1 para fornecedores de MDM, 1 para identidade de dispositivo (não documentado):
* O chamado [API de "serviço em nuvem" do DEP](https://developer.apple.com/enterprise/documentation/MDM-Protocol-Reference.pdf). Isso é usado por servidores MDM para associar perfis DEP a dispositivos específicos.
* A [API DEP usada por Revendedores Autorizados da Apple](https://applecareconnect.apple.com/api-docs/depuat/html/WSImpManual.html) para inscrever dispositivos, verificar o status de inscrição e verificar o status da transação.
* A API DEP privada não documentada. Isso é usado por Dispositivos Apple para solicitar seu perfil DEP. No macOS, o binário `cloudconfigurationd` é responsável por se comunicar por essa API.
* Mais moderno e baseado em **JSON** (em vez de plist)
* A Apple concede um **token OAuth** ao fornecedor de MDM

**API de "serviço em nuvem" do DEP**

* RESTful
* sincroniza registros de dispositivos da Apple para o servidor MDM
* sincroniza perfis DEP para a Apple a partir do servidor MDM (entregue pela Apple para o dispositivo posteriormente)
* Um "perfil" DEP contém:
* URL do servidor do fornecedor de MDM
* Certificados adicionais confiáveis para URL do servidor (ponteiro opcional)
* Configurações adicionais (por exemplo, quais telas pular no Assistente de Configuração)

## Número de Série

Dispositivos Apple fabricados após 2010 geralmente possuem números de série alfanuméricos de **12 caracteres**, com os **três primeiros dígitos representando o local de fabricação**, os **dois seguintes indicando o **ano** e a **semana** de fabricação, os **três** dígitos seguintes fornecendo um **identificador** **único**, e os **últimos** **quatro** dígitos representando o **número do modelo**.

{% content-ref url="macos-serial-number.md" %}
[macos-serial-number.md](macos-serial-number.md)
{% endcontent-ref %}

## Etapas para inscrição e gerenciamento

1. Criação de registro do dispositivo (Revendedor, Apple): O registro do novo dispositivo é criado
2. Atribuição de registro do dispositivo (Cliente): O dispositivo é atribuído a um servidor MDM
3. Sincronização de registro do dispositivo (Fornecedor de MDM): O MDM sincroniza os registros do dispositivo e envia os perfis DEP para a Apple
4. Check-in DEP (Dispositivo): O dispositivo obtém seu perfil DEP
5. Recuperação de perfil (Dispositivo)
6. Instalação de perfil (Dispositivo) a. incl. payloads MDM, SCEP e root CA
7. Emissão de comando MDM (Dispositivo)

![](<../../../.gitbook/assets/image (694).png>)

O arquivo `/Library/Developer/CommandLineTools/SDKs/MacOSX10.15.sdk/System/Library/PrivateFrameworks/ConfigurationProfiles.framework/ConfigurationProfiles.tbd` exporta funções que podem ser consideradas **"etapas" de alto nível** do processo de inscrição.
### Passo 4: Verificação DEP - Obtendo o Registro de Ativação

Esta parte do processo ocorre quando um **usuário inicializa um Mac pela primeira vez** (ou após uma limpeza completa)

![](<../../../.gitbook/assets/image (1044).png>)

ou ao executar `sudo profiles show -type enrollment`

* Determinar se o dispositivo está habilitado para DEP
* Registro de Ativação é o nome interno para o **"perfil" DEP**
* Começa assim que o dispositivo é conectado à Internet
* Conduzido por **`CPFetchActivationRecord`**
* Implementado por **`cloudconfigurationd`** via XPC. O **"Assistente de Configuração"** (quando o dispositivo é inicializado pela primeira vez) ou o comando **`profiles`** irão **contatar esse daemon** para recuperar o registro de ativação.
* LaunchDaemon (sempre em execução como root)

Segue alguns passos para obter o Registro de Ativação realizado por **`MCTeslaConfigurationFetcher`**. Este processo utiliza uma criptografia chamada **Absinthe**

1. Recuperar **certificado**
1. GET [https://iprofiles.apple.com/resource/certificate.cer](https://iprofiles.apple.com/resource/certificate.cer)
2. **Inicializar** estado a partir do certificado (**`NACInit`**)
1. Usa vários dados específicos do dispositivo (por exemplo, **Número de Série via `IOKit`**)
3. Recuperar **chave de sessão**
1. POST [https://iprofiles.apple.com/session](https://iprofiles.apple.com/session)
4. Estabelecer a sessão (**`NACKeyEstablishment`**)
5. Fazer a solicitação
1. POST para [https://iprofiles.apple.com/macProfile](https://iprofiles.apple.com/macProfile) enviando os dados `{ "action": "RequestProfileConfiguration", "sn": "" }`
2. A carga JSON é criptografada usando Absinthe (**`NACSign`**)
3. Todas as solicitações são feitas via HTTPs, certificados raiz embutidos são usados

![](<../../../.gitbook/assets/image (566) (1).png>)

A resposta é um dicionário JSON com alguns dados importantes como:

* **url**: URL do host do fornecedor MDM para o perfil de ativação
* **anchor-certs**: Array de certificados DER usados como âncoras confiáveis

### **Passo 5: Recuperação do Perfil**

![](<../../../.gitbook/assets/image (444).png>)

* Solicitação enviada para **url fornecido no perfil DEP**.
* **Certificados âncora** são usados para **avaliar a confiança** se fornecidos.
* Lembrete: a propriedade **anchor\_certs** do perfil DEP
* **Solicitação é um .plist** simples com identificação do dispositivo
* Exemplos: **UDID, versão do SO**.
* Assinado por CMS, codificado em DER
* Assinado usando o **certificado de identidade do dispositivo (do APNS)**
* **Cadeia de certificados** inclui o expirado **Apple iPhone Device CA**

![](<../../../.gitbook/assets/image (567) (1) (2) (2) (2) (2) (2) (2) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2) (2).png>)

### Passo 6: Instalação do Perfil

* Uma vez recuperado, **o perfil é armazenado no sistema**
* Este passo começa automaticamente (se estiver no **assistente de configuração**)
* Conduzido por **`CPInstallActivationProfile`**
* Implementado pelo mdmclient via XPC
* LaunchDaemon (como root) ou LaunchAgent (como usuário), dependendo do contexto
* Os perfis de configuração têm vários payloads para instalar
* O framework tem uma arquitetura baseada em plugins para instalar perfis
* Cada tipo de payload está associado a um plugin
* Pode ser XPC (no framework) ou Cocoa clássico (no ManagedClient.app)
* Exemplo:
* Os payloads de Certificado usam CertificateService.xpc

Tipicamente, o **perfil de ativação** fornecido por um fornecedor MDM irá **incluir os seguintes payloads**:

* `com.apple.mdm`: para **inscrever** o dispositivo no MDM
* `com.apple.security.scep`: para fornecer de forma segura um **certificado de cliente** ao dispositivo.
* `com.apple.security.pem`: para **instalar certificados CA confiáveis** no Cadeado do Sistema do dispositivo.
* Instalando o payload MDM equivalente ao **check-in MDM na documentação**
* Payload **contém propriedades-chave**:
*
* URL de Check-In MDM (**`CheckInURL`**)
* URL de Polling de Comando MDM (**`ServerURL`**) + tópico APNs para ativá-lo
* Para instalar o payload MDM, a solicitação é enviada para **`CheckInURL`**
* Implementado em **`mdmclient`**
* O payload MDM pode depender de outros payloads
* Permite **solicitações serem fixadas em certificados específicos**:
* Propriedade: **`CheckInURLPinningCertificateUUIDs`**
* Propriedade: **`ServerURLPinningCertificateUUIDs`**
* Entregue via payload PEM
* Permite que o dispositivo seja atribuído com um certificado de identidade:
* Propriedade: IdentityCertificateUUID
* Entregue via payload SCEP

### **Passo 7: Escutando comandos MDM**

* Após o check-in MDM ser concluído, o fornecedor pode **emitir notificações push usando APNs**
* Ao receber, tratado por **`mdmclient`**
* Para consultar comandos MDM, a solicitação é enviada para ServerURL
* Faz uso do payload MDM previamente instalado:
* **`ServerURLPinningCertificateUUIDs`** para fixar a solicitação
* **`IdentityCertificateUUID`** para certificado de cliente TLS

## Ataques

### Inscrição de Dispositivos em Outras Organizações

Como mencionado anteriormente, para tentar inscrever um dispositivo em uma organização **é necessário apenas um Número de Série pertencente a essa Organização**. Uma vez que o dispositivo é inscrito, várias organizações irão instalar dados sensíveis no novo dispositivo: certificados, aplicativos, senhas de WiFi, configurações de VPN [e assim por diante](https://developer.apple.com/enterprise/documentation/Configuration-Profile-Reference.pdf).\
Portanto, este poderia ser um ponto de entrada perigoso para os atacantes se o processo de inscrição não estiver corretamente protegido:

{% content-ref url="enrolling-devices-in-other-organisations.md" %}
[enrolling-devices-in-other-organisations.md](enrolling-devices-in-other-organisations.md)
{% endcontent-ref %}

{% hint style="success" %}
Aprenda e pratique Hacking em AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking em GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoie o HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para os repositórios** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
