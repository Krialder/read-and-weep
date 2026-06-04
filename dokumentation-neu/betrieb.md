# Betrieb und Skalierung

## Deployment

Die Go-Dienste laufen auf Kubernetes, jeder als eigenes Deployment mit eigener Autoskalierung, davor ein Ingress als Edge mit TLS. Die zustandsbehafteten Teile (PostgreSQL, Redis, Objektspeicher, NATS und der KT-Merkle-Store) laufen als verwaltete Dienste oder über erprobte Operatoren. Multi-Region kommt später.

Die Sprache der Dienste ist Go, auch für die Realtime-Schicht. Elixir auf der BEAM war eine ernsthafte Option für Presence und Fanout (WhatsApp skaliert historisch damit), wir bleiben aber bei einem Stack, statt zwei Laufzeiten zu pflegen. Wenn die Realtime-Last das später erzwingt, ist der Wechsel ein begrenztes, isoliertes Stück.

## Der Krypto-Core im Betrieb

Die Krypto läuft auf den Clients, nicht im Backend. Fürs Backend heißt das: keine privaten Schlüssel, kein Entschlüsseln, kleinere Angriffsfläche. Der Rust-Core wird als WebAssembly fürs Web und als native Bibliothek über FFI gebaut. Die Builds sind reproducible und signiert, ihre Hashes landen im Binary-Transparency-Log, damit der Server keinem Zielkonto eine abweichende App unterschieben kann (siehe [ADR-0007](entscheidungen/0007-binary-transparency.md)).

## Skalierung und die teuren Pfade

Die zustandslosen Dienste skalieren waagerecht. Die heißen Pfade Echtzeit und Zustellung skalieren unabhängig. Presence liegt in Redis mit gebündelten Updates, das Fanout läuft über NATS.

Der Sonderfall ist das Schlüssel-Verzeichnis. PQXDH-Handshakes sind teuer, ML-KEM-Schlüssel rund 1,2 KB, und der Prekey-Pool ist erschöpfbar. Dagegen brauchen wir Rate-Limits pro Abrufer, den Pool von rund 100 Einmal-Prekeys mit Nachfüllen ab 20, und die atomare Konsumierung aus dem [Datenmodell](datenmodell.md). Auch die Epochen-Berechnung der Key Transparency ist Last und bekommt einen eigenen Pfad.

## Echtzeit-Zustellung

Die WebSocket-Knoten halten die Verbindungen und sind austauschbar. Eine eingehende Nachricht findet über NATS den Knoten mit der Zielverbindung. Offline-Geräte weckt Push, die Sendung wartet im Store-and-forward bis zur Abholung.

## Push

Push weckt nur, der Inhalt kommt per Pull. APNs und FCM sehen trotzdem Geräte-Token und Timing (siehe [sicherheit.md](sicherheit.md)). Auf iOS prüfen wir eine Notification Service Extension, die den Inhalt im Geräte-Sandbox entschlüsselt, das hält den Inhalt aus dem Push, ohne das Token-Tracking zu lösen. Für Android steht UnifiedPush als FCM-Alternative auf der Liste.

## Beobachtbarkeit

Strukturierte Logs nach stdout, niemals mit Nachrichteninhalt. Metriken im Prometheus-Format, Health-Probes je Dienst (`live` und `ready`), Tracing über die Dienstgrenzen.

## Audit-Timing

Nicht erst am Ende. Ein externes Audit gehört mindestens zweimal eingeplant, einmal sobald der Krypto-Kern steht und nochmal vor dem Launch. Davor läuft ein leichterer Krypto-Review auf die Specs, bevor Phase 1 beginnt.

## Datenhaltung und Löschung

Knappe Aufbewahrung, zugestellte Nachrichten verschwinden, die Tabelle ist nach Monat partitioniert. Verschlüsselte Backups liegen im Objektspeicher des Betreibers, das Geheimnis zu ihrer Entschlüsselung steckt in der SVR-Enclave und nicht beim Betreiber. Gelöscht wird über Crypto-Shredding, eine Kontolöschung läuft mit Widerrufsfrist und vernichtet danach die zugehörigen Schlüssel.
