# 0005: Krypto-Bibliotheken und ein gemeinsamer Rust-Core

**Status:** akzeptiert

## Kontext

Die Wahl der Krypto-Bibliothek entscheidet einen Großteil der Architektur, und sie war lange unausgesprochen. "Keine Eigenkrypto" allein reicht nicht, man muss sagen, woher die Verfahren konkret kommen. Außerdem soll dieselbe Krypto auf vier Client-Plattformen laufen, ohne vier leicht unterschiedliche Implementierungen zu pflegen.

## Entscheidung

PQXDH, der Double Ratchet und SPQR kommen aus libsignal. MLS kommt aus OpenMLS. Beide sind Rust. Wir bauen daraus einen gemeinsamen Krypto-Core, der über alle Clients geteilt wird: als WebAssembly im Browser, als native Bibliothek über FFI in den mobilen und Desktop-Apps.

## Konsequenzen

Die heiklen Teile existieren genau einmal, in geprüftem Code, und ein Fix gilt überall. Das senkt das Risiko an der Stelle, an der Fehler am teuersten sind.

Der Aufwand verlagert sich ins Tooling: WASM- und FFI-Builds, eine saubere Schnittstelle zwischen Core und der jeweiligen UI-Sprache, und die Pflege der Bindings. Außerdem bindet es uns an die Release-Zyklen von libsignal und OpenMLS, was beim Thema PQ-MLS sogar von Vorteil ist, weil wir nachziehen können, sobald OpenMLS es unterstützt.
