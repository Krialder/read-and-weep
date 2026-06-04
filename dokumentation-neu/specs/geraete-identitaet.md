# Spec: Geräte-Identitätsbindung

Ein Konto hat einen langlebigen Identitätsschlüssel und mehrere Geräte mit eigenen Schlüsseln. Daraus folgen zwei Fragen, die in der Übersicht leicht untergehen: Wie wird verhindert, dass ein freigeschaltetes, aber bösartiges Gerät als der Nutzer auftritt? Und wie taucht die Gerätemenge in Key Transparency auf, dass ein untergeschobenes Gerät auffällt? Weil die Geräteverwaltung in Phase 1 steht, gehört das nach vorn.

## Die Kette

Der Konto-Identitätsschlüssel (Ed25519) ist die Wurzel. Er signiert für jedes Gerät ein Credential, das den öffentlichen Geräteschlüssel, eine Geräte-ID und eine Gültigkeit bindet. Ein Gerät weist seine Zugehörigkeit nach, indem es sein Credential vorzeigt. Der private Wurzelschlüssel liegt beim Nutzer, nie beim Server, und wird bei der Freischaltung eines neuen Geräts verschlüsselt mit übertragen.

## Freischaltung

Ein neues Gerät erzeugt seine Schlüssel und legt sie einem bestehenden Gerät vor (QR-Code oder Bestätigung). Das bestehende Gerät hat den Wurzelschlüssel und signiert damit das neue Credential. Erst das Credential macht das Gerät zu einem gültigen Mitglied.

## Sichtbar in Key Transparency

Das KT-Commitment pro Konto deckt die aktuelle Menge gültiger Geräte-Credentials ab. Schiebt der Server ein Gerät unter, müsste das im Log auftauchen, und die Selbstprüfung des Clients (der seine wahre Gerätemenge kennt) schlägt an. Derselbe Schutz wie bei einem getauschten Schlüssel.

## Rotation des Wurzelschlüssels

Der Wurzelschlüssel ist der heikelste Punkt im ganzen Baum. Fällt er, ist alles darunter offen. Die Rotation läuft als Übergabekette: Ein neuer Wurzelschlüssel wird vom alten signiert, alle Geräte-Credentials werden neu ausgestellt, und der Wechsel landet als eigener Eintrag in der Konto-Historie der Key Transparency, sodass Kontakte ihn sehen und prüfen. Was passiert, wenn der alte Wurzelschlüssel kompromittiert statt nur abgelaufen ist, ist der harte Teil und noch offen.

## Cross-Signing

Optional bestätigen sich Geräte gegenseitig, statt nur am Wurzelschlüssel zu hängen. Das isoliert ein kompromittiertes Gerät besser. Matrix hat damit über Jahre schmerzhafte Erfahrungen gemacht, vor allem verlorene Cross-Signing-Schlüssel und Verifikations-Loops, in denen Nutzer ihre eigenen Geräte nicht mehr verifizieren konnten. Wir sehen uns das an, bevor wir Cross-Signing festlegen.

## Entfernen

Ein entferntes Gerät fällt aus der gültigen Menge. Das ist ein neuer Eintrag in der Konto-Historie (Revocation) und löst in allen Gruppen ein MLS-Neuschlüsseln aus. Ab dem Schnitt ist das Credential ungültig.

## Offen

- Verhalten bei kompromittiertem (nicht nur abgelaufenem) Wurzelschlüssel.
- Cross-Signing ja oder nein, und wie man die Matrix-Fehler vermeidet.
- Credential-Format im Detail, Gültigkeitsdauer, Rotationsintervall.
