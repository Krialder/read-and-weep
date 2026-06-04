# Betrieb und Skalierung

## Deployment

Die Go-Dienste laufen auf Kubernetes, jeder als eigenes Deployment mit eigener Autoskalierung, davor ein Ingress als Edge mit TLS. Konfiguration und Geheimnisse aus der Plattform. Die zustandsbehafteten Teile (PostgreSQL, Redis, Objektspeicher, Queue und der KT-Merkle-Store) laufen als verwaltete Dienste oder über erprobte Operatoren. Multi-Region kommt später, die Architektur steht dem nicht im Weg.

## Der Krypto-Core im Betrieb

Die Krypto läuft auf den Clients, nicht im Backend. Fürs Backend heißt das: keine privaten Schlüssel, kein Entschlüsseln, kleinere Angriffsfläche. Der Rust-Core wird als WebAssembly fürs Web und als native Bibliothek über FFI für mobile und Desktop gebaut. Ein gemeinsamer Core heißt auch, ein Bug wird an einer Stelle gefixt und gilt überall.

## Skalierung und die teuren Pfade

Die zustandslosen Dienste skalieren waagerecht. Die heißen Pfade Echtzeit und Zustellung skalieren unabhängig vom Rest. Presence liegt in Redis mit gebündelten Updates, geshardet wird nach Konto-ID, das Fanout läuft über die Queue.

Ein eigener Punkt ist das Schlüssel-Verzeichnis. PQXDH-Handshakes sind teuer, ML-KEM-Schlüssel sind rund ein Kilobyte, und Einmal-Prekeys können erschöpft werden. Wer das Verzeichnis flutet, erzeugt Last und leert Vorräte. Dagegen brauchen wir Rate-Limits pro Abrufer, Quotas und ein Monitoring auf Vorratsstände. Die genauen Schwellen stehen in [offene-fragen.md](offene-fragen.md). Auch die Epochen-Berechnung der Key Transparency ist Last und bekommt einen eigenen Pfad.

## Echtzeit-Zustellung

Die WebSocket-Knoten halten die Verbindungen und sind austauschbar. Eine eingehende Nachricht findet über Queue und Backplane den Knoten mit der Zielverbindung. Offline-Geräte weckt Push, die Sendung wartet im Store-and-forward bis zur Abholung.

## Push

Push weckt nur, der Inhalt kommt per Pull. Trotzdem sehen APNs und FCM den Geräte-Token und das Timing jeder Benachrichtigung (siehe [sicherheit.md](sicherheit.md)). Mildern lässt sich das mit gebündelten oder verzögerten Pushes und einem generischen "du hast Neues" statt einer Meldung pro Nachricht. Ganz wegbekommen kann man es bei einer mobilen App nicht.

## Beobachtbarkeit

Strukturierte Logs nach stdout, niemals mit Nachrichteninhalt. Metriken im Prometheus-Format, Health-Probes je Dienst (`live` und `ready`), Tracing über die Dienstgrenzen.

## Audit-Timing

Nicht erst am Ende. Ein externes Audit gehört mindestens zweimal eingeplant: einmal, sobald der Krypto-Kern steht (PQXDH, Ratchet, MLS-Integration), und nochmal vor dem Launch. Audits ganz am Schluss finden Architekturfehler, die dann niemand mehr anfasst.

## Datenhaltung und Löschung

Knappe Aufbewahrung, zugestellte und bestätigte Nachrichten verschwinden vom Server. Backups rotieren mit fester Frist. Gelöscht wird über Crypto-Shredding (siehe [sicherheit.md](sicherheit.md)), eine Kontolöschung läuft mit Widerrufsfrist und vernichtet danach die zugehörigen Schlüssel.
