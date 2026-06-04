# Spec: Key Transparency

Das hier ist das Subsystem, das am leichtesten unterschätzt wird, und an dem ein E2E-System tatsächlich steht oder fällt. Verschlüsseln ist gelöst. Die offene Frage ist: Woher weiß mein Client, dass der Schlüssel, den der Server mir für einen Kontakt ausliefert, wirklich dem Kontakt gehört, und nicht dem Betreiber, der mitlesen will?

Ohne eine Antwort darauf ist jede E2E-Verschlüsselung gegen einen aktiven, bösartigen Server wertlos. Er liefert dir einfach seinen eigenen Schlüssel, du verschlüsselst an ihn, er liest und reicht weiter. Key Transparency macht genau diesen Tausch entdeckbar.

## Die Idee

Der Server führt alle Schlüssel in einem öffentlich nachprüfbaren, nur-anhängbaren Verzeichnis. Technisch ist das eine verifiable map (Label zu Wert) auf Basis eines Merkle-Baums, deren Wurzel der Server regelmäßig signiert veröffentlicht. Jeder kann zu einem Handle einen Beweis verlangen, dass der ausgelieferte Schlüssel genau der ist, der im Verzeichnis steht. Der Server kann also nicht zwei Wahrheiten fahren, ohne dass es auffällt.

Das Modell folgt der Linie von CONIKS und den späteren Systemen (Google Key Transparency, Metas KT für WhatsApp 2023 und Messenger 2025, Keybase). Wir erfinden hier nichts, wir setzen ein bekanntes Design um.

## VRF gegen das Durchzählen

Ein naiver Merkle-Baum, der direkt auf dem Handle aufsetzt, hätte ein Datenschutzproblem: Jeder könnte den Baum ablaufen und die komplette Nutzerliste herausziehen. Bei selbstgewählten Usernamen ist das besonders übel, weil die wenig Entropie haben und sich raten lassen.

Deshalb läuft das Label nicht über den Klartext-Handle, sondern über eine VRF (verifiable random function): `label = VRF(handle)`. Die VRF bildet den Handle auf einen undurchsichtigen Index ab und liefert gleichzeitig einen Beweis, dass die Abbildung korrekt ist. Der Server kann Inklusion beweisen, ohne zu verraten, welche Handles überhaupt existieren. Wer den Handle nicht kennt, kann nichts nachschlagen.

## Epochen und signierte Wurzeln

Das Verzeichnis arbeitet in Epochen. Am Ende jeder Epoche veröffentlicht der Server eine signierte Wurzel (signed tree head, STH) mit Epochennummer und Zeitstempel. Zwei Eigenschaften müssen über die Epochen hinweg beweisbar sein:

1. Append-only. Eine neue Wurzel enthält alles aus der alten. Der Server kann keine Historie umschreiben, nur anhängen.
2. Konsistenz. Zwischen zwei Wurzeln gibt es einen Konsistenzbeweis, der zeigt, dass die neue aus der alten hervorgegangen ist.

## Selbstprüfung durch den Client

Jeder Client überwacht seine eigene Schlüsselhistorie. Er weiß, welche Schlüssel er selbst veröffentlicht hat, und prüft regelmäßig, dass im Verzeichnis genau die stehen und kein fremder dazugekommen ist. Schiebt der Server einen Rogue-Key für dich ein, um in deinem Namen zu empfangen, taucht der in deinem eigenen Eintrag auf, und dein Client schlägt Alarm. Das ist der entscheidende Hebel: Der Angriff lässt sich nicht verstecken, weil das Opfer selbst mitliest.

## Auditoren und Gossip

Bleibt ein Angriff übrig: der split view. Der Server zeigt dir eine Verzeichnis-Version und deinem Kontakt eine andere. Dagegen helfen unabhängige Auditoren (Witnesses), die die Folge der signierten Wurzeln beobachten und auf Append-only und Konsistenz prüfen, plus Gossip: Clients und Auditoren vergleichen die Wurzeln, die sie sehen. Tauchen zwei widersprüchliche Wurzeln zur selben Epoche auf, ist der Betreiber überführt.

Wer die Auditoren betreibt, ist eine Vertrauens- und Betriebsfrage, und sie ist noch offen (siehe unten und [offene-fragen.md](../offene-fragen.md)).

## Was der Server hält

- Den Merkle-Baum der verifiable map (Label zu Schlüssel-Commitment).
- Die Folge der signierten Wurzeln pro Epoche.
- Den VRF-Schlüssel (privat) und den Signaturschlüssel für die Wurzeln.
- Pro Konto die aktuelle Geräte- und Schlüsselmenge, auf die das Commitment zeigt.

Der grobe Tabellen-Rahmen dazu steht in [datenmodell.md](../datenmodell.md). Die genaue Merkle-Bauart (Sparse Merkle Tree, Indexierung, Commitment-Schema) zurren wir gegen eine konkrete Bibliothek fest, bevor sich das Datenmodell zementiert.

## Was es schlägt, was nicht

Geschlagen: der stille Schlüsseltausch durch den Server, sowohl offen (Selbstprüfung) als auch versteckt über zwei Wahrheiten (Gossip plus Auditoren).

Nicht geschlagen: Ein Erstkontakt bleibt ein Vertrauen-beim-ersten-Sehen, bis genug Epochen und Audits vergangen sind. Und KT sagt dir, dass ein Schlüssel echt im Verzeichnis steht, nicht, ob die Person dahinter die ist, für die du sie hältst. Dafür gibt es zusätzlich die Safety Number zum manuellen Abgleich.

## Offen

- Wer betreibt die Auditoren: wir selbst, ein Konsortium, oder Dritte. Ohne unabhängige Instanz ist der split-view-Schutz schwächer.
- Transport für Gossip (in-band über andere Clients, oder out-of-band).
- Epochenlänge (Minuten gegen Stunden), Abwägung Latenz der Sichtbarkeit gegen Last.
- Wie ein neuer Client dem Signaturschlüssel der Wurzeln überhaupt vertraut (Bootstrapping).
- Länge und Beschneidung der Schlüsselhistorie pro Konto.
- Genaues Commitment-Schema für mehrere Geräte pro Konto.
