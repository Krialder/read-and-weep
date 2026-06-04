# 0004: Voll native Clients plus Web, nativ von Anfang an parallel

**Status:** akzeptiert

## Kontext

Ein Web-Client ist schnell ausgerollt und überall erreichbar, hat im Browser aber harte Grenzen: Schlüssel lassen sich nicht zuverlässig an die Hardware binden, Hintergrund-Sync und Push sind eingeschränkt, der lokale Speicher ist schwächer geschützt als nativ. Die naheliegende Reihenfolge "erst Web, native Clients später" hat eine Falle: Der schwächste Client wird zum Default für die meisten Nutzer und bleibt es, weil "später" oft nie kommt.

## Entscheidung

Web, iOS, Android und Desktop sind gleichwertige Clients auf Funktionsparität. Mindestens ein nativer Client läuft von Anfang an parallel zum Web-Client, nicht erst in einer späten Phase. Die nativen binden private Schlüssel über den Keystore ihrer Plattform an die Hardware (Secure Enclave, Android Keystore, TPM).

## Konsequenzen

Das beste Schutzniveau steht dort zur Verfügung, wo es geht, und der Browser bleibt als niedrigschwelliger Zugang erhalten. Der Preis ist Entwicklungsbreite: zwei Client-Welten gleichzeitig ab Tag eins, statt erst eine und dann die andere. Den Aufwand nehmen wir in Kauf, weil die Alternative das Sicherheitsmodell auf den schwächsten Client verzerrt.
