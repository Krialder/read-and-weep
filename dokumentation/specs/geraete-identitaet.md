# Spec: Geräte-Identitätsbindung

Ein Konto hat einen langlebigen Identitätsschlüssel und mehrere Geräte, jedes mit eigenen Schlüsseln. Das wirft eine Frage auf, die in der Übersicht leicht untergeht: Wie wird verhindert, dass ein freigeschaltetes, aber bösartiges Gerät als "der Nutzer" auftritt? Und wie taucht die Gerätemenge so in Key Transparency auf, dass ein heimlich untergeschobenes Gerät auffällt?

Weil die Geräteverwaltung schon in Phase 1 steht, gehört das hierher und nicht in eine spätere Phase.

## Die Kette

Der Konto-Identitätsschlüssel (Ed25519) ist die Wurzel. Er signiert für jedes Gerät ein Geräte-Credential, das den öffentlichen Geräteschlüssel, eine Geräte-ID und eine Gültigkeit bindet. Ein Gerät weist seine Zugehörigkeit zum Konto nach, indem es sein signiertes Credential vorzeigt. Geräteschlüssel hängen damit am Konto-Identitätsschlüssel, statt frei in der Luft.

Der private Konto-Identitätsschlüssel liegt beim Nutzer, nie beim Server. Er entsteht bei der Registrierung und wird bei der Freischaltung eines neuen Geräts verschlüsselt mit übertragen.

## Freischaltung

Ein neues Gerät erzeugt seine eigenen Schlüssel und legt sie einem bereits eingerichteten Gerät vor, per QR-Code oder Bestätigung. Das bestehende Gerät hat Zugriff auf den Konto-Identitätsschlüssel und signiert damit das Credential des neuen Geräts. Erst dieses Credential macht das Gerät zu einem gültigen Mitglied des Kontos.

## Sichtbar in Key Transparency

Das KT-Commitment pro Konto deckt nicht nur den Konto-Identitätsschlüssel ab, sondern die aktuelle Menge gültiger Geräte-Credentials beziehungsweise deren Hashes. Schiebt der Server ein zusätzliches Gerät unter, müsste das im KT-Log auftauchen, und die Selbstprüfung des Clients (der seine wahre Gerätemenge kennt) schlägt an. Damit greift hier derselbe Schutz wie bei einem getauschten Schlüssel, siehe [key-transparency.md](key-transparency.md).

## Entfernen

Ein entferntes Gerät wird aus der gültigen Menge gestrichen. Das ist ein neuer Eintrag in der Konto-Historie im KT-Log (Revocation) und löst in allen Gruppen ein MLS-Neuschlüsseln aus. Ab dem Schnitt ist das Credential ungültig.

## Offen

- Genaues Credential-Format: Felder, Gültigkeitsdauer, Rotation.
- Ob zusätzlich ein Cross-Signing zwischen Geräten sinnvoll ist, für stärkere Isolation bei einem kompromittierten Gerät, nach dem Vorbild von Matrix.
- Rotation des Konto-Identitätsschlüssels selbst, und was das für alte Credentials bedeutet.
