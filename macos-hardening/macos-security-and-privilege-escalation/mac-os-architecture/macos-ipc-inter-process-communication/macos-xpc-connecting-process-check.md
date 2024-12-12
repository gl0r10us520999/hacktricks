# Verificação de Conexão de Processo XPC no macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? Ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Verificação de Conexão de Processo XPC

Quando uma conexão é estabelecida com um serviço XPC, o servidor verificará se a conexão é permitida. Estas são as verificações que normalmente são realizadas:

1. Verificar se o **processo de conexão está assinado com um certificado assinado pela Apple** (apenas fornecido pela Apple).
* Se isso **não for verificado**, um atacante pode criar um **certificado falso** para corresponder a qualquer outra verificação.
2. Verificar se o processo de conexão está assinado com o **certificado da organização** (verificação do ID da equipe).
* Se isso **não for verificado**, qualquer certificado de desenvolvedor da Apple pode ser usado para assinar e se conectar ao serviço.
3. Verificar se o processo de conexão **contém um ID de pacote adequado**.
* Se isso **não for verificado**, qualquer ferramenta **assinada pela mesma organização** pode ser usada para interagir com o serviço XPC.
4. (4 ou 5) Verificar se o processo de conexão possui um **número de versão de software adequado**.
* Se isso **não for verificado**, clientes antigos e inseguros, vulneráveis à injeção de processo, podem ser usados para se conectar ao serviço XPC, mesmo com as outras verificações em vigor.
5. (4 ou 5) Verificar se o processo de conexão possui um tempo de execução protegido sem privilégios perigosos (como aqueles que permitem carregar bibliotecas arbitrárias ou usar variáveis de ambiente DYLD).
1. Se isso **não for verificado**, o cliente pode estar **vulnerável à injeção de código**.
6. Verificar se o processo de conexão possui uma **autorização** que permite a conexão com o serviço. Isso é aplicável para binários da Apple.
7. A **verificação** deve ser **baseada** no **token de auditoria do cliente** de conexão **em vez** de seu ID de processo (**PID**), pois o primeiro impede ataques de reutilização de PID.
* Os desenvolvedores raramente usam a chamada de API de token de auditoria, pois ela é **privada**, então a Apple pode **alterá-la** a qualquer momento. Além disso, o uso de API privada não é permitido em aplicativos da Mac App Store.

Para obter mais informações sobre o ataque de reutilização de PID, consulte:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

### Trustcache - Prevenção de Ataques de Downgrade

Trustcache é um método defensivo introduzido em máquinas Apple Silicon que armazena um banco de dados de CDHSAH de binários da Apple, para que apenas binários não modificados permitidos possam ser executados. Isso impede a execução de versões de downgrade.

### Exemplos de Código

O servidor implementará essa **verificação** em uma função chamada **`shouldAcceptNewConnection`**.

{% code overflow="wrap" %}
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
//Check connection
return YES;
}
```
O objeto NSXPCConnection possui uma propriedade **privada** chamada **`auditToken`** (a que deve ser usada, mas pode mudar) e uma propriedade **pública** chamada **`processIdentifier`** (a que não deve ser usada).

O processo de conexão pode ser verificado da seguinte forma:

{% code overflow="wrap" %}
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements with the PID (vulnerable)
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);

// Check the requirements wuing the auditToken (secure)
SecTaskRef taskRef = SecTaskCreateWithAuditToken(NULL, ((ExtendedNSXPCConnection*)newConnection).auditToken);
SecTaskValidateForRequirement(taskRef, (__bridge CFStringRef)(requirementString))
```
Se um desenvolvedor não quiser verificar a versão do cliente, ele pode pelo menos verificar se o cliente não é vulnerável à injeção de processo:

{% code overflow="wrap" %}
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page.
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
return Yes; // Accept connection
}
```
{% endcode %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de cibersegurança**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Verifique os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**A Família PEASS**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Compartilhe seus truques de hacking enviando PRs para o** [**repositório hacktricks**](https://github.com/carlospolop/hacktricks) **e** [**repositório hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
