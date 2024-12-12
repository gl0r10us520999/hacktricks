<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs exclusivos**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga**-me no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios github do** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>


# Informações Básicas

**AppArmor** é um aprimoramento do kernel para confinar **programas** a um conjunto **limitado** de **recursos** com **perfis por programa**. Os perfis podem **permitir** **capacidades** como acesso à rede, acesso a socket raw e a permissão para ler, escrever ou executar arquivos em caminhos correspondentes.

É um Controle de Acesso Obrigatório ou **MAC** que vincula atributos de **controle de acesso** **aos programas em vez de aos usuários**.\
O confinamento do AppArmor é fornecido através de **perfis carregados no kernel**, normalmente durante a inicialização.\
Os perfis do AppArmor podem estar em um de **dois modos**:

* **Enforcement**: Perfis carregados no modo de enforcement resultarão na **execução da política** definida no perfil **assim como na notificação** de tentativas de violação da política (seja via syslog ou auditd).
* **Complain**: Perfis no modo complain **não executarão a política**, mas em vez disso **notificarão** tentativas de **violação** da política.

O AppArmor difere de alguns outros sistemas MAC no Linux: é **baseado em caminhos**, permite a mistura de perfis nos modos de enforcement e complain, usa arquivos de inclusão para facilitar o desenvolvimento e tem uma barreira de entrada muito menor do que outros sistemas MAC populares.

## Componentes do AppArmor

* **Módulo do kernel**: Realiza o trabalho efetivo
* **Políticas**: Define o comportamento e o confinamento
* **Parser**: Carrega as políticas no kernel
* **Utilitários**: Programas em modo usuário para interagir com o apparmor

## Caminho dos Perfis

Os perfis do apparmor são geralmente salvos em _**/etc/apparmor.d/**_\
Com `sudo aa-status` você poderá listar os binários que estão restritos por algum perfil. Se você puder mudar o caractere "/" por um ponto no caminho de cada binário listado, você obterá o nome do perfil do apparmor dentro da pasta mencionada.

Por exemplo, um perfil do **apparmor** para _/usr/bin/man_ estará localizado em _/etc/apparmor.d/usr.bin.man_

## Comandos
```bash
aa-status     #check the current status
aa-enforce    #set profile to enforce mode (from disable or complain)
aa-complain   #set profile to complain mode (from diable or enforcement)
apparmor_parser #to load/reload an altered policy
aa-genprof    #generate a new profile
aa-logprof    #used to change the policy when the binary/program is changed
aa-mergeprof  #used to merge the policies
```
# Criando um perfil

* Para indicar o executável afetado, **caminhos absolutos e curingas** são permitidos (para correspondência de arquivos) para especificar arquivos.
* Para indicar o acesso que o binário terá sobre **arquivos**, os seguintes **controles de acesso** podem ser usados:
* **r** (leitura)
* **w** (escrita)
* **m** (mapear memória como executável)
* **k** (bloqueio de arquivo)
* **l** (criação de links rígidos)
* **ix** (para executar outro programa com a nova política herdada)
* **Px** (executar sob outro perfil, após limpar o ambiente)
* **Cx** (executar sob um perfil filho, após limpar o ambiente)
* **Ux** (executar sem restrições, após limpar o ambiente)
* **Variáveis** podem ser definidas nos perfis e podem ser manipuladas de fora do perfil. Por exemplo: @{PROC} e @{HOME} (adicionar #include \<tunables/global> ao arquivo de perfil)
* **Regras de negação são suportadas para sobrescrever regras de permissão**.

## aa-genprof

Para começar a criar um perfil facilmente, o apparmor pode ajudá-lo. É possível fazer com que o **apparmor inspecione as ações realizadas por um binário e depois permita que você decida quais ações deseja permitir ou negar**.\
Você só precisa executar:
```bash
sudo aa-genprof /path/to/binary
```
Então, em um console diferente, execute todas as ações que o binário normalmente realiza:
```bash
/path/to/binary -a dosomething
```
Então, no primeiro console pressione "**s**" e depois nas ações gravadas indique se deseja ignorar, permitir ou o que for. Quando terminar, pressione "**f**" e o novo perfil será criado em _/etc/apparmor.d/caminho.para.binario_

{% hint style="info" %}
Usando as teclas de seta, você pode selecionar o que deseja permitir/negar/o que for
{% endhint %}

## aa-easyprof

Você também pode criar um modelo de perfil apparmor de um binário com:
```bash
sudo aa-easyprof /path/to/binary
# vim:syntax=apparmor
# AppArmor policy for binary
# ###AUTHOR###
# ###COPYRIGHT###
# ###COMMENT###

#include <tunables/global>

# No template variables specified

"/path/to/binary" {
#include <abstractions/base>

# No abstractions specified

# No policy groups specified

# No read paths specified

# No write paths specified
}
```
{% hint style="info" %}
Observe que, por padrão, em um perfil criado, nada é permitido, portanto, tudo é negado. Você precisará adicionar linhas como `/etc/passwd r,` para permitir que o binário leia `/etc/passwd`, por exemplo.
{% endhint %}

Você pode então **forçar** o novo perfil com
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
## Modificando um perfil a partir de logs

A seguinte ferramenta lerá os logs e perguntará ao usuário se ele deseja permitir algumas das ações proibidas detectadas:
```bash
sudo aa-logprof
```
{% hint style="info" %}
Utilizando as teclas de seta, você pode selecionar o que deseja permitir/negar/qualquer coisa
{% endhint %}

## Gerenciando um Perfil
```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```
# Logs

Exemplo de logs **AUDIT** e **DENIED** do _/var/log/audit/audit.log_ do executável **`service_bin`**:
```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
Você também pode obter essa informação utilizando:
```bash
sudo aa-notify -s 1 -v
Profile: /bin/service_bin
Operation: open
Name: /etc/passwd
Denied: r
Logfile: /var/log/audit/audit.log

Profile: /bin/service_bin
Operation: open
Name: /etc/hosts
Denied: r
Logfile: /var/log/audit/audit.log

AppArmor denials: 2 (since Wed Jan  6 23:51:08 2021)
For more information, please see: https://wiki.ubuntu.com/DebuggingApparmor
```
# Apparmor no Docker

Observe como o perfil **docker-profile** do docker é carregado por padrão:
```bash
sudo aa-status
apparmor module is loaded.
50 profiles are loaded.
13 profiles are in enforce mode.
/sbin/dhclient
/usr/bin/lxc-start
/usr/lib/NetworkManager/nm-dhcp-client.action
/usr/lib/NetworkManager/nm-dhcp-helper
/usr/lib/chromium-browser/chromium-browser//browser_java
/usr/lib/chromium-browser/chromium-browser//browser_openjdk
/usr/lib/chromium-browser/chromium-browser//sanitized_helper
/usr/lib/connman/scripts/dhclient-script
docker-default
```
Por padrão, o **perfil Apparmor docker-default** é gerado a partir de [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

**Resumo do perfil docker-default**:

* **Acesso** a toda a **rede**
* **Nenhuma capacidade** é definida (No entanto, algumas capacidades virão da inclusão de regras básicas de base, ou seja, #include \<abstractions/base>)
* **Escrita** em qualquer arquivo **/proc** **não é permitida**
* Outros **subdiretórios**/**arquivos** de /**proc** e /**sys** têm acesso de leitura/escrita/trava/link/execução **negado**
* **Montagem** **não é permitida**
* **Ptrace** só pode ser executado em um processo que está confinado pelo **mesmo perfil apparmor**

Uma vez que você **execute um contêiner docker**, você deve ver a seguinte saída:
```bash
1 processes are in enforce mode.
docker-default (825)
```
Observe que o **apparmor bloqueará até privilégios de capacidades** concedidos ao contêiner por padrão. Por exemplo, ele poderá **bloquear a permissão de escrever dentro de /proc mesmo se a capacidade SYS_ADMIN for concedida** porque, por padrão, o perfil de apparmor do docker nega este acesso:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
Você precisa **desativar o apparmor** para contornar suas restrições:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
Observe que, por padrão, o **AppArmor** também **proibirá o container de montar** pastas internamente, mesmo com a capacidade SYS_ADMIN.

Note que você pode **adicionar/remover** **capacidades** ao container docker (isso ainda será restrito por métodos de proteção como **AppArmor** e **Seccomp**):

* `--cap-add=SYS_ADMIN` _concede_ a capacidade `SYS_ADMIN`
* `--cap-add=ALL` _concede_ todas as capacidades
* `--cap-drop=ALL --cap-add=SYS_PTRACE` remove todas as capacidades e concede apenas `SYS_PTRACE`

{% hint style="info" %}
Geralmente, quando você **descobre** que tem uma **capacidade privilegiada** disponível **dentro** de um container **docker** **mas** alguma parte do **exploit não está funcionando**, isso ocorrerá porque o **apparmor do docker estará impedindo**.
{% endhint %}

## Fuga do Docker AppArmor

Você pode descobrir qual **perfil apparmor está executando um container** usando:
```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```
Então, você pode executar a seguinte linha para **encontrar o perfil exato em uso**:
```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```
No caso incomum de você poder **modificar o perfil do apparmor docker e recarregá-lo**, você poderia remover as restrições e "bypassá-las".

<details>

<summary><strong>Aprenda hacking no AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

* Se você quer ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**material oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Junte-se ao grupo** 💬 [**Discord**](https://discord.gg/hRep4RUj7f) ou ao grupo [**telegram**](https://t.me/peass) ou **siga-me** no **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para os repositórios do GitHub** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
