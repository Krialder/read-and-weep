# Spec: Sealed Sender und Abuse

Zwei Themen, die zusammengehören. Sealed Sender verbirgt den Absender vor dem Server. Genau das macht Spam-Abwehr schwerer, weil der Server nicht mehr nach Absender filtern kann. Deshalb steht beides hier in einem Dokument.

## Sealed Sender

Ziel: Der Server stellt eine Nachricht zu, ohne zu lernen, von wem sie kommt. Er kennt nur den Empfänger, weil er sonst nicht zustellen kann.

Ablauf: Der Absender packt seine Identität in ein Sender-Certificate und legt Certificate plus Inhalt in eine verschlüsselte Hülle, die nur der Empfänger öffnet. Der Server sieht die Hülle und die Zustelladresse, mehr nicht. Der Empfänger öffnet sie, liest das Certificate und weiß damit, wer geschrieben hat, obwohl der Server es nicht weiß.

## Sender-Certificate

Das Certificate bindet Identität und Gerät an einen vom Server signierten, kurzlebigen Nachweis. Der Empfänger prüft die Signatur und die Gültigkeit. Kurzlebig heißt, es rotiert (Größenordnung Tage), damit ein altes Certificate nicht ewig nachnutzbar ist. Die genaue Lebensdauer steht in den offenen Punkten.

Wichtig: Das Certificate sagt dem Empfänger, wer gesendet hat. Es sagt es nicht dem Server. Diese Trennung ist der ganze Witz an Sealed Sender.

## Zugang: wer darf überhaupt sealed senden

Wenn jeder anonym an jeden senden könnte, wäre Sealed Sender ein Spam-Traumland. Signal löst das über ein Zugangstoken, das aus dem Profil-Schlüssel des Empfängers abgeleitet ist. Nur wer den Profil-Schlüssel hat (also ein Kontakt, dem du ihn gegeben hast), kann dir sealed schicken. Für alle anderen gibt es zwei Modi:

- restricted: Fremde können nicht sealed senden. Ihre Nachricht läuft sichtbar (Server kennt den Absender) oder landet in einer Anfrage-Inbox.
- unrestricted: Der Nutzer erlaubt sealed auch von Fremden, auf eigenes Risiko.

Default ist restricted.

## Das Abuse-Problem ohne Telefonnummer

Ohne Telefonnummer als Anker fällt die natürliche Bremse gegen Massenkonten weg. Drei Hebel, die zusammen wirken müssen:

### Sybil-Widerstand bei der Registrierung

Das Ziel ist, einem Angreifer das Anlegen von zehntausend Konten teuer zu machen, ohne echte Nutzer zu nerven oder zu deanonymisieren. Kandidaten: Proof-of-Work bei der Registrierung, anonyme Credentials beziehungsweise Privacy Pass für rate-limitierte Tokens, Einladungs- oder Reputationssysteme. Welche Mischung, ist eine Produktentscheidung und noch offen.

### Message Franking

Sealed Sender macht Meldungen schwer: Wenn der Server den Absender nicht kennt, wie soll ein Empfänger eine missbräuchliche Nachricht beweisbar melden, ohne dass sich Meldungen fälschen lassen? Die Antwort ist kryptografisches Franking, wie es Facebook Messenger eingeführt hat. Der Absender hängt ein Franking-Tag an, der Server committet sich beim Transport darauf, ohne den Inhalt zu sehen. Meldet der Empfänger die Nachricht, deckt er den Tag auf, und der Server kann verifizieren, dass genau diese Nachricht so gesendet wurde. Kein Fälschen, kein Bruch der E2E.

### Empfängerseitige Kontrollen

Blockieren, "wer darf mir schreiben", Anfrage-Inbox für Erstkontakte, Rate-Limits pro Empfänger. Das ist die letzte und wichtigste Verteidigungslinie, weil sie beim Nutzer liegt und nicht auf Server-Einsicht angewiesen ist.

## Vorbilder

Signal hat Sealed Sender 2018 eingeführt und mehrfach über die Abuse-Folgen geschrieben. Das Franking-Konzept stammt aus Facebooks Arbeit zu Message Franking. Privacy Pass ist der Standard-Baustein für anonyme, rate-limitierte Tokens. Wir kombinieren bekannte Teile, statt etwas Eigenes zu erfinden.

## Offen

- Konkreter Sybil-Widerstand bei der Registrierung (PoW, Privacy Pass, Invite, oder Mischung).
- Lebensdauer und Rotation der Sender-Certificates.
- Schlüsselverwaltung fürs Franking und das genaue Commitment-Schema.
- Ob Erstkontakte grundsätzlich in eine Anfrage-Inbox laufen.
- Zusammenspiel von Zugangstoken und Multi-Device (mehrere Geräte, ein Profil-Schlüssel).
