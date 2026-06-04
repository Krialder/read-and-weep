# Spec: Backup und Recovery (Skizze)

Das ist eine Skizze, keine fertige Spec. Sie steht hier, weil die Recovery-Story das Schlüsselmodell von Phase 1 prägt. Eine später nachgerüstete Lösung bricht das Modell sonst wieder auf.

## Das Problem

Der lokale Verlauf hängt am Recovery-Key beziehungsweise an der Passphrase. Geht beides verloren, ist der Verlauf weg. In der Praxis verlieren damit viele Nutzer ihre History. Eine bequemere Lösung darf dem Server aber nicht das Geheimnis zeigen, sonst ist die Zero-Knowledge-Eigenschaft hin.

## Der Ansatz: Secure Value Recovery

Nach dem Vorbild von Signal (SVR2 und SVR3): Ein Recovery-Secret, das den Backup-Schlüssel entsperrt, liegt in einer abgesicherten Enclave. Der Nutzer entsperrt es mit einer niedrig-entropen PIN. Ein hartes Limit gegen Raten erzwingt der Counter-State der Enclave, nicht der Server. Der Betreiber kann die PIN also nicht durchprobieren.

SVR3 verteilt das Vertrauen über mehrere Enclaves oder Backends, damit eine einzelne Kompromittierung nicht reicht.

## Was Phase 1 dafür offenhalten muss

Die wichtigste Konsequenz für jetzt: Die Schlüsselableitung in Phase 1 darf nicht festschreiben, dass nur die Passphrase den lokalen Master-Key erzeugen kann. Es braucht von Anfang an einen Platz für ein zweites Entsperr-Geheimnis, das später aus der Enclave kommt. Sonst zementiert Phase 1 eine Recovery-Story, die hart zu ändern ist.

Gebaut wird SVR nach Phase 1. Offengehalten wird der Platz dafür schon jetzt.

## Offen

- Enclave-Wahl: SGX, AWS Nitro, OpenEnclave, oder eine Mischung.
- PIN-Politik und das genaue Brute-Force-Limit über den Counter-State.
- Replikation des Counter-States, ohne das Limit zu unterlaufen.
- Was SVR genau schützt: nur den Backup-Schlüssel, oder auch den Konto-Identitätsschlüssel.
