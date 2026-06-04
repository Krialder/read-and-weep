# Protokoll und Kryptografie

Hier steht, wie aus "der Server sieht nichts" echte Technik wird, im Überblick. Die Details, an denen es wirklich hängt (die Hybrid-Bindung bei PQXDH, die Cipher Suite bei MLS, die Versionierung), stehen in [specs/krypto-kern.md](specs/krypto-kern.md). Eine Regel vorweg: Nichts davon ist selbst gebaut. Die Krypto kommt aus libsignal (PQXDH, Double Ratchet) und OpenMLS (Gruppen), beide als geprüfter Rust-Code, geteilt über alle Clients.

## Schlüssel pro Nutzer, pro Gerät

Jeder Nutzer hat einen langlebigen Identitätsschlüssel, Ed25519 zum Signieren mit einem X25519-Anteil für den Austausch. Der ist die Wurzel der Identität und liegt nur auf dem Gerät, nie im Klartext woanders.

Ein Konto besteht aus mehreren Geräten, jedes mit eigenen Schlüsseln. Jedes lädt einen Vorrat an Prekeys ins Verzeichnis: einen signierten und viele Einmal-Prekeys, klassisch und post-quantum. Damit kann dir jemand eine erste Nachricht schicken, während dein Handy offline in der Tasche steckt.

## Sitzungsaufbau mit PQXDH

Schreibt jemand zum ersten Mal an ein Gerät, holt er dessen Prekey-Bündel und rechnet PQXDH: ein hybrider Schlüsselaustausch aus X25519 und dem PQ-KEM ML-KEM-768. Der Sinn des PQ-Teils ist ein konkreter Angriff. Jemand schneidet heute verschlüsselten Verkehr mit, legt ihn auf Halde und entschlüsselt ihn in zehn Jahren mit einem Quantencomputer. Hybrid heißt, ein Angreifer muss beide Anteile brechen, den klassischen und den post-quantum. Wie genau die beiden Geheimnisse gebunden werden, ist sicherheitskritisch und steht in der Krypto-Kern-Spec.

## Laufende Sitzung mit dem Ratchet

Sobald der Chat läuft, übernimmt der Double Ratchet. Er liefert zwei Eigenschaften. Forward Secrecy: Jede Nachricht bekommt einen frischen Schlüssel, alte werden weggeworfen, ein später gestohlenes Gerät gibt den alten Verlauf nicht mehr her. Post-Compromise Security: Wird doch mal ein Schlüssel abgegriffen, heilt sich die Sitzung mit der nächsten Runde selbst.

libsignal bringt dazu inzwischen einen post-quantum Ratchet mit (SPQR, von Signal Ende 2025 veröffentlicht), der parallel zum klassischen läuft. Damit wird auch die laufende 1:1-Sitzung quantenfest, nicht nur der Aufbau.

## Gruppen mit MLS

Für Gruppen nehmen wir MLS (RFC 9420) über OpenMLS. Der Kern ist ein Schlüsselbaum (TreeKEM): Tritt jemand bei oder geht, reicht ein Update entlang eines Astes, statt dass jeder mit jedem neu verhandelt. Der Aufwand wächst logarithmisch mit der Gruppengröße. Genau hier scheitern ältere Ansätze: Das Sender-Keys-Modell (WhatsApp, Signal-Gruppen) und Matrix' Megolm werden bei großen, aktiven Gruppen teuer oder schwach in der Post-Compromise-Eigenschaft. MLS ist dafür gebaut.

Ein ehrlicher Haken: MLS ist in den Standard-Suites von RFC 9420 klassisch. Unsere Gruppen sind damit aktuell nicht post-quantum. PQ-MLS läuft über einen IETF-Draft (Combiner) und ist noch nicht fertig. Das steht auf der [roadmap.md](roadmap.md) und in [offene-fragen.md](offene-fragen.md).

## Mehrere Geräte

Jedes Gerät führt eigene Sitzungen. An einen Nutzer zu schreiben heißt, an jedes seiner Geräte zu verschlüsseln, und an die eigenen Zweitgeräte gleich mit, damit der Verlauf überall derselbe ist. Ein neues Gerät wird von einem bestehenden freigeschaltet, ein entferntes verliert seine Sitzungen. Details in [identitaet-und-geraete.md](identitaet-und-geraete.md).

## Metadaten und Sealed Sender

Bei den meisten Nachrichten weiß der Server nicht, von wem sie kommen. Die Absenderkennung steckt verschlüsselt in der Sendung, geroutet wird über den Empfänger. Damit das nicht zum Spam-Einfallstor wird, hängt an Sealed Sender ein Sender-Certificate und eine Abuse-Mechanik. Der ganze Ablauf steht in [specs/sealed-sender-und-abuse.md](specs/sealed-sender-und-abuse.md).

## Echtheit der Schlüssel

Verschlüsseln nützt nichts, wenn der Server dir den falschen Schlüssel unterschiebt. Dagegen läuft Key Transparency: ein öffentlich prüfbares Log aller Schlüssel, gegen das jeder Client automatisch abgleicht. Das ist ein eigenes Subsystem, beschrieben in [specs/key-transparency.md](specs/key-transparency.md).

## Lokale Daten

Auf dem Gerät liegt alles verschlüsselt: private Schlüssel, der Ratchet- und MLS-Zustand, der entschlüsselte Nachrichten-Cache, die Einstellungen. Der Schlüssel dafür kommt aus dem Hardware-Keystore der Plattform (Secure Enclave, Android Keystore, TPM), wo es einen gibt, sonst aus einer Passphrase. Backups sind verschlüsselt und hängen an einem Recovery-Key. Wie viel Komfort die Wiederherstellung bekommt, ist noch offen (siehe [identitaet-und-geraete.md](identitaet-und-geraete.md)).
