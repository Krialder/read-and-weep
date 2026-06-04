# 0003: Identität über Username, Multi-Device von Anfang an

**Status:** akzeptiert

## Kontext

Viele Messenger hängen die Identität an die Telefonnummer. Das macht das Onboarding bequem, kostet aber Privatsphäre: Die Nummer ist ein direkter Personenbezug, und der Adressbuchabgleich legt den sozialen Graphen offen. Gleichzeitig erwarten Leute, dasselbe Konto auf mehreren Geräten zu haben.

## Entscheidung

Die Identität hängt an einem Username. Eine Telefonnummer braucht es nicht. Gefunden wird über Handles per privater Kontaktsuche, ohne Adressbuch-Upload. Multi-Device ist von Anfang an drin: Jedes Gerät hat eigene Schlüssel, ein neues wird von einem bestehenden freigeschaltet, ein entferntes verliert seine Sitzungen und löst Neuschlüsseln aus.

## Konsequenzen

Der Datenschutz ist von Haus aus besser, kein Personenbezug über die Nummer. Das kostet an zwei Stellen: Ohne Nummer als Anker ist Sybil-Abwehr schwerer (dazu [specs/sealed-sender-und-abuse.md](../specs/sealed-sender-und-abuse.md)), und das Datenmodell wird aufwändiger, weil pro Konto mehrere Geräte mit eigenen Schlüsseln laufen.

Die Wiederherstellung zerfällt in zwei Teile, die nicht durcheinandergeraten dürfen. Der Login ist über E-Mail und zweiten Faktor rettbar. Der lokale Klartext hängt am Recovery-Key. Ob es dafür eine bequemere, SVR-artige Lösung gibt, ist noch offen, und das gehört unübersehbar in die UI.
