{% hint style="success" %}
Lernen & Üben von AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & Üben von GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstützen Sie HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichen.

</details>
{% endhint %}


<a rel="license" href="https://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://licensebuttons.net/l/by-nc/4.0/88x31.png" /></a><br>Copyright © Carlos Polop 2021. Sofern nicht anders angegeben (die externen Informationen, die in das Buch kopiert wurden, gehören den ursprünglichen Autoren), ist der Text auf <a href="https://github.com/carlospolop/hacktricks">HACK TRICKS</a> von Carlos Polop lizenziert unter der <a href="https://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Namensnennung-NichtKommerziell 4.0 International (CC BY-NC 4.0)</a>.

Lizenz: Namensnennung-NichtKommerziell 4.0 International (CC BY-NC 4.0)<br>
Menschlich lesbare Lizenz: https://creativecommons.org/licenses/by-nc/4.0/<br>
Vollständige rechtliche Bedingungen: https://creativecommons.org/licenses/by-nc/4.0/legalcode<br>
Formatierung: https://github.com/jmatsushita/Creative-Commons-4.0-Markdown/blob/master/licenses/by-nc.markdown<br>

# creative commons

# Namensnennung-NichtKommerziell 4.0 International

Creative Commons Corporation (“Creative Commons”) ist keine Anwaltskanzlei und bietet keine juristischen Dienstleistungen oder Rechtsberatung an. Die Verbreitung von Creative Commons-Publik lizenzen schafft keine Anwalt- Mandanten- oder andere Beziehungen. Creative Commons stellt seine Lizenzen und verwandte Informationen auf einer „wie sie sind“-Basis zur Verfügung. Creative Commons gibt keine Garantien bezüglich seiner Lizenzen, des Materials, das unter ihren Bedingungen lizenziert ist, oder verwandter Informationen. Creative Commons schließt jegliche Haftung für Schäden, die aus ihrer Nutzung resultieren, im größtmöglichen Umfang aus.

## Verwendung von Creative Commons-Publiklizenzen

Creative Commons-Publiklizenzen bieten einen standardisierten Satz von Bedingungen, die von Kreativen und anderen Rechteinhabern verwendet werden können, um originale Werke der Urheberschaft und andere urheberrechtlich geschützte Materialien zu teilen. Die folgenden Überlegungen dienen nur zu Informationszwecken, sind nicht erschöpfend und sind kein Bestandteil unserer Lizenzen.

* __Überlegungen für Lizenzgeber:__ Unsere Publik lizenzen sind für die Verwendung durch diejenigen gedacht, die befugt sind, der Öffentlichkeit die Erlaubnis zu erteilen, Materialien auf eine Weise zu verwenden, die durch Urheberrecht und bestimmte andere Rechte eingeschränkt ist. Unsere Lizenzen sind unwiderruflich. Lizenzgeber sollten die Bedingungen der Lizenz, die sie wählen, lesen und verstehen, bevor sie sie anwenden. Lizenzgeber sollten auch alle notwendigen Rechte sichern, bevor sie unsere Lizenzen anwenden, damit die Öffentlichkeit das Material wie erwartet wiederverwenden kann. Lizenzgeber sollten deutlich kennzeichnen, welches Material nicht der Lizenz unterliegt. Dazu gehört auch anderes CC-lizenziertes Material oder Material, das unter einer Ausnahme oder Einschränkung des Urheberrechts verwendet wird. [Weitere Überlegungen für Lizenzgeber](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensors).

* __Überlegungen für die Öffentlichkeit:__ Durch die Verwendung einer unserer Publik lizenzen gewährt ein Lizenzgeber der Öffentlichkeit die Erlaubnis, das lizenzierte Material unter den angegebenen Bedingungen zu verwenden. Wenn die Erlaubnis des Lizenzgebers aus irgendeinem Grund nicht erforderlich ist – zum Beispiel aufgrund einer anwendbaren Ausnahme oder Einschränkung des Urheberrechts – dann wird diese Nutzung nicht durch die Lizenz geregelt. Unsere Lizenzen gewähren nur Erlaubnisse unter dem Urheberrecht und bestimmten anderen Rechten, die ein Lizenzgeber erteilen kann. Die Nutzung des lizenzierten Materials kann aus anderen Gründen eingeschränkt sein, einschließlich, weil andere Urheberrechte oder andere Rechte an dem Material haben. Ein Lizenzgeber kann besondere Anforderungen stellen, wie z.B. die Bitte, dass alle Änderungen gekennzeichnet oder beschrieben werden. Obwohl dies von unseren Lizenzen nicht gefordert wird, werden Sie ermutigt, diese Anforderungen, wo angemessen, zu respektieren. [Weitere Überlegungen für die Öffentlichkeit](http://wiki.creativecommons.org/Considerations_for_licensors_and_licensees#Considerations_for_licensees).

# Creative Commons Namensnennung-NichtKommerziell 4.0 International Public License

Durch die Ausübung der lizenzierten Rechte (wie unten definiert) akzeptieren und erklären Sie sich mit den Bedingungen dieser Creative Commons Namensnennung-NichtKommerziell 4.0 International Public License ("Öffentliche Lizenz") einverstanden. Soweit diese öffentliche Lizenz als Vertrag interpretiert werden kann, werden Ihnen die lizenzierten Rechte in Anbetracht Ihrer Akzeptanz dieser Bedingungen gewährt, und der Lizenzgeber gewährt Ihnen solche Rechte in Anbetracht der Vorteile, die der Lizenzgeber aus der Bereitstellung des lizenzierten Materials unter diesen Bedingungen erhält.

## Abschnitt 1 – Definitionen.

a. __Adaptives Material__ bedeutet Material, das dem Urheberrecht und ähnlichen Rechten unterliegt und das aus dem lizenzierten Material abgeleitet oder darauf basiert, in dem das lizenzierte Material übersetzt, verändert, angeordnet, transformiert oder anderweitig in einer Weise modifiziert wird, die eine Erlaubnis gemäß dem Urheberrecht und ähnlichen Rechten erfordert, die der Lizenzgeber hält. Für die Zwecke dieser öffentlichen Lizenz wird adaptives Material immer produziert, wenn das lizenzierte Material synchronisiert ist in zeitlicher Beziehung zu einem bewegten Bild.

b. __Lizenz des Adapters__ bedeutet die Lizenz, die Sie auf Ihre Urheberrechte und ähnlichen Rechte in Ihren Beiträgen zu adaptivem Material gemäß den Bedingungen dieser öffentlichen Lizenz anwenden.

c. __Urheberrecht und ähnliche Rechte__ bedeutet Urheberrecht und/oder ähnliche Rechte, die eng mit dem Urheberrecht verbunden sind, einschließlich, aber nicht beschränkt auf, Aufführung, Rundfunk, Tonaufnahme und Sui Generis-Datenbankrechte, unabhängig davon, wie die Rechte bezeichnet oder kategorisiert werden. Für die Zwecke dieser öffentlichen Lizenz sind die in Abschnitt 2(b)(1)-(2) angegebenen Rechte keine Urheberrechte und ähnlichen Rechte.

d. __Effektive technologische Maßnahmen__ bedeutet Maßnahmen, die in Abwesenheit der richtigen Autorität nicht umgangen werden dürfen gemäß den Gesetzen, die die Verpflichtungen gemäß Artikel 11 des WIPO-Urheberrechtsvertrags, der am 20. Dezember 1996 angenommen wurde, und/oder ähnlichen internationalen Vereinbarungen erfüllen.

e. __Ausnahmen und Einschränkungen__ bedeutet faire Nutzung, faire Behandlung und/oder jede andere Ausnahme oder Einschränkung des Urheberrechts und ähnlicher Rechte, die auf Ihre Nutzung des lizenzierten Materials anwendbar ist.

f. __Lizenziertes Material__ bedeutet das künstlerische oder literarische Werk, die Datenbank oder anderes Material, auf das der Lizenzgeber diese öffentliche Lizenz angewendet hat.

g. __Lizenzierte Rechte__ bedeutet die Ihnen unter den Bedingungen dieser öffentlichen Lizenz gewährten Rechte, die auf alle Urheberrechte und ähnlichen Rechte beschränkt sind, die auf Ihre Nutzung des lizenzierten Materials anwendbar sind und die der Lizenzgeber lizenziert.

h. __Lizenzgeber__ bedeutet die Person(en) oder Einheit(en), die Rechte gemäß dieser öffentlichen Lizenz gewähren.

i. __Nichtkommerziell__ bedeutet nicht hauptsächlich für kommerzielle Vorteile oder monetäre Entschädigung bestimmt oder darauf ausgerichtet. Für die Zwecke dieser öffentlichen Lizenz ist der Austausch des lizenzierten Materials gegen anderes Material, das dem Urheberrecht und ähnlichen Rechten unterliegt, durch digitalen Dateiaustausch oder ähnliche Mittel nichtkommerziell, sofern keine Zahlung von monetärer Entschädigung im Zusammenhang mit dem Austausch erfolgt.

j. __Teilen__ bedeutet, Material der Öffentlichkeit durch beliebige Mittel oder Prozesse zur Verfügung zu stellen, die eine Erlaubnis gemäß den lizenzierten Rechten erfordern, wie z.B. Vervielfältigung, öffentliche Anzeige, öffentliche Aufführung, Verteilung, Verbreitung, Kommunikation oder Einfuhr, und Material der Öffentlichkeit zur Verfügung zu stellen, einschließlich auf Wegen, die es Mitgliedern der Öffentlichkeit ermöglichen, das Material von einem Ort und zu einer Zeit, die sie individuell wählen, abzurufen.

k. __Sui Generis-Datenbankrechte__ bedeutet Rechte, die nicht Urheberrechte sind und aus der Richtlinie 96/9/EG des Europäischen Parlaments und des Rates vom 11. März 1996 über den rechtlichen Schutz von Datenbanken, in der geänderten oder nachfolgenden Fassung, sowie anderen im Wesentlichen gleichwertigen Rechten überall auf der Welt resultieren.

l. __Sie__ bedeutet die Person oder Einheit, die die lizenzierten Rechte gemäß dieser öffentlichen Lizenz ausübt. Ihr hat eine entsprechende Bedeutung.

## Abschnitt 2 – Umfang.

a. ___Lizenzgewährung.___

1. Vorbehaltlich der Bedingungen dieser öffentlichen Lizenz gewährt der Lizenzgeber Ihnen hiermit eine weltweite, gebührenfreie, nicht übertragbare, nicht-exklusive, unwiderrufliche Lizenz zur Ausübung der lizenzierten Rechte im lizenzierten Material, um:

A. das lizenzierte Material ganz oder teilweise für nichtkommerzielle Zwecke zu vervielfältigen und zu teilen; und

B. adaptiertes Material für nichtkommerzielle Zwecke zu produzieren, zu vervielfältigen und zu teilen.

2. __Ausnahmen und Einschränkungen.__ Um Zweifel auszuräumen, wo Ausnahmen und Einschränkungen auf Ihre Nutzung anwendbar sind, gilt diese öffentliche Lizenz nicht, und Sie müssen die Bedingungen nicht einhalten.

3. __Laufzeit.__ Die Laufzeit dieser öffentlichen Lizenz ist in Abschnitt 6(a) angegeben.

4. __Medien und Formate; technische Modifikationen erlaubt.__ Der Lizenzgeber autorisiert Sie, die lizenzierten Rechte in allen Medien und Formaten auszuüben, die jetzt bekannt sind oder künftig geschaffen werden, und technische Modifikationen vorzunehmen, die erforderlich sind, um dies zu tun. Der Lizenzgeber verzichtet auf und/oder erklärt sich nicht bereit, irgendein Recht oder eine Autorität geltend zu machen, um Ihnen zu verbieten, technische Modifikationen vorzunehmen, die erforderlich sind, um die lizenzierten Rechte auszuüben, einschließlich technischer Modifikationen, die erforderlich sind, um effektive technologische Maßnahmen zu umgehen. Für die Zwecke dieser öffentlichen Lizenz führt das bloße Vornehmen von Modifikationen, die durch diesen Abschnitt 2(a)(4) autorisiert sind, niemals zu adaptivem Material.

5. __Nachgelagerte Empfänger.__

A. __Angebot des Lizenzgebers – lizenziertes Material.__ Jeder Empfänger des lizenzierten Materials erhält automatisch ein Angebot des Lizenzgebers, die lizenzierten Rechte unter den Bedingungen dieser öffentlichen Lizenz auszuüben.

B. __Keine nachgelagerten Einschränkungen.__ Sie dürfen keine zusätzlichen oder anderen Bedingungen oder Einschränkungen auf das lizenzierte Material anbieten oder auferlegen oder irgendwelche effektiven technologischen Maßnahmen auf das lizenzierte Material anwenden, wenn dies die Ausübung der lizenzierten Rechte durch einen Empfänger des lizenzierten Materials einschränkt.

6. __Keine Billigung.__ Nichts in dieser öffentlichen Lizenz stellt eine Erlaubnis dar oder kann so ausgelegt werden, dass Sie oder Ihre Nutzung des lizenzierten Materials mit dem Lizenzgeber oder anderen, die zur Erhaltung der Namensnennung gemäß Abschnitt 3(a)(1)(A)(i) benannt sind, verbunden, gesponsert, gebilligt oder offiziell anerkannt sind.

b. ___Andere Rechte.___

1. Moralische Rechte, wie das Recht auf Integrität, sind nicht unter dieser öffentlichen Lizenz lizenziert, ebenso wenig wie Werbe-, Datenschutz- und/oder andere ähnliche Persönlichkeitsrechte; jedoch verzichtet der Lizenzgeber, soweit möglich, auf und/oder erklärt sich nicht bereit, solche Rechte, die der Lizenzgeber hält, in dem begrenzten Umfang geltend zu machen, der erforderlich ist, um Ihnen die Ausübung der lizenzierten Rechte zu ermöglichen, jedoch nicht darüber hinaus.

2. Patent- und Markenrechte sind nicht unter dieser öffentlichen Lizenz lizenziert.

3. Soweit möglich, verzichtet der Lizenzgeber auf jegliches Recht, von Ihnen Lizenzgebühren für die Ausübung der lizenzierten Rechte zu erheben, sei es direkt oder über eine Verwertungsgesellschaft im Rahmen eines freiwilligen oder abtretbaren gesetzlichen oder obligatorischen Lizenzierungsprogramms. In allen anderen Fällen behält sich der Lizenzgeber ausdrücklich das Recht vor, solche Lizenzgebühren zu erheben, einschließlich wenn das lizenzierte Material für andere als nichtkommerzielle Zwecke verwendet wird.

## Abschnitt 3 – Lizenzbedingungen.

Ihre Ausübung der lizenzierten Rechte unterliegt ausdrücklich den folgenden Bedingungen.

a. ___Namensnennung.___

1. Wenn Sie das lizenzierte Material (einschließlich in modifizierter Form) teilen, müssen Sie:

A. Folgendes beibehalten, wenn es vom Lizenzgeber mit dem lizenzierten Material bereitgestellt wird:

i. Identifizierung der Schöpfer des lizenzierten Materials und aller anderen, die zur Erhaltung der Namensnennung benannt sind, in einer angemessenen Weise, die vom Lizenzgeber angefordert wird (einschließlich durch Pseudonym, wenn benannt);

ii. einen Urheberrechtshinweis;

iii. einen Hinweis, der auf diese öffentliche Lizenz verweist;

iv. einen Hinweis, der auf den Haftungsausschluss verweist;

v. eine URI oder einen Hyperlink zum lizenzierten Material, soweit dies angemessen möglich ist;

B. angeben, ob Sie das lizenzierte Material modifiziert haben, und eine Angabe über vorherige Modifikationen beibehalten; und

C. angeben, dass das lizenzierte Material unter dieser öffentlichen Lizenz lizenziert ist, und den Text oder die URI oder den Hyperlink zu dieser öffentlichen Lizenz einfügen.

2. Sie können die Bedingungen in Abschnitt 3(a)(1) auf jede angemessene Weise erfüllen, die auf dem Medium, den Mitteln und dem Kontext basiert, in dem Sie das lizenzierte Material teilen. Zum Beispiel kann es angemessen sein, die Bedingungen zu erfüllen, indem Sie eine URI oder einen Hyperlink zu einer Ressource bereitstellen, die die erforderlichen Informationen enthält.

3. Wenn der Lizenzgeber dies anfordert, müssen Sie alle Informationen, die in Abschnitt 3(a)(1)(A) erforderlich sind, soweit dies angemessen möglich ist, entfernen.

4. Wenn Sie adaptiertes Material teilen, das Sie produzieren, darf die Lizenz des Adapters, die Sie anwenden, die Empfänger des adaptierten Materials nicht daran hindern, diese öffentliche Lizenz einzuhalten.

## Abschnitt 4 – Sui Generis-Datenbankrechte.

Wenn die lizenzierten Rechte Sui Generis-Datenbankrechte umfassen, die auf Ihre Nutzung des lizenzierten Materials anwendbar sind:

a. um Zweifel auszuräumen, gewährt Abschnitt 2(a)(1) Ihnen das Recht, alle oder einen wesentlichen Teil des Inhalts der Datenbank für nichtkommerzielle Zwecke zu extrahieren, wiederzuverwenden, zu reproduzieren und zu teilen;

b. wenn Sie alle oder einen wesentlichen Teil des Inhalts der Datenbank in einer Datenbank einfügen, in der Sie Sui Generis-Datenbankrechte haben, dann ist die Datenbank, in der Sie Sui Generis-Datenbankrechte haben (aber nicht deren einzelne Inhalte), adaptiertes Material; und

c. Sie müssen die Bedingungen in Abschnitt 3(a) einhalten, wenn Sie alle oder einen wesentlichen Teil des Inhalts der Datenbank teilen.

Um Zweifel auszuräumen, ergänzt dieser Abschnitt 4 Ihre Verpflichtungen unter dieser öffentlichen Lizenz, wenn die lizenzierten Rechte andere Urheberrechte und ähnliche Rechte umfassen.

## Abschnitt 5 – Haftungsausschluss und Haftungsbeschränkung.

a. __Sofern nicht anders vom Lizenzgeber gesondert übernommen, bietet der Lizenzgeber das lizenzierte Material nach dem Prinzip „wie es ist“ und „wie verfügbar“ an und gibt keine Zusicherungen oder Garantien jeglicher Art bezüglich des lizenzierten Materials, weder ausdrücklich, stillschweigend, gesetzlich noch anderweitig. Dies umfasst, ohne Einschränkung, Garantien bezüglich des Titels, der Marktfähigkeit, der Eignung für einen bestimmten Zweck, der Nichtverletzung, des Fehlens von versteckten oder anderen Mängeln, der Genauigkeit oder des Vorhandenseins oder Fehlens von Fehlern, unabhängig davon, ob bekannt oder entdeckbar. Wo Haftungsausschlüsse nicht vollständig oder teilweise zulässig sind, kann dieser Haftungsausschluss möglicherweise nicht auf Sie angewendet werden.__

b. __Soweit möglich, haftet der Lizenzgeber Ihnen in keinem Fall auf irgendeiner rechtlichen Theorie (einschließlich, aber nicht beschränkt auf, Fahrlässigkeit) oder anderweitig für direkte, spezielle, indirekte, zufällige, Folgeschäden, Strafschäden, exemplarische oder andere Verluste, Kosten, Ausgaben oder Schäden, die aus dieser öffentlichen Lizenz oder der Nutzung des lizenzierten Materials entstehen, selbst wenn der Lizenzgeber auf die Möglichkeit solcher Verluste, Kosten, Ausgaben oder Schäden hingewiesen wurde. Wo eine Haftungsbeschränkung nicht vollständig oder teilweise zulässig ist, kann diese Beschränkung möglicherweise nicht auf Sie angewendet werden.__

c. Der Haftungsausschluss und die Haftungsbeschränkung, die oben angegeben sind, werden so interpretiert, dass sie, soweit möglich, am ehesten einem absoluten Haftungsausschluss und Verzicht auf alle Haftungen entsprechen.

## Abschnitt 6 – Laufzeit und Kündigung.

a. Diese öffentliche Lizenz gilt für die Laufzeit der hier lizenzierten Urheberrechte und ähnlichen Rechte. Wenn Sie jedoch diese öffentliche Lizenz nicht einhalten, erlöschen Ihre Rechte unter dieser öffentlichen Lizenz automatisch.

b. Wenn Ihr Recht zur Nutzung des lizenzierten Materials gemäß Abschnitt 6(a) erloschen ist, wird es wiederhergestellt:

1. automatisch ab dem Datum, an dem der Verstoß behoben wird, vorausgesetzt, er wird innerhalb von 30 Tagen nach Ihrer Entdeckung des Verstoßes behoben; oder

2. nach ausdrücklicher Wiederherstellung durch den Lizenzgeber.

Um Zweifel auszuräumen, wirkt sich dieser Abschnitt 6(b) nicht auf das Recht des Lizenzgebers aus, Rechtsmittel für Ihre Verstöße gegen diese öffentliche Lizenz zu suchen.

c. Um Zweifel auszuräumen, kann der Lizenzgeber das lizenzierte Material auch unter separaten Bedingungen oder Konditionen anbieten oder die Verbreitung des lizenzierten Materials jederzeit einstellen; jedoch wird dadurch diese öffentliche Lizenz nicht beendet.

d. Die Abschnitte 1, 5, 6, 7 und 8 überdauern die Beendigung dieser öffentlichen Lizenz.

## Abschnitt 7 – Andere Bedingungen und Konditionen.

a. Der Lizenzgeber ist nicht an zusätzliche oder andere Bedingungen oder Konditionen gebunden, die von Ihnen kommuniziert werden, es sei denn, dies wurde ausdrücklich vereinbart.

b. Alle Vereinbarungen, Verständnisse oder Absprachen bezüglich des lizenzierten Materials, die hierin nicht angegeben sind, sind unabhängig von den Bedingungen dieser öffentlichen Lizenz.

## Abschnitt 8 – Auslegung.

a. Um Zweifel auszuräumen, reduziert, beschränkt, schränkt diese öffentliche Lizenz nicht ein oder legt keine Bedingungen für die Nutzung des lizenzierten Materials fest, die rechtmäßig ohne Erlaubnis gemäß dieser öffentlichen Lizenz vorgenommen werden könnte.

b. Soweit möglich, wenn eine Bestimmung dieser öffentlichen Lizenz als nicht durchsetzbar erachtet wird, wird sie automatisch in dem minimalen Umfang reformiert, der erforderlich ist, um sie durchsetzbar zu machen. Wenn die Bestimmung nicht reformiert werden kann, wird sie von dieser öffentlichen Lizenz getrennt, ohne die Durchsetzbarkeit der verbleibenden Bedingungen und Konditionen zu beeinträchtigen.

c. Keine Bedingung oder Klausel dieser öffentlichen Lizenz wird aufgegeben und kein Versäumnis, sich daran zu halten, wird akzeptiert, es sei denn, dies wurde ausdrücklich vom Lizenzgeber vereinbart.

d. Nichts in dieser öffentlichen Lizenz stellt eine Einschränkung oder einen Verzicht auf Privilegien und Immunitäten dar, die auf den Lizenzgeber oder Sie anwendbar sind, einschließlich der rechtlichen Verfahren einer Gerichtsbarkeit oder Behörde.
```
Creative Commons is not a party to its public licenses. Notwithstanding, Creative Commons may elect to apply one of its public licenses to material it publishes and in those instances will be considered the “Licensor.” Except for the limited purpose of indicating that material is shared under a Creative Commons public license or as otherwise permitted by the Creative Commons policies published at [creativecommons.org/policies](http://creativecommons.org/policies), Creative Commons does not authorize the use of the trademark “Creative Commons” or any other trademark or logo of Creative Commons without its prior written consent including, without limitation, in connection with any unauthorized modifications to any of its public licenses or any other arrangements, understandings, or agreements concerning use of licensed material. For the avoidance of doubt, this paragraph does not form part of the public licenses.

Creative Commons may be contacted at [creativecommons.org](http://creativecommons.org/).
```
{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}
