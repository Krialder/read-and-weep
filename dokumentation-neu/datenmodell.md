# Datenmodell

Was der Server speichert. Die Blöcke sind nah an dem, was wir wirklich anlegen, mit Indizes, Partitionierung und den Sperren, die der Sitzungsaufbau braucht. Klartext, private Schlüssel und der entschlüsselte Verlauf kommen nie hierher.

Geshardet wird nach Konto-ID, jeder Dienst hält sein eigenes Schema. Das KT-Verzeichnis ist zusätzlich ein Merkle-Speicher (siehe [specs/key-transparency.md](specs/key-transparency.md)).

## Konten und Geräte

```sql
CREATE TABLE accounts (
    id                     UUID PRIMARY KEY,
    handle                 TEXT UNIQUE NOT NULL,
    display_name           TEXT,
    status                 TEXT NOT NULL DEFAULT 'active',  -- active | locked | deletion_pending
    deletion_pending_until TIMESTAMPTZ,
    created_at             TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE devices (
    id              UUID PRIMARY KEY,
    account_id      UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    name            TEXT,
    identity_key    BYTEA NOT NULL,        -- öffentlicher Geräteschlüssel
    credential_blob BYTEA NOT NULL,        -- vom Konto-Identitätsschlüssel signiertes Geräte-Credential
    added_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_seen_at    TIMESTAMPTZ,
    revoked_at      TIMESTAMPTZ
);

CREATE INDEX idx_devices_account ON devices (account_id) WHERE revoked_at IS NULL;
```

## Prekeys

Pro Gerät ein Pool, klassisch und post-quantum. Wichtig ist die atomare Konsumierung: Zwei gleichzeitige Sitzungsaufbauten dürfen nicht denselben Einmal-Prekey ziehen. Das löst `SELECT ... FOR UPDATE SKIP LOCKED`.

```sql
CREATE TABLE prekeys (
    id         UUID PRIMARY KEY,
    device_id  UUID NOT NULL REFERENCES devices(id) ON DELETE CASCADE,
    kind       TEXT NOT NULL,        -- signed | one_time | pq_signed | pq_one_time
    key        BYTEA NOT NULL,
    used       BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Pool eines Geräts, ungenutzte zuerst, mit SKIP LOCKED beim Ziehen.
CREATE INDEX idx_prekeys_pool ON prekeys (device_id, kind) WHERE used = FALSE;
```

Zielwerte: rund 100 Einmal-Prekeys je Typ und Gerät, Nachfüllen ab 20 verbleibenden, signierte Prekeys wöchentlich rotiert.

## Key Transparency

Relationale Sicht sind die signierten Wurzeln pro Epoche. Die veränderliche verifiable map liegt im Merkle-Speicher. Lange Historien werden beschnitten, das Verfahren steht in der KT-Spec.

```sql
CREATE TABLE kt_epochs (
    epoch      BIGINT PRIMARY KEY,
    root_hash  BYTEA NOT NULL,
    signature  BYTEA NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## Nachrichten

Für den Server eine Blackbox mit Zustelladresse. Die `id` ist client-erzeugt und macht `POST /messages` idempotent. Die Tabelle ist nach Monat partitioniert (`PARTITION BY RANGE (received_at)`), alte Partitionen fallen per Retention weg.

```sql
CREATE TABLE messages (
    id            UUID NOT NULL,        -- client-erzeugt, dedupliziert Retries
    recipient_id  UUID NOT NULL,
    device_id     UUID NOT NULL,
    envelope      BYTEA NOT NULL,       -- Sealed-Sender-Hülle + Chiffretext
    franking_tag  BYTEA NOT NULL,       -- Commitment fürs Abuse-Reporting
    received_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    delivered_at  TIMESTAMPTZ,
    PRIMARY KEY (id, received_at)
) PARTITION BY RANGE (received_at);

CREATE INDEX idx_messages_inbox ON messages (device_id, received_at);
```

## Franking-Log

Damit der Server eine gemeldete Nachricht verifizieren kann, muss er sein eigenes Franking-Commitment aufbewahren, ohne den Inhalt zu kennen. Sonst lässt sich eine Meldung nicht prüfen.

```sql
CREATE TABLE franking_log (
    message_id   UUID NOT NULL,
    commitment   BYTEA NOT NULL,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (message_id)
);
```

## Gruppen-Zustellung

Der Server kann Gruppennachrichten nicht lesen, muss sie aber zustellen, und kennt dafür die Geräte einer Gruppe. Das ist der Metadaten-Punkt aus der [sicherheit.md](sicherheit.md).

```sql
CREATE TABLE group_devices (
    group_id  UUID NOT NULL,
    device_id UUID NOT NULL REFERENCES devices(id) ON DELETE CASCADE,
    PRIMARY KEY (group_id, device_id)
);
```

Der kryptografische Gruppenzustand lebt als TreeKEM-Zustand in den Clients. Der Server hält ihn nicht.

## Was auf dem Gerät bleibt

Nur lokal und verschlüsselt: private Schlüssel, der Ratchet- und MLS-Zustand, der entschlüsselte Cache, die Einstellungen. Auf einem gesperrten Gerät ist der Bestand ohne Geräte-Keystore oder Passphrase nutzlos.
