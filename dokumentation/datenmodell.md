# Datenmodell

Was der Server speichert, und warum er damit wenig anfangen kann.

Auf dem Server liegen Konten, Handles, Geräte, die öffentlichen Schlüssel samt Prekeys, das Key-Transparency-Verzeichnis, Chiffretext-Hüllen mit knappem Routing-Vermerk, die Geräteliste pro Gruppe (fürs Fanout) und verschlüsselte Medien. Klartext, private Schlüssel und der entschlüsselte Verlauf kommen nie hierher.

Geshardet wird nach Konto-ID. Jeder Dienst hält sein eigenes Schema. Die SQL-Blöcke zeigen die relationale Sicht. Das KT-Verzeichnis ist zusätzlich ein spezialisierter Merkle-Speicher, dazu unten.

## Konten und Geräte

```sql
CREATE TABLE accounts (
    id            UUID PRIMARY KEY,
    handle        TEXT UNIQUE NOT NULL,
    display_name  TEXT,
    status        TEXT NOT NULL DEFAULT 'active',   -- active | locked | deletion_pending
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE devices (
    id            UUID PRIMARY KEY,
    account_id    UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    name          TEXT,
    identity_key  BYTEA NOT NULL,                   -- öffentlicher Geräteschlüssel
    added_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_seen_at  TIMESTAMPTZ,
    revoked_at    TIMESTAMPTZ
);
```

## Prekeys

Pro Gerät ein Vorrat, klassisch und post-quantum. Die Einmal-Prekeys verbraucht der erste Kontakt, der signierte hält länger. Der Vorrat ist ein Ziel für Erschöpfungs-Angriffe, siehe [betrieb.md](betrieb.md).

```sql
CREATE TABLE prekeys (
    id          UUID PRIMARY KEY,
    device_id   UUID NOT NULL REFERENCES devices(id) ON DELETE CASCADE,
    kind        TEXT NOT NULL,        -- signed | one_time | pq_signed | pq_one_time
    key         BYTEA NOT NULL,
    used        BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_prekeys_pool ON prekeys (device_id, kind) WHERE used = FALSE;
```

## Key Transparency

Relationale Sicht sind die signierten Wurzeln pro Epoche. Die eigentliche verifiable map (Label aus VRF(handle), Wert ist ein Commitment auf die Schlüsselmenge) liegt in einem Merkle-Speicher, dessen Bauart wir gegen die KT-Bibliothek festzurren. Details in [specs/key-transparency.md](specs/key-transparency.md).

```sql
CREATE TABLE kt_epochs (
    epoch        BIGINT PRIMARY KEY,
    root_hash    BYTEA NOT NULL,
    signature    BYTEA NOT NULL,        -- Signatur über die Wurzel
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Nachrichten

Für den Server eine Blackbox mit Zustelladresse. Der Absender steht nicht drin, den führt Sealed Sender verschlüsselt in der Hülle mit. Die `id` ist vom Client erzeugt und macht `POST /messages` idempotent: Kommt dieselbe `id` zweimal an (Funkloch, Retry), wird nicht doppelt zugestellt.

```sql
CREATE TABLE messages (
    id            UUID PRIMARY KEY,    -- client-erzeugt, dedupliziert Retries
    recipient_id  UUID NOT NULL REFERENCES accounts(id),
    device_id     UUID NOT NULL REFERENCES devices(id),
    envelope      BYTEA NOT NULL,      -- Sealed-Sender-Hülle + Chiffretext
    franking_tag  BYTEA,               -- für meldbaren Abuse, siehe Abuse-Spec
    received_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    delivered_at  TIMESTAMPTZ
);

CREATE INDEX idx_messages_inbox ON messages (device_id, received_at);
```

## Gruppen-Zustellung

Der Server kann Gruppennachrichten nicht lesen, muss sie aber zustellen. Dafür kennt er, welche Geräte zu einer Gruppe gehören. Das ist der Metadaten-Punkt aus [sicherheit.md](sicherheit.md): Mitgliedschaft ist vor dem Betreiber nicht geheim.

```sql
CREATE TABLE group_devices (
    group_id   UUID NOT NULL,
    device_id  UUID NOT NULL REFERENCES devices(id) ON DELETE CASCADE,
    PRIMARY KEY (group_id, device_id)
);
```

Der kryptografische Gruppenzustand (wer Mitglied ist, welche Schlüssel gelten) lebt als TreeKEM-Zustand in den Clients. Der Server hält ihn nicht.

## Was auf dem Gerät bleibt

Nur lokal und verschlüsselt: private Schlüssel, der Ratchet- und MLS-Zustand, der entschlüsselte Nachrichten-Cache, die Einstellungen. Auf einem gesperrten Gerät ist der Bestand ohne Geräte-Keystore oder Passphrase nutzlos.
