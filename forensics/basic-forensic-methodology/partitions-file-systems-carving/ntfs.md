# NTFS

## NTFS

<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras formas de apoiar o HackTricks:

- Se você deseja ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF**, confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
- Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
- Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
- **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
- **Compartilhe suas dicas de hacking enviando PRs para os** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>

## **NTFS**

**NTFS** (**New Technology File System**) é um sistema de arquivos de registro proprietário desenvolvido pela Microsoft.

O cluster é a menor unidade de tamanho no NTFS e o tamanho do cluster depende do tamanho de uma partição.

| Tamanho da partição      | Setores por cluster | Tamanho do cluster |
| ------------------------ | ------------------- | ------------------ |
| 512MB ou menos           | 1                   | 512 bytes          |
| 513MB-1024MB (1GB)       | 2                   | 1KB                |
| 1025MB-2048MB (2GB)      | 4                   | 2KB                |
| 2049MB-4096MB (4GB)      | 8                   | 4KB                |
| 4097MB-8192MB (8GB)      | 16                  | 8KB                |
| 8193MB-16,384MB (16GB)   | 32                  | 16KB               |
| 16,385MB-32,768MB (32GB) | 64                  | 32KB               |
| Maior que 32,768MB       | 128                 | 64KB               |

### **Espaço de Reserva**

Como a menor unidade de tamanho do NTFS é um **cluster**, cada arquivo ocupará vários clusters completos. Assim, é altamente provável que **cada arquivo ocupe mais espaço do que o necessário**. Esses **espaços não utilizados** **reservados** por um arquivo são chamados de **espaço de reserva** e as pessoas podem aproveitar essa área para **ocultar** **informações**.

![](<../../../.gitbook/assets/image (498).png>)

### **Setor de Boot do NTFS**

Ao formatar um volume NTFS, o programa de formatação aloca os primeiros 16 setores para o arquivo de metadados de Boot. O primeiro setor é um setor de boot com um código "bootstrap" e os 15 setores seguintes são o IPL (Initial Program Loader) do setor de boot. Para aumentar a confiabilidade do sistema de arquivos, o último setor de uma partição NTFS contém uma cópia sobressalente do setor de boot.

### **Tabela de Arquivos Mestre (MFT)**

O sistema de arquivos NTFS contém um arquivo chamado Tabela de Arquivos Mestre (MFT). Existe pelo menos **uma entrada na MFT para cada arquivo em um volume do sistema de arquivos NTFS**, incluindo a própria MFT. Todas as informações sobre um arquivo, incluindo seu **tamanho, carimbos de data e hora, permissões e conteúdo de dados**, são armazenadas em entradas da MFT ou em espaço fora da MFT descrito por entradas da MFT.

À medida que **arquivos são adicionados** a um volume do sistema de arquivos NTFS, mais entradas são adicionadas à MFT e a **MFT aumenta de tamanho**. Quando os **arquivos** são **excluídos** de um volume do sistema de arquivos NTFS, suas **entradas da MFT são marcadas como livres** e podem ser reutilizadas. No entanto, o espaço em disco alocado para essas entradas não é realocado, e o tamanho da MFT não diminui.

O sistema de arquivos NTFS **reserva espaço para a MFT para mantê-la o mais contígua possível** à medida que cresce. O espaço reservado pelo sistema de arquivos NTFS para a MFT em cada volume é chamado de **zona da MFT**. O espaço para arquivos e diretórios também é alocado a partir deste espaço, mas somente depois que todo o espaço do volume fora da zona da MFT foi alocado.

Dependendo do tamanho médio do arquivo e de outras variáveis, **tanto a zona da MFT reservada quanto o espaço não reservado no disco podem ser alocados primeiro** à medida que o disco se enche. Volumes com um pequeno número de arquivos relativamente grandes alocarão primeiro o espaço não reservado, enquanto volumes com um grande número de arquivos relativamente pequenos alocarão primeiro a zona da MFT. Em ambos os casos, a fragmentação da MFT começa a ocorrer quando uma região ou outra se torna totalmente alocada. Se o espaço não reservado estiver completamente alocado, o espaço para arquivos e diretórios do usuário será alocado a partir da zona da MFT. Se a zona da MFT estiver completamente alocada, o espaço para novas entradas da MFT será alocado a partir do espaço não reservado.

Os sistemas de arquivos NTFS também geram um **$MFTMirror**. Esta é uma **cópia** dos **primeiros 4 registros** da MFT: $MFT, $MFT Mirror, $Log, $Volume.

O NTFS reserva os primeiros 16 registros da tabela para informações especiais:

| Arquivo de Sistema     | Nome do Arquivo | Registro MFT | Propósito do Arquivo                                                                                                                                                                                                         |
| ---------------------- | --------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tabela de arquivos mestre | $Mft          | 0            | Contém um registro de arquivo base para cada arquivo e pasta em um volume NTFS. Se as informações de alocação para um arquivo ou pasta forem muito grandes para caber em um único registro, outros registros de arquivo também são alocados. |
| Tabela de arquivos mestre 2 | $MftMirr     | 1            | Uma imagem duplicada dos primeiros quatro registros da MFT. Este arquivo garante acesso à MFT em caso de falha de um setor.                                                                                                 |
| Arquivo de log         | $LogFile       | 2            | Contém uma lista de etapas de transação usadas para recuperabilidade do NTFS. O tamanho do arquivo de log depende do tamanho do volume e pode ser tão grande quanto 4 MB. É usado pelo Windows NT/2000 para restaurar a consistência do NTFS após uma falha do sistema. |
| Volume                | $Volume        | 3            | Contém informações sobre o volume, como o rótulo do volume e a versão do volume.                                                                                                                                               |
| Definições de atributo | $AttrDef       | 4            | Uma tabela de nomes de atributos, números e descrições.                                                                                                                                                                        |
| Índice do nome do arquivo raiz | $           | 5            | A pasta raiz.                                                                                                                                                                                                                 |
| Bitmap de cluster      | $Bitmap        | 6            | Uma representação do volume mostrando quais clusters estão em uso.                                                                                                                                                             |
| Setor de boot          | $Boot          | 7            | Inclui o BPB usado para montar o volume e código de carregador de inicialização adicional usado se o volume for inicializável.                                                                                                 |
| Arquivo de cluster ruim | $BadClus      | 8            | Contém clusters ruins para o volume.                                                                                                                                                                                          |
| Arquivo de segurança    | $Secure       | 9            | Contém descritores de segurança exclusivos para todos os arquivos dentro de um volume.                                                                                                                                         |
| Tabela de conversão de maiúsculas | $Upcase | 10           | Converte caracteres minúsculos em caracteres maiúsculos Unicode correspondentes.                                                                                                                                               |
| Arquivo de extensão NTFS | $Extend       | 11           | Usado para várias extensões opcionais, como cotas, dados de ponto de reanálise e identificadores de objeto.                                                                                                                    |
|                        |               | 12-15        | Reservado para uso futuro.                                                                                                                                                                                                    |
| Arquivo de gerenciamento de cotas | $Quota | 24           | Contém limites de cota atribuídos pelo usuário no espaço do volume.                                                                                                                                                            |
| Arquivo de ID de objeto | $ObjId       | 25           | Contém IDs de objeto de arquivo.                                                                                                                                                                                              |
| Arquivo de ponto de reanálise | $Reparse  | 26           | Este arquivo contém informações sobre arquivos e pastas no volume, incluindo dados de ponto de reanálise.                                                                                                                     |

### Cada entrada da MFT se parece com o seguinte:

![](<../../../.gitbook/assets/image (499).png>)

Observe como cada entrada começa com "FILE". Cada entrada ocupa 1024 bits. Portanto, após 1024 bits do início de uma entrada da MFT, você encontrará a próxima.

Usando o [**Active Disk Editor**](https://www.disk-editor.org/index.html), é muito fácil inspecionar a entrada de um arquivo na MFT. Basta clicar com o botão direito no arquivo e depois clicar em "Inspect File Record"

![](<../../../.gitbook/assets/image (500).png>)

![](<../../../.gitbook/assets/image (501).png>)

Verificando a flag **"In use"** é muito fácil saber se um arquivo foi excluído (um valor de **0x0 significa excluído**).

![](<../../../.gitbook/assets/image (510).png>)

Também é possível recuperar arquivos excluídos usando o FTKImager:

![](<../../../.gitbook/assets/image (502).png>)

### Atributos da MFT

Cada entrada da MFT possui vários atributos, conforme a seguinte imagem indica:

![](<../../../.gitbook/assets/image (506).png>)

Cada atributo indica algumas informações da entrada identificadas pelo tipo:

| Identificador de Tipo | Nome                     | Descrição                                                                                                       |
| ---------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 16                     | $STANDARD\_INFORMATION   | Informações gerais, como flags; as últimas vezes de acesso, escrita e criação; e o proprietário e ID de segurança. |
| 32                     | $ATTRIBUTE\_LIST         | Lista onde outros atributos de um arquivo podem ser encontrados.                                                  |
| 48                     | $FILE\_NAME              | Nome do arquivo, em Unicode, e as últimas vezes de acesso, escrita e criação.                                     |
| 64                     | $VOLUME\_VERSION         | Informações do volume. Existe apenas na versão 1.2 (Windows NT).                                                  |
| 64                     | $OBJECT\_ID              | Um identificador único de 16 bytes para o arquivo ou diretório. Existe apenas nas versões 3.0+ e posteriores (Windows 2000+).    |
| 80                     | $SECURITY\_ DESCRIPTOR   | As propriedades de controle de acesso e segurança do arquivo.                                                     |
| 96                     | $VOLUME\_NAME            | Nome do volume.                                                                                                 |
| 112                    | $VOLUME\_ INFORMATION    | Versão do sistema de arquivos e outras flags.                                                                    |
| 128                    | $DATA                    | Conteúdo do arquivo.                                                                                             |
| 144                    | $INDEX\_ROOT             | Nó raiz de uma árvore de índice.                                                                                |
| 160                    | $INDEX\_ALLOCATION       | Nós de uma árvore de índice enraizada no atributo $INDEX\_ROOT.                                                   |
| 176                    | $BITMAP                  | Um mapa de bits para o arquivo $MFT e para índices.                                                              |
| 192                    | $SYMBOLIC\_LINK          | Informações de link simbólico. Existe apenas na versão 1.2 (Windows NT).                                         |
| 192                    | $REPARSE\_POINT          | Contém dados sobre um ponto de reanálise, que é usado como um link simbólico na versão 3.0+ (Windows 2000+).    |
| 208                    | $EA\_INFORMATION         | Usado para compatibilidade com aplicativos OS/2 (HPFS).                                                         |
| 224                    | $EA                      | Usado para compatibilidade com aplicativos OS/2 (HPFS).                                                         |
| 256                    | $LOGGED\_UTILITY\_STREAM | Contém chaves e informações sobre atributos criptografados na versão 3.0+ (Windows 2000+).                         |

Por exemplo, o **tipo 48 (0x30)** identifica o **nome do arquivo**:

![](<../../../.gitbook/assets/image (508).png>)

Também é útil entender que **esses atributos podem ser residentes** (ou seja, existem dentro de um determinado registro da MFT) ou **não residentes** (ou seja, existem fora de um determinado registro da MFT, em outro lugar no disco, e são simplesmente referenciados no registro). Por exemplo, se o atributo **$Data é residente**, isso significa que **todo o arquivo é salvo na MFT**, se for não residente, então o conteúdo do arquivo está em outra parte do sistema de arquivos.

Alguns atributos interessantes:

- [$STANDARD\_INFORMATION](https://flatcap.org/linux-ntfs/ntfs/attributes/standard\_information.html) (entre outros):
  - Data de criação
  - Data de modificação
  - Data de acesso
  - Data de atualização da MFT
  - Permissões de arquivo do DOS
- [$FILE\_NAME](https://flatcap.org/linux-ntfs/ntfs/attributes/file\_name.html) (entre outros):
  - Nome do arquivo
  - Data de criação
  - Data de modificação
  - Data de acesso
  - Data de atualização da MFT
  - Tamanho alocado
  - Tamanho real
  - [Referência de arquivo](https://flatcap.org/linux-ntfs/ntfs/concepts/file\_reference.html) para o diretório pai.
- [$Data](https://flatcap.org/linux-ntfs/ntfs/attributes/data.html) (entre outros):
  - Contém os dados do arquivo ou a indicação dos setores onde os dados residem. No exemplo a seguir, o atributo de dados não é residente, então o atributo fornece informações sobre os setores onde os dados residem.

![](<../../../.gitbook/assets/image (507) (1) (1).png>)

![](<../../../.gitbook/assets/image (509).png>)

### Carimbos de Data e Hora do NTFS

![](<../../../.gitbook/assets/image (512).png>)

Outra ferramenta útil para analisar a MFT é o [**MFT2csv**](https://github.com/jschicht/Mft2Csv) (selecione o arquivo mft ou a imagem e pressione dump all e extract para extrair todos os objetos).\
Este programa extrairá todos os dados da MFT e os apresentará em formato CSV. Também pode ser usado para extrair arquivos.

![](<../../../.gitbook/assets/image (513).png>)

### $LOGFILE

O arquivo **`$LOGFILE`** contém **logs** sobre as **ações** que foram **realizadas** **em** **arquivos**. Ele também **salva** a **ação** que precisaria realizar em caso de **refazer** e a ação necessária para **voltar** ao **estado anterior**.\
Esses logs são úteis para a MFT reconstruir o sistema de arquivos em caso de algum tipo de erro. O tamanho máximo deste arquivo é **65536KB**.

Para inspecionar o `$LOGFILE`, você precisa extrair e inspecionar o `$MFT` previamente com [**MFT2csv**](https://github.com/jschicht/Mft2Csv).\
Em seguida, execute [**LogFileParser**](https://github.com/jschicht/LogFileParser) neste arquivo e selecione o arquivo `$LOGFILE` exportado e o CVS da inspeção do `$MFT`. Você obterá um arquivo CSV com os logs da atividade do sistema de arquivos registrados pelo log `$LOGFILE`.

![](<../../../.gitbook/assets/image (515).png>)

Filtrando por nomes de arquivos, você pode ver **todas as ações realizadas em um arquivo**:

![](<../../../.gitbook/assets/image (514).png>)

### $USNJnrl

O arquivo `$EXTEND/$USNJnrl/$J` é um fluxo de dados alternativo do arquivo `$EXTEND$USNJnrl`. Este artefato contém um **registro de alterações produzidas dentro do volume NTFS com mais detalhes do que o `$LOGFILE`**.

Para inspecionar este arquivo, você pode usar a ferramenta [**UsnJrnl2csv**](https://github.com
