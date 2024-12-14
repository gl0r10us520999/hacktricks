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


## Conceitos Básicos

- **Smart Contracts** são definidos como programas que executam em uma blockchain quando certas condições são atendidas, automatizando a execução de acordos sem intermediários.
- **Aplicações Descentralizadas (dApps)** se baseiam em smart contracts, apresentando uma interface amigável e um back-end transparente e auditável.
- **Tokens & Moedas** diferenciam onde moedas servem como dinheiro digital, enquanto tokens representam valor ou propriedade em contextos específicos.
- **Utility Tokens** concedem acesso a serviços, e **Security Tokens** significam propriedade de ativos.
- **DeFi** significa Finanças Descentralizadas, oferecendo serviços financeiros sem autoridades centrais.
- **DEX** e **DAOs** referem-se a Plataformas de Troca Descentralizadas e Organizações Autônomas Descentralizadas, respectivamente.

## Mecanismos de Consenso

Mecanismos de consenso garantem validações de transações seguras e acordadas na blockchain:
- **Proof of Work (PoW)** depende de poder computacional para verificação de transações.
- **Proof of Stake (PoS)** exige que validadores mantenham uma certa quantidade de tokens, reduzindo o consumo de energia em comparação ao PoW.

## Essenciais do Bitcoin

### Transações

Transações de Bitcoin envolvem a transferência de fundos entre endereços. As transações são validadas através de assinaturas digitais, garantindo que apenas o proprietário da chave privada possa iniciar transferências.

#### Componentes Chave:

- **Transações Multisignature** requerem múltiplas assinaturas para autorizar uma transação.
- As transações consistem em **entradas** (fonte de fundos), **saídas** (destino), **taxas** (pagas aos mineradores) e **scripts** (regras da transação).

### Lightning Network

Tem como objetivo melhorar a escalabilidade do Bitcoin permitindo múltiplas transações dentro de um canal, transmitindo apenas o estado final para a blockchain.

## Preocupações com a Privacidade do Bitcoin

Ataques de privacidade, como **Common Input Ownership** e **UTXO Change Address Detection**, exploram padrões de transação. Estratégias como **Mixers** e **CoinJoin** melhoram a anonimidade ao obscurecer os vínculos de transação entre os usuários.

## Adquirindo Bitcoins Anonimamente

Métodos incluem trocas em dinheiro, mineração e uso de mixers. **CoinJoin** mistura múltiplas transações para complicar a rastreabilidade, enquanto **PayJoin** disfarça CoinJoins como transações regulares para maior privacidade.


# Ataques de Privacidade do Bitcoin

# Resumo dos Ataques de Privacidade do Bitcoin

No mundo do Bitcoin, a privacidade das transações e a anonimidade dos usuários são frequentemente assuntos de preocupação. Aqui está uma visão simplificada de vários métodos comuns pelos quais atacantes podem comprometer a privacidade do Bitcoin.

## **Assunção de Propriedade de Entrada Comum**

É geralmente raro que entradas de diferentes usuários sejam combinadas em uma única transação devido à complexidade envolvida. Assim, **dois endereços de entrada na mesma transação são frequentemente assumidos como pertencentes ao mesmo proprietário**.

## **Detecção de Endereço de Troca UTXO**

Um UTXO, ou **Unspent Transaction Output**, deve ser totalmente gasto em uma transação. Se apenas uma parte dele for enviada para outro endereço, o restante vai para um novo endereço de troca. Observadores podem assumir que este novo endereço pertence ao remetente, comprometendo a privacidade.

### Exemplo
Para mitigar isso, serviços de mistura ou o uso de múltiplos endereços podem ajudar a obscurecer a propriedade.

## **Exposição em Redes Sociais & Fóruns**

Os usuários às vezes compartilham seus endereços de Bitcoin online, tornando **fácil vincular o endereço ao seu proprietário**.

## **Análise de Gráficos de Transação**

As transações podem ser visualizadas como gráficos, revelando potenciais conexões entre usuários com base no fluxo de fundos.

## **Heurística de Entrada Desnecessária (Heurística de Troca Ótima)**

Esta heurística é baseada na análise de transações com múltiplas entradas e saídas para adivinhar qual saída é a troca retornando ao remetente.

### Exemplo
```bash
2 btc --> 4 btc
3 btc     1 btc
```
Se adicionar mais entradas faz com que a saída de troco seja maior do que qualquer entrada única, isso pode confundir a heurística.

## **Reutilização Forçada de Endereço**

Os atacantes podem enviar pequenas quantias para endereços já utilizados, na esperança de que o destinatário combine essas quantias com outras entradas em transações futuras, ligando assim os endereços.

### Comportamento Correto da Carteira
As carteiras devem evitar usar moedas recebidas em endereços já utilizados e vazios para prevenir essa vazamento de privacidade.

## **Outras Técnicas de Análise de Blockchain**

- **Quantias de Pagamento Exatas:** Transações sem troco são provavelmente entre dois endereços pertencentes ao mesmo usuário.
- **Números Redondos:** Um número redondo em uma transação sugere que é um pagamento, com a saída não redonda provavelmente sendo o troco.
- **Impressão Digital de Carteira:** Diferentes carteiras têm padrões únicos de criação de transações, permitindo que analistas identifiquem o software utilizado e potencialmente o endereço de troco.
- **Correlações de Quantidade e Tempo:** Divulgar horários ou quantias de transações pode tornar as transações rastreáveis.

## **Análise de Tráfego**

Ao monitorar o tráfego da rede, os atacantes podem potencialmente vincular transações ou blocos a endereços IP, comprometendo a privacidade do usuário. Isso é especialmente verdadeiro se uma entidade operar muitos nós Bitcoin, aumentando sua capacidade de monitorar transações.

## Mais
Para uma lista abrangente de ataques à privacidade e defesas, visite [Privacidade do Bitcoin na Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

# Transações Anônimas de Bitcoin

## Maneiras de Obter Bitcoins Anonimamente

- **Transações em Dinheiro**: Adquirir bitcoin através de dinheiro.
- **Alternativas em Dinheiro**: Comprar cartões-presente e trocá-los online por bitcoin.
- **Mineração**: O método mais privado para ganhar bitcoins é através da mineração, especialmente quando feito sozinho, pois pools de mineração podem conhecer o endereço IP do minerador. [Informações sobre Pools de Mineração](https://en.bitcoin.it/wiki/Pooled_mining)
- **Roubo**: Teoricamente, roubar bitcoin poderia ser outro método para adquiri-lo anonimamente, embora seja ilegal e não recomendado.

## Serviços de Mistura

Ao usar um serviço de mistura, um usuário pode **enviar bitcoins** e receber **bitcoins diferentes em troca**, o que dificulta o rastreamento do proprietário original. No entanto, isso requer confiança no serviço para não manter registros e realmente devolver os bitcoins. Opções alternativas de mistura incluem cassinos de Bitcoin.

## CoinJoin

**CoinJoin** mescla várias transações de diferentes usuários em uma, complicando o processo para qualquer um que tente combinar entradas com saídas. Apesar de sua eficácia, transações com tamanhos de entrada e saída únicos ainda podem ser potencialmente rastreadas.

Transações de exemplo que podem ter usado CoinJoin incluem `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` e `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Para mais informações, visite [CoinJoin](https://coinjoin.io/en). Para um serviço semelhante no Ethereum, confira [Tornado Cash](https://tornado.cash), que anonimiza transações com fundos de mineradores.

## PayJoin

Uma variante do CoinJoin, **PayJoin** (ou P2EP), disfarça a transação entre duas partes (por exemplo, um cliente e um comerciante) como uma transação regular, sem a característica distintiva de saídas iguais do CoinJoin. Isso torna extremamente difícil de detectar e pode invalidar a heurística de propriedade de entrada comum usada por entidades de vigilância de transações.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transações como a acima poderiam ser PayJoin, melhorando a privacidade enquanto permanecem indistinguíveis das transações padrão de bitcoin.

**A utilização de PayJoin poderia desestabilizar significativamente os métodos tradicionais de vigilância**, tornando-se um desenvolvimento promissor na busca pela privacidade transacional.


# Melhores Práticas para Privacidade em Criptomoedas

## **Técnicas de Sincronização de Carteiras**

Para manter a privacidade e a segurança, a sincronização de carteiras com a blockchain é crucial. Dois métodos se destacam:

- **Nó completo**: Ao baixar toda a blockchain, um nó completo garante máxima privacidade. Todas as transações já realizadas são armazenadas localmente, tornando impossível para adversários identificar quais transações ou endereços o usuário está interessado.
- **Filtragem de blocos do lado do cliente**: Este método envolve a criação de filtros para cada bloco na blockchain, permitindo que as carteiras identifiquem transações relevantes sem expor interesses específicos a observadores da rede. Carteiras leves baixam esses filtros, buscando blocos completos apenas quando uma correspondência com os endereços do usuário é encontrada.

## **Utilizando Tor para Anonimato**

Dado que o Bitcoin opera em uma rede peer-to-peer, é recomendado usar o Tor para ocultar seu endereço IP, melhorando a privacidade ao interagir com a rede.

## **Prevenindo Reutilização de Endereços**

Para proteger a privacidade, é vital usar um novo endereço para cada transação. Reutilizar endereços pode comprometer a privacidade ao vincular transações à mesma entidade. Carteiras modernas desencorajam a reutilização de endereços por meio de seu design.

## **Estratégias para Privacidade de Transações**

- **Múltiplas transações**: Dividir um pagamento em várias transações pode obscurecer o valor da transação, frustrando ataques à privacidade.
- **Evitar troco**: Optar por transações que não exigem saídas de troco melhora a privacidade ao interromper métodos de detecção de troco.
- **Múltiplas saídas de troco**: Se evitar troco não for viável, gerar várias saídas de troco ainda pode melhorar a privacidade.

# **Monero: Um Farol de Anonimato**

Monero aborda a necessidade de anonimato absoluto em transações digitais, estabelecendo um alto padrão para a privacidade.

# **Ethereum: Gas e Transações**

## **Entendendo Gas**

Gas mede o esforço computacional necessário para executar operações no Ethereum, precificado em **gwei**. Por exemplo, uma transação que custa 2.310.000 gwei (ou 0.00231 ETH) envolve um limite de gas e uma taxa base, com uma gorjeta para incentivar os mineradores. Os usuários podem definir uma taxa máxima para garantir que não paguem a mais, com o excesso reembolsado.

## **Executando Transações**

Transações no Ethereum envolvem um remetente e um destinatário, que podem ser endereços de usuário ou de contrato inteligente. Elas requerem uma taxa e devem ser mineradas. As informações essenciais em uma transação incluem o destinatário, a assinatura do remetente, o valor, dados opcionais, limite de gas e taxas. Notavelmente, o endereço do remetente é deduzido da assinatura, eliminando a necessidade de incluí-lo nos dados da transação.

Essas práticas e mecanismos são fundamentais para qualquer pessoa que deseje se envolver com criptomoedas enquanto prioriza a privacidade e a segurança.


## Referências

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)


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
