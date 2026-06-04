# Roadmap

Sortiert nach Abhängigkeiten, ohne Termine. Zwei Aspekte, auf die bei der Planung bewusst geachtet wurde: eine Spec-Phase vor dem ersten Code, und Audits laufen mehrfach, nach dem Krypto-Kern und vor Launch.

## Phase 0: Specs und Entscheidungen, vor dem Code

- [x] Krypto-Bibliotheken festgenagelt (libsignal, OpenMLS), siehe [ADR-0005](entscheidungen/0005-krypto-bibliotheken.md)
- [x] Key-Transparency-Modell entworfen, siehe [specs/key-transparency.md](specs/key-transparency.md)
- [x] Sealed Sender und Abuse entworfen, siehe [specs/sealed-sender-und-abuse.md](specs/sealed-sender-und-abuse.md)
- [x] Krypto-Kern (PQXDH-Bindung, MLS-Suite, Versionierung), siehe [specs/krypto-kern.md](specs/krypto-kern.md)
- [ ] Offene Entscheidungen aus [offene-fragen.md](offene-fragen.md) abarbeiten, bevor das jeweilige Stück gebaut wird

## Phase 1: Fundament

- [ ] Identität und Auth: Konten mit Handle, Login, rotierende Sessions
- [ ] Geräteverwaltung: erstes Gerät, weitere per Freischaltung, Entfernen mit Neuschlüsseln
- [ ] Schlüssel-Verzeichnis mit Prekey-Vorrat (klassisch und post-quantum)
- [ ] Key-Transparency-Grundgerüst: Epochen, Merkle-Map, Selbstprüfung im Client
- [ ] Web-Client und ein nativer Client parallel, beide registrieren sich und veröffentlichen Schlüssel

## Phase 2: 1:1-Nachrichten

- [ ] Sitzungsaufbau über PQXDH (libsignal)
- [ ] Double Ratchet plus SPQR
- [ ] Senden und Empfangen, Zustellstatus, Echtzeit-Push, idempotente Zustellung
- [ ] Multi-Device-Fanout, eigene Zweitgeräte bleiben synchron
- [ ] Erstes externes Audit des Krypto-Kerns

## Phase 3: Metadaten, Verifikation, Abuse

- [ ] Sealed Sender mit Sender-Certificates
- [ ] Key Transparency vollständig: Auditoren, Gossip, Konsistenzbeweise
- [ ] Safety Numbers und sichtbare Warnung bei Schlüsselwechsel
- [ ] Private Kontaktsuche über Handles
- [ ] Abuse v1: Message Franking, empfängerseitige Kontrollen, Sybil-Widerstand bei der Registrierung

## Phase 4: Gruppen

- [ ] MLS-Gruppen über OpenMLS (klassische Suite), Beitritt, Austritt, Neuschlüsseln
- [ ] Gruppen-Zustellung über die Mitgliedsgeräte

## Phase 5: Medien und Anrufe

- [ ] Verschlüsselter Dateiversand über den Medien-Dienst
- [ ] Anrufe als eigenes Subprojekt: SFrame, MLS-basiertes Group-Calling, DTLS-SRTP, Identitätsbindung der Call-Schlüssel. Vom Aufwand her in der Größe von Phase 2.

## Phase 6: Härtung

- [ ] PQ-MLS, sobald der Combiner-Draft stabil und in OpenMLS ist
- [ ] Krypto-Agilität und Wire-Versionierung gehärtet
- [ ] Formale Modelle der Handshakes (ProVerif oder Tamarin), Fuzzing, Property-Tests für den Krypto-Code
- [ ] Lasttests auf den heißen Pfaden
- [ ] Zweites externes Audit vor Launch

## Später

- Föderation: serverübergreifende Key Transparency, Server-Discovery, S2S-Auth
- Multi-Region für Latenz und Ausfallsicherheit
- Secure Value Recovery für Backups (entschieden, kommt nach dem Fundament)
- Client-seitige Suche über den lokalen, entschlüsselten Verlauf
- FIDO2 als phishing-resistenter Login-Faktor
