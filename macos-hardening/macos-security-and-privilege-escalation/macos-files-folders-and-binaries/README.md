# macOS Arquivos, Pastas, Binários e Memória

{% hint style="success" %}
Aprenda e pratique Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprenda e pratique Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Confira os [**planos de assinatura**](https://github.com/sponsors/carlospolop)!
* **Junte-se ao** 💬 [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga**-nos no **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>
{% endhint %}

## Layout da Hierarquia de Arquivos

* **/Applications**: Os aplicativos instalados devem estar aqui. Todos os usuários poderão acessá-los.
* **/bin**: Binários de linha de comando
* **/cores**: Se existir, é usado para armazenar dumps de núcleo
* **/dev**: Tudo é tratado como um arquivo, então você pode ver dispositivos de hardware armazenados aqui.
* **/etc**: Arquivos de configuração
* **/Library**: Muitas subpastas e arquivos relacionados a preferências, caches e logs podem ser encontrados aqui. Uma pasta Library existe na raiz e no diretório de cada usuário.
* **/private**: Não documentado, mas muitas das pastas mencionadas são links simbólicos para o diretório privado.
* **/sbin**: Binários essenciais do sistema (relacionados à administração)
* **/System**: Arquivo para fazer o OS X funcionar. Você deve encontrar principalmente apenas arquivos específicos da Apple aqui (não de terceiros).
* **/tmp**: Arquivos são excluídos após 3 dias (é um link simbólico para /private/tmp)
* **/Users**: Diretório inicial para usuários.
* **/usr**: Configuração e binários do sistema
* **/var**: Arquivos de log
* **/Volumes**: As unidades montadas aparecerão aqui.
* **/.vol**: Executando `stat a.txt` você obtém algo como `16777223 7545753 -rw-r--r-- 1 username wheel ...` onde o primeiro número é o número de id do volume onde o arquivo existe e o segundo é o número do inode. Você pode acessar o conteúdo deste arquivo através de /.vol/ com essa informação executando `cat /.vol/16777223/7545753`

### Pastas de Aplicativos

* **Aplicativos do sistema** estão localizados em `/System/Applications`
* **Aplicativos instalados** geralmente são instalados em `/Applications` ou em `~/Applications`
* **Dados de aplicativos** podem ser encontrados em `/Library/Application Support` para os aplicativos executando como root e `~/Library/Application Support` para aplicativos executando como o usuário.
* Daemons de aplicativos de terceiros que **precisam ser executados como root** geralmente estão localizados em `/Library/PrivilegedHelperTools/`
* Aplicativos **Sandboxed** são mapeados na pasta `~/Library/Containers`. Cada aplicativo tem uma pasta nomeada de acordo com o ID do bundle do aplicativo (`com.apple.Safari`).
* O **kernel** está localizado em `/System/Library/Kernels/kernel`
* **Extensões do kernel da Apple** estão localizadas em `/System/Library/Extensions`
* **Extensões do kernel de terceiros** são armazenadas em `/Library/Extensions`

### Arquivos com Informações Sensíveis

MacOS armazena informações como senhas em vários lugares:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Instaladores pkg Vulneráveis

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## Extensões Específicas do OS X

* **`.dmg`**: Arquivos de Imagem de Disco da Apple são muito frequentes para instaladores.
* **`.kext`**: Deve seguir uma estrutura específica e é a versão do OS X de um driver. (é um bundle)
* **`.plist`**: Também conhecido como lista de propriedades, armazena informações em formato XML ou binário.
* Pode ser XML ou binário. Os binários podem ser lidos com:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plist`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Aplicativos da Apple que seguem a estrutura de diretório (é um bundle).
* **`.dylib`**: Bibliotecas dinâmicas (como arquivos DLL do Windows)
* **`.pkg`**: São os mesmos que xar (formato de Arquivo Extensível). O comando de instalador pode ser usado para instalar o conteúdo desses arquivos.
* **`.DS_Store`**: Este arquivo está em cada diretório, ele salva os atributos e personalizações do diretório.
* **`.Spotlight-V100`**: Esta pasta aparece no diretório raiz de cada volume no sistema.
* **`.metadata_never_index`**: Se este arquivo estiver na raiz de um volume, o Spotlight não indexará esse volume.
* **`.noindex`**: Arquivos e pastas com esta extensão não serão indexados pelo Spotlight.
* **`.sdef`**: Arquivos dentro de bundles especificando como é possível interagir com o aplicativo a partir de um AppleScript.

### Bundles do macOS

Um bundle é um **diretório** que **parece um objeto no Finder** (um exemplo de Bundle são arquivos `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Cache de Biblioteca Compartilhada Dyld (SLC)

No macOS (e iOS) todas as bibliotecas compartilhadas do sistema, como frameworks e dylibs, são **combinadas em um único arquivo**, chamado de **cache compartilhado dyld**. Isso melhora o desempenho, já que o código pode ser carregado mais rapidamente.

Isso está localizado no macOS em `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` e em versões mais antigas você pode encontrar o **cache compartilhado** em **`/System/Library/dyld/`**.\
No iOS você pode encontrá-los em **`/System/Library/Caches/com.apple.dyld/`**.

Semelhante ao cache compartilhado dyld, o kernel e as extensões do kernel também são compilados em um cache do kernel, que é carregado na inicialização.

Para extrair as bibliotecas do único arquivo de cache compartilhado dylib, foi possível usar o binário [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) que pode não estar funcionando atualmente, mas você também pode usar [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
Observe que mesmo que a ferramenta `dyld_shared_cache_util` não funcione, você pode passar o **binário dyld compartilhado para o Hopper** e o Hopper será capaz de identificar todas as bibliotecas e permitir que você **selecione qual** deseja investigar:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1152).png" alt="" width="563"><figcaption></figcaption></figure>

Alguns extratores não funcionarão, pois os dylibs estão pré-vinculados com endereços codificados, portanto, podem estar pulando para endereços desconhecidos.

{% hint style="success" %}
Também é possível baixar o Cache de Biblioteca Compartilhada de outros dispositivos \*OS no macos usando um emulador no Xcode. Eles serão baixados dentro de: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, como: `$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Mapeamento SLC

**`dyld`** usa a syscall **`shared_region_check_np`** para saber se o SLC foi mapeado (o que retorna o endereço) e **`shared_region_map_and_slide_np`** para mapear o SLC.

Observe que mesmo que o SLC seja deslizante no primeiro uso, todos os **processos** usam a **mesma cópia**, o que **elimina a proteção ASLR** se o atacante conseguir executar processos no sistema. Isso foi, na verdade, explorado no passado e corrigido com o pager de região compartilhada.

Os pools de ramificação são pequenos dylibs Mach-O que criam pequenos espaços entre os mapeamentos de imagem, tornando impossível interpor as funções.

### Substituir SLCs

Usando as variáveis de ambiente:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Isso permitirá carregar um novo cache de biblioteca compartilhada.
* **`DYLD_SHARED_CACHE_DIR=avoid`** e substituir manualmente as bibliotecas por symlinks para o cache compartilhado com as reais (você precisará extraí-las).

## Permissões Especiais de Arquivo

### Permissões de Pasta

Em uma **pasta**, **ler** permite **listá-la**, **escrever** permite **deletar** e **escrever** arquivos nela, e **executar** permite **navegar** pelo diretório. Portanto, por exemplo, um usuário com **permissão de leitura sobre um arquivo** dentro de um diretório onde ele **não tem permissão de execução** **não poderá ler** o arquivo.

### Modificadores de Flag

Existem algumas flags que podem ser definidas nos arquivos que farão o arquivo se comportar de maneira diferente. Você pode **verificar as flags** dos arquivos dentro de um diretório com `ls -lO /path/directory`

* **`uchg`**: Conhecida como flag **uchange**, **impede qualquer ação** de alteração ou exclusão do **arquivo**. Para defini-la, faça: `chflags uchg file.txt`
* O usuário root pode **remover a flag** e modificar o arquivo.
* **`restricted`**: Esta flag faz com que o arquivo seja **protegido pelo SIP** (você não pode adicionar esta flag a um arquivo).
* **`Sticky bit`**: Se um diretório tiver o sticky bit, **apenas** o **proprietário do diretório ou root pode renomear ou deletar** arquivos. Normalmente, isso é definido no diretório /tmp para impedir que usuários comuns excluam ou movam arquivos de outros usuários.

Todas as flags podem ser encontradas no arquivo `sys/stat.h` (encontre-o usando `mdfind stat.h | grep stat.h`) e são:

* `UF_SETTABLE` 0x0000ffff: Máscara de flags alteráveis pelo proprietário.
* `UF_NODUMP` 0x00000001: Não despejar arquivo.
* `UF_IMMUTABLE` 0x00000002: O arquivo não pode ser alterado.
* `UF_APPEND` 0x00000004: Escritas no arquivo podem apenas adicionar.
* `UF_OPAQUE` 0x00000008: O diretório é opaco em relação à união.
* `UF_COMPRESSED` 0x00000020: O arquivo está comprimido (alguns sistemas de arquivos).
* `UF_TRACKED` 0x00000040: Sem notificações para exclusões/renomeações para arquivos com isso definido.
* `UF_DATAVAULT` 0x00000080: Direito necessário para leitura e escrita.
* `UF_HIDDEN` 0x00008000: Dica de que este item não deve ser exibido em uma GUI.
* `SF_SUPPORTED` 0x009f0000: Máscara de flags suportadas por superusuário.
* `SF_SETTABLE` 0x3fff0000: Máscara de flags alteráveis por superusuário.
* `SF_SYNTHETIC` 0xc0000000: Máscara de flags sintéticas somente leitura do sistema.
* `SF_ARCHIVED` 0x00010000: O arquivo está arquivado.
* `SF_IMMUTABLE` 0x00020000: O arquivo não pode ser alterado.
* `SF_APPEND` 0x00040000: Escritas no arquivo podem apenas adicionar.
* `SF_RESTRICTED` 0x00080000: Direito necessário para escrita.
* `SF_NOUNLINK` 0x00100000: O item não pode ser removido, renomeado ou montado.
* `SF_FIRMLINK` 0x00800000: O arquivo é um firmlink.
* `SF_DATALESS` 0x40000000: O arquivo é um objeto sem dados.

### **ACLs de Arquivo**

As **ACLs** de arquivo contêm **ACE** (Entradas de Controle de Acesso) onde permissões **mais granulares** podem ser atribuídas a diferentes usuários.

É possível conceder a um **diretório** essas permissões: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
E a um **arquivo**: `read`, `write`, `append`, `execute`.

Quando o arquivo contém ACLs, você encontrará um "+" ao listar as permissões, como em:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Você pode **ler os ACLs** do arquivo com:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Você pode encontrar **todos os arquivos com ACLs** com (isso é muito lento):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Extended Attributes

Atributos estendidos têm um nome e qualquer valor desejado, e podem ser vistos usando `ls -@` e manipulados usando o comando `xattr`. Alguns atributos estendidos comuns são:

* `com.apple.resourceFork`: Compatibilidade com fork de recurso. Também visível como `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Mecanismo de quarentena do Gatekeeper (III/6)
* `metadata:*`: MacOS: vários metadados, como `_backup_excludeItem`, ou `kMD*`
* `com.apple.lastuseddate` (#PS): Data da última utilização do arquivo
* `com.apple.FinderInfo`: MacOS: Informações do Finder (por exemplo, Tags de cor)
* `com.apple.TextEncoding`: Especifica a codificação de texto de arquivos de texto ASCII
* `com.apple.logd.metadata`: Usado pelo logd em arquivos em `/var/db/diagnostics`
* `com.apple.genstore.*`: Armazenamento geracional (`/.DocumentRevisions-V100` na raiz do sistema de arquivos)
* `com.apple.rootless`: MacOS: Usado pela Proteção de Integridade do Sistema para rotular arquivo (III/10)
* `com.apple.uuidb.boot-uuid`: Marcação do logd de épocas de inicialização com UUID único
* `com.apple.decmpfs`: MacOS: Compressão de arquivo transparente (II/7)
* `com.apple.cprotect`: \*OS: Dados de criptografia por arquivo (III/11)
* `com.apple.installd.*`: \*OS: Metadados usados pelo installd, por exemplo, `installType`, `uniqueInstallID`

### Resource Forks | macOS ADS

Esta é uma maneira de obter **Fluxos de Dados Alternativos em máquinas MacOS**. Você pode salvar conteúdo dentro de um atributo estendido chamado **com.apple.ResourceFork** dentro de um arquivo salvando-o em **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Você pode **encontrar todos os arquivos contendo este atributo estendido** com:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

O atributo estendido `com.apple.decmpfs` indica que o arquivo está armazenado criptografado, `ls -l` reportará um **tamanho de 0** e os dados comprimidos estão dentro desse atributo. Sempre que o arquivo for acessado, ele será descriptografado na memória.

Esse atributo pode ser visto com `ls -lO` indicado como comprimido porque arquivos comprimidos também são marcados com a flag `UF_COMPRESSED`. Se um arquivo comprimido for removido essa flag com `chflags nocompressed </path/to/file>`, o sistema não saberá que o arquivo foi comprimido e, portanto, não poderá descomprimir e acessar os dados (ele pensará que está realmente vazio).

A ferramenta afscexpand pode ser usada para forçar a descompressão de um arquivo.

## **Binaries Universais &** Formato Mach-o

Os binaries do Mac OS geralmente são compilados como **binaries universais**. Um **binary universal** pode **suportar múltiplas arquiteturas no mesmo arquivo**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## Memória do Processo macOS

## Dumping de memória do macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Categoria de Risco de Arquivos Mac OS

O diretório `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` é onde as informações sobre o **risco associado a diferentes extensões de arquivo são armazenadas**. Este diretório categoriza arquivos em vários níveis de risco, influenciando como o Safari lida com esses arquivos ao serem baixados. As categorias são as seguintes:

* **LSRiskCategorySafe**: Arquivos nesta categoria são considerados **completamente seguros**. O Safari abrirá automaticamente esses arquivos após serem baixados.
* **LSRiskCategoryNeutral**: Esses arquivos não vêm com avisos e **não são abertos automaticamente** pelo Safari.
* **LSRiskCategoryUnsafeExecutable**: Arquivos sob esta categoria **disparam um aviso** indicando que o arquivo é um aplicativo. Isso serve como uma medida de segurança para alertar o usuário.
* **LSRiskCategoryMayContainUnsafeExecutable**: Esta categoria é para arquivos, como arquivos compactados, que podem conter um executável. O Safari **disparará um aviso** a menos que possa verificar que todos os conteúdos são seguros ou neutros.

## Arquivos de log

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Contém informações sobre arquivos baixados, como a URL de onde foram baixados.
* **`/var/log/system.log`**: Log principal dos sistemas OSX. com.apple.syslogd.plist é responsável pela execução do syslogging (você pode verificar se está desativado procurando por "com.apple.syslogd" em `launchctl list`).
* **`/private/var/log/asl/*.asl`**: Estes são os Logs do Sistema da Apple que podem conter informações interessantes.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Armazena arquivos e aplicativos acessados recentemente através do "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Armazena itens para iniciar ao iniciar o sistema.
* **`$HOME/Library/Logs/DiskUtility.log`**: Arquivo de log para o aplicativo DiskUtility (informações sobre drives, incluindo USBs).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Dados sobre pontos de acesso sem fio.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lista de daemons desativados.

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
