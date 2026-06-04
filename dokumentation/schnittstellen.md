# Schnittstellen

Zwei Wege zwischen Client und Kern. REST über HTTPS für alles, was du aktiv auslöst. WebSocket für alles, was der Server von sich aus meldet. Bodies sind JSON, Binärdaten gehen Base64-kodiert über die Leitung. Authentifiziert wird per `Authorization: Bearer <token>`, Access-Token kurzlebig, Refresh-Token rotiert.

Die Übersicht zeigt die grobe Form. Jedes Feld steht verbindlich in der OpenAPI-Spec, die mit dem Backend ausgeliefert wird.

## Konto und Geräte

- `POST /accounts` legt ein Konto mit Handle an und registriert das erste Gerät.
- `POST /sessions`, `POST /sessions/refresh`, `DELETE /sessions` für Login, Erneuern, Logout.
- `GET /devices`, `POST /devices` (Freischaltung durch ein bestehendes Gerät), `DELETE /devices/{id}` (entfernt und stößt Gruppen-Neuschlüsseln an).

## Schlüssel und Zertifikate

- `GET /keys/{handle}` liefert die Prekey-Bündel aller Geräte, klassisch und post-quantum. Rate-limitiert gegen Erschöpfung.
- `POST /keys/prekeys` füllt den eigenen Vorrat nach.
- `POST /keys/sender-cert` holt ein frisches, kurzlebiges Sender-Certificate für Sealed Sender.

## Key Transparency

- `GET /kt/proof/{handle}` gibt den Inklusionsbeweis für den aktuellen Schlüssel eines Handles (inkl. VRF-Beweis).
- `GET /kt/epoch/{n}` liefert die signierte Wurzel einer Epoche.
- `GET /kt/consistency?from=a&to=b` gibt den Konsistenzbeweis zwischen zwei Epochen, für Clients und Auditoren.

## Nachrichten

- `POST /messages` nimmt eine oder mehrere Sealed-Sender-Hüllen entgegen (eine je Zielgerät). Idempotent über die client-erzeugte `id`.
- `GET /messages` holt die wartenden Sendungen für die eigenen Geräte ab.
- `POST /messages/ack` bestätigt den Empfang, danach räumt der Server die Sendung weg.
- `POST /messages/report` meldet Abuse unter Aufdeckung des Franking-Tags.

## Gruppen

- `POST /groups` erzeugt eine MLS-Gruppe.
- `POST /groups/{id}/commit` reicht einen MLS-Commit ein (Beitritt, Austritt, Schlüsselwechsel).
- `POST /groups/{id}/messages` verteilt eine Gruppennachricht an die Mitgliedsgeräte.

## Medien

- `POST /media` lädt einen verschlüsselten Blob hoch und gibt eine Referenz zurück.
- `GET /media/{ref}` lädt ihn herunter. Der Schlüssel reist in der Nachricht, nie über diesen Endpunkt.

## Echtzeit-Events

Über die WebSocket-Verbindung schickt der Server nur Hinweise. Den Chiffretext holt der Client danach selbst.

| Event | Bedeutung |
|-------|-----------|
| `message.waiting` | Es liegt etwas für ein Gerät bereit |
| `message.delivered` | Eine Nachricht wurde zugestellt |
| `message.read` | Eine Nachricht wurde gelesen |
| `presence` | Ein Kontakt ist online oder offline |
| `typing` | Das Gegenüber tippt |
| `key.changed` | Der Schlüssel eines Kontakts hat sich geändert (Anlass zur Anzeige) |

## Fehler und Drosselung

Fehler kommen einheitlich als JSON mit Code, Klartextmeldung und optionalen Detailfeldern. Krypto-Fehler bleiben nach außen generisch. Rate-Limiting läuft als Middleware mit Zählern in Redis. Streng auf Login, Registrierung und die Schlüssel-Endpunkte (die sind teuer und ein Angriffsziel), großzügiger auf den Versand. Bei Überschreitung kommt eine `429` mit `Retry-After`.
