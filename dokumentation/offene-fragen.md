# Offene Fragen

Was noch nicht entschieden ist. Die Liste ist bewusst lang und ehrlich. Wenn hier etwas steht, heißt das, wir wissen es noch nicht, und das ist in dieser Phase in Ordnung. Jeder Punkt sollte fallen, bevor das zugehörige Stück gebaut wird.

## Krypto-Kern

- AES-128-GCM gegen ChaCha20-Poly1305 als MLS-Suite. Standard-Kompatibilität gegen einen einheitlichen symmetrischen Stack.
- Zeitpunkt für PQ-MLS. Hängt am Combiner-Draft und an OpenMLS. Bis dahin sind Gruppen klassisch.
- Genaues Wire-Format der Versionsaushandlung und der Suite-IDs.
- Stichtags-Mechanik für erzwungene Re-Handshakes bei einem Suite-Wechsel.
- Wie SPQR-Versionen in unsere eigene Wire-Version eingehängt werden.
- Die exakten PQXDH-Labels und Byte-Layouts, gepinnt gegen die libsignal-Version, die wir nehmen.

## Key Transparency

- Wer betreibt die Auditoren. Selbst, Konsortium, oder Dritte. Ohne unabhängige Instanz ist der Split-View-Schutz schwächer.
- Transport fürs Gossip, in-band über Clients oder out-of-band.
- Epochenlänge. Kürzer heißt schnellere Sichtbarkeit, aber mehr Last.
- Wie ein frischer Client dem Signaturschlüssel der Wurzeln vertraut (Bootstrapping).
- Beschneidung langer Schlüsselhistorien pro Konto.
- Commitment-Schema für mehrere Geräte unter einem Konto.
- Konkrete Merkle-Bauart und KT-Bibliothek.

## Sealed Sender und Abuse

- Sybil-Widerstand bei der Registrierung. Proof-of-Work, Privacy Pass, Invite, oder eine Mischung. Das ist die wichtigste offene Produktfrage.
- Lebensdauer und Rotation der Sender-Certificates.
- Schlüsselverwaltung fürs Franking und das Commitment-Schema.
- Laufen Erstkontakte grundsätzlich in eine Anfrage-Inbox.
- Zugangstoken und Multi-Device. Ein Profil-Schlüssel, mehrere Geräte.

## Identität, Geräte, Recovery

- Protokoll fürs Geräte-zu-Geräte-Übertragen des Verlaufs an ein neues Gerät.
- Secure Value Recovery ja oder nein. Ohne wird die Retention schlecht, mit ist es ein eigenes Großthema (Enclaves, PIN, hartes Rate-Limit). Das ist eine der größten offenen Entscheidungen.
- Was passiert mit Gruppen, wenn ein Nutzer alle Geräte verliert.

## Betrieb und Skalierung

- Konkrete Rate-Limit-Schwellen fürs Schlüssel-Verzeichnis und Fallback-Verhalten bei leerem Prekey-Vorrat (Fallback auf den Signed Prekey schwächt PQXDH für diese Sitzungen).
- Bündelungs- und Verzögerungsstrategie für Push, um Timing-Metadaten zu drücken.
- Endgültige Sprache der Realtime-Schicht. Go überall, oder Elixir/BEAM für den Realtime- und Presence-Teil, der historisch (WhatsApp) damit gut skaliert.
- Verschlüsselungsschema für Medien im Detail (Chunking, Schlüssel pro Datei, Resumable Uploads).
- Quotas und DoS-Schwellen auf den teuren Pfaden.

## Compliance und Recht

- Position zu UK Online Safety Act und der EU-Chat-Kontrolle (CSAR). Ein System, das technisch keine Inhalte herausgeben kann, ist eine politische Haltung, die man begründen muss.
- Umgang mit behördlichen Anfragen, die sich nur auf Metadaten richten können.
- Aufbewahrungsfristen rechtlich sauber festlegen.

## Anrufe

- SFrame oder eine Alternative für die Medienverschlüsselung.
- Group-Calling über den MLS-Exporter, Schlüsselableitung pro Call.
- Identitätsbindung der Call-Schlüssel, damit niemand sich in einen Call schiebt.

## Föderation (falls überhaupt)

- Serverübergreifende Key Transparency, das ist deutlich schwerer als bei einem Betreiber.
- Server-Discovery-Protokoll.
- S2S-Authentifizierung und die Vertrauensannahmen zwischen Betreibern.

## Qualität und Verifikation

- Umfang der formalen Modelle (ProVerif, Tamarin) und welche Eigenschaften wir beweisen wollen.
- Fuzzing-Strategie für die Parser und den Wire-Code.
- Property-Tests für die Krypto-Integration (Round-Trip, Tampering, Out-of-Order).
