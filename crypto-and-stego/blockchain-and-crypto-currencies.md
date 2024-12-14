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


## Grundkonzepte

- **Smart Contracts** sind Programme, die auf einer Blockchain ausgeführt werden, wenn bestimmte Bedingungen erfüllt sind, und automatisieren die Ausführung von Vereinbarungen ohne Zwischenhändler.
- **Dezentralisierte Anwendungen (dApps)** basieren auf Smart Contracts und verfügen über ein benutzerfreundliches Front-End sowie ein transparentes, prüfbares Back-End.
- **Tokens & Coins** unterscheiden sich darin, dass Coins als digitales Geld dienen, während Tokens Wert oder Eigentum in bestimmten Kontexten repräsentieren.
- **Utility Tokens** gewähren Zugang zu Dienstleistungen, und **Security Tokens** signalisieren den Besitz von Vermögenswerten.
- **DeFi** steht für Dezentrale Finanzen und bietet Finanzdienstleistungen ohne zentrale Autoritäten.
- **DEX** und **DAOs** beziehen sich auf Dezentrale Handelsplattformen und Dezentrale Autonome Organisationen.

## Konsensmechanismen

Konsensmechanismen gewährleisten sichere und vereinbarte Transaktionsvalidierungen auf der Blockchain:
- **Proof of Work (PoW)** basiert auf Rechenleistung zur Verifizierung von Transaktionen.
- **Proof of Stake (PoS)** verlangt von Validierern, eine bestimmte Menge an Tokens zu halten, was den Energieverbrauch im Vergleich zu PoW reduziert.

## Bitcoin-Grundlagen

### Transaktionen

Bitcoin-Transaktionen beinhalten die Übertragung von Geldern zwischen Adressen. Transaktionen werden durch digitale Signaturen validiert, die sicherstellen, dass nur der Besitzer des privaten Schlüssels Überweisungen initiieren kann.

#### Schlüsselkomponenten:

- **Multisignatur-Transaktionen** erfordern mehrere Unterschriften, um eine Transaktion zu autorisieren.
- Transaktionen bestehen aus **Inputs** (Quelle der Gelder), **Outputs** (Ziel), **Gebühren** (die an Miner gezahlt werden) und **Scripts** (Transaktionsregeln).

### Lightning-Netzwerk

Zielt darauf ab, die Skalierbarkeit von Bitcoin zu verbessern, indem mehrere Transaktionen innerhalb eines Kanals ermöglicht werden, wobei nur der endgültige Zustand an die Blockchain gesendet wird.

## Bitcoin-Privatsphäre-Bedenken

Privatsphäreangriffe, wie **Common Input Ownership** und **UTXO Change Address Detection**, nutzen Transaktionsmuster aus. Strategien wie **Mixers** und **CoinJoin** verbessern die Anonymität, indem sie die Transaktionsverbindungen zwischen Benutzern verschleiern.

## Bitcoins anonym erwerben

Methoden umfassen Bargeschäfte, Mining und die Verwendung von Mixern. **CoinJoin** mischt mehrere Transaktionen, um die Rückverfolgbarkeit zu erschweren, während **PayJoin** CoinJoins als reguläre Transaktionen tarnt, um die Privatsphäre zu erhöhen.


# Bitcoin-Privatsphäre-Angriffe

# Zusammenfassung der Bitcoin-Privatsphäre-Angriffe

In der Welt von Bitcoin sind die Privatsphäre von Transaktionen und die Anonymität der Benutzer oft Gegenstand von Bedenken. Hier ist eine vereinfachte Übersicht über mehrere gängige Methoden, durch die Angreifer die Bitcoin-Privatsphäre gefährden können.

## **Annahme des gemeinsamen Eingangsbesitzes**

Es ist allgemein selten, dass Eingänge von verschiedenen Benutzern in einer einzigen Transaktion kombiniert werden, aufgrund der damit verbundenen Komplexität. Daher wird **angenommen, dass zwei Eingangsadressen in derselben Transaktion oft demselben Eigentümer gehören**.

## **UTXO-Wechseladresse-Erkennung**

Ein UTXO, oder **Unspent Transaction Output**, muss in einer Transaktion vollständig ausgegeben werden. Wenn nur ein Teil davon an eine andere Adresse gesendet wird, geht der Rest an eine neue Wechseladresse. Beobachter können annehmen, dass diese neue Adresse dem Absender gehört, was die Privatsphäre gefährdet.

### Beispiel
Um dies zu mildern, können Mischdienste oder die Verwendung mehrerer Adressen helfen, den Besitz zu verschleiern.

## **Exposition in sozialen Netzwerken & Foren**

Benutzer teilen manchmal ihre Bitcoin-Adressen online, was es **einfach macht, die Adresse mit ihrem Eigentümer zu verknüpfen**.

## **Transaktionsgraphanalyse**

Transaktionen können als Graphen visualisiert werden, die potenzielle Verbindungen zwischen Benutzern basierend auf dem Fluss von Geldern offenbaren.

## **Unnötige Eingangsheuristik (Optimale Wechselheuristik)**

Diese Heuristik basiert auf der Analyse von Transaktionen mit mehreren Eingängen und Ausgängen, um zu erraten, welcher Ausgang das Wechselgeld ist, das an den Absender zurückgegeben wird.

### Beispiel
```bash
2 btc --> 4 btc
3 btc     1 btc
```
If adding more inputs makes the change output larger than any single input, it can confuse the heuristic.

## **Erzwungene Adressennutzung**

Angreifer können kleine Beträge an zuvor verwendete Adressen senden, in der Hoffnung, dass der Empfänger diese mit anderen Eingaben in zukünftigen Transaktionen kombiniert und somit Adressen miteinander verknüpft.

### Korrektes Wallet-Verhalten
Wallets sollten vermeiden, Münzen zu verwenden, die auf bereits verwendeten, leeren Adressen empfangen wurden, um diesen Datenschutzleck zu verhindern.

## **Andere Blockchain-Analyse-Techniken**

- **Exakte Zahlungsbeträge:** Transaktionen ohne Wechselgeld sind wahrscheinlich zwischen zwei Adressen, die demselben Benutzer gehören.
- **Runde Zahlen:** Eine runde Zahl in einer Transaktion deutet darauf hin, dass es sich um eine Zahlung handelt, wobei der nicht-runde Ausgang wahrscheinlich das Wechselgeld ist.
- **Wallet-Fingerprinting:** Verschiedene Wallets haben einzigartige Muster bei der Transaktionsgenerierung, die es Analysten ermöglichen, die verwendete Software und potenziell die Wechseladresse zu identifizieren.
- **Betrags- und Zeitkorrelationen:** Die Offenlegung von Transaktionszeiten oder -beträgen kann Transaktionen nachverfolgbar machen.

## **Verkehrsanalyse**

Durch die Überwachung des Netzwerkverkehrs können Angreifer potenziell Transaktionen oder Blöcke mit IP-Adressen verknüpfen, was die Privatsphäre der Benutzer gefährdet. Dies gilt insbesondere, wenn eine Entität viele Bitcoin-Knoten betreibt, was ihre Fähigkeit zur Überwachung von Transaktionen erhöht.

## Mehr
Für eine umfassende Liste von Datenschutzangriffen und -abwehrmaßnahmen besuchen Sie [Bitcoin Privacy on Bitcoin Wiki](https://en.bitcoin.it/wiki/Privacy).

# Anonyme Bitcoin-Transaktionen

## Möglichkeiten, Bitcoins anonym zu erhalten

- **Bargeldtransaktionen**: Erwerb von Bitcoin durch Bargeld.
- **Bargeldalternativen**: Kauf von Geschenkkarten und deren Online-Einlösung gegen Bitcoin.
- **Mining**: Die privateste Methode, um Bitcoins zu verdienen, ist das Mining, insbesondere wenn es alleine durchgeführt wird, da Mining-Pools möglicherweise die IP-Adresse des Miners kennen. [Mining-Pools-Informationen](https://en.bitcoin.it/wiki/Pooled_mining)
- **Diebstahl**: Theoretisch könnte das Stehlen von Bitcoin eine weitere Methode sein, um es anonym zu erwerben, obwohl es illegal und nicht empfohlen ist.

## Mischdienste

Durch die Nutzung eines Mischdienstes kann ein Benutzer **Bitcoins senden** und **andere Bitcoins im Gegenzug erhalten**, was die Rückverfolgung des ursprünglichen Eigentümers erschwert. Dies erfordert jedoch Vertrauen in den Dienst, dass er keine Protokolle führt und die Bitcoins tatsächlich zurückgibt. Alternative Mischoptionen sind Bitcoin-Casinos.

## CoinJoin

**CoinJoin** kombiniert mehrere Transaktionen von verschiedenen Benutzern in eine, was den Prozess für jeden, der versucht, Eingaben mit Ausgaben abzugleichen, kompliziert. Trotz seiner Effektivität können Transaktionen mit einzigartigen Eingabe- und Ausgabengrößen dennoch potenziell zurückverfolgt werden.

Beispieltransaktionen, die CoinJoin verwendet haben könnten, sind `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` und `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Für weitere Informationen besuchen Sie [CoinJoin](https://coinjoin.io/en). Für einen ähnlichen Dienst auf Ethereum, schauen Sie sich [Tornado Cash](https://tornado.cash) an, der Transaktionen mit Mitteln von Minern anonymisiert.

## PayJoin

Eine Variante von CoinJoin, **PayJoin** (oder P2EP), tarnt die Transaktion zwischen zwei Parteien (z. B. einem Kunden und einem Händler) als reguläre Transaktion, ohne die charakteristischen gleichmäßigen Ausgaben von CoinJoin. Dies macht es extrem schwer zu erkennen und könnte die gängige Eingabe-Eigentumsheuristik, die von Transaktionsüberwachungsstellen verwendet wird, ungültig machen.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Transaktionen wie die oben genannten könnten PayJoin sein, was die Privatsphäre verbessert und gleichzeitig von standardmäßigen Bitcoin-Transaktionen nicht zu unterscheiden ist.

**Die Nutzung von PayJoin könnte traditionelle Überwachungsmethoden erheblich stören**, was es zu einer vielversprechenden Entwicklung im Streben nach transaktionaler Privatsphäre macht.


# Best Practices für Privatsphäre in Kryptowährungen

## **Wallet-Synchronisationstechniken**

Um Privatsphäre und Sicherheit zu gewährleisten, ist die Synchronisierung von Wallets mit der Blockchain entscheidend. Zwei Methoden stechen hervor:

- **Vollknoten**: Durch das Herunterladen der gesamten Blockchain gewährleistet ein Vollknoten maximale Privatsphäre. Alle jemals getätigten Transaktionen werden lokal gespeichert, was es Gegnern unmöglich macht, zu identifizieren, an welchen Transaktionen oder Adressen der Benutzer interessiert ist.
- **Client-seitige Blockfilterung**: Diese Methode beinhaltet die Erstellung von Filtern für jeden Block in der Blockchain, sodass Wallets relevante Transaktionen identifizieren können, ohne spezifische Interessen gegenüber Netzwerkbeobachtern offenzulegen. Leichte Wallets laden diese Filter herunter und holen sich nur vollständige Blöcke, wenn eine Übereinstimmung mit den Adressen des Benutzers gefunden wird.

## **Tor zur Anonymität nutzen**

Da Bitcoin in einem Peer-to-Peer-Netzwerk arbeitet, wird empfohlen, Tor zu verwenden, um Ihre IP-Adresse zu maskieren und die Privatsphäre bei der Interaktion mit dem Netzwerk zu verbessern.

## **Vermeidung von Adresswiederverwendung**

Um die Privatsphäre zu schützen, ist es wichtig, für jede Transaktion eine neue Adresse zu verwenden. Die Wiederverwendung von Adressen kann die Privatsphäre gefährden, indem Transaktionen mit derselben Entität verknüpft werden. Moderne Wallets entmutigen die Wiederverwendung von Adressen durch ihr Design.

## **Strategien für Transaktionsprivatsphäre**

- **Mehrere Transaktionen**: Eine Zahlung in mehrere Transaktionen aufzuteilen, kann den Transaktionsbetrag verschleiern und Privatsphäre-Angriffe vereiteln.
- **Vermeidung von Wechselgeld**: Transaktionen zu wählen, die keine Wechselgeldausgaben erfordern, verbessert die Privatsphäre, indem Methoden zur Wechselgelddetektion gestört werden.
- **Mehrere Wechselgeldausgaben**: Wenn die Vermeidung von Wechselgeld nicht möglich ist, kann die Generierung mehrerer Wechselgeldausgaben dennoch die Privatsphäre verbessern.

# **Monero: Ein Leuchtturm der Anonymität**

Monero adressiert das Bedürfnis nach absoluter Anonymität in digitalen Transaktionen und setzt einen hohen Standard für Privatsphäre.

# **Ethereum: Gas und Transaktionen**

## **Gas verstehen**

Gas misst den Rechenaufwand, der erforderlich ist, um Operationen auf Ethereum auszuführen, und wird in **gwei** bewertet. Zum Beispiel beinhaltet eine Transaktion, die 2.310.000 gwei (oder 0,00231 ETH) kostet, ein Gaslimit und eine Grundgebühr, mit einem Trinkgeld, um Miner zu incentivieren. Benutzer können eine Höchstgebühr festlegen, um sicherzustellen, dass sie nicht zu viel bezahlen, wobei der Überschuss zurückerstattet wird.

## **Transaktionen ausführen**

Transaktionen in Ethereum beinhalten einen Absender und einen Empfänger, die entweder Benutzer- oder Smart-Contract-Adressen sein können. Sie erfordern eine Gebühr und müssen geschürft werden. Wesentliche Informationen in einer Transaktion umfassen den Empfänger, die Unterschrift des Absenders, den Wert, optionale Daten, das Gaslimit und die Gebühren. Bemerkenswert ist, dass die Adresse des Absenders aus der Unterschrift abgeleitet wird, wodurch die Notwendigkeit entfällt, sie in den Transaktionsdaten anzugeben.

Diese Praktiken und Mechanismen sind grundlegend für jeden, der mit Kryptowährungen interagieren möchte, während er Privatsphäre und Sicherheit priorisiert.


## Referenzen

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
