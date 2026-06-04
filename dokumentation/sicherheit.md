# Sicherheit und Bedrohungsmodell

Bevor es um Features geht, die wichtigere Frage: gegen wen schützt das System überhaupt? Hier die Antwort als Tabelle, samt der Zeilen, in denen wir verlieren.

## Was geschützt ist, was nicht

| Szenario | Geschützt? | Warum |
|----------|-----------|-------|
| Server-Datenbank wird geleakt | ja | Nur Chiffretext und öffentliche Schlüssel, keine privaten, kein Klartext |
| Betreiber liest Inhalte mit | ja | Kein Klartext im Kern, Sealed Sender verbirgt zusätzlich den Absender |
| Bösartiger Server schiebt falschen Schlüssel unter | gestaffelt | Offenen Tausch fängt die Selbstprüfung sofort, Split-View erst mit Auditoren (ab Phase 3) |
| Netzwerk-Mitschnitt | ja | TLS im Transport, darüber die Ende-zu-Ende-Schicht |
| Mitschnitt heute, Quantencomputer in ein paar Jahren | 1:1 ja, Gruppen noch nicht | PQXDH ist hybrid mit PQ-KEM. MLS ist aktuell klassisch, siehe unten |
| Gestohlenes, gesperrtes Gerät | ja | Lokale Daten verschlüsselt, an den Geräte-Keystore gebunden |
| Verlorenes Gerät | ja | Aus der Ferne entfernbar, seine Sitzungen sterben, Gruppen schlüsseln neu |
| Schadsoftware auf dem Endgerät | nein | Wer das Gerät kontrolliert, sieht den Klartext wie der Nutzer |
| Betreiber will wissen, wer in welcher Gruppe ist | nein | Der Zustelldienst kennt die Empfängerliste pro Zustellung |
| Verkehrsanalyse (Timing, Datenmenge) | teilweise | Inhalt und meist Absender verborgen, der Verkehr als Muster bleibt |
| Spam und Bot-Konten | teilweise | Eigenes Thema, siehe Abuse-Spec |

Vier Zeilen brauchen ein Wort dazu.

Die Malware-Zeile ist die, die wehtut. Sitzt ein Schädling erst mal auf deinem Gerät, mit Zugriff auf Tastatur und Speicher, dann sieht er, was du siehst. Dagegen hilft keine Verschlüsselung der Welt. Das ist eine Grenze der ganzen Gattung, die hat jeder Messenger.

Gruppen und Post-Quantum: 1:1-Chats laufen über PQXDH und sind gegen das "heute mitschneiden, später entschlüsseln"-Szenario abgesichert. Gruppen laufen über MLS, und MLS ist in den Standard-Suites von RFC 9420 klassisch. PQ-MLS gibt es noch nicht fertig, nur als IETF-Draft (Combiner). Heißt konkret: Wer heute Gruppen-Chiffretext mitschneidet und in zehn Jahren einen Quantencomputer hat, kommt ran. Das steht so in [protokoll.md](protokoll.md) und auf der Roadmap. Und selbst mit PQ-MLS bliebe zunächst die Signaturschicht (Ed25519) klassisch. Das betrifft die Authentizität und zählt erst, wenn Quantencomputer real sind, Details in [specs/krypto-kern.md](specs/krypto-kern.md).

Gruppenmitgliedschaft: Der Server kann keine Gruppennachrichten lesen, aber er muss sie zustellen, und dafür kennt er die Empfänger. Wer mit wem in einer Gruppe ist, ist also für den Betreiber sichtbar. Das lässt sich mit MLS nicht wegzaubern, der Delivery Service braucht die Routing-Information.

Push: "Ohne Inhalt im Push" stimmt, aber APNs und FCM sehen den Geräte-Token und das Timing jeder Benachrichtigung. Apple und Google sind damit passive Beobachter eines Teils deiner Metadaten. Das ist bei jeder mobilen App so und nur begrenzt zu entschärfen (gebündelte oder verzögerte Pushes), nicht abzuschalten.

Key Transparency, gestaffelt: Die Selbstprüfung im Client fängt ab Phase 1 jeden offenen Schlüsseltausch. Den Split-View, bei dem der Server dir und deinem Kontakt verschiedene Verzeichnisse zeigt, fängt erst der unabhängige Auditor ab Phase 3. Bis dahin ist der Schutz gegen einen aktiv bösartigen Server eingeschränkt.

## Wo es eng wird

Metadaten lassen sich runterdrücken, nicht abschalten. Sealed Sender und der Verzicht auf Telefonnummern helfen viel, aber Zustelladresse, Timing und Gruppenrouting bleiben. Wer das lange genug beobachtet, zieht Schlüsse.

Schlüsselverlust trifft hart. Sind Recovery-Key und Passphrase weg, sind die lokal verschlüsselten Daten und der alte Verlauf futsch. Eine Konto-Wiederherstellung holt den Login zurück, an den alten Klartext kommt man damit nicht mehr. Eine bequemere Lösung über PIN und Enclave (SVR) ist eingeplant und kommt nach Phase 1, die Skizze steht in [specs/backup-und-recovery.md](specs/backup-und-recovery.md).

Prekey-Erschöpfung ist nicht nur ein Betriebsthema. Sind die Einmal-Prekeys eines Geräts leer, fällt der Sitzungsaufbau auf den Signed Prekey zurück, und das schwächt die PQXDH-Eigenschaften für genau diese Sitzungen. Die Politik dazu (Nachfüllen, Limits, Verhalten bei Leerstand) gehört ins Krypto-Design, siehe [offene-fragen.md](offene-fragen.md).

Föderation weicht den Metadatenschutz auf, sobald fremde Server zustellen. Deshalb ist sie standardmäßig aus.

## Compliance, offen und bewusst so

Wir bauen ein System, in dem der Betreiber technisch keine Inhalte herausgeben kann. Das ist eine politische Position, keine technische Selbstverständlichkeit. UK Online Safety Act, die EU-Diskussion um Chat-Kontrolle (CSAR), Vorratsdaten: All das berührt so ein Produkt direkt. In dieser Phase (ernst gemeint, aber noch nicht im Betrieb) ist das ein bewusst offener Punkt und kein gelöstes Kapitel. Es steht in [offene-fragen.md](offene-fragen.md), damit es niemand übersieht. Eine Position muss vor Phase 3 stehen, weil sie das Datenmodell und die Server-Fähigkeiten berührt.

## Löschen heißt Schlüssel vernichten

Daten auf einer Datenbank "sicher zu überschreiben" ist Augenwischerei, sobald Replikation, Write-Ahead-Log, Backups und das Wear-Leveling der SSD mitspielen. Die eine Kopie, die du überschreibst, ist nie die einzige.

Der Hebel ist die Verschlüsselung. Vernichtest du die Schlüssel zu einem Datensatz, ist der Chiffretext wertloser Müll, egal wie viele Kopien herumliegen. Crypto-Shredding nennt sich das. Stammdaten werden gelöscht und laufen über die normale Backup-Rotation aus dem Bestand. Eine Löschfrist erlaubt den Rückzieher, danach ist Schluss.

## Standards, an denen wir uns messen

Eine Orientierung, keine Zertifizierung. ML-KEM aus der NIST-PQC-Auswahl, hybrid mit X25519. ChaCha20-Poly1305 und AES-256-GCM symmetrisch. Ed25519 und X25519 für Signatur und Austausch. Argon2id für Passwörter und Schlüsselableitung. MLS nach RFC 9420 für Gruppen. Ein externes Audit hat nicht stattgefunden und muss vor jedem ernsthaften Einsatz her, nach dem Krypto-Kern und nochmal vor Launch.

## Threat-Model-Reife

Die Tabelle oben ist eine Szenario-Liste, kein systematisches Modell. Für ein Audit reicht das nicht. Vor Phase 2 kommt ein strukturiertes Threat Model dazu, für einen metadaten-bewussten Messenger passt LINDDUN neben einer Angreifer-mal-Asset-Sicht. Die formalen Handshake-Modelle (ProVerif oder Tamarin) laufen schon ab Phase 2 parallel zum Krypto-Kern.
