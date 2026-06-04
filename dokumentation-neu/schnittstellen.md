# Schnittstellen

Zwei Wege zwischen Client und Kern. REST über HTTPS für alles, was du auslöst, WebSocket für alles, was der Server von sich aus meldet. Bodies sind JSON, Binärdaten gehen Base64-kodiert über die Leitung. Authentifiziert wird per `Authorization: Bearer <token>`, Access-Token kurzlebig, Refresh-Token rotiert.

Die Übersicht zeigt die grobe Form. Jedes Feld steht verbindlich in der OpenAPI-Spec, die mit dem Backend ausgeliefert wird.

## Konto und Geräte

- `POST /accounts` legt ein Konto mit Handle an und registriert das erste Gerät.
- `POST /sessions`, `POST /sessions/refresh`, `DELETE /sessions` für Login, Erneuern, Logout.
- `GET /devices`, `POST /devices` (Freischaltung durch ein bestehendes Gerät, mit Geräte-Credential), `DELETE /devices/{id}` (entfernt und stößt Gruppen-Neuschlüsseln an).

## Schlüssel, Credentials, Zertifikate

- `GET /keys/{handle}` liefert die Prekey-Bündel aller Geräte, klassisch und post-quantum. Rate-limitiert pro Abrufer.
- `POST /keys/prekeys` füllt den eigenen Pool nach.
- `POST /credentials/sender` gibt ein anonymes Sende-Credential (zkgroup-Linie) aus, das beweist, dass der Abruf von einem gültigen Konto kommt, ohne zu verraten, von welchem.

## Key Transparency

- `GET /kt/proof/{handle}` gibt den Inklusionsbeweis samt VRF-Beweis.
- `GET /kt/epoch/{n}` liefert die signierte Wurzel einer Epoche.
- `GET /kt/consistency?from=a&to=b` gibt den Konsistenzbeweis zwischen zwei Epochen.

## Nachrichten

`POST /messages` ist die heikle Stelle. Die Sendung darf den Absender nicht preisgeben, der Server muss aber prüfen können, dass sie von einem berechtigten Konto kommt. Deshalb trägt der Request kein Absender-Feld, sondern ein anonymes Sende-Credential und die Sealed-Sender-Hülle. Der Server verifiziert das Credential, ohne den Absender zu lernen, und stellt zu.

- `POST /messages` nimmt eine oder mehrere Hüllen entgegen (eine je Zielgerät), idempotent über die client-erzeugte `id`.
- `GET /messages` holt die wartenden Sendungen für die eigenen Geräte.
- `POST /messages/ack` bestätigt den Empfang, danach wird die Sendung weggeräumt.
- `POST /messages/report` meldet Abuse unter Aufdeckung des Franking-Tags.

## Gruppen

- `POST /groups` erzeugt eine MLS-Gruppe.
- `POST /groups/{id}/commit` reicht einen MLS-Commit ein. Der Delivery Service ordnet konkurrierende Commits, siehe [specs/mls-delivery-service.md](specs/mls-delivery-service.md).
- `POST /groups/{id}/messages` verteilt eine Gruppennachricht an die Mitgliedsgeräte.

## Medien

- `POST /media` lädt einen verschlüsselten Blob hoch und gibt eine Referenz zurück.
- `GET /media/{ref}` lädt ihn herunter. Der Schlüssel reist in der Nachricht.

## Echtzeit-Events

Über die WebSocket-Verbindung schickt der Server nur Hinweise, den Chiffretext holt der Client danach selbst.

| Event | Bedeutung |
|-------|-----------|
| `message.waiting` | Es liegt etwas für ein Gerät bereit |
| `message.delivered` | Eine Nachricht wurde zugestellt |
| `message.read` | Eine Nachricht wurde gelesen |
| `presence` | Ein Kontakt ist online oder offline |
| `typing` | Das Gegenüber tippt |
| `key.changed` | Der Schlüssel eines Kontakts hat sich geändert |

## Fehler und Drosselung

Fehler kommen einheitlich als JSON mit Code, Klartextmeldung und optionalen Detailfeldern. Krypto-Fehler bleiben nach außen generisch. Rate-Limiting läuft als Middleware mit Zählern in Redis, streng auf Login, Registrierung und die Schlüssel-Endpunkte. Bei Überschreitung kommt eine `429` mit `Retry-After`.
