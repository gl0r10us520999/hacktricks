# Chaves de Registro do Windows Interessantes

### Chaves de Registro do Windows Interessantes

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


### **Informações sobre a Versão do Windows e Proprietário**
- Localizado em **`Software\Microsoft\Windows NT\CurrentVersion`**, você encontrará a versão do Windows, Service Pack, hora da instalação e o nome do proprietário registrado de forma direta.

### **Nome do Computador**
- O nome do host é encontrado em **`System\ControlSet001\Control\ComputerName\ComputerName`**.

### **Configuração do Fuso Horário**
- O fuso horário do sistema é armazenado em **`System\ControlSet001\Control\TimeZoneInformation`**.

### **Rastreamento do Tempo de Acesso**
- Por padrão, o rastreamento do último tempo de acesso está desativado (**`NtfsDisableLastAccessUpdate=1`**). Para ativá-lo, use:
`fsutil behavior set disablelastaccess 0`

### Versões do Windows e Service Packs
- A **versão do Windows** indica a edição (por exemplo, Home, Pro) e seu lançamento (por exemplo, Windows 10, Windows 11), enquanto os **Service Packs** são atualizações que incluem correções e, às vezes, novos recursos.

### Habilitando o Tempo de Último Acesso
- Habilitar o rastreamento do último tempo de acesso permite que você veja quando os arquivos foram abertos pela última vez, o que pode ser crítico para análise forense ou monitoramento de sistema.

### Detalhes da Informação de Rede
- O registro contém dados extensivos sobre configurações de rede, incluindo **tipos de redes (sem fio, cabo, 3G)** e **categorias de rede (Pública, Privada/Casa, Domínio/Trabalho)**, que são vitais para entender as configurações de segurança e permissões da rede.

### Cache do Lado do Cliente (CSC)
- **CSC** melhora o acesso a arquivos offline armazenando cópias de arquivos compartilhados. Diferentes configurações de **CSCFlags** controlam como e quais arquivos são armazenados em cache, afetando o desempenho e a experiência do usuário, especialmente em ambientes com conectividade intermitente.

### Programas de Inicialização Automática
- Programas listados em várias chaves de registro `Run` e `RunOnce` são lançados automaticamente na inicialização, afetando o tempo de inicialização do sistema e potencialmente sendo pontos de interesse para identificar malware ou software indesejado.

### Shellbags
- **Shellbags** não apenas armazenam preferências para visualizações de pastas, mas também fornecem evidências forenses de acesso a pastas, mesmo que a pasta não exista mais. Eles são inestimáveis para investigações, revelando atividades do usuário que não são óbvias por outros meios.

### Informações e Forense de USB
- Os detalhes armazenados no registro sobre dispositivos USB podem ajudar a rastrear quais dispositivos foram conectados a um computador, potencialmente ligando um dispositivo a transferências de arquivos sensíveis ou incidentes de acesso não autorizado.

### Número de Série do Volume
- O **Número de Série do Volume** pode ser crucial para rastrear a instância específica de um sistema de arquivos, útil em cenários forenses onde a origem do arquivo precisa ser estabelecida entre diferentes dispositivos.

### **Detalhes do Desligamento**
- O tempo e a contagem de desligamento (esta última apenas para XP) são mantidos em **`System\ControlSet001\Control\Windows`** e **`System\ControlSet001\Control\Watchdog\Display`**.

### **Configuração de Rede**
- Para informações detalhadas sobre a interface de rede, consulte **`System\ControlSet001\Services\Tcpip\Parameters\Interfaces{GUID_INTERFACE}`**.
- Os tempos de primeira e última conexão de rede, incluindo conexões VPN, são registrados em vários caminhos em **`Software\Microsoft\Windows NT\CurrentVersion\NetworkList`**.

### **Pastas Compartilhadas**
- Pastas e configurações compartilhadas estão em **`System\ControlSet001\Services\lanmanserver\Shares`**. As configurações de Cache do Lado do Cliente (CSC) ditam a disponibilidade de arquivos offline.

### **Programas que Iniciam Automaticamente**
- Caminhos como **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run`** e entradas semelhantes em `Software\Microsoft\Windows\CurrentVersion` detalham programas configurados para serem executados na inicialização.

### **Pesquisas e Caminhos Digitados**
- Pesquisas do Explorer e caminhos digitados são rastreados no registro sob **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer`** para WordwheelQuery e TypedPaths, respectivamente.

### **Documentos Recentes e Arquivos do Office**
- Documentos recentes e arquivos do Office acessados são anotados em `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` e caminhos específicos da versão do Office.

### **Itens Mais Recentemente Usados (MRU)**
- Listas MRU, indicando caminhos e comandos de arquivos recentes, são armazenadas em várias subchaves `ComDlg32` e `Explorer` sob `NTUSER.DAT`.

### **Rastreamento de Atividade do Usuário**
- O recurso User Assist registra estatísticas detalhadas de uso de aplicativos, incluindo contagem de execuções e último tempo de execução, em **`NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\{GUID}\Count`**.

### **Análise de Shellbags**
- Shellbags, revelando detalhes de acesso a pastas, são armazenados em `USRCLASS.DAT` e `NTUSER.DAT` sob `Software\Microsoft\Windows\Shell`. Use **[Shellbag Explorer](https://ericzimmerman.github.io/#!index.md)** para análise.

### **Histórico de Dispositivos USB**
- **`HKLM\SYSTEM\ControlSet001\Enum\USBSTOR`** e **`HKLM\SYSTEM\ControlSet001\Enum\USB`** contêm detalhes ricos sobre dispositivos USB conectados, incluindo fabricante, nome do produto e timestamps de conexão.
- O usuário associado a um dispositivo USB específico pode ser identificado pesquisando os hives `NTUSER.DAT` pelo **{GUID}** do dispositivo.
- O último dispositivo montado e seu número de série de volume podem ser rastreados através de `System\MountedDevices` e `Software\Microsoft\Windows NT\CurrentVersion\EMDMgmt`, respectivamente.

Este guia condensa os caminhos e métodos cruciais para acessar informações detalhadas sobre sistema, rede e atividade do usuário em sistemas Windows, visando clareza e usabilidade.



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
