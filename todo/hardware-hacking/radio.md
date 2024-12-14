# Radio

{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}

## SigDigger

[**SigDigger** ](https://github.com/BatchDrake/SigDigger)ist ein kostenloser digitaler Signalanalysator für GNU/Linux und macOS, der entwickelt wurde, um Informationen aus unbekannten Funksignalen zu extrahieren. Er unterstützt eine Vielzahl von SDR-Geräten über SoapySDR und ermöglicht eine anpassbare Demodulation von FSK-, PSK- und ASK-Signalen, dekodiert analoges Video, analysiert burstige Signale und hört analoge Sprachkanäle (alles in Echtzeit).

### Grundkonfiguration

Nach der Installation gibt es einige Dinge, die Sie in Betracht ziehen könnten, zu konfigurieren.\
In den Einstellungen (der zweite Tab-Button) können Sie das **SDR-Gerät** auswählen oder **eine Datei** zum Lesen auswählen und die Frequenz, auf die Sie syntonisieren möchten, sowie die Abtastrate (empfohlen bis zu 2,56 Msps, wenn Ihr PC dies unterstützt)\\

![](<../../.gitbook/assets/image (245).png>)

Im GUI-Verhalten wird empfohlen, einige Dinge zu aktivieren, wenn Ihr PC dies unterstützt:

![](<../../.gitbook/assets/image (472).png>)

{% hint style="info" %}
Wenn Sie feststellen, dass Ihr PC keine Signale erfasst, versuchen Sie, OpenGL zu deaktivieren und die Abtastrate zu senken.
{% endhint %}

### Anwendungen

* Um **einige Zeit eines Signals zu erfassen und zu analysieren**, halten Sie einfach die Taste "Push to capture" so lange gedrückt, wie Sie benötigen.

![](<../../.gitbook/assets/image (960).png>)

* Der **Tuner** von SigDigger hilft, **bessere Signale zu erfassen** (aber er kann sie auch verschlechtern). Idealerweise beginnen Sie mit 0 und erhöhen **es, bis** Sie feststellen, dass das **Rauschen**, das eingeführt wird, **größer** ist als die **Verbesserung des Signals**, die Sie benötigen.

![](<../../.gitbook/assets/image (1099).png>)

### Synchronisieren mit dem Funkkanal

Mit [**SigDigger** ](https://github.com/BatchDrake/SigDigger) synchronisieren Sie sich mit dem Kanal, den Sie hören möchten, konfigurieren die Option "Baseband audio preview", konfigurieren die Bandbreite, um alle gesendeten Informationen zu erhalten, und stellen dann den Tuner auf das Niveau ein, bevor das Rauschen wirklich zu steigen beginnt:

![](<../../.gitbook/assets/image (585).png>)

## Interessante Tricks

* Wenn ein Gerät Informationsbursts sendet, ist normalerweise der **erste Teil ein Präambel**, sodass Sie sich **keine Sorgen machen müssen**, wenn Sie **keine Informationen** darin **finden oder wenn es einige Fehler** gibt.
* In Informationsrahmen sollten Sie normalerweise **verschiedene Rahmen gut ausgerichtet zueinander finden**:

![](<../../.gitbook/assets/image (1076).png>)

![](<../../.gitbook/assets/image (597).png>)

* **Nachdem Sie die Bits wiederhergestellt haben, müssen Sie sie möglicherweise irgendwie verarbeiten**. Zum Beispiel bedeutet in der Manchester-Codierung ein up+down eine 1 oder 0 und ein down+up wird die andere sein. Paare von 1s und 0s (ups und downs) werden eine echte 1 oder eine echte 0 sein.
* Selbst wenn ein Signal die Manchester-Codierung verwendet (es ist unmöglich, mehr als zwei 0s oder 1s hintereinander zu finden), könnten Sie **mehrere 1s oder 0s zusammen in der Präambel finden**!

### Aufdecken des Modulationstyps mit IQ

Es gibt 3 Möglichkeiten, Informationen in Signalen zu speichern: Modulation der **Amplitude**, **Frequenz** oder **Phase**.\
Wenn Sie ein Signal überprüfen, gibt es verschiedene Möglichkeiten, um herauszufinden, was verwendet wird, um Informationen zu speichern (finden Sie mehr Möglichkeiten unten), aber eine gute Möglichkeit ist, das IQ-Diagramm zu überprüfen.

![](<../../.gitbook/assets/image (788).png>)

* **AM erkennen**: Wenn im IQ-Diagramm beispielsweise **2 Kreise** erscheinen (wahrscheinlich einer bei 0 und der andere bei einer anderen Amplitude), könnte das bedeuten, dass es sich um ein AM-Signal handelt. Dies liegt daran, dass im IQ-Diagramm der Abstand zwischen 0 und dem Kreis die Amplitude des Signals ist, sodass es einfach ist, verschiedene verwendete Amplituden zu visualisieren.
* **PM erkennen**: Wie im vorherigen Bild, wenn Sie kleine Kreise finden, die nicht miteinander verbunden sind, bedeutet das wahrscheinlich, dass eine Phasenmodulation verwendet wird. Dies liegt daran, dass im IQ-Diagramm der Winkel zwischen dem Punkt und dem 0,0 die Phase des Signals ist, was bedeutet, dass 4 verschiedene Phasen verwendet werden.
* Beachten Sie, dass, wenn die Informationen im Faktum verborgen sind, dass eine Phase geändert wird und nicht in der Phase selbst, Sie keine klar differenzierten Phasen sehen werden.
* **FM erkennen**: IQ hat kein Feld zur Identifizierung von Frequenzen (Abstand zum Zentrum ist Amplitude und Winkel ist Phase).\
Daher sollten Sie zur Identifizierung von FM **grundsätzlich nur einen Kreis** in diesem Diagramm sehen.\
Darüber hinaus wird eine andere Frequenz im IQ-Diagramm durch eine **Geschwindigkeitsbeschleunigung über den Kreis** "repräsentiert" (wenn Sie in SysDigger das Signal auswählen, wird das IQ-Diagramm gefüllt; wenn Sie eine Beschleunigung oder Richtungsänderung im erzeugten Kreis finden, könnte das bedeuten, dass dies FM ist):

## AM-Beispiel

{% file src="../../.gitbook/assets/sigdigger_20220308_165547Z_2560000_433500000_float32_iq.raw" %}

### Aufdecken von AM

#### Überprüfen der Hüllkurve

Überprüfen von AM-Informationen mit [**SigDigger** ](https://github.com/BatchDrake/SigDigger) und nur durch Betrachten der **Hüllkurve** können Sie verschiedene klare Amplitudenebenen sehen. Das verwendete Signal sendet Pulse mit Informationen in AM, so sieht ein Puls aus:

![](<../../.gitbook/assets/image (590).png>)

Und so sieht ein Teil des Symbols mit der Wellenform aus:

![](<../../.gitbook/assets/image (734).png>)

#### Überprüfen des Histogramms

Sie können **das gesamte Signal auswählen**, in dem sich die Informationen befinden, den **Amplitude**-Modus und die **Auswahl** auswählen und auf **Histogramm** klicken. Sie können beobachten, dass nur 2 klare Ebenen gefunden werden.

![](<../../.gitbook/assets/image (264).png>)

Wenn Sie beispielsweise Frequenz anstelle von Amplitude in diesem AM-Signal auswählen, finden Sie nur 1 Frequenz (keine Möglichkeit, dass Informationen, die in Frequenz moduliert sind, nur 1 Frequenz verwenden).

![](<../../.gitbook/assets/image (732).png>)

Wenn Sie viele Frequenzen finden, wird dies wahrscheinlich kein FM sein; wahrscheinlich wurde die Frequenz des Signals nur aufgrund des Kanals modifiziert.

#### Mit IQ

In diesem Beispiel sehen Sie, wie es einen **großen Kreis** gibt, aber auch **viele Punkte im Zentrum**.

![](<../../.gitbook/assets/image (222).png>)

### Symbolrate ermitteln

#### Mit einem Symbol

Wählen Sie das kleinste Symbol aus, das Sie finden können (damit Sie sicher sind, dass es nur 1 ist), und überprüfen Sie die "Auswahlfreq". In diesem Fall wäre es 1,013 kHz (also 1 kHz).

![](<../../.gitbook/assets/image (78).png>)

#### Mit einer Gruppe von Symbolen

Sie können auch die Anzahl der Symbole angeben, die Sie auswählen möchten, und SigDigger wird die Frequenz von 1 Symbol berechnen (je mehr ausgewählte Symbole, desto besser wahrscheinlich). In diesem Szenario habe ich 10 Symbole ausgewählt und die "Auswahlfreq" beträgt 1,004 kHz:

![](<../../.gitbook/assets/image (1008).png>)

### Bits erhalten

Nachdem Sie festgestellt haben, dass es sich um ein **AM-moduliertes** Signal handelt und die **Symbolrate** (und wissen, dass in diesem Fall etwas up eine 1 und etwas down eine 0 bedeutet), ist es sehr einfach, die **Bits** zu erhalten, die im Signal codiert sind. Wählen Sie also das Signal mit Informationen aus und konfigurieren Sie die Abtastung und Entscheidung und drücken Sie auf Abtasten (stellen Sie sicher, dass **Amplitude** ausgewählt ist, die entdeckte **Symbolrate** konfiguriert ist und die **Gadner-Uhrensynchronisation** ausgewählt ist):

![](<../../.gitbook/assets/image (965).png>)

* **Sync zu Auswahlintervallen** bedeutet, dass, wenn Sie zuvor Intervalle ausgewählt haben, um die Symbolrate zu finden, diese Symbolrate verwendet wird.
* **Manuell** bedeutet, dass die angegebene Symbolrate verwendet wird.
* In der **Festintervallauswahl** geben Sie die Anzahl der Intervalle an, die ausgewählt werden sollen, und es berechnet die Symbolrate daraus.
* **Gadner-Uhrensynchronisation** ist normalerweise die beste Option, aber Sie müssen immer noch eine ungefähre Symbolrate angeben.

Wenn Sie auf Abtasten drücken, erscheint dies:

![](<../../.gitbook/assets/image (644).png>)

Jetzt, um SigDigger zu verstehen, **wo der Bereich** des Niveaus liegt, das Informationen trägt, müssen Sie auf das **untere Niveau** klicken und gedrückt halten, bis das größte Niveau erreicht ist:

![](<../../.gitbook/assets/image (439).png>)

Wenn es beispielsweise **4 verschiedene Amplitudenebenen** gegeben hätte, müssten Sie die **Bits pro Symbol auf 2** konfigurieren und von der kleinsten bis zur größten auswählen.

Schließlich können Sie durch **Erhöhen** des **Zooms** und **Ändern der Zeilenhöhe** die Bits sehen (und Sie können alles auswählen und kopieren, um alle Bits zu erhalten):

![](<../../.gitbook/assets/image (276).png>)

Wenn das Signal mehr als 1 Bit pro Symbol hat (zum Beispiel 2), hat SigDigger **keine Möglichkeit zu wissen, welches Symbol** 00, 01, 10, 11 ist, sodass es verschiedene **Graustufen** verwendet, um jedes darzustellen (und wenn Sie die Bits kopieren, verwendet es **Zahlen von 0 bis 3**, die Sie behandeln müssen).

Verwenden Sie auch **Codierungen** wie **Manchester**, und **up+down** kann **1 oder 0** sein und ein down+up kann eine 1 oder 0 sein. In diesen Fällen müssen Sie die erhaltenen Ups (1) und Downs (0) behandeln, um die Paare von 01 oder 10 als 0s oder 1s zu ersetzen.

## FM-Beispiel

{% file src="../../.gitbook/assets/sigdigger_20220308_170858Z_2560000_433500000_float32_iq.raw" %}

### Aufdecken von FM

#### Überprüfen der Frequenzen und Wellenform

Signalbeispiel, das Informationen moduliert in FM sendet:

![](<../../.gitbook/assets/image (725).png>)

Im vorherigen Bild können Sie ziemlich gut beobachten, dass **2 Frequenzen verwendet werden**, aber wenn Sie die **Wellenform** beobachten, könnten Sie **nicht in der Lage sein, die 2 verschiedenen Frequenzen korrekt zu identifizieren**:

![](<../../.gitbook/assets/image (717).png>)

Das liegt daran, dass ich das Signal in beiden Frequenzen erfasst habe, daher ist eine ungefähr die andere in negativ:

![](<../../.gitbook/assets/image (942).png>)

Wenn die synchronisierte Frequenz **näher an einer Frequenz als an der anderen** ist, können Sie die 2 verschiedenen Frequenzen leicht sehen:

![](<../../.gitbook/assets/image (422).png>)

![](<../../.gitbook/assets/image (488).png>)

#### Überprüfen des Histogramms

Wenn Sie das Frequenzhistogramm des Signals mit Informationen überprüfen, können Sie leicht 2 verschiedene Signale sehen:

![](<../../.gitbook/assets/image (871).png>)

In diesem Fall, wenn Sie das **Amplitude-Histogramm** überprüfen, werden Sie **nur eine Amplitude** finden, sodass es **nicht AM sein kann** (wenn Sie viele Amplituden finden, könnte es daran liegen, dass das Signal entlang des Kanals an Leistung verloren hat):

![](<../../.gitbook/assets/image (817).png>)

Und dies wäre das Phasenhistogramm (was sehr klar macht, dass das Signal nicht in Phase moduliert ist):

![](<../../.gitbook/assets/image (996).png>)

#### Mit IQ

IQ hat kein Feld zur Identifizierung von Frequenzen (Abstand zum Zentrum ist Amplitude und Winkel ist Phase).\
Daher sollten Sie zur Identifizierung von FM **grundsätzlich nur einen Kreis** in diesem Diagramm sehen.\
Darüber hinaus wird eine andere Frequenz im IQ-Diagramm durch eine **Geschwindigkeitsbeschleunigung über den Kreis** "repräsentiert" (wenn Sie in SysDigger das Signal auswählen, wird das IQ-Diagramm gefüllt; wenn Sie eine Beschleunigung oder Richtungsänderung im erzeugten Kreis finden, könnte das bedeuten, dass dies FM ist):

![](<../../.gitbook/assets/image (81).png>)

### Symbolrate ermitteln

Sie können die **gleiche Technik wie im AM-Beispiel verwenden**, um die Symbolrate zu ermitteln, sobald Sie die Frequenzen gefunden haben, die Symbole tragen.

### Bits erhalten

Sie können die **gleiche Technik wie im AM-Beispiel verwenden**, um die Bits zu erhalten, sobald Sie **festgestellt haben, dass das Signal in Frequenz moduliert ist** und die **Symbolrate**.

{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}
