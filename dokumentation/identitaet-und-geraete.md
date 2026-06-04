# Identität und Geräte

## Konten ohne Telefonnummer

Ein Konto hängt an einem Username (Handle). Eine Telefonnummer braucht es nicht. Der Anzeigename ist davon getrennt und jederzeit änderbar, das Handle bleibt eindeutig und stabil. Bei der Registrierung entstehen der Identitätsschlüssel des Nutzers und sein erstes Gerät. Eine E-Mail kann man hinterlegen, um den Login wiederherzustellen, gegenüber Kontakten ist sie kein Teil der Identität.

Der Verzicht auf die Telefonnummer ist eine Datenschutz-Entscheidung. Threema fährt seit Jahren ein ID-basiertes Modell und zeigt, dass es geht. Der Preis ist bekannt: Ohne Nummer als Anker ist es schwerer, Bots und Wegwerf-Konten zu bremsen. Was wir dagegen tun, steht in [specs/sealed-sender-und-abuse.md](specs/sealed-sender-und-abuse.md).

## Kontakte finden

Die Kontaktsuche löst ein Handle zu einem Konto auf, ohne dass der Server ein Adressbuch mitliest. Telefonnummern gibt es keine, also auch keinen Abgleich ganzer Adressbücher. Wer einen Kontakt hinzufügt, holt sich dessen Schlüssel aus dem Verzeichnis und prüft sie gegen das Key-Transparency-Log, bevor eine Sitzung startet.

## Mehrere Geräte

Ein Konto darf mehrere Geräte haben, jedes mit eigenen Schlüsseln. Ein neues Gerät kommt nicht von allein rein: Ein schon eingerichtetes muss es freischalten, per QR-Code oder Bestätigung. Erst danach zählt es als vertrauenswürdiges Mitglied des Kontos und taucht im Verzeichnis auf.

Den bisherigen Verlauf zieht sich das neue Gerät verschlüsselt vom alten, der Server reicht dabei keinen Klartext durch. Wie genau dieser Geräte-zu-Geräte-Transfer abläuft, ist noch nicht festgezurrt (siehe [offene-fragen.md](offene-fragen.md)). Wird ein Gerät entfernt, etwa weil es verloren ging, werden seine Sitzungen ungültig, und alle Gruppen, in denen es war, schlüsseln über MLS neu. Ab dem Schnitt liest es nichts mehr mit, auch nicht rückwirkend.

## Verifikation über Key Transparency

Das härteste Problem bei E2E ist nicht die Verschlüsselung, sondern die Frage, ob der Schlüssel, den der Server für einen Kontakt ausliefert, wirklich dem Kontakt gehört. Die ausführliche Antwort steht in [specs/key-transparency.md](specs/key-transparency.md), hier die Kurzform.

Im Alltag prüft der Client jeden ausgelieferten Schlüssel gegen ein öffentliches, nur-anhängbares Log. Tauscht der Server heimlich einen Schlüssel aus, fällt das beim Abgleich auf, ohne dass jemand von Hand etwas tun muss. Wer es zusätzlich absichern will, vergleicht die Safety Number mit dem Gegenüber, eine kurze Prüfsumme über beide Schlüssel, beim Treffen oder am Telefon. Ändert sich der Schlüssel eines Kontakts, zeigt der Client das deutlich an, statt es stillschweigend zu schlucken.

## Wiederherstellung

Zwei Dinge gehören getrennt gehalten. Der Login ist wiederherstellbar, etwa über die hinterlegte E-Mail und einen zweiten Faktor. Der lokale Klartext hängt am Recovery-Key oder an der Passphrase. Geht beides verloren, kommt man zwar wieder ins Konto, der alte verschlüsselte Verlauf bleibt aber zu.

Das ist die ehrliche, aber unbequeme Variante, und in der Praxis verlieren damit viele Nutzer ihre History. Signal hat dafür Secure Value Recovery gebaut (SVR2 und SVR3, ein PIN, der einen Schlüssel in einer abgesicherten Enclave entsperrt, mit hartem Rate-Limit gegen Raten). Ob wir etwas Vergleichbares bauen, ist eine der größeren offenen Entscheidungen, siehe [offene-fragen.md](offene-fragen.md). Bis dahin gilt: Recovery-Key sichern, sonst ist der Verlauf weg.
