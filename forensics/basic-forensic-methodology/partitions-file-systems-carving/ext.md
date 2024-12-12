<details>

<summary><strong>Aprenda hacking AWS do zero ao herói com</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Outras maneiras de apoiar o HackTricks:

* Se você quiser ver sua **empresa anunciada no HackTricks** ou **baixar o HackTricks em PDF** Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Adquira o [**swag oficial PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Junte-se ao** 💬 [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-nos** no **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositórios do github.

</details>


# Ext - Sistema de Arquivos Estendido

**Ext2** é o sistema de arquivos mais comum para **partições sem journaling** (**partições que não mudam muito**) como a partição de inicialização. **Ext3/4** são **com journaling** e são usados geralmente para as **outras partições**.

Todos os grupos de blocos no sistema de arquivos têm o mesmo tamanho e são armazenados sequencialmente. Isso permite ao kernel derivar facilmente a localização de um grupo de blocos em um disco a partir de seu índice inteiro.

Cada grupo de blocos contém as seguintes informações:

* Uma cópia do superbloco do sistema de arquivos
* Uma cópia dos descritores do grupo de blocos
* Um mapa de bits de blocos de dados que é usado para identificar os blocos livres dentro do grupo
* Um mapa de bits de inodes, que é usado para identificar os inodes livres dentro do grupo
* tabela de inodes: consiste em uma série de blocos consecutivos, cada um dos quais contém um número predefinido de inodes do Ext2. Todos os inodes têm o mesmo tamanho: 128 bytes. Um bloco de 1.024 bytes contém 8 inodes, enquanto um bloco de 4.096 bytes contém 32 inodes. Note que no Ext2, não é necessário armazenar em disco um mapeamento entre um número de inode e o número de bloco correspondente, porque o último valor pode ser derivado do número do grupo de blocos e da posição relativa dentro da tabela de inodes. Por exemplo, suponha que cada grupo de blocos contenha 4.096 inodes e que desejamos saber o endereço no disco do inode 13.021. Neste caso, o inode pertence ao terceiro grupo de blocos e seu endereço no disco é armazenado na 733ª entrada da tabela de inodes correspondente. Como pode ver, o número do inode é apenas uma chave usada pelos rotinas do Ext2 para recuperar rapidamente o descritor de inode apropriado no disco
* blocos de dados, contendo arquivos. Qualquer bloco que não contenha informações significativas é considerado livre.

![](<../../../.gitbook/assets/image (406).png>)

## Recursos Opcionais do Ext

**Recursos afetam onde** os dados estão localizados, **como** os dados são armazenados nos inodes e alguns deles podem fornecer **metadados adicionais** para análise, portanto os recursos são importantes no Ext.

O Ext possui recursos opcionais que seu sistema operacional pode ou não suportar, existem 3 possibilidades:

* Compatível
* Incompatível
* Compatível Somente Leitura: Pode ser montado, mas não para escrita

Se houver recursos **incompatíveis**, você não poderá montar o sistema de arquivos, pois o sistema operacional não saberá como acessar os dados.

{% hint style="info" %}
Um atacante suspeito pode ter extensões não padrão
{% endhint %}

**Qualquer utilitário** que leia o **superbloco** será capaz de indicar os **recursos** de um **sistema de arquivos Ext**, mas você também pode usar `file -sL /dev/sd*`

## Superbloco

O superbloco é os primeiros 1024 bytes do início e é repetido no primeiro bloco de cada grupo e contém:

* Tamanho do bloco
* Total de blocos
* Blocos por grupo de blocos
* Blocos reservados antes do primeiro grupo de blocos
* Total de inodes
* Inodes por grupo de blocos
* Nome do volume
* Última hora de gravação
* Última hora de montagem
* Caminho onde o sistema de arquivos foi montado pela última vez
* Status do sistema de arquivos (limpo?)

É possível obter essas informações de um arquivo de sistema de arquivos Ext usando:
```bash
fsstat -o <offsetstart> /pat/to/filesystem-file.ext
#You can get the <offsetstart> with the "p" command inside fdisk
```
Você também pode usar o aplicativo GUI gratuito: [https://www.disk-editor.org/index.html](https://www.disk-editor.org/index.html)\
Ou você também pode usar **python** para obter as informações do superbloco: [https://pypi.org/project/superblock/](https://pypi.org/project/superblock/)

## inodes

Os **inodes** contêm a lista de **blocos** que **contêm** os dados reais de um **arquivo**.\
Se o arquivo for grande, um inode **pode conter ponteiros** para **outros inodes** que apontam para os blocos/mais inodes que contêm os dados do arquivo.

![](<../../../.gitbook/assets/image (416).png>)

Nos sistemas de arquivos **Ext2** e **Ext3**, os inodes têm tamanho de **128B**, o **Ext4** atualmente usa **156B** mas aloca **256B** no disco para permitir uma expansão futura.

Estrutura do inode:

| Offset | Tamanho | Nome              | Descrição                                       |
| ------ | ------- | ----------------- | ----------------------------------------------- |
| 0x0    | 2       | Modo do Arquivo    | Modo do arquivo e tipo                          |
| 0x2    | 2       | UID               | 16 bits inferiores do ID do proprietário        |
| 0x4    | 4       | Tamanho Il         | 32 bits inferiores do tamanho do arquivo        |
| 0x8    | 4       | Atime             | Tempo de acesso em segundos desde a época       |
| 0xC    | 4       | Ctime             | Tempo de alteração em segundos desde a época    |
| 0x10   | 4       | Mtime             | Tempo de modificação em segundos desde a época  |
| 0x14   | 4       | Dtime             | Tempo de exclusão em segundos desde a época     |
| 0x18   | 2       | GID               | 16 bits inferiores do ID do grupo               |
| 0x1A   | 2       | Contagem de Hlink  | Contagem de links rígidos                      |
| 0xC    | 4       | Blocos Io         | 32 bits inferiores da contagem de blocos       |
| 0x20   | 4       | Flags             | Flags                                           |
| 0x24   | 4       | União osd1        | Linux: Versão I                                 |
| 0x28   | 69      | Bloco\[15]        | 15 aponta para bloco de dados                  |
| 0x64   | 4       | Versão            | Versão do arquivo para NFS                      |
| 0x68   | 4       | ACL de Arquivo baixo | 32 bits inferiores de atributos estendidos (ACL, etc) |
| 0x6C   | 4       | Tamanho do Arquivo hi | 32 bits superiores do tamanho do arquivo (somente ext4) |
| 0x70   | 4       | Fragmento obsoleto | Um endereço de fragmento obsoleto               |
| 0x74   | 12      | Osd 2             | Segunda união dependente do sistema operacional |
| 0x74   | 2       | Blocos hi         | 16 bits superiores da contagem de blocos       |
| 0x76   | 2       | ACL de Arquivo hi  | 16 bits superiores de atributos estendidos (ACL, etc) |
| 0x78   | 2       | UID hi            | 16 bits superiores do ID do proprietário        |
| 0x7A   | 2       | GID hi            | 16 bits superiores do ID do grupo               |
| 0x7C   | 2       | Checksum Io       | 16 bits inferiores do checksum do inode        |

"Modificar" é o horário da última vez que o _conteúdo_ do arquivo foi modificado. Isso é frequentemente chamado de "_mtime_".\
"Alterar" é o horário da última vez que o _inode_ do arquivo foi alterado, como ao alterar permissões, propriedade, nome do arquivo e o número de links rígidos. Frequentemente é chamado de "_ctime_".

Estrutura do inode estendida (Ext4):

| Offset | Tamanho | Nome         | Descrição                                     |
| ------ | ------- | ------------ | --------------------------------------------- |
| 0x80   | 2       | Tamanho Extra | Quantos bytes além dos 128 padrão são usados  |
| 0x82   | 2       | Checksum hi  | 16 bits superiores do checksum do inode       |
| 0x84   | 4       | Ctime extra  | Bits extras de tempo de alteração             |
| 0x88   | 4       | Mtime extra  | Bits extras de tempo de modificação           |
| 0x8C   | 4       | Atime extra  | Bits extras de tempo de acesso                |
| 0x90   | 4       | Crtime       | Tempo de criação do arquivo (segundos desde a época) |
| 0x94   | 4       | Crtime extra | Bits extras de tempo de criação do arquivo    |
| 0x98   | 4       | Versão hi    | 32 bits superiores da versão                  |
| 0x9C   |         | Não utilizado | Espaço reservado para expansões futuras       |

Inodes especiais:

| Inode | Propósito Especial                                   |
| ----- | ----------------------------------------------------- |
| 0     | Nenhum inode, a numeração começa em 1                 |
| 1     | Lista de blocos defeituosos                           |
| 2     | Diretório raiz                                       |
| 3     | Quotas de usuário                                    |
| 4     | Quotas de grupo                                      |
| 5     | Carregador de inicialização                           |
| 6     | Diretório de recuperação                              |
| 7     | Descritores de grupo reservados (para redimensionar o sistema de arquivos) |
| 8     | Diário                                               |
| 9     | Inode de exclusão (para snapshots)                    |
| 10    | Inode de réplica                                     |
| 11    | Primeiro inode não reservado (frequentemente lost + found) |

{% hint style="info" %}
Note que o tempo de criação só aparece no Ext4.
{% endhint %}

Ao saber o número do inode, você pode facilmente encontrar seu índice:

* **Grupo de blocos** ao qual um inode pertence: (Número do inode - 1) / (Inodes por grupo)
* **Índice dentro do grupo**: (Número do inode - 1) mod (Inodes/grupos)
* **Deslocamento** na **tabela de inodes**: Número do inode \* (Tamanho do inode)
* O "-1" é porque o inode 0 é indefinido (não utilizado)
```bash
ls -ali /bin | sort -n #Get all inode numbers and sort by them
stat /bin/ls #Get the inode information of a file
istat -o <start offset> /path/to/image.ext 657103 #Get information of that inode inside the given ext file
icat -o <start offset> /path/to/image.ext 657103 #Cat the file
```
Modo de Arquivo

| Número | Descrição                                                                                         |
| ------ | --------------------------------------------------------------------------------------------------- |
| **15** | **Reg/Slink-13/Socket-14**                                                                          |
| **14** | **Diretório/Bit de Bloco 13**                                                                          |
| **13** | **Dispositivo de Caractere/Bit de Bloco 14**                                                                        |
| **12** | **FIFO**                                                                                            |
| 11     | Set UID                                                                                             |
| 10     | Set GID                                                                                             |
| 9      | Bit Sticky (sem ele, qualquer pessoa com permissões de escrita e execução em um diretório pode excluir e renomear arquivos)  |
| 8      | Leitura do Proprietário                                                                                          |
| 7      | Escrita do Proprietário                                                                                         |
| 6      | Execução do Proprietário                                                                                          |
| 5      | Leitura do Grupo                                                                                          |
| 4      | Escrita do Grupo                                                                                         |
| 3      | Execução do Grupo                                                                                          |
| 2      | Leitura de Outros                                                                                         |
| 1      | Escrita de Outros                                                                                        |
| 0      | Execução de Outros                                                                                         |

Os bits em negrito (12, 13, 14, 15) indicam o tipo de arquivo que o arquivo é (um diretório, soquete...) apenas uma das opções em negrito pode existir.

Diretórios

| Deslocamento | Tamanho | Nome      | Descrição                                                                                                                                                  |
| ------ | ---- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0x0    | 4    | Inode     |                                                                                                                                                              |
| 0x4    | 2    | Comprimento do Registro   | Comprimento do registro                                                                                                                                                |
| 0x6    | 1    | Comprimento do Nome  | Comprimento do nome                                                                                                                                                  |
| 0x7    | 1    | Tipo de Arquivo | <p>0x00 Desconhecido<br>0x01 Regular</p><p>0x02 Diretório</p><p>0x03 Dispositivo de Caractere</p><p>0x04 Dispositivo de Bloco</p><p>0x05 FIFO</p><p>0x06 Soquete</p><p>0x07 Link Simbólico</p> |
| 0x8    |      | Nome      | String do nome (até 255 caracteres)                                                                                                                           |

**Para aumentar o desempenho, blocos de diretório de hash raiz podem ser usados.**

**Atributos Estendidos**

Podem ser armazenados em

* Espaço extra entre inodes (256 - tamanho do inode, geralmente = 100)
* Um bloco de dados apontado por file\_acl no inode

Podem ser usados para armazenar qualquer coisa como um atributo de usuário se o nome começar com "usuário". Dessa forma, os dados podem ser ocultados.

Entradas de Atributos Estendidos

| Deslocamento | Tamanho | Nome         | Descrição                                                                                                                                                                                                        |
| ------ | ---- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0x0    | 1    | Comprimento do Nome     | Comprimento do nome do atributo                                                                                                                                                                                           |
| 0x1    | 1    | Índice do Nome   | <p>0x0 = sem prefixo</p><p>0x1 = prefixo user.</p><p>0x2 = system.posix_acl_access</p><p>0x3 = system.posix_acl_default</p><p>0x4 = trusted.</p><p>0x6 = security.</p><p>0x7 = system.</p><p>0x8 = system.richacl</p> |
| 0x2    | 2    | Deslocamento do Valor   | Deslocamento a partir da primeira entrada de inode ou início do bloco                                                                                                                                                                    |
| 0x4    | 4    | Blocos de Valor | Bloco de disco onde o valor é armazenado ou zero para este bloco                                                                                                                                                               |
| 0x8    | 4    | Tamanho do Valor   | Comprimento do valor                                                                                                                                                                                                    |
| 0xC    | 4    | Hash         | Hash para atributos no bloco ou zero se no inode                                                                                                                                                                      |
| 0x10   |      | Nome         | Nome do atributo sem NULL final                                                                                                                                                                                   |
```bash
setfattr -n 'user.secret' -v 'This is a secret' file.txt #Save a secret using extended attributes
getfattr file.txt #Get extended attribute names of a file
getdattr -n 'user.secret' file.txt #Get extended attribute called "user.secret"
```
## Visualização do Sistema de Arquivos

Para ver o conteúdo do sistema de arquivos, você pode **usar a ferramenta gratuita**: [https://www.disk-editor.org/index.html](https://www.disk-editor.org/index.html)\
Ou você pode montá-lo em seu linux usando o comando `mount`.

[https://piazza.com/class\_profile/get\_resource/il71xfllx3l16f/inz4wsb2m0w2oz#:\~:text=O%20sistema%20de%20arquivos%20Ext2%20divide,menor%20tempo%20médio%20de%20busca%20no%20disco.](https://piazza.com/class\_profile/get\_resource/il71xfllx3l16f/inz4wsb2m0w2oz#:\~:text=O%20sistema%20de%20arquivos%20Ext2%20divide,menor%20tempo%20médio%20de%20busca%20no%20disco.)
