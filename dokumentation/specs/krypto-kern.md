# Spec: Krypto-Kern

Hier liegen die Details, die über sicher und angreifbar entscheiden. Die Verfahren selbst kommen aus libsignal (PQXDH, Double Ratchet, SPQR) und OpenMLS. Was wir festlegen müssen, ist die Integration: wie die hybriden Anteile gebunden werden, welche MLS-Suite läuft, und wie wir Versionen wechseln, ohne dass die Welt stehenbleibt.

## PQXDH und die Hybrid-Bindung

PQXDH kombiniert einen klassischen Schlüsselaustausch (mehrere X25519-DH-Schritte im X3DH-Stil) mit einem post-quantum KEM (ML-KEM-768). Heraus kommt ein Sitzungsgeheimnis, grob:

```
SK = KDF( F || DH1 || DH2 || DH3 || DH4 || SS )
```

`DH1..DH4` sind die X25519-Ergebnisse (das vierte nur, wenn ein Einmal-Prekey dabei ist), `SS` ist das KEM-Geheimnis, `F` ein fester Domain-Präfix. Der KEM-Chiffretext `CT` wird mitgesendet.

Der ganze Sicherheitsgewinn hängt an der Bindung. Wenn ein Angreifer den PQ-Anteil unbemerkt strippen oder den KEM-Chiffretext austauschen könnte, fällt man auf reine X25519-Sicherheit zurück, und der ganze Aufwand war umsonst. Deshalb:

- Alle Anteile (klassisch und PQ) gehen gemeinsam durch die KDF. Keiner ist optional weglassbar.
- Die Associated Data der ersten verschlüsselten Nachricht bindet die Identitätsschlüssel beider Seiten und den KEM-Chiffretext.
- Domain-Separation-Labels trennen diese Ableitung sauber von jeder anderen.

Die genauen Byte-Layouts, Labels und der `F`-Präfix übernehmen wir 1:1 aus der PQXDH-Spezifikation von libsignal. Hier nichts selbst basteln, das ist genau die Stelle, an der eigene Hybrid-Konstruktionen in den letzten Jahren reihenweise gebrochen sind.

## Double Ratchet und SPQR

Die laufende 1:1-Sitzung fährt den Double Ratchet. Der Nachrichtenschlüssel zieht seine Sicherheit aus der DH-Kette plus der symmetrischen Kette. libsignal ergänzt das um SPQR, einen parallelen post-quantum Ratchet. Der Nachrichtenschlüssel mischt dann beide Anteile, klassisch und PQ. Damit ist die fortlaufende Sitzung quantenfest und nicht nur der Aufbau.

## MLS-Cipher-Suite

OpenMLS bringt mehrere Suites mit. Als klassische Basis setzen wir auf:

```
MLS_128_DHKEMX25519_AES128GCM_SHA256_Ed25519   (Suite 0x0001)
```

Das ist eine breit unterstützte RFC-9420-Suite. Die ChaCha20-Variante ist eine Alternative, falls wir den symmetrischen Teil über alles vereinheitlichen wollen, das steht in den offenen Punkten.

Ehrlich und wichtig: Diese Suite ist klassisch. Gruppen sind damit nicht post-quantum. Ein PQ-Pfad für MLS existiert nur als IETF-Draft (der Combiner, `draft-ietf-mls-combiner`, der einen klassischen und einen PQ-KEM in TreeKEM kombiniert). Sobald das stabil ist und in OpenMLS landet, ziehen wir nach. Bis dahin gilt für Gruppen: Forward Secrecy und Post-Compromise Security ja, Post-Quantum nein.

## Krypto-Agilität und Wire-Versionierung

Das ist die Frage, an der die meisten Projekte scheitern: Wie kommt man von Suite v1 auf v2, ohne dass alte Nachrichten unlesbar werden oder ein Angreifer ein Downgrade erzwingt?

Unser Ansatz:

1. Jede Hülle und jede Sitzung trägt eine Protokoll-Version und eine Suite-ID. Nichts ist implizit.
2. Das Prekey-Bündel im Verzeichnis nennt die unterstützten Suites. Beim Sitzungsaufbau wird die höchste gemeinsam unterstützte gewählt.
3. Downgrade-Schutz: Die ausgehandelte Version wird in den Transkript-Hash des Handshakes gebunden. Ein MITM, der eine schwächere Suite erzwingen will, zerstört damit den Handshake.
4. Migration: Neue Sitzungen nehmen die neue Suite. Bestehende laufen auf ihrer Suite weiter, bis sie natürlich enden oder an einem Stichtag zum Neu-Handshake gezwungen werden. Server dürfen eine alte Suite erst abschalten, wenn die Clients migriert sind, sonst werden In-flight-Nachrichten unlesbar.

## Zufall und Nonces

Schlüssel und Nonces kommen ausschließlich aus dem CSPRNG der Bibliothek. Nonce-Wiederverwendung ist bei AEAD tödlich und wird durch die Library-Nutzung vermieden, nicht durch eigenen Code.

## Offen

- AES-GCM gegen ChaCha20-Poly1305 als MLS-Suite, also Standard-Kompatibilität gegen einheitlichen symmetrischen Stack.
- Zeitpunkt für PQ-MLS, abhängig vom Combiner-Draft und der OpenMLS-Unterstützung.
- Genaues Wire-Format der Versionsaushandlung und der Suite-IDs.
- Stichtags-Mechanik für erzwungene Re-Handshakes.
- Wie SPQR-Versionen in unsere Wire-Version eingehängt werden.
