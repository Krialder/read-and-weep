# 0002: Krypto-Stack

**Status:** akzeptiert

## Kontext

Eine sichere Messaging-Plattform steht und fällt mit der Wahl der Verfahren. Zwei Dinge waren gesetzt: nichts selbst bauen, und der Schutz muss auch dann halten, wenn in ein paar Jahren ein Quantencomputer alte Mitschnitte angeht.

## Entscheidung

Der Sitzungsaufbau läuft über PQXDH, hybrid aus X25519 und ML-KEM-768. Die laufende 1:1-Sitzung sichert der Double Ratchet plus der post-quantum Ratchet SPQR. Gruppen verschlüsselt MLS (RFC 9420) über TreeKEM. Den Absender verbirgt Sealed Sender, die Echtheit der Schlüssel sichert Key Transparency mit Safety Numbers als manuellem Zusatz. Symmetrisch ChaCha20-Poly1305 und AES-256-GCM, abgeleitet mit Argon2id und HKDF. Die Details der Bindung und der Suites stehen in [specs/krypto-kern.md](../specs/krypto-kern.md).

## Konsequenzen

1:1 ist damit post-quantum, mit Forward Secrecy und Post-Compromise Security. Die Echtheits-Frage beim Schlüsselaustausch ist über Key Transparency abgedeckt, nicht dem Nutzer allein aufgebürdet.

Ein ehrlicher Haken bleibt: Gruppen sind über MLS noch klassisch, weil die PQ-Variante (Combiner) erst als IETF-Draft existiert. Gruppen-Chiffretext von heute ist also gegen einen späteren Quantencomputer nicht sicher. Das ziehen wir nach, sobald der Draft steht, und es ist bis dahin offen ausgewiesen.
