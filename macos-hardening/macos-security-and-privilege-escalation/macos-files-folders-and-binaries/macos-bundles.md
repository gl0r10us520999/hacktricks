# macOS Bundles

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

## Informações Básicas

Os bundles no macOS servem como contêineres para uma variedade de recursos, incluindo aplicativos, bibliotecas e outros arquivos necessários, fazendo com que apareçam como objetos únicos no Finder, como os arquivos `*.app`. O bundle mais comumente encontrado é o bundle `.app`, embora outros tipos como `.framework`, `.systemextension` e `.kext` também sejam prevalentes.

### Componentes Essenciais de um Bundle

Dentro de um bundle, particularmente no diretório `<application>.app/Contents/`, uma variedade de recursos importantes está armazenada:

* **\_CodeSignature**: Este diretório armazena detalhes de assinatura de código vitais para verificar a integridade do aplicativo. Você pode inspecionar as informações de assinatura de código usando comandos como: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Contém o binário executável do aplicativo que é executado ao interagir com o usuário.
* **Resources**: Um repositório para os componentes da interface do usuário do aplicativo, incluindo imagens, documentos e descrições da interface (arquivos nib/xib).
* **Info.plist**: Funciona como o arquivo de configuração principal do aplicativo, crucial para que o sistema reconheça e interaja com o aplicativo de forma apropriada.

#### Chaves Importantes no Info.plist

O arquivo `Info.plist` é uma pedra angular para a configuração do aplicativo, contendo chaves como:

* **CFBundleExecutable**: Especifica o nome do arquivo executável principal localizado no diretório `Contents/MacOS`.
* **CFBundleIdentifier**: Fornece um identificador global para o aplicativo, amplamente utilizado pelo macOS para gerenciamento de aplicativos.
* **LSMinimumSystemVersion**: Indica a versão mínima do macOS necessária para o aplicativo ser executado.

### Explorando Bundles

Para explorar o conteúdo de um bundle, como `Safari.app`, o seguinte comando pode ser usado: `bash ls -lR /Applications/Safari.app/Contents`

Essa exploração revela diretórios como `_CodeSignature`, `MacOS`, `Resources`, e arquivos como `Info.plist`, cada um servindo a um propósito único, desde a segurança do aplicativo até a definição de sua interface do usuário e parâmetros operacionais.

#### Diretórios Adicionais de Bundle

Além dos diretórios comuns, os bundles também podem incluir:

* **Frameworks**: Contém frameworks agrupados usados pelo aplicativo. Frameworks são como dylibs com recursos extras.
* **PlugIns**: Um diretório para plug-ins e extensões que aprimoram as capacidades do aplicativo.
* **XPCServices**: Contém serviços XPC usados pelo aplicativo para comunicação fora do processo.

Essa estrutura garante que todos os componentes necessários estejam encapsulados dentro do bundle, facilitando um ambiente de aplicativo modular e seguro.

Para informações mais detalhadas sobre as chaves `Info.plist` e seus significados, a documentação do desenvolvedor da Apple fornece recursos extensivos: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

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
