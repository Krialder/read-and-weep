# Sichere Messaging-Plattform

Verschlüsselte Kommunikation, die heute hält und auch dann noch, wenn in ein paar Jahren die ersten Quantencomputer ernst werden. Ein Konto über Web, Mobile und Desktop. Der Server bekommt nur Chiffretext zu sehen, die Schlüssel bleiben beim Nutzer.

Diese Doku ist eine Übersicht plus die paar Specs, an denen so ein System wirklich steht oder fällt. Was noch nicht entschieden ist, hängt sichtbar in [offene-fragen.md](offene-fragen.md). Das ist Absicht, kein Versäumnis.

## Die Grundentscheidungen

1. Krypto aus geprüften Bibliotheken, über einen gemeinsamen Rust-Core. libsignal für PQXDH und den Double Ratchet, OpenMLS für Gruppen. Einmal geschrieben, über alle Clients geteilt (WASM im Web, FFI nativ). Keine Eigenkrypto. Begründung in [ADR-0005](entscheidungen/0005-krypto-bibliotheken.md).
2. Zentraler Kern, der sich den Weg zu Föderation nicht verbaut. Erstmal ein Betreiber, das schützt Metadaten am besten. Föderation ist später möglich, aber kein Schalter, sondern eine andere Vertrauenstopologie. Die Abwägung steht in [ADR-0001](entscheidungen/0001-hybrid-kern.md).
3. Multi-Plattform, native Clients von Anfang an parallel. Wer mit dem Web-Client allein startet, zementiert den schwächsten Client als Default. Das machen wir nicht.
4. Identität über Username, ohne Telefonnummer. Kontakte über Handles, die Echtheit der Schlüssel sichert Key Transparency.

## Was die Plattform kann

- 1:1-Chats und Gruppen, Ende-zu-Ende verschlüsselt (Double Ratchet beziehungsweise MLS).
- Schlüsselaustausch, der gegen künftige Quantencomputer hält (PQXDH, hybrid).
- Mehrere Geräte pro Konto, jedes mit eigenen Schlüsseln.
- Metadaten so weit runtergedrückt, wie es geht (Sealed Sender, keine Telefonnummern).
- Eine Warnung, wenn jemand beim Schlüsselaustausch zu tricksen versucht (Key Transparency).
- Dateiversand und verschlüsselte Anrufe sind geplant, beides ein eigenes Kapitel für sich.

## Wegweiser

| Datei | Inhalt |
|-------|--------|
| [architektur.md](architektur.md) | Dienste, der Krypto-Core, Echtzeit, Skalierung |
| [protokoll.md](protokoll.md) | Krypto im Überblick, mit Verweis auf die tiefen Specs |
| [sicherheit.md](sicherheit.md) | Bedrohungsmodell und die Grenzen |
| [identitaet-und-geraete.md](identitaet-und-geraete.md) | Konten, Handles, Multi-Device, Verifikation |
| [datenmodell.md](datenmodell.md) | Was der Server speichert |
| [schnittstellen.md](schnittstellen.md) | APIs und Echtzeit-Events |
| [betrieb.md](betrieb.md) | Deployment, Skalierung, die teuren Pfade |
| [roadmap.md](roadmap.md) | Reihenfolge, mit einer Spec-Phase vor dem Code |
| [offene-fragen.md](offene-fragen.md) | Was noch nicht entschieden ist |
| [specs/](specs/) | Key Transparency, Sealed Sender und Abuse, Krypto-Kern |
| [entscheidungen/](entscheidungen/) | Die ADRs zu den großen Festlegungen |

## Stand

Frisch aus der Planung. Die Architektur steht, die kritischen Specs (Key Transparency, Sealed Sender und Abuse, die Krypto-Bindung) sind ausgearbeitet, der Rest hängt sichtbar in [offene-fragen.md](offene-fragen.md). Drei Dinge gehören vorneweg gesagt, weil sie leicht überlesen werden:

- Gruppen sind aktuell nicht post-quantum. MLS nach RFC 9420 ist in den Standard-Suites klassisch, PQ-MLS läuft noch über einen IETF-Draft. 1:1 ist über PQXDH schon post-quantum.
- Backup und Wiederherstellung sind nur grob skizziert. Ohne eine ordentliche Lösung verlieren reale Nutzer ihre History. Das ist offen.
- Ein externes Sicherheitsaudit hat nicht stattgefunden und muss vor jedem echten Einsatz her, mindestens zweimal.
