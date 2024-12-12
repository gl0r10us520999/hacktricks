# Análise de Firmware

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

## **Introdução**

O firmware é um software essencial que permite que os dispositivos operem corretamente, gerenciando e facilitando a comunicação entre os componentes de hardware e o software com o qual os usuários interagem. Ele é armazenado em memória permanente, garantindo que o dispositivo possa acessar instruções vitais a partir do momento em que é ligado, levando ao lançamento do sistema operacional. Examinar e potencialmente modificar o firmware é um passo crítico na identificação de vulnerabilidades de segurança.

## **Coleta de Informações**

**Coletar informações** é um passo inicial crítico para entender a composição de um dispositivo e as tecnologias que ele utiliza. Esse processo envolve a coleta de dados sobre:

* A arquitetura da CPU e o sistema operacional que ele executa
* Especificações do bootloader
* Layout de hardware e folhas de dados
* Métricas de código e localizações de origem
* Bibliotecas externas e tipos de licença
* Históricos de atualização e certificações regulatórias
* Diagramas arquiteturais e de fluxo
* Avaliações de segurança e vulnerabilidades identificadas

Para esse fim, ferramentas de **inteligência de código aberto (OSINT)** são inestimáveis, assim como a análise de quaisquer componentes de software de código aberto disponíveis por meio de processos de revisão manuais e automatizados. Ferramentas como [Coverity Scan](https://scan.coverity.com) e [LGTM da Semmle](https://lgtm.com/#explore) oferecem análise estática gratuita que pode ser aproveitada para encontrar possíveis problemas.

## **Obtenção do Firmware**

Obter o firmware pode ser abordado por vários meios, cada um com seu próprio nível de complexidade:

* **Diretamente** da fonte (desenvolvedores, fabricantes)
* **Construindo** a partir de instruções fornecidas
* **Baixando** de sites de suporte oficiais
* Utilizando consultas **Google dork** para encontrar arquivos de firmware hospedados
* Acessando armazenamento **em nuvem** diretamente, com ferramentas como [S3Scanner](https://github.com/sa7mon/S3Scanner)
* Interceptando **atualizações** por meio de técnicas de homem-no-meio
* **Extraindo** do dispositivo por meio de conexões como **UART**, **JTAG** ou **PICit**
* **Capturando** solicitações de atualização na comunicação do dispositivo
* Identificando e usando **pontos de atualização codificados**
* **Despejando** do bootloader ou rede
* **Removendo e lendo** o chip de armazenamento, quando tudo mais falha, usando ferramentas de hardware apropriadas

## Analisando o firmware

Agora que você **tem o firmware**, você precisa extrair informações sobre ele para saber como tratá-lo. Diferentes ferramentas que você pode usar para isso:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #print offsets in hex
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head # might find signatures in header
fdisk -lu <bin> #lists a drives partition and filesystems if multiple
```
Se não encontrar muito com essas ferramentas, verifique a **entropia** da imagem com `binwalk -E <bin>`, se a entropia for baixa, então é pouco provável que esteja criptografada. Se a entropia for alta, é provável que esteja criptografada (ou compactada de alguma forma).

Além disso, você pode usar essas ferramentas para extrair **arquivos incorporados no firmware**:

{% content-ref url="../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../../generic-methodologies-and-resources/basic-forensic-methodology/partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

Ou [**binvis.io**](https://binvis.io/#/) ([código](https://code.google.com/archive/p/binvis/)) para inspecionar o arquivo.

### Obtendo o Sistema de Arquivos

Com as ferramentas mencionadas anteriormente, como `binwalk -ev <bin>`, você deveria ter sido capaz de **extrair o sistema de arquivos**.\
O Binwalk geralmente extrai dentro de uma **pasta com o nome do tipo de sistema de arquivos**, que geralmente é um dos seguintes: squashfs, ubifs, romfs, rootfs, jffs2, yaffs2, cramfs, initramfs.

#### Extração Manual do Sistema de Arquivos

Às vezes, o binwalk **não terá o byte mágico do sistema de arquivos em suas assinaturas**. Nestes casos, use o binwalk para **encontrar o deslocamento do sistema de arquivos e esculpir o sistema de arquivos comprimido** do binário e **extrair manualmente** o sistema de arquivos de acordo com seu tipo usando os passos abaixo.
```
$ binwalk DIR850L_REVB.bin

DECIMAL HEXADECIMAL DESCRIPTION
----------------------------------------------------------------------------- ---

0 0x0 DLOB firmware header, boot partition: """"dev=/dev/mtdblock/1""""
10380 0x288C LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 5213748 bytes
1704052 0x1A0074 PackImg section delimiter tag, little endian size: 32256 bytes; big endian size: 8257536 bytes
1704084 0x1A0094 Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 8256900 bytes, 2688 inodes, blocksize: 131072 bytes, created: 2016-07-12 02:28:41
```
Execute o seguinte comando **dd** esculpindo o sistema de arquivos Squashfs.
```
$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs

8257536+0 records in

8257536+0 records out

8257536 bytes (8.3 MB, 7.9 MiB) copied, 12.5777 s, 657 kB/s
```
Alternativamente, o seguinte comando também pode ser executado.

`$ dd if=DIR850L_REVB.bin bs=1 skip=$((0x1A0094)) of=dir.squashfs`

* Para squashfs (usado no exemplo acima)

`$ unsquashfs dir.squashfs`

Os arquivos estarão no diretório "`squashfs-root`" posteriormente.

* Arquivos de arquivo CPIO

`$ cpio -ivd --no-absolute-filenames -F <bin>`

* Para sistemas de arquivos jffs2

`$ jefferson rootfsfile.jffs2`

* Para sistemas de arquivos ubifs com flash NAND

`$ ubireader_extract_images -u UBI -s <start_offset> <bin>`

`$ ubidump.py <bin>`

## Analisando Firmware

Uma vez que o firmware é obtido, é essencial dissecá-lo para entender sua estrutura e vulnerabilidades potenciais. Esse processo envolve a utilização de várias ferramentas para analisar e extrair dados valiosos da imagem do firmware.

### Ferramentas de Análise Inicial

Um conjunto de comandos é fornecido para inspeção inicial do arquivo binário (referido como `<bin>`). Esses comandos ajudam na identificação de tipos de arquivos, extração de strings, análise de dados binários e compreensão dos detalhes de partição e sistema de arquivos:
```bash
file <bin>
strings -n8 <bin>
strings -tx <bin> #prints offsets in hexadecimal
hexdump -C -n 512 <bin> > hexdump.out
hexdump -C <bin> | head #useful for finding signatures in the header
fdisk -lu <bin> #lists partitions and filesystems, if there are multiple
```
Para avaliar o status de criptografia da imagem, a **entropia** é verificada com `binwalk -E <bin>`. Baixa entropia sugere falta de criptografia, enquanto alta entropia indica possível criptografia ou compressão.

Para extrair **arquivos embutidos**, são recomendadas ferramentas e recursos como a documentação de **file-data-carving-recovery-tools** e o **binvis.io** para inspeção de arquivos.

### Extraindo o Sistema de Arquivos

Usando `binwalk -ev <bin>`, geralmente é possível extrair o sistema de arquivos, frequentemente para um diretório nomeado de acordo com o tipo de sistema de arquivos (por exemplo, squashfs, ubifs). No entanto, quando o **binwalk** falha em reconhecer o tipo de sistema de arquivos devido à ausência de bytes mágicos, a extração manual é necessária. Isso envolve usar o `binwalk` para localizar o deslocamento do sistema de arquivos, seguido pelo comando `dd` para extrair o sistema de arquivos:
```bash
$ binwalk DIR850L_REVB.bin

$ dd if=DIR850L_REVB.bin bs=1 skip=1704084 of=dir.squashfs
```
### Análise do Sistema de Arquivos

Com o sistema de arquivos extraído, a busca por falhas de segurança começa. A atenção é dada aos daemons de rede inseguros, credenciais codificadas, endpoints de API, funcionalidades do servidor de atualização, código não compilado, scripts de inicialização e binários compilados para análise offline.

**Locais** e **itens** chave a serem inspecionados incluem:

- **etc/shadow** e **etc/passwd** para credenciais de usuário
- Certificados SSL e chaves em **etc/ssl**
- Arquivos de configuração e scripts em busca de vulnerabilidades potenciais
- Binários embutidos para análise adicional
- Servidores web comuns de dispositivos IoT e binários

Várias ferramentas auxiliam na descoberta de informações sensíveis e vulnerabilidades dentro do sistema de arquivos:

- [**LinPEAS**](https://github.com/carlospolop/PEASS-ng) e [**Firmwalker**](https://github.com/craigz28/firmwalker) para busca de informações sensíveis
- [**The Firmware Analysis and Comparison Tool (FACT)**](https://github.com/fkie-cad/FACT\_core) para análise abrangente de firmware
- [**FwAnalyzer**](https://github.com/cruise-automation/fwanalyzer), [**ByteSweep**](https://gitlab.com/bytesweep/bytesweep), [**ByteSweep-go**](https://gitlab.com/bytesweep/bytesweep-go) e [**EMBA**](https://github.com/e-m-b-a/emba) para análise estática e dinâmica

### Verificações de Segurança em Binários Compilados

Tanto o código-fonte quanto os binários compilados encontrados no sistema de arquivos devem ser examinados em busca de vulnerabilidades. Ferramentas como **checksec.sh** para binários Unix e **PESecurity** para binários do Windows ajudam a identificar binários desprotegidos que poderiam ser explorados.

## Emulação de Firmware para Análise Dinâmica

O processo de emulação de firmware permite a **análise dinâmica** da operação de um dispositivo ou de um programa individual. Essa abordagem pode enfrentar desafios com dependências de hardware ou arquitetura, mas transferir o sistema de arquivos raiz ou binários específicos para um dispositivo com arquitetura e endianness correspondentes, como um Raspberry Pi, ou para uma máquina virtual pré-construída, pode facilitar testes adicionais.

### Emulação de Binários Individuais

Para examinar programas individuais, é crucial identificar a endianness e a arquitetura da CPU do programa.

#### Exemplo com Arquitetura MIPS

Para emular um binário de arquitetura MIPS, pode-se usar o comando:
```bash
file ./squashfs-root/bin/busybox
```
E para instalar as ferramentas de emulação necessárias:
```bash
sudo apt-get install qemu qemu-user qemu-user-static qemu-system-arm qemu-system-mips qemu-system-x86 qemu-utils
```
### Emulação de Arquitetura ARM

Para binários ARM, o processo é semelhante, com o emulador `qemu-arm` sendo utilizado para a emulação.

### Emulação de Sistema Completo

Ferramentas como [Firmadyne](https://github.com/firmadyne/firmadyne), [Firmware Analysis Toolkit](https://github.com/attify/firmware-analysis-toolkit) e outras facilitam a emulação completa de firmware, automatizando o processo e auxiliando na análise dinâmica.

## Análise Dinâmica na Prática

Nesta etapa, é utilizado um ambiente de dispositivo real ou emulado para análise. É essencial manter acesso ao shell do sistema operacional e ao sistema de arquivos. A emulação pode não reproduzir perfeitamente as interações de hardware, exigindo reinicializações ocasionais da emulação. A análise deve revisitar o sistema de arquivos, explorar páginas da web expostas e serviços de rede, e investigar vulnerabilidades no bootloader. Testes de integridade do firmware são críticos para identificar possíveis vulnerabilidades de backdoor.

## Técnicas de Análise em Tempo de Execução

A análise em tempo de execução envolve interagir com um processo ou binário em seu ambiente operacional, utilizando ferramentas como gdb-multiarch, Frida e Ghidra para definir pontos de interrupção e identificar vulnerabilidades por meio de fuzzing e outras técnicas.

## Exploração de Binários e Prova de Conceito

Desenvolver uma PoC para vulnerabilidades identificadas requer um entendimento profundo da arquitetura alvo e programação em linguagens de baixo nível. Proteções de tempo de execução de binários em sistemas embarcados são raras, mas quando presentes, técnicas como Programação Orientada a Retorno (ROP) podem ser necessárias.

## Sistemas Operacionais Preparados para Análise de Firmware

Sistemas operacionais como [AttifyOS](https://github.com/adi0x90/attifyos) e [EmbedOS](https://github.com/scriptingxss/EmbedOS) fornecem ambientes pré-configurados para testes de segurança de firmware, equipados com ferramentas necessárias.

## Sistemas Operacionais Preparados para Analisar Firmware

* [**AttifyOS**](https://github.com/adi0x90/attifyos): AttifyOS é uma distribuição destinada a ajudar na avaliação de segurança e testes de penetração de dispositivos da Internet das Coisas (IoT). Economiza tempo fornecendo um ambiente pré-configurado com todas as ferramentas necessárias carregadas.
* [**EmbedOS**](https://github.com/scriptingxss/EmbedOS): Sistema operacional de teste de segurança embarcado baseado no Ubuntu 18.04 pré-carregado com ferramentas de teste de segurança de firmware.

## Firmware Vulnerável para Prática

Para praticar a descoberta de vulnerabilidades em firmware, utilize os seguintes projetos de firmware vulneráveis como ponto de partida.

* OWASP IoTGoat
* [https://github.com/OWASP/IoTGoat](https://github.com/OWASP/IoTGoat)
* The Damn Vulnerable Router Firmware Project
* [https://github.com/praetorian-code/DVRF](https://github.com/praetorian-code/DVRF)
* Damn Vulnerable ARM Router (DVAR)
* [https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html](https://blog.exploitlab.net/2018/01/dvar-damn-vulnerable-arm-router.html)
* ARM-X
* [https://github.com/therealsaumil/armx#downloads](https://github.com/therealsaumil/armx#downloads)
* Azeria Labs VM 2.0
* [https://azeria-labs.com/lab-vm-2-0/](https://azeria-labs.com/lab-vm-2-0/)
* Damn Vulnerable IoT Device (DVID)
* [https://github.com/Vulcainreo/DVID](https://github.com/Vulcainreo/DVID)

## Referências

* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)
* [Practical IoT Hacking: The Definitive Guide to Attacking the Internet of Things](https://www.amazon.co.uk/Practical-IoT-Hacking-F-Chantzis/dp/1718500904)

## Treinamento e Certificação

* [https://www.attify-store.com/products/offensive-iot-exploitation](https://www.attify-store.com/products/offensive-iot-exploitation)
