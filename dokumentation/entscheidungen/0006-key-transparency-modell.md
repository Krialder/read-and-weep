# 0006: Key-Transparency-Modell

**Status:** akzeptiert

## Kontext

Ende-zu-Ende-Verschlüsselung gegen einen aktiven, bösartigen Server hält nur, wenn der Client prüfen kann, dass der ausgelieferte Schlüssel echt ist. Ein naheliegender erster Versuch wäre ein simples, nur-anhängbares Log der Schlüssel. Das ist aber bloß ein Audit-Trail, kein Schutz: Es verhindert nicht, dass der Server einem Opfer einen anderen Schlüssel zeigt als dem Rest der Welt.

## Entscheidung

Wir setzen ein vollwertiges Key-Transparency-System um, in der Linie von CONIKS und den späteren Systemen (Google KT, Metas KT). Die Bausteine: eine verifiable Merkle-Map, eine VRF über das Handle gegen das Durchzählen der Nutzerliste, signierte Wurzeln pro Epoche mit Konsistenzbeweisen, Selbstprüfung der eigenen Schlüsselhistorie im Client, und unabhängige Auditoren plus Gossip gegen den Split-View-Angriff. Ausgearbeitet in [specs/key-transparency.md](../specs/key-transparency.md).

## Konsequenzen

Damit fällt ein stiller Schlüsseltausch auf, offen wie versteckt, ohne dass jeder Nutzer von Hand verifizieren muss. Die Safety Number bleibt als manueller Zusatz für die Vorsichtigen.

Das ist ein eigenes Subsystem mit eigenem Datenmodell und eigener Betriebslast, kein Feature, das man nebenbei einbaut. Für den Start ist entschieden: Selbstprüfung im Client ab Phase 1, ein unabhängiger Auditor ab Phase 3. Offen bleibt die Governance, also wer den Auditor betreibt. Ohne eine unabhängige Instanz ist der Schutz gegen den Split View schwächer. Diese Frage steht in [offene-fragen.md](../offene-fragen.md).
